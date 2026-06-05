# gemm_testbed_3x_ptr_array.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/unit/gemm/device/gemm_testbed_3x_ptr_array.hpp`
- **Purpose / 用途:** Grouped GEMM pointer-array testbed for validating batched Intel Xe kernels.

## Line-by-Line Analysis / 逐行分析

### Lines 1-34

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * Copyright (C) 2025 Intel Corporation, All rights reserved.
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
    \brief Testbed for Ptr-Array and Grouped GEMM interface
*/
```
- **EN:** Provides the standard BSD-3-Clause license header and ownership notice for this source file.
- **CN:** 给出该源文件的标准 BSD-3-Clause 许可证头和版权归属说明。

### Lines 36-36

```cpp
#pragma once
```
- **EN:** Uses `#pragma once` to ensure the header is included only once per translation unit.
- **CN:** 使用 `#pragma once`，确保该头文件在一个编译单元中只被包含一次。

### Lines 38-42

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <algorithm>
#include <random>
```
- **EN:** Imports dependencies such as `iostream`, `fstream`, `sstream`, `algorithm`, `random` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `iostream`, `fstream`, `sstream`, `algorithm`, `random`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 44-57

```cpp
#include "../../common/cutlass_unit_test.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/gett.hpp"
#include "cutlass/epilogue/collective/default_epilogue.hpp"
#include "cutlass/epilogue/fusion/operations.hpp"
#include "cutlass/complex.h"
#include "testbed_utils.h"
```
- **EN:** Imports dependencies such as `cutlass_unit_test.h`, `host_tensor.h`, `tensor_view_io.h`, `distribution.h`, `packed_stride.hpp`, `tensor_fill.h`, ... (+8) so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `cutlass_unit_test.h`, `host_tensor.h`, `tensor_view_io.h`, `distribution.h`, `packed_stride.hpp`, `tensor_fill.h`, ... (+8)，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 59-62

```cpp
#include "cutlass/kernel_hardware_info.hpp"
#include "cutlass/layout/matrix.h"
#include "cutlass/matrix_coord.h"
#include "cutlass/gemm/gemm.h"
```
- **EN:** Imports dependencies such as `kernel_hardware_info.hpp`, `matrix.h`, `matrix_coord.h`, `gemm.h` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `kernel_hardware_info.hpp`, `matrix.h`, `matrix_coord.h`, `gemm.h`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 64-66

```cpp
#include "cute/int_tuple.hpp"
#include "cute/layout.hpp"
#include "cute/numeric/int.hpp"
```
- **EN:** Imports dependencies such as `int_tuple.hpp`, `layout.hpp`, `int.hpp` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `int_tuple.hpp`, `layout.hpp`, `int.hpp`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 68-70

```cpp
namespace test {
namespace gemm {
namespace device {
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 72-72

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 74-77

```cpp
enum class ScalarLoc {
  ON_HOST = 0,
  ON_DEVICE = 1
};
```
- **EN:** Declares enumeration types or constants (`enum`, `ScalarLoc`, `ON_HOST`, `ON_DEVICE`) so later code can express modes and options explicitly.
- **CN:** 声明枚举类型或常量（`enum`, `ScalarLoc`, `ON_HOST`, `ON_DEVICE`），使后续代码能够更明确地表达模式与选项。

### Lines 79-82

```cpp
enum class VectorScale {
  DISABLED = 0,
  ENABLED = 1
};
```
- **EN:** Declares enumeration types or constants (`enum`, `VectorScale`, `DISABLED`, `ENABLED`) so later code can express modes and options explicitly.
- **CN:** 声明枚举类型或常量（`enum`, `VectorScale`, `DISABLED`, `ENABLED`），使后续代码能够更明确地表达模式与选项。

### Lines 84-87

```cpp
enum class CheckEquality {
  EXACT = 0,
  RELATIVE = 1
};
```
- **EN:** Declares enumeration types or constants (`enum`, `CheckEquality`, `EXACT`, `RELATIVE`) so later code can express modes and options explicitly.
- **CN:** 声明枚举类型或常量（`enum`, `CheckEquality`, `EXACT`, `RELATIVE`），使后续代码能够更明确地表达模式与选项。

### Lines 89-89

```cpp
namespace detail{
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 91-98

```cpp
// Helper classes that take default data type when
// the Gemm::EpilogueOutputOp does not have ElementCompute
// and ElementScalar.
// (e.g. when Sm90TreeVisitor is used as FusionCallbacks)
template <typename Gemm, typename Default, typename = void>
struct ElementComputeType {
  using Type = Default;
};
```
- **EN:** Defines templated type `ElementComputeType` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ElementComputeType`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 100-103

```cpp
template <typename Gemm, typename Default>
struct ElementComputeType<Gemm, Default, std::void_t<typename Gemm::EpilogueOutputOp::ElementCompute>> {
  using Type = typename Gemm::EpilogueOutputOp::ElementCompute;
};
```
- **EN:** Defines templated type `ElementComputeType` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ElementComputeType`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 105-108

```cpp
template <typename Gemm, typename Default, typename = void>
struct ElementScalarType {
  using Type = Default;
};
```
- **EN:** Defines templated type `ElementScalarType` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ElementScalarType`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 110-113

```cpp
template <typename Gemm, typename Default>
struct ElementScalarType<Gemm, Default, std::void_t<typename Gemm::EpilogueOutputOp::ElementScalar>> {
  using Type = typename Gemm::EpilogueOutputOp::ElementScalar;
};
```
- **EN:** Defines templated type `ElementScalarType` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ElementScalarType`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 116-119

```cpp
template <typename Gemm, typename = void>
struct IsF8F6F4Kernel {
  static constexpr bool value = false;
};
```
- **EN:** Defines templated type `IsF8F6F4Kernel` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsF8F6F4Kernel`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 121-124

```cpp
template <typename Gemm>
struct IsF8F6F4Kernel<Gemm, std::void_t<decltype(Gemm::GemmKernel::CollectiveMainloop::IsF8F6F4)>> {
  static constexpr bool value = true;
};
```
- **EN:** Defines templated type `IsF8F6F4Kernel` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsF8F6F4Kernel`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 127-133

```cpp
// The maximum swizzle size to use
//
// This class, like Splits above makes it harder to confuse
// the order of arguments of the various run(...) functions in this file.
class MaxSwizzleSize {
public:
  MaxSwizzleSize() = default;
```
- **EN:** Declares `MaxSwizzleSize` to hold related state or helper behavior for the surrounding test infrastructure.
- **CN:** 声明 `MaxSwizzleSize`，用于保存周边测试基础设施所需的状态或辅助行为。

### Lines 135-142

```cpp
  template<class IntegralNotBool,
    __CUTE_REQUIRES((std::is_integral_v<IntegralNotBool> &&
      !cute::is_same_v<IntegralNotBool, bool>)) >
  explicit MaxSwizzleSize(IntegralNotBool max_swizzle_size) : max_swizzle_size_(max_swizzle_size) {}
  explicit operator int() const { return max_swizzle_size_; }
private:
  int max_swizzle_size_ = 1;
};
```
- **EN:** Defines templated type `IntegralNotBool` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IntegralNotBool`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 144-147

```cpp
template <typename T>
auto make_iterator(T* ptr) {
  return cute::recast_ptr<T>(ptr);
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 149-152

```cpp
template<class T>
struct IsDefaultEpilogue {
  static constexpr bool value = false;
};
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 154-157

```cpp
template<class ...args>
struct IsDefaultEpilogue<cutlass::epilogue::collective::DefaultEpilogue<args...>> {
  static constexpr bool value = true;
};
```
- **EN:** Defines templated type `IsDefaultEpilogue` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsDefaultEpilogue`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 159-162

```cpp
template<class ...args>
struct IsDefaultEpilogue<cutlass::epilogue::collective::detail::Sm90TmaWarpSpecializedAdapter<args...>> {
  static constexpr bool value = true;
};
```
- **EN:** Defines templated type `IsDefaultEpilogue` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsDefaultEpilogue`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 164-167

```cpp
template <typename Epilogue, typename = void>
struct IsLegacyEpiloguePolicy {
  static constexpr bool value = false;
};
```
- **EN:** Defines templated type `IsLegacyEpiloguePolicy` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsLegacyEpiloguePolicy`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 169-176

```cpp
template <typename Epilogue>
struct IsLegacyEpiloguePolicy<Epilogue, cute::void_t<decltype(Epilogue::DispatchPolicy::FragmentSize)>> {
  using EpiloguePolicy = typename Epilogue::DispatchPolicy;
  static constexpr bool value = cute::is_same_v<
                                      EpiloguePolicy,
                                      cutlass::epilogue::Sm90TmaWarpSpecializedBiasElementwise<
                                        EpiloguePolicy::StagesC, EpiloguePolicy::StagesD, EpiloguePolicy::FragmentSize>>;
};
```
- **EN:** Defines templated type `IsLegacyEpiloguePolicy` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsLegacyEpiloguePolicy`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 178-193

```cpp
// The number of splits to test.
//
// This class makes it harder to confuse the order of arguments
// of the various run(...) functions in this file.  The constructor
// is explicit, so one can't just type 42 (or false, which the
// compiler unhelpfully turns into 0); one has to type Splits(42).
// Splits() picks the default number of splits, 1.
//
// The conversion-to-int operator (operator int()) MUST be explicit!
// Conversion to int MUST require static_cast<int>.
// Otherwise, that defeats a key purpose of this class,
// which is to catch common errors of confusing the order
// of function arguments.
class Splits {
public:
  Splits() = default;
```
- **EN:** Declares `makes` to hold related state or helper behavior for the surrounding test infrastructure.
- **CN:** 声明 `makes`，用于保存周边测试基础设施所需的状态或辅助行为。

### Lines 195-202

```cpp
  template<class IntegralNotBool,
    __CUTE_REQUIRES((std::is_integral_v<IntegralNotBool> &&
      !cute::is_same_v<IntegralNotBool, bool>)) >
  explicit Splits(IntegralNotBool splits) : splits_(splits) {}
  explicit operator int() const { return splits_; }
private:
  int splits_ = 1;
};
```
- **EN:** Defines templated type `IntegralNotBool` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IntegralNotBool`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 204-211

```cpp
// The number of iterations to test.
//
// This class, like Splits above makes it harder to confuse
// the order of arguments of the various run(...) functions in this file.
// Iterations() picks the default number of iterations, 20.
class Iterations {
public:
  Iterations() = default;
```
- **EN:** Declares `Iterations` to hold related state or helper behavior for the surrounding test infrastructure.
- **CN:** 声明 `Iterations`，用于保存周边测试基础设施所需的状态或辅助行为。

### Lines 213-220

```cpp
  template<class IntegralNotBool,
    __CUTE_REQUIRES((std::is_integral_v<IntegralNotBool> &&
      !cute::is_same_v<IntegralNotBool, bool>)) >
  explicit Iterations(IntegralNotBool iterations) : iterations_(iterations) {}
  explicit operator int() const { return iterations_; }
private:
  int iterations_ = 20;
};
```
- **EN:** Defines templated type `IntegralNotBool` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IntegralNotBool`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 222-226

```cpp
template <typename Element, typename Layout>
bool initialize_tensor(
  cutlass::TensorView<Element, Layout> view,
  cutlass::Distribution::Kind dist_kind,
  uint64_t seed) {
```
- **EN:** Implements or wires together logic around `Element`, `initialize_tensor`, `TensorView`, `view`, `Distribution` for the current test scenario.
- **CN:** 围绕 `Element`, `initialize_tensor`, `TensorView`, `view`, `Distribution` 实现或连接当前测试场景所需的逻辑。

### Lines 228-230

```cpp
  if (dist_kind == cutlass::Distribution::Uniform) {
    double scope_max, scope_min;
    int bits_input = cutlass::sizeof_bits<Element>::value;
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 232-235

```cpp
    if (bits_input == 1) {
      scope_max = 2;
      scope_min = 0;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 237-240

```cpp
    else if (bits_input <= 6) {
      scope_max = 2;
      scope_min = -2;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 242-242

```cpp
    else if (bits_input <= 8) {
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 244-249

```cpp
      if constexpr (
                    cute::is_same_v<Element, cutlass::float_ue8m0_t>){
        scope_max = 4;
        scope_min = 1;
      }
      else {
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 251-254

```cpp
        scope_max = 1;
        scope_min = -1;
      }
```
- **EN:** Implements or wires together logic around `scope_max`, `scope_min` for the current test scenario.
- **CN:** 围绕 `scope_max`, `scope_min` 实现或连接当前测试场景所需的逻辑。

### Lines 256-263

```cpp
    }
    else{
      scope_max = 4;
      scope_min = -4;
    }
    cutlass::reference::host::TensorFillRandomUniform(
      view, seed, scope_max, scope_min, 0);
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 265-267

```cpp
  else if (dist_kind == cutlass::Distribution::Identity) {
    cutlass::reference::host::TensorFillIdentity(view);
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 269-271

```cpp
  else if (dist_kind == cutlass::Distribution::Gaussian) {
    cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 273-276

```cpp
  else if (dist_kind == cutlass::Distribution::Sequential) {
    cutlass::reference::host::BlockFillSequential(
      view.data(), view.capacity());
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 278-280

```cpp
  else if (dist_kind == cutlass::Distribution::AllOnes) {
    cutlass::reference::host::TensorFill(view, Element(1));
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 282-285

```cpp
  else {
    EXPECT_TRUE(false) << "Not implemented";
    return false;
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 287-288

```cpp
  return true;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 290-297

```cpp
// Looks at Cute Stride to check Row / Column Major
template<typename Stride>
static constexpr bool is_row_or_col_major(){
  int stride_0 = int(cute::size<0>(Stride{}));
  int stride_1 = int(cute::size<1>(Stride{}));
  int depth = cute::depth(Stride{});
  return ((stride_0 == 1) || (stride_1 == 1)) && (depth == 1);
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 300-317

```cpp
//
// Default MMA input Operands : A , B
//
template<
  class ScheduleType_,
  class Gemm,
  class ElementA_ = typename Gemm::GemmKernel::ElementA,
  class ElementB_ = typename Gemm::GemmKernel::ElementB>
struct HostCollectiveMainloop {
  // Kernel data types
  using ElementA = ElementA_;
  using StrideA  = typename Gemm::GemmKernel::StrideA;
  using InternalStrideA  = typename Gemm::GemmKernel::InternalStrideA;
  using ElementB = ElementB_;
  using StrideB  = typename Gemm::GemmKernel::StrideB;
  using InternalStrideB  = typename Gemm::GemmKernel::InternalStrideB;
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  using LayoutTagA = cutlass::detail::StrideToLayoutTagA_t<StrideA>;
```
- **EN:** Defines templated type `ScheduleType_` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ScheduleType_`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 318-318

```cpp
  using LayoutTagB = cutlass::detail::StrideToLayoutTagB_t<StrideB>;
```
- **EN:** Defines aliases such as `LayoutTagB` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `LayoutTagB`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 320-320

```cpp
  static constexpr bool IsGroupGemm = !cute::is_same_v<StrideA, InternalStrideA>;
```
- **EN:** Declares member fields or local variables related to `IsGroupGemm`, `is_same_v`, `StrideA`, `InternalStrideA` for later setup, execution, or verification.
- **CN:** 声明与 `IsGroupGemm`, `is_same_v`, `StrideA`, `InternalStrideA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 322-325

```cpp
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using EpilogueOutputOp = typename Gemm::EpilogueOutputOp;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 327-327

```cpp
  using Arguments = typename Gemm::GemmKernel::MainloopArguments;
```
- **EN:** Defines aliases such as `Arguments` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Arguments`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 329-330

```cpp
  cutlass::ComplexTransform TransformA = Gemm::kTransformA;
  cutlass::ComplexTransform TransformB = Gemm::kTransformB;
```
- **EN:** Declares member fields or local variables related to `ComplexTransform`, `TransformA`, `Gemm`, `kTransformA`, `TransformB` for later setup, execution, or verification.
- **CN:** 声明与 `ComplexTransform`, `TransformA`, `Gemm`, `kTransformA`, `TransformB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 332-333

```cpp
  std::vector<InternalStrideA> stride_a_host;
  std::vector<InternalStrideB> stride_b_host;
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `InternalStrideA`, `stride_a_host`, `InternalStrideB` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `InternalStrideA`, `stride_a_host`, `InternalStrideB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 335-336

```cpp
  cutlass::DeviceAllocation<InternalStrideA> stride_a_device;
  cutlass::DeviceAllocation<InternalStrideB> stride_b_device;
```
- **EN:** Declares member fields or local variables related to `DeviceAllocation`, `InternalStrideA`, `stride_a_device`, `InternalStrideB`, `stride_b_device` for later setup, execution, or verification.
- **CN:** 声明与 `DeviceAllocation`, `InternalStrideA`, `stride_a_device`, `InternalStrideB`, `stride_b_device` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 338-339

```cpp
  typename LayoutTagA::Stride stride_factor_A;
  typename LayoutTagB::Stride stride_factor_B;
```
- **EN:** Declares member fields or local variables related to `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 341-342

```cpp
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
```
- **EN:** Declares member fields or local variables related to `Distribution`, `Kind`, `init_A`, `init_B` for later setup, execution, or verification.
- **CN:** 声明与 `Distribution`, `Kind`, `init_A`, `init_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 344-349

```cpp
  std::vector<cutlass::HostTensor<ElementA, LayoutTagA>> tensors_A;
  std::vector<cutlass::HostTensor<ElementB, LayoutTagB>> tensors_B;
  cutlass::DeviceAllocation<const ElementA *> device_tensors_A;
  cutlass::DeviceAllocation<const ElementB *> device_tensors_B;
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
```
- **EN:** Implements or wires together logic around `std`, `vector`, `HostTensor`, `ElementA`, `LayoutTagA` for the current test scenario.
- **CN:** 围绕 `std`, `vector`, `HostTensor`, `ElementA`, `LayoutTagA` 实现或连接当前测试场景所需的逻辑。

### Lines 351-352

```cpp
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
```
- **EN:** Declares member fields or local variables related to `uint64_t`, `seed`, `kDefaultSeed` for later setup, execution, or verification.
- **CN:** 声明与 `uint64_t`, `seed`, `kDefaultSeed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 354-358

```cpp
  // Note: this limitation comes from testbed / not the library
  static_assert(is_row_or_col_major<InternalStrideA>(),
    "ERROR : A Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<InternalStrideB>(),
    "ERROR : B Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 360-371

```cpp
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = kDefaultSeed,
    typename LayoutTagA::Stride stride_factor_A_ = typename LayoutTagA::Stride(),
    typename LayoutTagB::Stride stride_factor_B_ = typename LayoutTagB::Stride()
  ):
    stride_factor_A(stride_factor_A_),
    stride_factor_B(stride_factor_B_),
    init_A(init_A_), init_B(init_B_), seed(seed_),
    check_relative_equality(check_relative_equality_) { }
```
- **EN:** Implements or wires together logic around `HostCollectiveMainloop`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution` for the current test scenario.
- **CN:** 围绕 `HostCollectiveMainloop`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution` 实现或连接当前测试场景所需的逻辑。

### Lines 373-377

```cpp
  bool initialize(ProblemShapeType problem_shapes) {
    //
    // Allocate the GEMM workspace
    //
    // for pointer array problem_shapes.groups() is 1
```
- **EN:** Begins function or method `initialize`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `initialize`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 379-382

```cpp
    tensors_A.clear();
    tensors_B.clear();
    stride_a_host.clear();
    stride_b_host.clear();
```
- **EN:** Declares member fields or local variables related to `tensors_A`, `clear`, `tensors_B`, `stride_a_host`, `stride_b_host` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_A`, `clear`, `tensors_B`, `stride_a_host`, `stride_b_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 384-385

```cpp
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = cutlass::platform::max(problem_shapes.groups(), L);
```
- **EN:** Declares member fields or local variables related to `append`, `problem_shapes`, `get_host_problem_shape`, `platform`, `max` for later setup, execution, or verification.
- **CN:** 声明与 `append`, `problem_shapes`, `get_host_problem_shape`, `platform`, `max` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 387-388

```cpp
    for(int32_t i = 0; i < L; ++i) {
      auto [M, N, K, mock_L] = cute::append<4>(problem_shapes.get_host_problem_shape(i), 1);
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 390-391

```cpp
      stride_a_host.push_back(cutlass::make_cute_packed_stride(InternalStrideA{}, {M, K, 1}));
      stride_b_host.push_back(cutlass::make_cute_packed_stride(InternalStrideB{}, {N, K, 1}));
```
- **EN:** Declares member fields or local variables related to `stride_a_host`, `push_back`, `make_cute_packed_stride`, `InternalStrideA`, `stride_b_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a_host`, `push_back`, `make_cute_packed_stride`, `InternalStrideA`, `stride_b_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 393-397

```cpp
      // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
      auto a_coord = cutlass::make_Coord(M, K);
      // Cutlass has Row/Col major refers to MxK times KxN matrix product,
      // so the HostTensorB should be treated as KxN in "coord"'s view
      auto b_coord = cutlass::make_Coord(K, N);
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 399-400

```cpp
      tensors_A.push_back(cutlass::HostTensor<ElementA, LayoutTagA>(a_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_coord, stride_factor_A)));
      tensors_B.push_back(cutlass::HostTensor<ElementB, LayoutTagB>(b_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(b_coord, stride_factor_B)));
```
- **EN:** Declares member fields or local variables related to `tensors_A`, `push_back`, `HostTensor`, `ElementA`, `LayoutTagA` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_A`, `push_back`, `HostTensor`, `ElementA`, `LayoutTagA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 402-403

```cpp
      EXPECT_TRUE(initialize_tensor(tensors_A[i].host_view(), init_A, seed + 2022 + i));
      EXPECT_TRUE(initialize_tensor(tensors_B[i].host_view(), init_B, seed + 2021 + i));
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `initialize_tensor`, `tensors_A`, `host_view`, `init_A` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `initialize_tensor`, `tensors_A`, `host_view`, `init_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 405-408

```cpp
      // It is possible to randomly initialize to all zeros, so override this with non-zeros
      // in the upper left corner of each operand.
      tensors_A[i].host_view().at({0, 0}) = ElementA(1);
      tensors_B[i].host_view().at({0, 0}) = ElementB(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 410-412

```cpp
      tensors_A[i].sync_device();
      tensors_B[i].sync_device();
    }
```
- **EN:** Implements or wires together logic around `tensors_A`, `sync_device`, `tensors_B` for the current test scenario.
- **CN:** 围绕 `tensors_A`, `sync_device`, `tensors_B` 实现或连接当前测试场景所需的逻辑。

### Lines 414-415

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 417-419

```cpp
  Arguments to_args(ProblemShapeType problem_shapes) {
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = cutlass::platform::max(problem_shapes.groups(), L);
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 421-422

```cpp
    std::vector<ElementA *> ptr_A_host(L);
    std::vector<ElementB *> ptr_B_host(L);
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `ElementA`, `ptr_A_host`, `ElementB` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `ElementA`, `ptr_A_host`, `ElementB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 424-427

```cpp
    for (int32_t i = 0; i < L; ++i) {
      ptr_A_host.at(i) = tensors_A[i].device_data();
      ptr_B_host.at(i) = tensors_B[i].device_data();
    }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 429-430

```cpp
    device_tensors_A.reset(L);
    device_tensors_A.copy_from_host(ptr_A_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_A`, `reset`, `copy_from_host`, `ptr_A_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_A`, `reset`, `copy_from_host`, `ptr_A_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 432-433

```cpp
    device_tensors_B.reset(L);
    device_tensors_B.copy_from_host(ptr_B_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_B`, `reset`, `copy_from_host`, `ptr_B_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_B`, `reset`, `copy_from_host`, `ptr_B_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 435-438

```cpp
    stride_a_device.reset(problem_shapes.groups());
    stride_a_device.copy_from_host(stride_a_host.data());
    stride_b_device.reset(problem_shapes.groups());
    stride_b_device.copy_from_host(stride_b_host.data());
```
- **EN:** Declares member fields or local variables related to `stride_a_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 440-440

```cpp
    Arguments arguments;
```
- **EN:** Declares member fields or local variables related to `Arguments`, `arguments` for later setup, execution, or verification.
- **CN:** 声明与 `Arguments`, `arguments` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 442-454

```cpp
    if constexpr (IsGroupGemm) {
      arguments
      =
      {
        device_tensors_A.get(), stride_a_device.get(), device_tensors_B.get(), stride_b_device.get()
      };
    }
    else {
      arguments =
      {
        device_tensors_A.get(), stride_a_host[0], device_tensors_B.get(), stride_b_host[0]
      };
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 456-457

```cpp
    return arguments;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 459-468

```cpp
  auto to_host_args(ProblemShapeType problem_shapes, int batch) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(batch), 1);
    auto A = make_tensor(make_iterator(tensors_A[batch].host_data()),
          make_layout(make_shape(M, K, 1), stride_a_host[batch]));
    auto B = make_tensor(make_iterator(tensors_B[batch].host_data()),
        make_layout(make_shape(N, K, 1), stride_b_host[batch]));
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 470-473

```cpp
    cutlass::reference::host::GettMainloopParams<ElementAccumulator,
                                                 decltype(A),
                                                 decltype(B)
                                                 > mainloop_params{};
```
- **EN:** Implements or wires together logic around `reference`, `host`, `GettMainloopParams`, `ElementAccumulator`, `decltype` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `GettMainloopParams`, `ElementAccumulator`, `decltype` 实现或连接当前测试场景所需的逻辑。

### Lines 475-478

```cpp
    mainloop_params.A = A;
    mainloop_params.B = B;
    mainloop_params.transform_A = TransformA;
    mainloop_params.transform_B = TransformB;
```
- **EN:** Declares member fields or local variables related to `mainloop_params`, `transform_A`, `TransformA`, `transform_B`, `TransformB` for later setup, execution, or verification.
- **CN:** 声明与 `mainloop_params`, `transform_A`, `TransformA`, `transform_B`, `TransformB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 480-481

```cpp
    return mainloop_params;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 483-486

```cpp
  void print_tensors(std::ofstream& file, int batch) {
    file << "A =\n" << tensors_A[batch].host_view()
         << "\nB =\n" << tensors_B[batch].host_view();
  }
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 488-494

```cpp
  template <
    class Element,
    class Layout
  >
  bool equality_check(
    cutlass::TensorView<Element, Layout> const& lhs,
    cutlass::TensorView<Element, Layout> const& rhs) const {
```
- **EN:** Defines templated type `Element` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Element`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 496-501

```cpp
    // Factors used for calculating relative equality. CUTLASS's relative-equality
    // checks in include/cutlass/relatively_equal.h  are inspired by
    // https://floating-point-gui.de/errors/comparison/. This reference suggests using
    // the minimum normal value of a given type as the nonzero_floor.
    Element epsilon(static_cast<Element>(0.1f));
    Element nonzero_floor(std::numeric_limits<Element>::min());
```
- **EN:** Implements or wires together logic around `Factors`, `used`, `calculating`, `relative`, `equality` for the current test scenario.
- **CN:** 围绕 `Factors`, `used`, `calculating`, `relative`, `equality` 实现或连接当前测试场景所需的逻辑。

### Lines 503-515

```cpp
    if constexpr (!cutlass::is_complex<Element>::value) {
      if (check_relative_equality == CheckEquality::RELATIVE) {
        return cutlass::reference::host::TensorRelativelyEquals(
          lhs, rhs, epsilon, nonzero_floor);
      }
      else {
        return cutlass::reference::host::TensorEquals(lhs, rhs);
      }
    }
    else {
      return cutlass::reference::host::TensorEquals(lhs, rhs);
    }
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 517-520

```cpp
  bool compare_reference(
      ProblemShapeType problem_shapes, int batch) {
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_A[batch].host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_B[batch].host_view()), 0);
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 522-525

```cpp
    bool passed = true;
    return passed;
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 528-545

```cpp
//
// Block Scaled Gemm Input Operands : A , B, scalefactorA, scalefactorB
//
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  int AccumulatorPipelineStageCount_,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockScaledSm100<
                                SchedulerPipelineStageCount_,
                                AccumulatorPipelineStageCount_>,
                                Gemm, ElementA_, ElementB_> {
  // Kernel data types
  using ElementA = ElementA_;
  using StrideA  = typename Gemm::GemmKernel::StrideA;
  using InternalStrideA  = typename Gemm::GemmKernel::InternalStrideA;
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 546-551

```cpp
  using ElementB = ElementB_;
  using StrideB  = typename Gemm::GemmKernel::StrideB;
  using InternalStrideB  = typename Gemm::GemmKernel::InternalStrideB;
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  using LayoutTagA = cutlass::detail::StrideToLayoutTagA_t<StrideA>;
  using LayoutTagB = cutlass::detail::StrideToLayoutTagB_t<StrideB>;
```
- **EN:** Defines aliases such as `ElementB`, `StrideB`, `InternalStrideB`, `ScheduleType`, `LayoutTagA`, `LayoutTagB` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementB`, `StrideB`, `InternalStrideB`, `ScheduleType`, `LayoutTagA`, `LayoutTagB`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 553-553

```cpp
  static constexpr bool IsGroupGemm = !cute::is_same_v<StrideA, InternalStrideA>;
```
- **EN:** Declares member fields or local variables related to `IsGroupGemm`, `is_same_v`, `StrideA`, `InternalStrideA` for later setup, execution, or verification.
- **CN:** 声明与 `IsGroupGemm`, `is_same_v`, `StrideA`, `InternalStrideA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 555-558

```cpp
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using EpilogueOutputOp = typename Gemm::EpilogueOutputOp;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 560-560

```cpp
  static constexpr int SFVecSize = Gemm::GemmKernel::CollectiveMainloop::SFVecSize;
```
- **EN:** Declares member fields or local variables related to `SFVecSize`, `Gemm`, `GemmKernel`, `CollectiveMainloop` for later setup, execution, or verification.
- **CN:** 声明与 `SFVecSize`, `Gemm`, `GemmKernel`, `CollectiveMainloop` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 562-570

```cpp
  using ElementSF = typename Gemm::GemmKernel::CollectiveMainloop::ElementSF;
  using Sm1xxBlkScaledConfig =  typename Gemm::GemmKernel::CollectiveMainloop::Sm1xxBlkScaledConfig;
  using Blk_MN   = typename Sm1xxBlkScaledConfig::Blk_MN;
  using Blk_SF   = typename Sm1xxBlkScaledConfig::Blk_SF;
  using SfAtom   = typename Sm1xxBlkScaledConfig::SfAtom;
  using LayoutSFA = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFA;
  using InternalLayoutSFA = typename Gemm::GemmKernel::CollectiveMainloop::InternalLayoutSFA;
  using LayoutSFB = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFB;
  using InternalLayoutSFB = typename Gemm::GemmKernel::CollectiveMainloop::InternalLayoutSFB;
```
- **EN:** Defines aliases such as `ElementSF`, `Sm1xxBlkScaledConfig`, `Blk_MN`, `Blk_SF`, `SfAtom`, `LayoutSFA`, ... (+3) to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementSF`, `Sm1xxBlkScaledConfig`, `Blk_MN`, `Blk_SF`, `SfAtom`, `LayoutSFA`, ... (+3)，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 572-572

```cpp
  using Arguments = typename Gemm::GemmKernel::MainloopArguments;
```
- **EN:** Defines aliases such as `Arguments` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Arguments`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 574-575

```cpp
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
```
- **EN:** Implements or wires together logic around `Whether`, `use`, `relative`, `equality`, `checks` for the current test scenario.
- **CN:** 围绕 `Whether`, `use`, `relative`, `equality`, `checks` 实现或连接当前测试场景所需的逻辑。

### Lines 577-580

```cpp
  std::vector<InternalStrideA> stride_a_host;
  std::vector<InternalStrideB> stride_b_host;
  cutlass::DeviceAllocation<InternalStrideA> stride_a_device;
  cutlass::DeviceAllocation<InternalStrideB> stride_b_device;
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `InternalStrideA`, `stride_a_host`, `InternalStrideB` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `InternalStrideA`, `stride_a_host`, `InternalStrideB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 582-585

```cpp
  std::vector<InternalLayoutSFA> layout_sfa_host;
  std::vector<InternalLayoutSFB> layout_sfb_host;
  cutlass::DeviceAllocation<InternalLayoutSFA> layout_sfa_device;
  cutlass::DeviceAllocation<InternalLayoutSFB> layout_sfb_device;
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `InternalLayoutSFA`, `layout_sfa_host`, `InternalLayoutSFB` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `InternalLayoutSFA`, `layout_sfa_host`, `InternalLayoutSFB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 587-588

```cpp
  typename LayoutTagA::Stride stride_factor_A;
  typename LayoutTagB::Stride stride_factor_B;
```
- **EN:** Declares member fields or local variables related to `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 590-591

```cpp
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
```
- **EN:** Declares member fields or local variables related to `Distribution`, `Kind`, `init_A`, `init_B` for later setup, execution, or verification.
- **CN:** 声明与 `Distribution`, `Kind`, `init_A`, `init_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 593-596

```cpp
  std::vector<cutlass::HostTensor<ElementA, LayoutTagA>> tensors_A;
  std::vector<cutlass::HostTensor<ElementB, LayoutTagB>> tensors_B;
  std::vector<cutlass::HostTensor<ElementSF, LayoutTagA>> tensors_SFA;
  std::vector<cutlass::HostTensor<ElementSF, LayoutTagB>> tensors_SFB;
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `HostTensor`, `ElementA`, `LayoutTagA` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `HostTensor`, `ElementA`, `LayoutTagA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 598-601

```cpp
  cutlass::DeviceAllocation<const ElementA *> device_tensors_A;
  cutlass::DeviceAllocation<const ElementB *> device_tensors_B;
  cutlass::DeviceAllocation<const ElementSF *> device_tensors_SFA;
  cutlass::DeviceAllocation<const ElementSF *> device_tensors_SFB;
```
- **EN:** Declares member fields or local variables related to `DeviceAllocation`, `ElementA`, `device_tensors_A`, `ElementB`, `device_tensors_B` for later setup, execution, or verification.
- **CN:** 声明与 `DeviceAllocation`, `ElementA`, `device_tensors_A`, `ElementB`, `device_tensors_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 603-604

```cpp
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
```
- **EN:** Declares member fields or local variables related to `uint64_t`, `seed`, `kDefaultSeed` for later setup, execution, or verification.
- **CN:** 声明与 `uint64_t`, `seed`, `kDefaultSeed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 606-610

```cpp
  // Note: this limitation comes from testbed / not the library
  static_assert(is_row_or_col_major<InternalStrideA>(),
    "ERROR : A Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<InternalStrideB>(),
    "ERROR : B Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 612-623

```cpp
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = kDefaultSeed,
    typename LayoutTagA::Stride stride_factor_A_ = typename LayoutTagA::Stride(),
    typename LayoutTagB::Stride stride_factor_B_ = typename LayoutTagB::Stride()
  ):
    check_relative_equality(check_relative_equality_),
    stride_factor_A(stride_factor_A_),
    stride_factor_B(stride_factor_B_),
    init_A(init_A_), init_B(init_B_), seed(seed_) { }
```
- **EN:** Implements or wires together logic around `HostCollectiveMainloop`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution` for the current test scenario.
- **CN:** 围绕 `HostCollectiveMainloop`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution` 实现或连接当前测试场景所需的逻辑。

### Lines 625-629

```cpp
  template<class ProblemShapeType>
  bool initialize(ProblemShapeType problem_shapes) {
    //
    // Allocate the GEMM workspace
    //
```
- **EN:** Defines templated type `ProblemShapeType` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ProblemShapeType`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 631-638

```cpp
    tensors_A.clear();
    tensors_B.clear();
    stride_a_host.clear();
    stride_b_host.clear();
    tensors_SFA.clear();
    tensors_SFB.clear();
    layout_sfa_host.clear();
    layout_sfb_host.clear();
```
- **EN:** Declares member fields or local variables related to `tensors_A`, `clear`, `tensors_B`, `stride_a_host`, `stride_b_host` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_A`, `clear`, `tensors_B`, `stride_a_host`, `stride_b_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 640-641

```cpp
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = std::max(problem_shapes.groups(), L);
```
- **EN:** Declares member fields or local variables related to `append`, `problem_shapes`, `get_host_problem_shape`, `std`, `max` for later setup, execution, or verification.
- **CN:** 声明与 `append`, `problem_shapes`, `get_host_problem_shape`, `std`, `max` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 643-644

```cpp
    for (int32_t i = 0; i < L; ++i) {
      auto [M, N, K, mock_L] = cute::append<4>(problem_shapes.get_host_problem_shape(i), 1);
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 646-647

```cpp
      stride_a_host.push_back(cutlass::make_cute_packed_stride(InternalStrideA{}, {M, K, 1}));
      stride_b_host.push_back(cutlass::make_cute_packed_stride(InternalStrideB{}, {N, K, 1}));
```
- **EN:** Declares member fields or local variables related to `stride_a_host`, `push_back`, `make_cute_packed_stride`, `InternalStrideA`, `stride_b_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a_host`, `push_back`, `make_cute_packed_stride`, `InternalStrideA`, `stride_b_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 649-653

```cpp
      // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
      auto a_coord = cutlass::make_Coord(M, K);
      // Cutlass has Row/Col major refers to MxK times KxN matrix product,
      // so the HostTensorB should be treated as KxN in "coord"'s view
      auto b_coord = cutlass::make_Coord(K, N);
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 655-656

```cpp
      tensors_A.push_back(cutlass::HostTensor<ElementA, LayoutTagA>(a_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_coord, stride_factor_A)));
      tensors_B.push_back(cutlass::HostTensor<ElementB, LayoutTagB>(b_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(b_coord, stride_factor_B)));
```
- **EN:** Declares member fields or local variables related to `tensors_A`, `push_back`, `HostTensor`, `ElementA`, `LayoutTagA` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_A`, `push_back`, `HostTensor`, `ElementA`, `LayoutTagA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 658-659

```cpp
      EXPECT_TRUE(initialize_tensor(tensors_A[i].host_view(), init_A, seed + 2022 + i));
      EXPECT_TRUE(initialize_tensor(tensors_B[i].host_view(), init_B, seed + 2021 + i));
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `initialize_tensor`, `tensors_A`, `host_view`, `init_A` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `initialize_tensor`, `tensors_A`, `host_view`, `init_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 661-664

```cpp
      // It is possible to randomly initialize to all zeros, so override this with non-zeros
      // in the upper left corner of each operand.
      tensors_A[i].host_view().at({0, 0}) = ElementA(1);
      tensors_B[i].host_view().at({0, 0}) = ElementB(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 666-667

```cpp
      tensors_A[i].sync_device();
      tensors_B[i].sync_device();
```
- **EN:** Declares member fields or local variables related to `tensors_A`, `sync_device`, `tensors_B` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_A`, `sync_device`, `tensors_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 669-669

```cpp
      using namespace cute;
```
- **EN:** Defines aliases such as `namespace` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `namespace`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 671-675

```cpp
      auto k_blks = cutlass::ceil_div(K, size<1>(shape(SfAtom{})));
      auto m_blks = cutlass::ceil_div(M, Blk_MN{});
      auto n_blks = cutlass::ceil_div(N, Blk_MN{});
      layout_sfa_host.push_back(Sm1xxBlkScaledConfig::tile_atom_to_shape_SFA(cute::make_shape(M, N, K, 1)));
      layout_sfb_host.push_back(Sm1xxBlkScaledConfig::tile_atom_to_shape_SFB(cute::make_shape(M, N, K, 1)));
```
- **EN:** Declares member fields or local variables related to `k_blks`, `ceil_div`, `size`, `shape`, `SfAtom` for later setup, execution, or verification.
- **CN:** 声明与 `k_blks`, `ceil_div`, `size`, `shape`, `SfAtom` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 677-679

```cpp
      // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
      auto sfa_coord   = cutlass::make_Coord(m_blks * Blk_MN{}, k_blks * Blk_SF{});
      auto sfb_coord   = cutlass::make_Coord(n_blks * Blk_MN{}, k_blks * Blk_SF{});
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 681-682

```cpp
      tensors_SFA.push_back(cutlass::HostTensor<ElementSF, LayoutTagA>(sfa_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(sfa_coord, stride_factor_A)));
      tensors_SFB.push_back(cutlass::HostTensor<ElementSF, LayoutTagB>(sfb_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(sfb_coord, stride_factor_B)));
```
- **EN:** Declares member fields or local variables related to `tensors_SFA`, `push_back`, `HostTensor`, `ElementSF`, `LayoutTagA` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_SFA`, `push_back`, `HostTensor`, `ElementSF`, `LayoutTagA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 684-685

```cpp
      EXPECT_TRUE(initialize_tensor(tensors_SFA[i].host_view(), init_A, seed + 2024 + i));
      EXPECT_TRUE(initialize_tensor(tensors_SFB[i].host_view(), init_B, seed + 2025 + i));
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `initialize_tensor`, `tensors_SFA`, `host_view`, `init_A` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `initialize_tensor`, `tensors_SFA`, `host_view`, `init_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 687-690

```cpp
      // It is possible to randomly initialize to all zeros, so override this with non-zeros
      // in the upper left corner of each operand.
      tensors_SFA[i].host_view().at({0, 0}) = ElementSF(1);
      tensors_SFB[i].host_view().at({0, 0}) = ElementSF(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 692-694

```cpp
      tensors_SFA[i].sync_device();
      tensors_SFB[i].sync_device();
    }
```
- **EN:** Implements or wires together logic around `tensors_SFA`, `sync_device`, `tensors_SFB` for the current test scenario.
- **CN:** 围绕 `tensors_SFA`, `sync_device`, `tensors_SFB` 实现或连接当前测试场景所需的逻辑。

### Lines 696-697

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 699-701

```cpp
  Arguments to_args(ProblemShapeType problem_shapes) {
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = std::max(problem_shapes.groups(), L);
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 703-706

```cpp
    std::vector<ElementA *> ptr_A_host(L);
    std::vector<ElementB *> ptr_B_host(L);
    std::vector<ElementSF *> ptr_SFA_host(L);
    std::vector<ElementSF *> ptr_SFB_host(L);
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `ElementA`, `ptr_A_host`, `ElementB` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `ElementA`, `ptr_A_host`, `ElementB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 708-713

```cpp
    for (int32_t i = 0; i < L; ++i) {
      ptr_A_host.at(i) = tensors_A[i].device_data();
      ptr_B_host.at(i) = tensors_B[i].device_data();
      ptr_SFA_host.at(i) = tensors_SFA[i].device_data();
      ptr_SFB_host.at(i) = tensors_SFB[i].device_data();
    }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 715-716

```cpp
    device_tensors_A.reset(L);
    device_tensors_A.copy_from_host(ptr_A_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_A`, `reset`, `copy_from_host`, `ptr_A_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_A`, `reset`, `copy_from_host`, `ptr_A_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 718-719

```cpp
    device_tensors_B.reset(L);
    device_tensors_B.copy_from_host(ptr_B_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_B`, `reset`, `copy_from_host`, `ptr_B_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_B`, `reset`, `copy_from_host`, `ptr_B_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 721-722

```cpp
    device_tensors_SFA.reset(L);
    device_tensors_SFA.copy_from_host(ptr_SFA_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_SFA`, `reset`, `copy_from_host`, `ptr_SFA_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_SFA`, `reset`, `copy_from_host`, `ptr_SFA_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 724-725

```cpp
    device_tensors_SFB.reset(L);
    device_tensors_SFB.copy_from_host(ptr_SFB_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_SFB`, `reset`, `copy_from_host`, `ptr_SFB_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_SFB`, `reset`, `copy_from_host`, `ptr_SFB_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 727-728

```cpp
    stride_a_device.reset(problem_shapes.groups());
    stride_a_device.copy_from_host(stride_a_host.data());
```
- **EN:** Declares member fields or local variables related to `stride_a_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 730-731

```cpp
    stride_b_device.reset(problem_shapes.groups());
    stride_b_device.copy_from_host(stride_b_host.data());
```
- **EN:** Declares member fields or local variables related to `stride_b_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_b_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 733-734

```cpp
    layout_sfa_device.reset(problem_shapes.groups());
    layout_sfa_device.copy_from_host(layout_sfa_host.data());
```
- **EN:** Declares member fields or local variables related to `layout_sfa_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `layout_sfa_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 736-737

```cpp
    layout_sfb_device.reset(problem_shapes.groups());
    layout_sfb_device.copy_from_host(layout_sfb_host.data());
```
- **EN:** Declares member fields or local variables related to `layout_sfb_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `layout_sfb_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 739-755

```cpp
    if constexpr (IsGroupGemm) {
      return Arguments{
        device_tensors_A.get(), stride_a_device.get(),
        device_tensors_B.get(), stride_b_device.get(),
        device_tensors_SFA.get(), layout_sfa_device.get(),
        device_tensors_SFB.get(), layout_sfb_device.get()
      };
    }
    else {
      return Arguments{
        device_tensors_A.get(), stride_a_host[0],
        device_tensors_B.get(), stride_b_host[0],
        device_tensors_SFA.get(), layout_sfa_host[0],
        device_tensors_SFB.get(), layout_sfb_host[0]
      };
    }
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 757-765

```cpp
  auto to_host_args(ProblemShapeType problem_shapes, int batch) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(batch), 1);
    auto A = make_tensor(make_iterator(tensors_A[batch].host_data()),
          make_layout(make_shape(M, K, 1), stride_a_host[batch]));
    auto SfA = make_tensor(tensors_SFA[batch].host_data(), layout_sfa_host[batch]);
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 767-769

```cpp
    auto B = make_tensor(make_iterator(tensors_B[batch].host_data()),
        make_layout(make_shape(N, K, 1), stride_b_host[batch]));
    auto SfB = make_tensor(tensors_SFB[batch].host_data(), layout_sfb_host[batch]);
```
- **EN:** Implements or wires together logic around `make_tensor`, `make_iterator`, `tensors_B`, `batch`, `host_data` for the current test scenario.
- **CN:** 围绕 `make_tensor`, `make_iterator`, `tensors_B`, `batch`, `host_data` 实现或连接当前测试场景所需的逻辑。

### Lines 771-778

```cpp
    return cutlass::reference::host::GettMainloopParams<ElementAccumulator,
        decltype(A),
        decltype(B),
        decltype(SfA),
        decltype(SfB)
      >
      {A, SfA, B, SfB};
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 780-785

```cpp
  void print_tensors(std::ofstream& file, int batch) {
    file << "A =\n" << tensors_A[batch].host_view()
         << "\nB =\n" << tensors_B[batch].host_view()
         << "\nSFA =\n" << tensors_SFA[batch].host_view()
         << "\nSFB =\n" << tensors_SFB[batch].host_view();
  }
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 787-788

```cpp
  bool compare_reference(
      ProblemShapeType problem_shapes, int batch) {
```
- **EN:** Implements or wires together logic around `compare_reference`, `ProblemShapeType`, `problem_shapes`, `batch` for the current test scenario.
- **CN:** 围绕 `compare_reference`, `ProblemShapeType`, `problem_shapes`, `batch` 实现或连接当前测试场景所需的逻辑。

### Lines 790-796

```cpp
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_A[batch].host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_B[batch].host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_SFA[batch].host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_SFB[batch].host_view()), 0);
    return true;
  }
};
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 798-815

```cpp
//
// Block Scaled Gemm Input Operands : A , B, scalefactorA, scalefactorB
//
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongBlockScaledSm120<SchedulerPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockScaledSm100<0,0>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockScaledSm100<0,0>,
                                      Gemm, ElementA_, ElementB_>;
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 816-821

```cpp
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = Base::kDefaultSeed,
    typename Base::LayoutTagA::Stride stride_factor_A_ = typename Base::LayoutTagA::Stride(),
    typename Base::LayoutTagB::Stride stride_factor_B_ = typename Base::LayoutTagB::Stride()
  ) : Base::HostCollectiveMainloop(check_relative_equality_, init_A_, init_B_, seed_, stride_factor_A_, stride_factor_B_) {}
};
```
- **EN:** Implements or wires together logic around `Distribution`, `Kind`, `init_B_`, `Uniform`, `uint64_t` for the current test scenario.
- **CN:** 围绕 `Distribution`, `Kind`, `init_B_`, `Uniform`, `uint64_t` 实现或连接当前测试场景所需的逻辑。

### Lines 823-840

```cpp
//
// Block Scaled Gemm Input Operands : A , B, scalefactorA, scalefactorB
//
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeBlockScaledSm120<SchedulerPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockScaledSm100<0,0>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockScaledSm100<0,0>,
                                      Gemm, ElementA_, ElementB_>;
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 841-846

```cpp
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = Base::kDefaultSeed,
    typename Base::LayoutTagA::Stride stride_factor_A_ = typename Base::LayoutTagA::Stride(),
    typename Base::LayoutTagB::Stride stride_factor_B_ = typename Base::LayoutTagB::Stride()
  ) : Base::HostCollectiveMainloop(check_relative_equality_, init_A_, init_B_, seed_, stride_factor_A_, stride_factor_B_) {}
};
```
- **EN:** Implements or wires together logic around `Distribution`, `Kind`, `init_B_`, `Uniform`, `uint64_t` for the current test scenario.
- **CN:** 围绕 `Distribution`, `Kind`, `init_B_`, `Uniform`, `uint64_t` 实现或连接当前测试场景所需的逻辑。

### Lines 848-865

```cpp
//
// Block Scaled Gemm Input Operands : A , B, scalefactorA, scalefactorB
//
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  int AccumulatorPipelineStageCount_,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockScaledSm103<SchedulerPipelineStageCount_,
                                                                                              AccumulatorPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_,AccumulatorPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelPtrArrayTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_,AccumulatorPipelineStageCount_>,
                                      Gemm, ElementA_, ElementB_>;
  HostCollectiveMainloop(
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 866-873

```cpp
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = Base::kDefaultSeed,
    typename Base::LayoutTagA::Stride stride_factor_A_ = typename Base::LayoutTagA::Stride(),
    typename Base::LayoutTagB::Stride stride_factor_B_ = typename Base::LayoutTagB::Stride()
  ) : Base::HostCollectiveMainloop(check_relative_equality_, init_A_, init_B_, seed_, stride_factor_A_, stride_factor_B_) {}
};
```
- **EN:** Implements or wires together logic around `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution`, `Kind` for the current test scenario.
- **CN:** 围绕 `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution`, `Kind` 实现或连接当前测试场景所需的逻辑。

### Lines 875-880

```cpp
template<class Gemm>
struct HostCollectiveDefaultEpilogue {
  // fusion types are potentially void if the fusion is not supported
  // helper so we don't try to construct HostTensor with void type
  template <typename T, typename U = uint8_t>
  using non_void_t = cute::conditional_t<cute::is_void_v<T>, U, T>;
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 882-884

```cpp
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  using kernel   = typename Gemm::GemmKernel;
  using Epilogue = typename kernel::CollectiveEpilogue;
```
- **EN:** Defines aliases such as `ScheduleType`, `kernel`, `Epilogue` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ScheduleType`, `kernel`, `Epilogue`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 886-891

```cpp
  using ElementD = typename kernel::ElementD;
  using StrideD  = typename kernel::StrideD;
  using InternalStrideD  = typename kernel::InternalStrideD;
  using ElementC = non_void_t<typename kernel::ElementC, ElementD>;
  using StrideC  = typename kernel::StrideC;
  using InternalStrideC  = typename kernel::InternalStrideC;
```
- **EN:** Defines aliases such as `ElementD`, `StrideD`, `InternalStrideD`, `ElementC`, `StrideC`, `InternalStrideC` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementD`, `StrideD`, `InternalStrideD`, `ElementC`, `StrideC`, `InternalStrideC`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 893-893

```cpp
  static constexpr bool IsGroupGemm = !cute::is_same_v<StrideD, InternalStrideD>;
```
- **EN:** Declares member fields or local variables related to `IsGroupGemm`, `is_same_v`, `StrideD`, `InternalStrideD` for later setup, execution, or verification.
- **CN:** 声明与 `IsGroupGemm`, `is_same_v`, `StrideD`, `InternalStrideD` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 895-895

```cpp
  using FusionOp = typename Gemm::EpilogueOutputOp;
```
- **EN:** Defines aliases such as `FusionOp` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `FusionOp`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 897-898

```cpp
  static_assert(rank(InternalStrideC{}) == 3, "StrideCD must be rank-3: [M, N, L]");
  static_assert(rank(InternalStrideD{}) == 3, "StrideCD must be rank-3: [M, N, L]");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 900-903

```cpp
  static_assert(is_row_or_col_major<InternalStrideC>(),
    "ERROR : C Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<InternalStrideD>(),
    "ERROR : D Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 905-909

```cpp
  // Deduce Cutlass Layouts (RowMajor & ColumnMajor)
  using LayoutTagC = cutlass::detail::StrideToLayoutTagC_t<StrideC>;
  using LayoutTagD = cutlass::detail::StrideToLayoutTagC_t<StrideD>;
  using LayoutTagScalar = cutlass::layout::PackedVectorLayout; // scalars are size-1 vectors
  using LayoutTagVector = cutlass::layout::PackedVectorLayout;
```
- **EN:** Begins function or method `Layouts`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `Layouts`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 911-915

```cpp
  using ElementAccumulator = typename kernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename kernel::ProblemShape;
  using ElementCompute = typename ElementComputeType<Gemm, ElementAccumulator>::Type;
  using ElementScalar = typename ElementScalarType<Gemm, ElementCompute>::Type;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `ElementCompute`, `ElementScalar` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `ElementCompute`, `ElementScalar`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 917-917

```cpp
  using Arguments = typename Gemm::GemmKernel::EpilogueArguments;
```
- **EN:** Defines aliases such as `Arguments` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Arguments`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 919-921

```cpp
  /// Initialization
  cutlass::DeviceAllocation<InternalStrideC> stride_c_device;
  cutlass::DeviceAllocation<InternalStrideD> stride_d_device;
```
- **EN:** Implements or wires together logic around `Initialization`, `DeviceAllocation`, `InternalStrideC`, `stride_c_device`, `InternalStrideD` for the current test scenario.
- **CN:** 围绕 `Initialization`, `DeviceAllocation`, `InternalStrideC`, `stride_c_device`, `InternalStrideD` 实现或连接当前测试场景所需的逻辑。

### Lines 923-924

```cpp
  std::vector<InternalStrideC> stride_c_host;
  std::vector<InternalStrideD> stride_d_host;
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `InternalStrideC`, `stride_c_host`, `InternalStrideD` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `InternalStrideC`, `stride_c_host`, `InternalStrideD` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 926-927

```cpp
  typename LayoutTagC::Stride stride_factor_C;
  typename LayoutTagD::Stride stride_factor_D;
```
- **EN:** Declares member fields or local variables related to `LayoutTagC`, `Stride`, `stride_factor_C`, `LayoutTagD`, `stride_factor_D` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagC`, `Stride`, `stride_factor_C`, `LayoutTagD`, `stride_factor_D` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 929-931

```cpp
  // Inputs
  ElementScalar alpha;
  ElementScalar beta;
```
- **EN:** Implements or wires together logic around `Inputs`, `ElementScalar`, `alpha`, `beta` for the current test scenario.
- **CN:** 围绕 `Inputs`, `ElementScalar`, `alpha`, `beta` 实现或连接当前测试场景所需的逻辑。

### Lines 933-937

```cpp
  std::vector<cutlass::HostTensor<ElementC, LayoutTagC>> tensors_C;
  std::vector<cutlass::HostTensor<ElementD, LayoutTagD>> tensors_D;
  std::vector<cutlass::HostTensor<ElementD, LayoutTagD>> references_D;
  cutlass::DeviceAllocation<const ElementC *> device_tensors_C;
  cutlass::DeviceAllocation<ElementD *> device_tensors_D;
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `HostTensor`, `ElementC`, `LayoutTagC` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `HostTensor`, `ElementC`, `LayoutTagC` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 939-944

```cpp
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
  // Are scalars copied to device memory before kernel launch
  ScalarLoc use_device_scalars = ScalarLoc::ON_HOST;
  // If per-row scale is enabled and this is disabled, alpha/beta are passed as a host or device scalar instead of device vector
  VectorScale vector_scale_mode = VectorScale::DISABLED;
```
- **EN:** Implements or wires together logic around `Whether`, `use`, `relative`, `equality`, `checks` for the current test scenario.
- **CN:** 围绕 `Whether`, `use`, `relative`, `equality`, `checks` 实现或连接当前测试场景所需的逻辑。

### Lines 946-948

```cpp
  cutlass::Distribution::Kind init_C;
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
```
- **EN:** Declares member fields or local variables related to `Distribution`, `Kind`, `init_C`, `uint64_t`, `seed` for later setup, execution, or verification.
- **CN:** 声明与 `Distribution`, `Kind`, `init_C`, `uint64_t`, `seed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 950-962

```cpp
  HostCollectiveDefaultEpilogue(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    ScalarLoc use_device_scalars_ = ScalarLoc::ON_HOST,
    VectorScale vector_scale_mode_ = VectorScale::DISABLED,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_scale_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_bias_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = kDefaultSeed
  ): init_C(init_C_), seed(seed_),
     stride_factor_C(typename LayoutTagC::Stride()),
     stride_factor_D(typename LayoutTagD::Stride()),
     check_relative_equality(check_relative_equality_),
     use_device_scalars(use_device_scalars_){ }
```
- **EN:** Implements or wires together logic around `HostCollectiveDefaultEpilogue`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `ScalarLoc` for the current test scenario.
- **CN:** 围绕 `HostCollectiveDefaultEpilogue`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `ScalarLoc` 实现或连接当前测试场景所需的逻辑。

### Lines 964-965

```cpp
  bool initialize(ProblemShapeType problem_shapes, ElementScalar alpha_=1.f, ElementScalar beta_=0.f) {
    // Initialize Epilogue tensors
```
- **EN:** Begins function or method `initialize`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `initialize`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 967-971

```cpp
    tensors_C.clear();
    tensors_D.clear();
    references_D.clear();
    stride_c_host.clear();
    stride_d_host.clear();
```
- **EN:** Declares member fields or local variables related to `tensors_C`, `clear`, `tensors_D`, `references_D`, `stride_c_host` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_C`, `clear`, `tensors_D`, `references_D`, `stride_c_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 973-974

```cpp
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = cutlass::platform::max(problem_shapes.groups(), L);
```
- **EN:** Declares member fields or local variables related to `append`, `problem_shapes`, `get_host_problem_shape`, `platform`, `max` for later setup, execution, or verification.
- **CN:** 声明与 `append`, `problem_shapes`, `get_host_problem_shape`, `platform`, `max` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 976-977

```cpp
    for (int32_t i = 0; i < L; ++i) {
      auto [M, N, K, mock_L] = cute::append<4>(problem_shapes.get_host_problem_shape(i), 1);
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 979-980

```cpp
      stride_c_host.push_back(cutlass::make_cute_packed_stride(InternalStrideC{}, {M, N, 1}));
      stride_d_host.push_back(cutlass::make_cute_packed_stride(InternalStrideD{}, {M, N, 1}));
```
- **EN:** Declares member fields or local variables related to `stride_c_host`, `push_back`, `make_cute_packed_stride`, `InternalStrideC`, `stride_d_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_c_host`, `push_back`, `make_cute_packed_stride`, `InternalStrideC`, `stride_d_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 982-983

```cpp
      // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
      auto c_coord = cutlass::make_Coord(M, N);
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 985-989

```cpp
      tensors_C.push_back(cutlass::HostTensor<ElementC, LayoutTagC>(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagC>::layout_factory(c_coord, stride_factor_C)));
      tensors_D.push_back(cutlass::HostTensor<ElementD, LayoutTagD>(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, stride_factor_D)));
      references_D.push_back(cutlass::HostTensor<ElementD, LayoutTagD>(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, stride_factor_D), false));
      EXPECT_TRUE(initialize_tensor(tensors_C[i].host_view(), init_C, seed + 2020));
      tensors_C[i].host_view().at({0, 0}) = ElementC(1);
```
- **EN:** Declares member fields or local variables related to `tensors_C`, `push_back`, `HostTensor`, `ElementC`, `LayoutTagC` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_C`, `push_back`, `HostTensor`, `ElementC`, `LayoutTagC` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 991-996

```cpp
      cutlass::reference::host::TensorCopy(references_D[i].host_view(), tensors_C[i].host_view());
      tensors_C[i].sync_device();
      tensors_D[i].sync_device();
    }
    alpha = alpha_;
    beta = beta_;
```
- **EN:** Implements or wires together logic around `reference`, `host`, `TensorCopy`, `references_D`, `host_view` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `TensorCopy`, `references_D`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 998-999

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1001-1007

```cpp
  template <
    class Element,
    class Layout
  >
  bool equality_check(
    cutlass::TensorView<Element, Layout> const& lhs,
    cutlass::TensorView<Element, Layout> const& rhs) const {
```
- **EN:** Defines templated type `Element` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Element`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1009-1014

```cpp
    // Factors used for calculating relative equality. CUTLASS's relative-equality
    // checks in include/cutlass/relatively_equal.h  are inspired by
    // https://floating-point-gui.de/errors/comparison/. This reference suggests using
    // the minimum normal value of a given type as the nonzero_floor.
    Element epsilon(static_cast<Element>(0.1f));
    Element nonzero_floor(std::numeric_limits<Element>::min());
```
- **EN:** Implements or wires together logic around `Factors`, `used`, `calculating`, `relative`, `equality` for the current test scenario.
- **CN:** 围绕 `Factors`, `used`, `calculating`, `relative`, `equality` 实现或连接当前测试场景所需的逻辑。

### Lines 1016-1028

```cpp
    if constexpr (!cutlass::is_complex<Element>::value) {
      if (check_relative_equality == CheckEquality::RELATIVE) {
        return cutlass::reference::host::TensorRelativelyEquals(
          lhs, rhs, epsilon, nonzero_floor);
      }
      else {
        return cutlass::reference::host::TensorEquals(lhs, rhs);
      }
    }
    else {
      return cutlass::reference::host::TensorEquals(lhs, rhs);
    }
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1030-1036

```cpp
  bool compare_reference(
      ProblemShapeType problem_shapes,
      ElementScalar alpha,
      ElementScalar beta,
      int batch) {
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = cutlass::platform::max(problem_shapes.groups(), L);
```
- **EN:** Implements or wires together logic around `compare_reference`, `ProblemShapeType`, `problem_shapes`, `ElementScalar`, `alpha` for the current test scenario.
- **CN:** 围绕 `compare_reference`, `ProblemShapeType`, `problem_shapes`, `ElementScalar`, `alpha` 实现或连接当前测试场景所需的逻辑。

### Lines 1038-1039

```cpp
    tensors_D[batch].sync_host();
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_C[batch].host_view()), 0);
```
- **EN:** Declares member fields or local variables related to `tensors_D`, `batch`, `sync_host`, `EXPECT_GT`, `reference` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_D`, `batch`, `sync_host`, `EXPECT_GT`, `reference` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1041-1043

```cpp
    if (tensors_D[batch].size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_D[batch].host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1045-1047

```cpp
    if (references_D[batch].size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(references_D[batch].host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1049-1054

```cpp
    bool passed = equality_check(references_D[batch].host_view(), tensors_D[batch].host_view());
    if(!passed) {
      std::cout<<"D is incorrect"<<std::endl;
    }
    return passed;
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1056-1061

```cpp
  void print_tensors(std::ofstream& file, int batch) {
    file
    << "\nC =\n" << tensors_C[batch].host_view()
    << "\n\nReference =\n" << references_D[batch].host_view()
    << "\n\nComputed =\n" << tensors_D[batch].host_view();
  }
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1063-1065

```cpp
  Arguments to_args(ProblemShapeType problem_shapes) {
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = cutlass::platform::max(problem_shapes.groups(), L);
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1067-1068

```cpp
    std::vector<ElementC *> ptr_C_host(L);
    std::vector<ElementD *> ptr_D_host(L);
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `ElementC`, `ptr_C_host`, `ElementD` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `ElementC`, `ptr_C_host`, `ElementD` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1070-1073

```cpp
    for (int32_t i = 0; i < L; ++i) {
      ptr_C_host.at(i) = tensors_C[i].device_data();
      ptr_D_host.at(i) = tensors_D[i].device_data();
    }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 1075-1076

```cpp
    device_tensors_C.reset(L);
    device_tensors_C.copy_from_host(ptr_C_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_C`, `reset`, `copy_from_host`, `ptr_C_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_C`, `reset`, `copy_from_host`, `ptr_C_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1078-1079

```cpp
    device_tensors_D.reset(L);
    device_tensors_D.copy_from_host(ptr_D_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_D`, `reset`, `copy_from_host`, `ptr_D_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_D`, `reset`, `copy_from_host`, `ptr_D_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1081-1082

```cpp
    stride_c_device.reset(problem_shapes.groups());
    stride_c_device.copy_from_host(stride_c_host.data());
```
- **EN:** Declares member fields or local variables related to `stride_c_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_c_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1084-1085

```cpp
    stride_d_device.reset(problem_shapes.groups());
    stride_d_device.copy_from_host(stride_d_host.data());
```
- **EN:** Declares member fields or local variables related to `stride_d_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_d_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1087-1101

```cpp
    Arguments arguments;
    if constexpr (IsGroupGemm) {
      arguments =
      {
        {alpha, beta},
        device_tensors_C.get(), stride_c_device.get(), device_tensors_D.get(), stride_d_device.get()
      };
    }
    else {
      arguments =
      {
        {alpha, beta},
        device_tensors_C.get(), stride_c_host[0], device_tensors_D.get(), stride_d_host[0]
      };
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1103-1104

```cpp
    return arguments;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1106-1112

```cpp
  auto to_host_args(ProblemShapeType problem_shapes, int batch) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(batch), 1);
    L = std::max(problem_shapes.groups(), L);
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1114-1118

```cpp
    auto coord_0 = cutlass::make_Coord(0);
    auto C = cute::make_tensor(detail::make_iterator(tensors_C[batch].host_data()),
        cute::make_layout(cute::make_shape(M, N, 1), stride_c_host[batch]));
    auto D = cute::make_tensor(detail::make_iterator(references_D[batch].host_data()),
        cute::make_layout(cute::make_shape(M, N, 1), stride_d_host[batch]));
```
- **EN:** Implements or wires together logic around `coord_0`, `make_Coord`, `make_tensor`, `detail`, `make_iterator` for the current test scenario.
- **CN:** 围绕 `coord_0`, `make_Coord`, `make_tensor`, `detail`, `make_iterator` 实现或连接当前测试场景所需的逻辑。

### Lines 1120-1127

```cpp
    cutlass::reference::host::GettEpilogueParams<
      ElementScalar,
      ElementScalar,
      ElementAccumulator,
      ElementCompute,
      decltype(C),
      decltype(D)>
        epilogue_params{};
```
- **EN:** Implements or wires together logic around `reference`, `host`, `GettEpilogueParams`, `ElementScalar`, `ElementAccumulator` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `GettEpilogueParams`, `ElementScalar`, `ElementAccumulator` 实现或连接当前测试场景所需的逻辑。

### Lines 1129-1132

```cpp
    epilogue_params.C = C;
    epilogue_params.D = D;
    epilogue_params.alpha = alpha;
    epilogue_params.beta = beta;
```
- **EN:** Declares member fields or local variables related to `epilogue_params`, `alpha`, `beta` for later setup, execution, or verification.
- **CN:** 声明与 `epilogue_params`, `alpha`, `beta` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1134-1136

```cpp
    return epilogue_params;
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1138-1143

```cpp
template<class Gemm>
struct HostCollectiveEpilogue {
  // fusion types are potentially void if the fusion is not supported
  // helper so we don't try to construct HostTensor with void type
  template <typename T, typename U = uint8_t>
  using non_void_t = cute::conditional_t<cute::is_void_v<T>, U, T>;
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1145-1148

```cpp
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  using kernel   = typename Gemm::GemmKernel;
  using Epilogue = typename kernel::CollectiveEpilogue;
  static_assert(IsDefaultEpilogue<Epilogue>::value == false, "Default Epilogue is not supported");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1150-1155

```cpp
  using ElementD = typename kernel::ElementD;
  using StrideD  = typename kernel::StrideD;
  using InternalStrideD  = typename kernel::InternalStrideD;
  using ElementC = non_void_t<typename kernel::ElementC, ElementD>;
  using StrideC  = typename kernel::StrideC;
  using InternalStrideC  = typename kernel::InternalStrideC;
```
- **EN:** Defines aliases such as `ElementD`, `StrideD`, `InternalStrideD`, `ElementC`, `StrideC`, `InternalStrideC` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementD`, `StrideD`, `InternalStrideD`, `ElementC`, `StrideC`, `InternalStrideC`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1157-1157

```cpp
  static constexpr bool IsGroupGemm = !cute::is_same_v<StrideD, InternalStrideD>;
```
- **EN:** Declares member fields or local variables related to `IsGroupGemm`, `is_same_v`, `StrideD`, `InternalStrideD` for later setup, execution, or verification.
- **CN:** 声明与 `IsGroupGemm`, `is_same_v`, `StrideD`, `InternalStrideD` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1159-1160

```cpp
  static_assert(rank(InternalStrideC{}) == 3, "StrideCD must be rank-3: [M, N, L]");
  static_assert(rank(InternalStrideD{}) == 3, "StrideCD must be rank-3: [M, N, L]");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1162-1165

```cpp
  static_assert(is_row_or_col_major<InternalStrideC>(),
    "ERROR : C Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<InternalStrideD>(),
    "ERROR : D Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1167-1171

```cpp
  // Deduce Cutlass Layouts (RowMajor & ColumnMajor)
  using LayoutTagC = cutlass::detail::StrideToLayoutTagC_t<StrideC>;
  using LayoutTagD = cutlass::detail::StrideToLayoutTagC_t<StrideD>;
  using LayoutTagScalar = cutlass::layout::PackedVectorLayout; // scalars are size-1 vectors
  using LayoutTagVector = cutlass::layout::PackedVectorLayout;
```
- **EN:** Begins function or method `Layouts`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `Layouts`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1173-1175

```cpp
  using ElementAccumulator = typename kernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename kernel::ProblemShape;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1177-1180

```cpp
  //
  // FusionOperation derived types/queries
  //
  static constexpr bool IsLegacy = detail::IsLegacyEpiloguePolicy<Epilogue>::value;
```
- **EN:** Implements or wires together logic around `FusionOperation`, `derived`, `types`, `queries`, `IsLegacy` for the current test scenario.
- **CN:** 围绕 `FusionOperation`, `derived`, `types`, `queries`, `IsLegacy` 实现或连接当前测试场景所需的逻辑。

### Lines 1182-1183

```cpp
  using FusionOp = typename Gemm::EpilogueOutputOp;
  static_assert(cute::is_base_of_v<cutlass::epilogue::fusion::FusionOperation, FusionOp>);
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1186-1200

```cpp
  // Scale factor Generation related
  using SfStrategy = cutlass::reference::host::SfStrategy;
  static constexpr bool IsBlockScaleSupported            = FusionOp::IsBlockScaleSupported;
  static constexpr SfStrategy SfGenStrategy              = (!IsBlockScaleSupported) ? SfStrategy::None : SfStrategy::SfDGen;
  static constexpr int32_t SFD_VectorSize = IsBlockScaleSupported ? FusionOp::SFVecSize : 1;
  using ElementSFD = non_void_t<cute::remove_pointer_t<typename FusionOp::ElementBlockScaleFactor>, ElementD>;
  using Sm1xxBlockScaledOutputConfig= cutlass::detail::Sm1xxBlockScaledOutputConfig<
                                          SFD_VectorSize
                                        >;
  using Blk_MN = typename Sm1xxBlockScaledOutputConfig::Blk_MN;
  using Blk_SF = typename Sm1xxBlockScaledOutputConfig::Blk_SF;
  using OutputSFAtom = typename Sm1xxBlockScaledOutputConfig::SfAtom;
  std::vector<cutlass::HostTensor<ElementSFD, LayoutTagD>> tensors_SFD;
  std::vector<cutlass::HostTensor<ElementSFD, LayoutTagD>> references_SFD;
  cutlass::DeviceAllocation<ElementSFD *> device_tensors_SFD;
```
- **EN:** Introduces type aliases like `SfStrategy`, `ElementSFD`, `Sm1xxBlockScaledOutputConfig`, `Blk_MN`, `Blk_SF`, `OutputSFAtom` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `SfStrategy`, `ElementSFD`, `Sm1xxBlockScaledOutputConfig`, `Blk_MN`, `Blk_SF`, `OutputSFAtom`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 1202-1209

```cpp
  using ElementCompute    = typename FusionOp::ElementCompute;
  using ElementScalar     = typename FusionOp::ElementScalar;
  using ElementBias       = non_void_t<typename FusionOp::ElementBias>;
  using ElementAux        = non_void_t<typename FusionOp::ElementAux>;
  using ElementAmax       = non_void_t<typename FusionOp::ElementAmax>;
  using LayoutTagAux      = non_void_t<typename FusionOp::GmemLayoutTagAux, LayoutTagD>;
  using ActivationFunctor = non_void_t<typename FusionOp::ActivationFn,
                              cutlass::epilogue::thread::Identity<ElementCompute>>;
```
- **EN:** Introduces type aliases like `ElementCompute`, `ElementScalar`, `ElementBias`, `ElementAux`, `ElementAmax`, `LayoutTagAux`, ... (+1) so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ElementCompute`, `ElementScalar`, `ElementBias`, `ElementAux`, `ElementAmax`, `LayoutTagAux`, ... (+1)，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 1211-1222

```cpp
  static constexpr bool IsBiasEnabled        = FusionOp::IsPerRowBiasSupported;
  static constexpr bool IsDeBiasEnabled      = FusionOp::IsDePerRowBiasSupported;
  static constexpr bool IsPerRowScaleEnabled = FusionOp::IsPerRowScaleSupported;
  static constexpr bool IsScaleFactorEnabled = FusionOp::IsScaleFactorSupported;
  static constexpr bool IsAuxInEnabled       = FusionOp::IsAuxInSupported;
  static constexpr bool IsAuxOutEnabled      = FusionOp::IsAuxOutSupported;
  static constexpr bool IsAbsMaxEnabledD     = FusionOp::IsAbsMaxSupported &&
                                                (cute::is_same_v<ElementD, cutlass::float_e4m3_t> ||
                                                 cute::is_same_v<ElementD, cutlass::float_e5m2_t>);
  static constexpr bool IsAbsMaxEnabledAux   = IsAuxOutEnabled && FusionOp::IsAbsMaxSupported &&
                                                (cute::is_same_v<ElementAux, cutlass::float_e4m3_t> ||
                                                 cute::is_same_v<ElementAux, cutlass::float_e5m2_t>);
```
- **EN:** Implements or wires together logic around `IsBiasEnabled`, `FusionOp`, `IsPerRowBiasSupported`, `IsDeBiasEnabled`, `IsDePerRowBiasSupported` for the current test scenario.
- **CN:** 围绕 `IsBiasEnabled`, `FusionOp`, `IsPerRowBiasSupported`, `IsDeBiasEnabled`, `IsDePerRowBiasSupported` 实现或连接当前测试场景所需的逻辑。

### Lines 1224-1224

```cpp
  using Arguments = typename Gemm::GemmKernel::EpilogueArguments;
```
- **EN:** Defines aliases such as `Arguments` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Arguments`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1226-1228

```cpp
  /// Initialization
  cutlass::DeviceAllocation<InternalStrideC> stride_c_device;
  cutlass::DeviceAllocation<InternalStrideD> stride_d_device;
```
- **EN:** Implements or wires together logic around `Initialization`, `DeviceAllocation`, `InternalStrideC`, `stride_c_device`, `InternalStrideD` for the current test scenario.
- **CN:** 围绕 `Initialization`, `DeviceAllocation`, `InternalStrideC`, `stride_c_device`, `InternalStrideD` 实现或连接当前测试场景所需的逻辑。

### Lines 1230-1231

```cpp
  std::vector<InternalStrideC> stride_c_host;
  std::vector<InternalStrideD> stride_d_host;
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `InternalStrideC`, `stride_c_host`, `InternalStrideD` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `InternalStrideC`, `stride_c_host`, `InternalStrideD` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1233-1234

```cpp
  typename LayoutTagC::Stride stride_factor_C;
  typename LayoutTagD::Stride stride_factor_D;
```
- **EN:** Declares member fields or local variables related to `LayoutTagC`, `Stride`, `stride_factor_C`, `LayoutTagD`, `stride_factor_D` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagC`, `Stride`, `stride_factor_C`, `LayoutTagD`, `stride_factor_D` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1236-1247

```cpp
  // Inputs
  cutlass::HostTensor<ElementScalar, LayoutTagScalar> alpha;
  cutlass::HostTensor<ElementScalar, LayoutTagScalar> beta;
  cutlass::HostTensor<ElementScalar, LayoutTagScalar> scale_A;
  cutlass::HostTensor<ElementScalar, LayoutTagScalar> scale_B;
  cutlass::HostTensor<ElementScalar, LayoutTagScalar> scale_C;
  cutlass::HostTensor<ElementScalar, LayoutTagScalar> scale_D;
  cutlass::HostTensor<ElementScalar, LayoutTagScalar> scale_Aux;
  cutlass::HostTensor<ElementBias  , LayoutTagVector> bias;
  std::vector<cutlass::HostTensor<ElementC, LayoutTagC>> tensors_C;
  cutlass::DeviceAllocation<const ElementC *> device_tensors_C;
  cutlass::HostTensor<ElementCompute, LayoutTagScalar> norm_constant;
```
- **EN:** Implements or wires together logic around `Inputs`, `HostTensor`, `ElementScalar`, `LayoutTagScalar`, `alpha` for the current test scenario.
- **CN:** 围绕 `Inputs`, `HostTensor`, `ElementScalar`, `LayoutTagScalar`, `alpha` 实现或连接当前测试场景所需的逻辑。

### Lines 1249-1257

```cpp
  // Outputs
  cutlass::HostTensor<ElementAmax, LayoutTagScalar> abs_max_Aux;
  cutlass::HostTensor<ElementAmax, LayoutTagScalar> abs_max_D;
  std::vector<cutlass::HostTensor<ElementAux , LayoutTagAux>> tensors_Aux;
  cutlass::DeviceAllocation<ElementAux *> device_tensors_Aux;
  cutlass::gemm::TagToStrideC_t<   LayoutTagAux   > stride_Aux;
  std::vector<cutlass::HostTensor<ElementD, LayoutTagD>> tensors_D;
  std::vector<cutlass::HostTensor<ElementD, LayoutTagD>> references_D;
  cutlass::DeviceAllocation<ElementD *> device_tensors_D;
```
- **EN:** Implements or wires together logic around `Outputs`, `HostTensor`, `ElementAmax`, `LayoutTagScalar`, `abs_max_Aux` for the current test scenario.
- **CN:** 围绕 `Outputs`, `HostTensor`, `ElementAmax`, `LayoutTagScalar`, `abs_max_Aux` 实现或连接当前测试场景所需的逻辑。

### Lines 1259-1263

```cpp
  // References
  cutlass::HostTensor<ElementBias, LayoutTagVector> reference_dbias;
  std::vector<cutlass::HostTensor<ElementAux , LayoutTagAux>> references_Aux;
  cutlass::HostTensor<ElementAmax, LayoutTagScalar> reference_abs_max_Aux;
  cutlass::HostTensor<ElementAmax, LayoutTagScalar> reference_abs_max_D;
```
- **EN:** Implements or wires together logic around `References`, `HostTensor`, `ElementBias`, `LayoutTagVector`, `reference_dbias` for the current test scenario.
- **CN:** 围绕 `References`, `HostTensor`, `ElementBias`, `LayoutTagVector`, `reference_dbias` 实现或连接当前测试场景所需的逻辑。

### Lines 1265-1270

```cpp
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
  // Are scalars copied to device memory before kernel launch
  ScalarLoc use_device_scalars = ScalarLoc::ON_HOST;
  // If per-row scale is enabled and this is disabled, alpha/beta are passed as a host or device scalar instead of device vector
  VectorScale vector_scale_mode = VectorScale::DISABLED;
```
- **EN:** Implements or wires together logic around `Whether`, `use`, `relative`, `equality`, `checks` for the current test scenario.
- **CN:** 围绕 `Whether`, `use`, `relative`, `equality`, `checks` 实现或连接当前测试场景所需的逻辑。

### Lines 1272-1278

```cpp
  // Random distribution with which to initialize the A/B/C/D/Aux scaling factors
  cutlass::Distribution::Kind init_scale = cutlass::Distribution::Uniform;
  // Random distribution with which to initialize the bias vector
  cutlass::Distribution::Kind init_bias = cutlass::Distribution::Uniform;
  cutlass::Distribution::Kind init_C;
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
```
- **EN:** Implements or wires together logic around `Random`, `distribution`, `with`, `which`, `initialize` for the current test scenario.
- **CN:** 围绕 `Random`, `distribution`, `with`, `which`, `initialize` 实现或连接当前测试场景所需的逻辑。

### Lines 1280-1293

```cpp
  HostCollectiveEpilogue(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    ScalarLoc use_device_scalars_ = ScalarLoc::ON_HOST,
    VectorScale vector_scale_mode_ = VectorScale::DISABLED,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_scale_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_bias_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = kDefaultSeed
  ): init_scale(init_scale_), init_bias(init_bias_),
     init_C(init_C_), seed(seed_),
     stride_factor_C(typename LayoutTagC::Stride()),
     stride_factor_D(typename LayoutTagD::Stride()),
     check_relative_equality(check_relative_equality_),
     use_device_scalars(use_device_scalars_){ }
```
- **EN:** Implements or wires together logic around `HostCollectiveEpilogue`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `ScalarLoc` for the current test scenario.
- **CN:** 围绕 `HostCollectiveEpilogue`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `ScalarLoc` 实现或连接当前测试场景所需的逻辑。

### Lines 1295-1296

```cpp
  bool initialize(ProblemShapeType problem_shapes, ElementScalar alpha_=1.f, ElementScalar beta_=0.f) {
    // Initialize Epilogue tensors
```
- **EN:** Begins function or method `initialize`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `initialize`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1298-1302

```cpp
    tensors_C.clear();
    tensors_D.clear();
    references_D.clear();
    stride_c_host.clear();
    stride_d_host.clear();
```
- **EN:** Declares member fields or local variables related to `tensors_C`, `clear`, `tensors_D`, `references_D`, `stride_c_host` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_C`, `clear`, `tensors_D`, `references_D`, `stride_c_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1304-1305

```cpp
    tensors_SFD.clear();
    references_SFD.clear();
```
- **EN:** Declares member fields or local variables related to `tensors_SFD`, `clear`, `references_SFD` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_SFD`, `clear`, `references_SFD` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1308-1309

```cpp
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = std::max(problem_shapes.groups(), L);
```
- **EN:** Declares member fields or local variables related to `append`, `problem_shapes`, `get_host_problem_shape`, `std`, `max` for later setup, execution, or verification.
- **CN:** 声明与 `append`, `problem_shapes`, `get_host_problem_shape`, `std`, `max` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1311-1312

```cpp
    for (int32_t i = 0; i < L; ++i) {
      auto [M, N, K, mock_L] = cute::append<4>(problem_shapes.get_host_problem_shape(i), 1);
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 1314-1315

```cpp
      stride_c_host.push_back(cutlass::make_cute_packed_stride(InternalStrideC{}, {M, N, 1}));
      stride_d_host.push_back(cutlass::make_cute_packed_stride(InternalStrideD{}, {M, N, 1}));
```
- **EN:** Declares member fields or local variables related to `stride_c_host`, `push_back`, `make_cute_packed_stride`, `InternalStrideC`, `stride_d_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_c_host`, `push_back`, `make_cute_packed_stride`, `InternalStrideC`, `stride_d_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1317-1322

```cpp
      auto c_coord = cutlass::make_Coord(M, N);
      tensors_C.push_back(cutlass::HostTensor<ElementC, LayoutTagC>(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagC>::layout_factory(c_coord, stride_factor_C)));
      tensors_D.push_back(cutlass::HostTensor<ElementD, LayoutTagD>(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, stride_factor_D)));
      references_D.push_back(cutlass::HostTensor<ElementD, LayoutTagD>(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, stride_factor_D), false));
      EXPECT_TRUE(initialize_tensor(tensors_C[i].host_view(), init_C, seed + 2020));
      tensors_C[i].host_view().at({0, 0}) = ElementC(1);
```
- **EN:** Declares member fields or local variables related to `c_coord`, `make_Coord`, `tensors_C`, `push_back`, `HostTensor` for later setup, execution, or verification.
- **CN:** 声明与 `c_coord`, `make_Coord`, `tensors_C`, `push_back`, `HostTensor` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1324-1327

```cpp
      cutlass::reference::host::TensorCopy(references_D[i].host_view(), tensors_C[i].host_view());
      tensors_C[i].sync_device();
      tensors_D[i].sync_device();
    }
```
- **EN:** Implements or wires together logic around `reference`, `host`, `TensorCopy`, `references_D`, `host_view` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `TensorCopy`, `references_D`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 1329-1346

```cpp
    auto scalar_coord = cutlass::make_Coord(1);
    auto col_vector_coord = cutlass::make_Coord(M);
    if constexpr (IsPerRowScaleEnabled) {
      alpha.resize(col_vector_coord);
      EXPECT_TRUE(initialize_tensor(alpha.host_view(), init_scale, seed + 2023));
      if (vector_scale_mode == VectorScale::DISABLED) {
        beta.resize(scalar_coord, false);
        cutlass::reference::host::TensorFill(beta.host_view(), beta_);
      }
      else {
        beta.resize(col_vector_coord);
        EXPECT_TRUE(initialize_tensor(beta.host_view(), init_scale, seed + 2024));
      }
    }
    else {
      alpha.resize(scalar_coord, (use_device_scalars == ScalarLoc::ON_DEVICE));
      beta.resize(scalar_coord, (use_device_scalars == ScalarLoc::ON_DEVICE));
      cutlass::reference::host::TensorFill(alpha.host_view(), alpha_);
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1347-1350

```cpp
      cutlass::reference::host::TensorFill(beta.host_view(), beta_);
    }
    alpha.sync_device();
    beta.sync_device();
```
- **EN:** Implements or wires together logic around `reference`, `host`, `TensorFill`, `beta`, `host_view` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `TensorFill`, `beta`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 1352-1365

```cpp
    if constexpr (IsScaleFactorEnabled) {
      scale_A.resize(scalar_coord, (use_device_scalars == ScalarLoc::ON_DEVICE));
      scale_B.resize(scalar_coord, (use_device_scalars == ScalarLoc::ON_DEVICE));
      scale_C.resize(scalar_coord, (use_device_scalars == ScalarLoc::ON_DEVICE));
      scale_D.resize(scalar_coord, (use_device_scalars == ScalarLoc::ON_DEVICE));
      EXPECT_TRUE(initialize_tensor(scale_A.host_view(), init_scale, seed + 2023));
      EXPECT_TRUE(initialize_tensor(scale_B.host_view(), init_scale, seed + 2024));
      EXPECT_TRUE(initialize_tensor(scale_C.host_view(), init_scale, seed + 2025));
      EXPECT_TRUE(initialize_tensor(scale_D.host_view(), init_scale, seed + 2026));
      scale_A.sync_device();
      scale_B.sync_device();
      scale_C.sync_device();
      scale_D.sync_device();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1367-1371

```cpp
    if constexpr (IsBiasEnabled) {
      bias.resize(col_vector_coord);
      EXPECT_TRUE(initialize_tensor(bias.host_view(), init_bias, seed + 2023));
      bias.sync_device();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1373-1379

```cpp
    if constexpr (IsDeBiasEnabled) {
      bias.resize(col_vector_coord);
      reference_dbias.resize(col_vector_coord);
      cutlass::reference::host::TensorFill(bias.host_view(), ElementBias(0));
      cutlass::reference::host::TensorFill(reference_dbias.host_view(), ElementBias(0));
      bias.sync_device();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1381-1389

```cpp
    if constexpr (IsAbsMaxEnabledD) {
      abs_max_D.resize(scalar_coord);
      // ensure in-place device reductions perform their own initialization
      cutlass::reference::host::TensorFill(abs_max_D.host_view(),
                                           CUTLASS_STL_NAMESPACE::numeric_limits<ElementAmax>::max());
      abs_max_D.sync_device();
      reference_abs_max_D.resize(scalar_coord);
      cutlass::reference::host::TensorFill(reference_abs_max_D.host_view(), ElementAmax(0));
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1391-1392

```cpp
    tensors_Aux.clear();
    references_Aux.clear();
```
- **EN:** Declares member fields or local variables related to `tensors_Aux`, `clear`, `references_Aux` for later setup, execution, or verification.
- **CN:** 声明与 `tensors_Aux`, `clear`, `references_Aux` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1394-1394

```cpp
    static_assert(!IsGroupGemm or (IsGroupGemm and !IsAuxInEnabled));
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1396-1405

```cpp
    if constexpr (IsAuxInEnabled) {
      auto aux_coord = cutlass::make_Coord(M, N);
      auto aux_layout = cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(aux_coord, typename LayoutTagAux::Stride{});
      for (int32_t i = 0; i < L; ++i) {
        tensors_Aux.push_back(cutlass::HostTensor<ElementAux , LayoutTagAux>(aux_coord, aux_layout));
        EXPECT_TRUE(initialize_tensor(tensors_Aux[i].host_view(), init_C, seed + 2023));
        tensors_Aux[i].sync_device();
      }
      stride_Aux = cutlass::make_cute_packed_stride(cutlass::gemm::TagToStrideC_t<LayoutTagAux>{}, cute::make_shape(M, N, 1));
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1407-1407

```cpp
    static_assert(!IsGroupGemm or (IsGroupGemm and !IsAuxOutEnabled));
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1409-1417

```cpp
    if constexpr (IsAuxOutEnabled) {
      for (int32_t i = 0; i < L; ++i) {
        auto [M, N, K, mock_L] = cute::append<4>(problem_shapes.get_host_problem_shape(i), 1);
        auto aux_coord = cutlass::make_Coord(M, N);
        auto aux_layout = cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(aux_coord, typename LayoutTagAux::Stride{});
        tensors_Aux.push_back(cutlass::HostTensor<ElementAux , LayoutTagAux>(aux_coord, aux_layout));
        references_Aux.push_back(cutlass::HostTensor<ElementAux , LayoutTagAux>(aux_coord, aux_layout, false));
        tensors_Aux[i].sync_device();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1419-1419

```cpp
      stride_Aux = cutlass::make_cute_packed_stride(cutlass::gemm::TagToStrideC_t<LayoutTagAux>{}, cute::make_shape(M, N, 1));
```
- **EN:** Declares member fields or local variables related to `stride_Aux`, `make_cute_packed_stride`, `gemm`, `TagToStrideC_t`, `LayoutTagAux` for later setup, execution, or verification.
- **CN:** 声明与 `stride_Aux`, `make_cute_packed_stride`, `gemm`, `TagToStrideC_t`, `LayoutTagAux` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1421-1425

```cpp
      if constexpr (IsScaleFactorEnabled) {
        scale_Aux.resize(scalar_coord, (use_device_scalars == ScalarLoc::ON_DEVICE));
        EXPECT_TRUE(initialize_tensor(scale_Aux.host_view(), init_scale, seed + 2027));
        scale_Aux.sync_device();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1427-1436

```cpp
      if constexpr (IsAbsMaxEnabledAux) {
        abs_max_Aux.resize(scalar_coord);
        // ensure in-place device reductions perform their own initialization
        cutlass::reference::host::TensorFill(abs_max_Aux.host_view(),
                                             CUTLASS_STL_NAMESPACE::numeric_limits<ElementAmax>::max());
        abs_max_Aux.sync_device();
        reference_abs_max_Aux.resize(scalar_coord);
        cutlass::reference::host::TensorFill(reference_abs_max_Aux.host_view(), ElementAmax(0));
      }
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1439-1455

```cpp
    if constexpr (IsBlockScaleSupported) {
      for (int32_t i = 0; i < L; ++i) {
        auto [M, N, K, _] = cute::append<4>(problem_shapes.get_host_problem_shape(i), 1);
        // If block scaled output is supported we always have at least 1 SFD
        auto m_blks = cutlass::ceil_div(M, cute::size<0>(cute::shape(OutputSFAtom{})));
        auto n_blks = cutlass::ceil_div(N, cute::size<1>(cute::shape(OutputSFAtom{})));
        auto sfd_coord = [&] () {
            return cutlass::make_Coord(m_blks * Blk_MN{}, n_blks * Blk_SF{});
        }();
        tensors_SFD.push_back(cutlass::HostTensor<ElementSFD, LayoutTagD>(sfd_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(sfd_coord, stride_factor_D)));
        references_SFD.push_back(cutlass::HostTensor<ElementSFD, LayoutTagD>(sfd_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(sfd_coord, stride_factor_D), false));
        tensors_SFD[i].sync_device();
      }
      norm_constant.resize(scalar_coord, true);
      EXPECT_TRUE(initialize_tensor(norm_constant.host_view(), init_scale, seed + 2023));
      norm_constant.sync_device();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1458-1459

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1461-1467

```cpp
  template <
    class Element,
    class Layout
  >
  bool equality_check(
    cutlass::TensorView<Element, Layout> const& lhs,
    cutlass::TensorView<Element, Layout> const& rhs) const {
```
- **EN:** Defines templated type `Element` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Element`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1469-1474

```cpp
    // Factors used for calculating relative equality. CUTLASS's relative-equality
    // checks in include/cutlass/relatively_equal.h  are inspired by
    // https://floating-point-gui.de/errors/comparison/. This reference suggests using
    // the minimum normal value of a given type as the nonzero_floor.
    Element epsilon(static_cast<Element>(0.1f));
    Element nonzero_floor(std::numeric_limits<Element>::min());
```
- **EN:** Implements or wires together logic around `Factors`, `used`, `calculating`, `relative`, `equality` for the current test scenario.
- **CN:** 围绕 `Factors`, `used`, `calculating`, `relative`, `equality` 实现或连接当前测试场景所需的逻辑。

### Lines 1476-1488

```cpp
    if constexpr (!cutlass::is_complex<Element>::value) {
      if (check_relative_equality == CheckEquality::RELATIVE) {
        return cutlass::reference::host::TensorRelativelyEquals(
          lhs, rhs, epsilon, nonzero_floor);
      }
      else {
        return cutlass::reference::host::TensorEquals(lhs, rhs);
      }
    }
    else {
      return cutlass::reference::host::TensorEquals(lhs, rhs);
    }
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1490-1496

```cpp
  bool compare_reference(
      ProblemShapeType problem_shapes,
      ElementScalar alpha,
      ElementScalar beta,
      int batch) {
    tensors_D[batch].sync_host();
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_C[batch].host_view()), 0);
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 1498-1500

```cpp
    if (tensors_D[batch].size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_D[batch].host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1502-1504

```cpp
    if (references_D[batch].size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(references_D[batch].host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1506-1509

```cpp
    bool passed = equality_check(references_D[batch].host_view(), tensors_D[batch].host_view());
    if(!passed) {
      std::cout<<"D is incorrect"<<std::endl;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1511-1514

```cpp
    if constexpr (IsAbsMaxEnabledD) {
      abs_max_D.sync_host();
      passed &= equality_check(reference_abs_max_D.host_view(), abs_max_D.host_view());
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1516-1521

```cpp
    if constexpr (IsDeBiasEnabled) {
      bias.sync_host();
      EXPECT_GT(cutlass::reference::host::TensorNorm(bias.host_view()), 0);
      EXPECT_GT(cutlass::reference::host::TensorNorm(reference_dbias.host_view()), 0);
      passed &= equality_check(reference_dbias.host_view(), bias.host_view());
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1523-1539

```cpp
    if constexpr (IsAuxOutEnabled) {
      tensors_Aux[batch].sync_host();
      EXPECT_GT(cutlass::reference::host::TensorNorm(tensors_Aux[batch].host_view()), 0);
      EXPECT_GT(cutlass::reference::host::TensorNorm(references_Aux[batch].host_view()), 0);
      passed &= equality_check(references_Aux[batch].host_view(), tensors_Aux[batch].host_view());
      if(!passed) {
        std::cout<<"Aux is incorrect"<<std::endl;
      }
      if constexpr (IsAbsMaxEnabledAux) {
        abs_max_Aux.sync_host();
        bool tmp =  equality_check(reference_abs_max_Aux.host_view(), abs_max_Aux.host_view());
        if(!tmp) {
          std::cout<<"AbsMax of Aux is incorrect"<<std::endl;
        }
        passed &= tmp;
      }
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1541-1548

```cpp
    if constexpr (IsBlockScaleSupported) {
      tensors_SFD[batch].sync_host();
      bool passed_sf = equality_check(references_SFD[batch].host_view(), tensors_SFD[batch].host_view());
      if(!passed_sf) {
        std::cout<<"SF is incorrect"<<std::endl;
      }
      passed &= passed_sf;
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1551-1552

```cpp
    return passed;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1554-1570

```cpp
  void print_tensors(std::ofstream& file, int batch) {
    auto coord_0 = cutlass::make_Coord(0);
    if constexpr (IsScaleFactorEnabled) {
      file
        << ", scale_a: " << scale_A.at(coord_0)
        << ", scale_b: " << scale_B.at(coord_0)
        << ", scale_c: " << scale_C.at(coord_0);
    }
    if constexpr (IsPerRowScaleEnabled) {
      file << "\n\nvalpha = \n" << alpha.host_view();
      file << "\n\nvbeta = \n" << beta.host_view();
    }
    else {
      file
        << ", alpha: " << alpha.at(coord_0) << ", beta: " << beta.at(coord_0);
    }
    file << "\n\n";
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1572-1575

```cpp
    if constexpr (IsAbsMaxEnabledD) {
      file << "scale_d: " << float(scale_D.at(coord_0));
      file << "\nReference abs_max_D :";
      file << " " << float(reference_abs_max_D.at(coord_0));
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1577-1580

```cpp
      file << "\nComputed abs_max_D :";
      file << " " << float(abs_max_D.at(coord_0));
      file << "\n\n";
    }
```
- **EN:** Implements or wires together logic around `file`, `nComputed`, `abs_max_D`, `coord_0` for the current test scenario.
- **CN:** 围绕 `file`, `nComputed`, `abs_max_D`, `coord_0` 实现或连接当前测试场景所需的逻辑。

### Lines 1582-1585

```cpp
    if constexpr (IsAbsMaxEnabledAux) {
      file << "scale_aux: " << float(scale_Aux.at(coord_0));
      file << "\nReference abs_max_Aux :";
      file << " " << float(reference_abs_max_Aux.at(coord_0));
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1587-1590

```cpp
      file << "\nComputed abs_max_Aux :";
      file << " " << float(abs_max_Aux.at(coord_0));
      file << "\n\n";
    }
```
- **EN:** Implements or wires together logic around `file`, `nComputed`, `abs_max_Aux`, `coord_0` for the current test scenario.
- **CN:** 围绕 `file`, `nComputed`, `abs_max_Aux`, `coord_0` 实现或连接当前测试场景所需的逻辑。

### Lines 1592-1594

```cpp
    if constexpr (IsBiasEnabled) {
      file << "\n\nBias = \n" << bias.host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1596-1598

```cpp
    if constexpr (IsAuxInEnabled) {
      file << "\n\nAux Input = \n" << tensors_Aux[batch].host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1600-1603

```cpp
    if constexpr (IsDeBiasEnabled) {
      file << "\n\nReference dBias = \n" << reference_dbias.host_view();
      file << "\n\nComputed dBias = \n" << bias.host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1605-1609

```cpp
    if constexpr (IsAuxOutEnabled) {
      file
        << "\n\nReference Aux =\n" << references_Aux[batch].host_view()
        << "\n\nComputed Aux =\n" << tensors_Aux[batch].host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1611-1615

```cpp
    if constexpr (IsBlockScaleSupported) {
      file
        << "\n\nReference SFD =\n" << references_SFD[batch].host_view()
        << "\n\nComputed SFD =\n" << tensors_SFD[batch].host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1617-1622

```cpp
    file
    << "\nC =\n" << tensors_C[batch].host_view()
    << "\n\nReference =\n" << references_D[batch].host_view()
    << "\n\nComputed =\n" << tensors_D[batch].host_view();
  }
```
- **EN:** Implements or wires together logic around `file`, `tensors_C`, `batch`, `host_view`, `nReference` for the current test scenario.
- **CN:** 围绕 `file`, `tensors_C`, `batch`, `host_view`, `nReference` 实现或连接当前测试场景所需的逻辑。

### Lines 1624-1627

```cpp
  Arguments to_args(ProblemShapeType problem_shapes) {
    auto coord_0 = cutlass::make_Coord(0);
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = std::max(problem_shapes.groups(), L);
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1629-1630

```cpp
    std::vector<ElementC *> ptr_C_host(L);
    std::vector<ElementD *> ptr_D_host(L);
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `ElementC`, `ptr_C_host`, `ElementD` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `ElementC`, `ptr_C_host`, `ElementD` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1632-1635

```cpp
    for (int32_t i = 0; i < L; ++i) {
      ptr_C_host.at(i) = tensors_C[i].device_data();
      ptr_D_host.at(i) = tensors_D[i].device_data();
    }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 1637-1638

```cpp
    device_tensors_C.reset(L);
    device_tensors_C.copy_from_host(ptr_C_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_C`, `reset`, `copy_from_host`, `ptr_C_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_C`, `reset`, `copy_from_host`, `ptr_C_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1640-1641

```cpp
    device_tensors_D.reset(L);
    device_tensors_D.copy_from_host(ptr_D_host.data());
```
- **EN:** Declares member fields or local variables related to `device_tensors_D`, `reset`, `copy_from_host`, `ptr_D_host`, `data` for later setup, execution, or verification.
- **CN:** 声明与 `device_tensors_D`, `reset`, `copy_from_host`, `ptr_D_host`, `data` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1643-1644

```cpp
    stride_c_device.reset(problem_shapes.groups());
    stride_c_device.copy_from_host(stride_c_host.data());
```
- **EN:** Declares member fields or local variables related to `stride_c_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_c_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1646-1647

```cpp
    stride_d_device.reset(problem_shapes.groups());
    stride_d_device.copy_from_host(stride_d_host.data());
```
- **EN:** Declares member fields or local variables related to `stride_d_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_d_device`, `reset`, `problem_shapes`, `groups`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1649-1656

```cpp
    std::vector<ElementAux *> ptr_Aux_host(L);
    if constexpr (IsAuxInEnabled || IsAuxOutEnabled) {
      for (int32_t i = 0; i < L; ++i) {
        ptr_Aux_host.at(i) = tensors_Aux[i].device_data();
      }
      device_tensors_Aux.reset(L);
      device_tensors_Aux.copy_from_host(ptr_Aux_host.data());
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1658-1659

```cpp
    auto device_tensors_C_ptr = cute::is_void_v<typename kernel::ElementC> ? nullptr :
                                  reinterpret_cast<typename kernel::ElementC const**>(device_tensors_C.get());
```
- **EN:** Implements or wires together logic around `device_tensors_C_ptr`, `is_void_v`, `kernel`, `ElementC`, `nullptr` for the current test scenario.
- **CN:** 围绕 `device_tensors_C_ptr`, `is_void_v`, `kernel`, `ElementC`, `nullptr` 实现或连接当前测试场景所需的逻辑。

### Lines 1661-1675

```cpp
    Arguments arguments;
    if constexpr (IsGroupGemm) {
      arguments =
      {
        {},
        device_tensors_C_ptr, stride_c_device.get(), device_tensors_D.get(), stride_d_device.get()
      };
    }
    else {
      arguments =
      {
        {},
        device_tensors_C_ptr, stride_c_host[0], device_tensors_D.get(), stride_d_host[0]
      };
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1677-1690

```cpp
    auto &fusion_args = arguments.thread;
    if constexpr (IsLegacy) {
      arguments.thread = {
        alpha.at(coord_0),
        beta.at(coord_0),
        alpha.device_data(),
        beta.device_data()
      };
      arguments.ptr_Bias = bias.device_data();
      arguments.ptr_T = device_tensors_Aux.get();
    }
    else {
      fusion_args.alpha = alpha.at(coord_0);
      fusion_args.beta = beta.at(coord_0);
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1692-1695

```cpp
      fusion_args.alpha_ptr = alpha.device_data();
      // can_implement requires beta_ptr to not be set if its voidC
      fusion_args.beta_ptr = cute::is_void_v<typename kernel::ElementC> ? nullptr :
                               beta.device_data();
```
- **EN:** Implements or wires together logic around `fusion_args`, `alpha_ptr`, `alpha`, `device_data`, `can_implement` for the current test scenario.
- **CN:** 围绕 `fusion_args`, `alpha_ptr`, `alpha`, `device_data`, `can_implement` 实现或连接当前测试场景所需的逻辑。

### Lines 1697-1706

```cpp
      if constexpr (IsScaleFactorEnabled) {
        fusion_args.scale_a = scale_A.at(coord_0);
        fusion_args.scale_b = scale_B.at(coord_0);
        fusion_args.scale_c = scale_C.at(coord_0);
        fusion_args.scale_d = scale_D.at(coord_0);
        fusion_args.scale_a_ptr = scale_A.device_data();
        fusion_args.scale_b_ptr = scale_B.device_data();
        fusion_args.scale_c_ptr = scale_C.device_data();
        fusion_args.scale_d_ptr = scale_D.device_data();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1708-1710

```cpp
      if constexpr (IsBiasEnabled) {
        fusion_args.bias_ptr = bias.device_data();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1712-1714

```cpp
      if constexpr (IsDeBiasEnabled) {
        fusion_args.dbias_ptr = bias.device_data();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1716-1721

```cpp
      // example of how to set kernel activation arguments
      // see ActivationFunctor::Arguments in activation.h for definition
      // if Arguments doesn't exist then fusion_args.activation is empty
      if constexpr (cute::is_same_v<ActivationFunctor, cutlass::epilogue::thread::ScaledGELU_taylor<ElementCompute>>) {
        fusion_args.activation.scale = ElementCompute(1);
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1723-1727

```cpp
      // Treat Clamp as ReLU
      if constexpr (cute::is_same_v<ActivationFunctor, cutlass::epilogue::thread::Clamp<ElementCompute>>) {
        fusion_args.activation.lower_bound = 0;
        fusion_args.activation.upper_bound = std::numeric_limits<ElementCompute>::max();
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1729-1731

```cpp
      if constexpr (IsAbsMaxEnabledD) {
        fusion_args.amax_D_ptr = abs_max_D.device_data();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1733-1736

```cpp
      if constexpr (IsAuxInEnabled) {
        fusion_args.aux_ptr = device_tensors_Aux.get();
        fusion_args.dAux = stride_Aux;
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1738-1748

```cpp
      if constexpr (IsAuxOutEnabled) {
        fusion_args.aux_ptr = device_tensors_Aux.get();
        fusion_args.dAux = stride_Aux;
        if constexpr (IsScaleFactorEnabled) {
          fusion_args.scale_aux = scale_Aux.at(coord_0);
          fusion_args.scale_aux_ptr = scale_Aux.device_data();
        }
        if constexpr (IsAbsMaxEnabledAux) {
          fusion_args.amax_aux_ptr = abs_max_Aux.device_data();
        }
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1750-1756

```cpp
      if constexpr (IsBlockScaleSupported) {
        std::vector<ElementSFD *> ptr_SFD_host(L);
        for (int32_t i = 0; i < L; ++i) {
          ptr_SFD_host.at(i) = tensors_SFD[i].device_data();
        }
        device_tensors_SFD.reset(L);
        device_tensors_SFD.copy_from_host(ptr_SFD_host.data());
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1758-1762

```cpp
        arguments.thread.block_scale_factor_ptr = device_tensors_SFD.get();
        arguments.thread.norm_constant_ptr = norm_constant.device_data();
      }
    }
```
- **EN:** Implements or wires together logic around `arguments`, `thread`, `block_scale_factor_ptr`, `device_tensors_SFD`, `get` for the current test scenario.
- **CN:** 围绕 `arguments`, `thread`, `block_scale_factor_ptr`, `device_tensors_SFD`, `get` 实现或连接当前测试场景所需的逻辑。

### Lines 1764-1765

```cpp
    return arguments;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1767-1784

```cpp
  auto to_host_args(ProblemShapeType problem_shapes, int batch) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_shapes.get_host_problem_shape(batch), 1);
    auto [M, N, K, L] = problem_shape_MNKL;
    auto coord_0 = cutlass::make_Coord(0);
    auto C = cute::make_tensor(detail::make_iterator(tensors_C[batch].host_data()),
        cute::make_layout(cute::make_shape(M, N, 1), stride_c_host[batch]));
    auto D = cute::make_tensor(detail::make_iterator(references_D[batch].host_data()),
        cute::make_layout(cute::make_shape(M, N, 1), stride_d_host[batch]));
    auto Bias = cute::make_tensor(detail::make_iterator(IsDeBiasEnabled ? reference_dbias.host_data() : bias.host_data()),
        cute::make_layout(cute::make_shape(M, cute::_1{})));
    auto Aux_layout = cute::make_layout(cute::make_shape(M, N, 1), stride_Aux);
    auto Aux = [&]() {
      auto ptr = recast_ptr<ElementAux>(nullptr);
      if (IsAuxInEnabled) {
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1785-1794

```cpp
        ptr = detail::make_iterator(tensors_Aux[batch].host_data());
      } else if (IsAuxOutEnabled) {
        ptr = detail::make_iterator(references_Aux[batch].host_data());
      }
      return cute::make_tensor(ptr, Aux_layout);
    }();
    auto Valpha = cute::make_tensor(detail::make_iterator(alpha.host_data()),
        cute::make_layout(cute::make_shape(M, N, cute::_1{}), cute::make_stride(cute::_1{}, cute::_0{}, M)));
    auto Vbeta = cute::make_tensor(detail::make_iterator(beta.host_data()),
        cute::make_layout(cute::make_shape(M, N, cute::_1{}), cute::make_stride(cute::_1{}, cute::_0{}, N)));
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1796-1806

```cpp
    auto SfD = [&](){
      if constexpr (IsBlockScaleSupported) {
        auto tensor = make_tensor(detail::make_iterator(references_SFD[batch].host_data()),
          Sm1xxBlockScaledOutputConfig::tile_atom_to_shape_SFD(problem_shape_MNKL));
        return tensor;
      }
      else {
        // Reference kernel has a logic to ignore scalefactor computation if we pass the tensor type same as output D tensor.
        return D;
      }
    }();
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1809-1826

```cpp
    cutlass::reference::host::GettEpilogueParams<
      ElementScalar,
      ElementScalar,
      ElementAccumulator,
      ElementCompute,
      decltype(C),
      decltype(D),
      decltype(Bias),
      decltype(Aux),
      decltype(Valpha),
      decltype(Vbeta),
      ActivationFunctor
      , decltype(SfD)
      , Int<SFD_VectorSize>
      , cutlass::plus<ElementCompute>
      , false
      , SfGenStrategy
    > epilogue_params{};
```
- **EN:** Implements or wires together logic around `reference`, `host`, `GettEpilogueParams`, `ElementScalar`, `ElementAccumulator` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `GettEpilogueParams`, `ElementScalar`, `ElementAccumulator` 实现或连接当前测试场景所需的逻辑。

### Lines 1828-1831

```cpp
    epilogue_params.C = C;
    epilogue_params.D = D;
    epilogue_params.alpha = alpha.at(coord_0);
    epilogue_params.beta = beta.at(coord_0);
```
- **EN:** Declares member fields or local variables related to `epilogue_params`, `alpha`, `coord_0`, `beta` for later setup, execution, or verification.
- **CN:** 声明与 `epilogue_params`, `alpha`, `coord_0`, `beta` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1833-1838

```cpp
    if constexpr (IsScaleFactorEnabled) {
      epilogue_params.scale_a = scale_A.at(coord_0);
      epilogue_params.scale_b = scale_B.at(coord_0);
      epilogue_params.scale_c = scale_C.at(coord_0);
      epilogue_params.scale_d = scale_D.at(coord_0);
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1840-1842

```cpp
    if constexpr (IsBiasEnabled or IsDeBiasEnabled) {
      epilogue_params.Bias = Bias;
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1844-1846

```cpp
    if constexpr (IsAbsMaxEnabledD) {
      epilogue_params.abs_max_D = reference_abs_max_D.host_data();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1848-1850

```cpp
    if constexpr (IsAuxInEnabled) {
      epilogue_params.Aux = Aux;
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1852-1860

```cpp
    if constexpr (IsAuxOutEnabled) {
      epilogue_params.Aux = Aux;
      if constexpr (IsScaleFactorEnabled) {
        epilogue_params.scale_aux = scale_Aux.at(coord_0);
      }
      if constexpr (IsAbsMaxEnabledAux) {
        epilogue_params.abs_max_Aux = reference_abs_max_Aux.host_data();
      }
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1862-1867

```cpp
    if constexpr (IsPerRowScaleEnabled) {
      epilogue_params.Valpha = Valpha;
      if (vector_scale_mode == VectorScale::ENABLED) {
        epilogue_params.Vbeta = Vbeta;
      }
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1869-1872

```cpp
    if constexpr (IsBlockScaleSupported) {
      epilogue_params.SfD = SfD;
      epilogue_params.st = norm_constant.at(coord_0);
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1874-1876

```cpp
    return epilogue_params;
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1878-1892

```cpp
template <
  typename Gemm,
  template <class T> class ActivationFunctor_ = cutlass::epilogue::thread::Identity,
  bool force_legacy_epilogue = false,
  typename ElementA = typename Gemm::GemmKernel::ElementA,
  typename ElementB = typename Gemm::GemmKernel::ElementB
>
struct TestbedImpl {
  // Kernel data types
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  // All Collective MMA operands are defined by HostCollectiveMainloopType based on the schedule type
  using HostCollectiveMainloopType = HostCollectiveMainloop<ScheduleType, Gemm, ElementA, ElementB>;
  using CollectiveEpilogue = cute::conditional_t<IsDefaultEpilogue<typename Gemm::GemmKernel::CollectiveEpilogue>::value || force_legacy_epilogue,
                                                HostCollectiveDefaultEpilogue<Gemm>,
                                                HostCollectiveEpilogue<Gemm>>;
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1894-1897

```cpp
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementCompute = typename ElementComputeType<Gemm, ElementAccumulator>::Type;
  using ElementScalar = typename ElementScalarType<Gemm, ElementCompute>::Type;
```
- **EN:** Defines aliases such as `ProblemShapeType`, `ElementAccumulator`, `ElementCompute`, `ElementScalar` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ProblemShapeType`, `ElementAccumulator`, `ElementCompute`, `ElementScalar`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1899-1902

```cpp
  using LayoutTagA = typename HostCollectiveMainloopType::LayoutTagA;
  using LayoutTagB = typename HostCollectiveMainloopType::LayoutTagB;
  using LayoutTagC = typename CollectiveEpilogue::LayoutTagC;
  using LayoutTagD = typename CollectiveEpilogue::LayoutTagD;
```
- **EN:** Defines aliases such as `LayoutTagA`, `LayoutTagB`, `LayoutTagC`, `LayoutTagD` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `LayoutTagA`, `LayoutTagB`, `LayoutTagC`, `LayoutTagD`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1904-1910

```cpp
  uint32_t sm_count;
  // Used to force multi-wave tests for persistent kernel schedules
  constexpr static int MaxSmCount = 16;
  static constexpr uint64_t kDefaultSeed = 4096;
  static constexpr uint32_t mma_promotion_interval = 4;
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
```
- **EN:** Introduces type aliases like `RasterOrderOptions`, `DecompositionMode` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `RasterOrderOptions`, `DecompositionMode`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 1912-1913

```cpp
  HostCollectiveMainloopType collective_mma_inputs;
  CollectiveEpilogue collective_epilogue;
```
- **EN:** Declares member fields or local variables related to `HostCollectiveMainloopType`, `collective_mma_inputs`, `CollectiveEpilogue`, `collective_epilogue` for later setup, execution, or verification.
- **CN:** 声明与 `HostCollectiveMainloopType`, `collective_mma_inputs`, `CollectiveEpilogue`, `collective_epilogue` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1915-1915

```cpp
  static constexpr bool IsGroupGemm = CollectiveEpilogue::IsGroupGemm;
```
- **EN:** Declares member fields or local variables related to `IsGroupGemm`, `CollectiveEpilogue` for later setup, execution, or verification.
- **CN:** 声明与 `IsGroupGemm`, `CollectiveEpilogue` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1917-1919

```cpp
  //
  // Methods
  //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 1921-1932

```cpp
  TestbedImpl(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    ScalarLoc use_device_scalars_ = ScalarLoc::ON_HOST,
    VectorScale vector_scale_mode_ = VectorScale::DISABLED,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_scale_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_bias_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = kDefaultSeed
  ): collective_mma_inputs(HostCollectiveMainloopType(check_relative_equality_, init_A_, init_B_, seed_)),
     collective_epilogue(CollectiveEpilogue(check_relative_equality_, use_device_scalars_, vector_scale_mode_, init_C_, init_scale_, init_bias_, seed_)) { }
```
- **EN:** Implements or wires together logic around `TestbedImpl`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `ScalarLoc` for the current test scenario.
- **CN:** 围绕 `TestbedImpl`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `ScalarLoc` 实现或连接当前测试场景所需的逻辑。

### Lines 1934-1949

```cpp
  TestbedImpl(
    typename LayoutTagA::Stride stride_factor_A_,
    typename LayoutTagB::Stride stride_factor_B_,
    typename LayoutTagC::Stride stride_factor_C_,
    typename LayoutTagD::Stride stride_factor_D_,
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    ScalarLoc use_device_scalars_ = ScalarLoc::ON_HOST,
    VectorScale vector_scale_mode_ = VectorScale::DISABLED,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_scale_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_bias_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = kDefaultSeed
  ): collective_mma_inputs(HostCollectiveMainloopType(check_relative_equality_, stride_factor_A_, stride_factor_B_, init_A_, init_B_, seed_)),
     collective_epilogue(CollectiveEpilogue(check_relative_equality_, use_device_scalars_, vector_scale_mode_, init_C_, init_scale_, init_bias_, seed_)) { }
```
- **EN:** Implements or wires together logic around `TestbedImpl`, `LayoutTagA`, `Stride`, `stride_factor_A_`, `LayoutTagB` for the current test scenario.
- **CN:** 围绕 `TestbedImpl`, `LayoutTagA`, `Stride`, `stride_factor_A_`, `LayoutTagB` 实现或连接当前测试场景所需的逻辑。

### Lines 1951-1954

```cpp
  /// Initializes data structures
  bool initialize(ProblemShapeType problem_shapes, ElementScalar alpha_=1.f, ElementScalar beta_=0.f) {
    collective_mma_inputs.initialize(problem_shapes);
    collective_epilogue.initialize(problem_shapes, alpha_, beta_);
```
- **EN:** Implements or wires together logic around `Initializes`, `data`, `structures`, `initialize`, `ProblemShapeType` for the current test scenario.
- **CN:** 围绕 `Initializes`, `data`, `structures`, `initialize`, `ProblemShapeType` 实现或连接当前测试场景所需的逻辑。

### Lines 1956-1957

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1959-1966

```cpp
  /// Compares computed reference with device reference and outputs to a file if incorrect
  bool compare_reference(
      ProblemShapeType problem_shapes,
      ElementScalar alpha,
      ElementScalar beta,
      int batch)
  {
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(batch), 1);
```
- **EN:** Implements or wires together logic around `Compares`, `computed`, `reference`, `with`, `device` for the current test scenario.
- **CN:** 围绕 `Compares`, `computed`, `reference`, `with`, `device` 实现或连接当前测试场景所需的逻辑。

### Lines 1968-1977

```cpp
    bool passed = collective_mma_inputs.compare_reference(problem_shapes, batch);
    passed &= collective_epilogue.compare_reference(problem_shapes, alpha, beta, batch);
    EXPECT_TRUE(passed);
    if (!passed) {
      std::stringstream fname;
      fname << "error_Gemm_device_"
        << M << "x" << N << "x" << K << "x" << batch << "_"
        << cute::get<0>(typename Gemm::GemmKernel::TileShape{}) << "_"
        << cute::get<1>(typename Gemm::GemmKernel::TileShape{}) << "_"
        << cute::get<2>(typename Gemm::GemmKernel::TileShape{}) << ".txt";
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1979-1982

```cpp
      std::ofstream file(fname.str());
      file
        << "problem: " << ' ' << M << "x" << N << "x" << K << ", Batch count = " << batch
        << ", alpha: " << alpha << ", beta: " << beta << "\n\n";
```
- **EN:** Implements or wires together logic around `std`, `ofstream`, `file`, `fname`, `str` for the current test scenario.
- **CN:** 围绕 `std`, `ofstream`, `file`, `fname`, `str` 实现或连接当前测试场景所需的逻辑。

### Lines 1984-1986

```cpp
      collective_mma_inputs.print_tensors(file, batch);
      collective_epilogue.print_tensors(file, batch);
    }
```
- **EN:** Implements or wires together logic around `collective_mma_inputs`, `print_tensors`, `file`, `batch`, `collective_epilogue` for the current test scenario.
- **CN:** 围绕 `collective_mma_inputs`, `print_tensors`, `file`, `batch`, `collective_epilogue` 实现或连接当前测试场景所需的逻辑。

### Lines 1988-1989

```cpp
    return passed;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1991-1999

```cpp
  /// Verifies the result is a GEMM
  bool verify(
      ProblemShapeType problem_shapes,
      ElementScalar alpha,
      ElementScalar beta)
  {
    using namespace cute;
    auto [M, N, K, L] = cute::append<4>(problem_shapes.get_host_problem_shape(0), 1);
    L = std::max(problem_shapes.groups(), L);
```
- **EN:** Implements or wires together logic around `Verifies`, `the`, `result`, `GEMM`, `verify` for the current test scenario.
- **CN:** 围绕 `Verifies`, `the`, `result`, `GEMM`, `verify` 实现或连接当前测试场景所需的逻辑。

### Lines 2001-2004

```cpp
    bool passed = true;
    for (int32_t i = 0; i < L; ++i) {
      auto mainloop_params = collective_mma_inputs.to_host_args(problem_shapes, i);
      auto epilogue_params = collective_epilogue.to_host_args(problem_shapes, i);
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 2006-2006

```cpp
      cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
```
- **EN:** Declares member fields or local variables related to `reference`, `host`, `Gemm3x`, `mainloop_params`, `epilogue_params` for later setup, execution, or verification.
- **CN:** 声明与 `reference`, `host`, `Gemm3x`, `mainloop_params`, `epilogue_params` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2008-2011

```cpp
      passed &= compare_reference(problem_shapes, alpha, beta, i);
    }
    return passed;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2013-2018

```cpp
#ifndef SYCL_INTEL_TARGET
  /// Determine if the CUDA device is sufficient to run the kernel
  bool sufficient() {
    //
    // Determine SMEM requirements and waive if not satisfied
    //
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2020-2020

```cpp
    size_t smem_size = static_cast<size_t>(Gemm::GemmKernel::SharedStorageSize);
```
- **EN:** Declares member fields or local variables related to `size_t`, `smem_size`, `static_cast`, `Gemm`, `GemmKernel` for later setup, execution, or verification.
- **CN:** 声明与 `size_t`, `smem_size`, `static_cast`, `Gemm`, `GemmKernel` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2022-2023

```cpp
    int device_idx;
    cudaError_t result = cudaGetDevice(&device_idx);
```
- **EN:** Declares member fields or local variables related to `device_idx`, `cudaError_t`, `result`, `cudaGetDevice` for later setup, execution, or verification.
- **CN:** 声明与 `device_idx`, `cudaError_t`, `result`, `cudaGetDevice` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2025-2027

```cpp
    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDevice() API call failed.");
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2029-2031

```cpp
    cudaDeviceProp properties;
    result = cudaGetDeviceProperties(&properties, device_idx);
    this->sm_count = properties.multiProcessorCount;
```
- **EN:** Declares member fields or local variables related to `cudaDeviceProp`, `properties`, `result`, `cudaGetDeviceProperties`, `device_idx` for later setup, execution, or verification.
- **CN:** 声明与 `cudaDeviceProp`, `properties`, `result`, `cudaGetDeviceProperties`, `device_idx` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2033-2035

```cpp
    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDeviceProperties() failed");
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2037-2041

```cpp
    if (properties.sharedMemPerBlockOptin < smem_size) {
      printf("failed due to smem_size\n");
      printf("hardware smem_size: %d, required smem_size: %d\n\n", int(properties.sharedMemPerBlockOptin), int(smem_size));
      return false;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2043-2045

```cpp
    return true;
  }
  #endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2047-2054

```cpp
  /// Executes one test
  bool run(
    ProblemShapeType problem_shapes,
    ElementScalar alpha = ElementScalar(1),
    ElementScalar beta = ElementScalar(0),
    detail::Iterations iterations = detail::Iterations{}
    )
  {
```
- **EN:** Implements or wires together logic around `Executes`, `one`, `test`, `run`, `ProblemShapeType` for the current test scenario.
- **CN:** 围绕 `Executes`, `one`, `test`, `run`, `ProblemShapeType` 实现或连接当前测试场景所需的逻辑。

### Lines 2056-2063

```cpp
    using namespace cutlass;
#ifndef SYCL_INTEL_TARGET
    // Fail test if insufficient CUDA device
    if (!sufficient()) {
      std::cout << "Test failed due to insufficient CUDA device." << std::endl;
      return false;
    }
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2065-2068

```cpp
    if (!this->initialize(problem_shapes, alpha, beta)) {
      std::cerr << "Initialization failed \n";
      return false;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2070-2072

```cpp
    //
    // Initialize the GEMM operator
    //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 2074-2078

```cpp
    typename Gemm::Arguments arguments;
    cutlass::KernelHardwareInfo hw_info;
    hw_info.device_id = 0;
    this->sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
    hw_info.sm_count = this->sm_count;
```
- **EN:** Declares member fields or local variables related to `Gemm`, `Arguments`, `arguments`, `KernelHardwareInfo`, `hw_info` for later setup, execution, or verification.
- **CN:** 声明与 `Gemm`, `Arguments`, `arguments`, `KernelHardwareInfo`, `hw_info` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2080-2080

```cpp
    typename HostCollectiveMainloopType::Arguments mainloop_args;
```
- **EN:** Declares member fields or local variables related to `HostCollectiveMainloopType`, `Arguments`, `mainloop_args` for later setup, execution, or verification.
- **CN:** 声明与 `HostCollectiveMainloopType`, `Arguments`, `mainloop_args` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2082-2082

```cpp
    mainloop_args = collective_mma_inputs.to_args(problem_shapes);
```
- **EN:** Declares member fields or local variables related to `mainloop_args`, `collective_mma_inputs`, `to_args`, `problem_shapes` for later setup, execution, or verification.
- **CN:** 声明与 `mainloop_args`, `collective_mma_inputs`, `to_args`, `problem_shapes` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2084-2101

```cpp
    if constexpr (IsGroupGemm) {
      arguments =
      {
        cutlass::gemm::GemmUniversalMode::kGrouped,
        problem_shapes,
        mainloop_args,
        collective_epilogue.to_args(problem_shapes),
        hw_info
      };
    }
    else {
      arguments =
      {
        cutlass::gemm::GemmUniversalMode::kArray,
        problem_shapes,
        mainloop_args,
        collective_epilogue.to_args(problem_shapes),
        hw_info
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2102-2103

```cpp
      };
    }
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 2106-2106

```cpp
    Gemm gemm_op;
```
- **EN:** Declares member fields or local variables related to `Gemm`, `gemm_op` for later setup, execution, or verification.
- **CN:** 声明与 `Gemm`, `gemm_op` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2108-2109

```cpp
    size_t workspace_size = Gemm::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```
- **EN:** Declares member fields or local variables related to `size_t`, `workspace_size`, `Gemm`, `get_workspace_size`, `arguments` for later setup, execution, or verification.
- **CN:** 声明与 `size_t`, `workspace_size`, `Gemm`, `get_workspace_size`, `arguments` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2111-2111

```cpp
    cutlass::Status status = gemm_op.can_implement(arguments);
```
- **EN:** Declares member fields or local variables related to `Status`, `status`, `gemm_op`, `can_implement`, `arguments` for later setup, execution, or verification.
- **CN:** 声明与 `Status`, `status`, `gemm_op`, `can_implement`, `arguments` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2113-2117

```cpp
    if (status != cutlass::Status::kSuccess) {
      cudaError_t error = cudaGetLastError();
      std::cerr << "This test is not supported: " << cudaGetErrorString(error) << "\n";
      return false;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2119-2121

```cpp
    //
    // Run the GEMM
    //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 2123-2135

```cpp
    cudaError_t result;
    status = gemm_op.initialize(arguments, workspace.get());
    status = gemm_op.run();
#if defined SYCL_INTEL_TARGET
    result = cudaSuccess;
    compat::wait();
#else
    result = cudaDeviceSynchronize();
#endif
    if (result != cudaSuccess) {
      EXPECT_EQ(result, cudaSuccess) << "Error at Kernel Sync.";
      return false;
    }
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2137-2137

```cpp
    EXPECT_TRUE(status == cutlass::Status::kSuccess) << to_string(status);
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `status`, `Status`, `kSuccess`, `to_string` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `status`, `Status`, `kSuccess`, `to_string` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2139-2146

```cpp
    //
    // Verify
    //
    bool passed = this->verify(problem_shapes, alpha, beta);
    if (!passed) {
      std::cout << "Error : Failed : with alpha: " << alpha << ", beta: " << beta
                << "\n";
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2148-2152

```cpp
    return passed;
  }
};
} // namespace detail
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2154-2154

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 2157-2157

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 2159-2166

```cpp
template <
  typename Gemm,
  template <class T> class ActivationFunctor = cutlass::epilogue::thread::Identity,
  bool force_legacy_epilogue = false,
  typename ElementA = typename Gemm::GemmKernel::ElementA,
  typename ElementB = typename Gemm::GemmKernel::ElementB
>
struct Testbed3x {
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 2168-2176

```cpp
  using TestBedImpl = typename detail::TestbedImpl<
                        Gemm,
                        ActivationFunctor,
                        force_legacy_epilogue,
                        ElementA,
                        ElementB
                        >;
  using Kernel      = typename Gemm::GemmKernel;
  using Epilogue    = typename Gemm::GemmKernel::CollectiveEpilogue;
```
- **EN:** Introduces type aliases like `TestBedImpl`, `Kernel`, `Epilogue` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `TestBedImpl`, `Kernel`, `Epilogue`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 2178-2180

```cpp
  using ElementAccumulator   = typename TestBedImpl::ElementAccumulator;
  using ElementCompute       = typename TestBedImpl::ElementCompute;
  using ElementScalar        = typename TestBedImpl::ElementScalar;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementCompute`, `ElementScalar` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementCompute`, `ElementScalar`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 2182-2183

```cpp
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
```
- **EN:** Defines aliases such as `RasterOrderOptions`, `DecompositionMode` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `RasterOrderOptions`, `DecompositionMode`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 2185-2185

```cpp
  static constexpr bool IsGroupGemm = TestBedImpl::IsGroupGemm;
```
- **EN:** Declares member fields or local variables related to `IsGroupGemm`, `TestBedImpl` for later setup, execution, or verification.
- **CN:** 声明与 `IsGroupGemm`, `TestBedImpl` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2187-2188

```cpp
  // Detail Implementation
  TestBedImpl impl_;
```
- **EN:** Implements or wires together logic around `Detail`, `Implementation`, `TestBedImpl`, `impl_` for the current test scenario.
- **CN:** 围绕 `Detail`, `Implementation`, `TestBedImpl`, `impl_` 实现或连接当前测试场景所需的逻辑。

### Lines 2190-2203

```cpp
  //
  // Methods
  //
  Testbed3x(
      CheckEquality check_relative_equality_ = CheckEquality::EXACT,
      ScalarLoc use_device_scalars_ = ScalarLoc::ON_DEVICE,
      VectorScale vector_scale_mode_ = VectorScale::DISABLED,
      cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
      cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
      cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
      cutlass::Distribution::Kind init_scale_ = cutlass::Distribution::Uniform,
      cutlass::Distribution::Kind init_bias_ = cutlass::Distribution::Uniform,
      uint64_t seed_ = TestBedImpl::kDefaultSeed)
      : impl_(check_relative_equality_, use_device_scalars_, vector_scale_mode_, init_A_, init_B_, init_C_, init_scale_, init_bias_, seed_) {}
```
- **EN:** Implements or wires together logic around `Methods`, `Testbed3x`, `CheckEquality`, `check_relative_equality_`, `EXACT` for the current test scenario.
- **CN:** 围绕 `Methods`, `Testbed3x`, `CheckEquality`, `check_relative_equality_`, `EXACT` 实现或连接当前测试场景所需的逻辑。

### Lines 2205-2216

```cpp
  /// Executes one test
  bool run(
   typename TestBedImpl::ProblemShapeType problem_shapes,
    ElementScalar alpha = ElementScalar(1),
    ElementScalar beta = ElementScalar(0),
    detail::Iterations iterations = detail::Iterations{}
    )
  {
    return impl_.run(
        problem_shapes, alpha, beta, iterations);
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2218-2224

```cpp
template <
  typename Gemm,
  template <class T> class ActivationFunctor = cutlass::epilogue::thread::Identity
>
bool TestAll(double alpha = 1.0, double beta = 0.0, CheckEquality check_relative_equality = CheckEquality::RELATIVE) {
  using ElementScalar = typename Gemm::EpilogueOutputOp::ElementScalar;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 2226-2226

```cpp
  Testbed3x<Gemm, ActivationFunctor> testbed(check_relative_equality, ScalarLoc::ON_DEVICE, VectorScale::DISABLED);
```
- **EN:** Declares member fields or local variables related to `Testbed3x`, `Gemm`, `ActivationFunctor`, `testbed`, `check_relative_equality` for later setup, execution, or verification.
- **CN:** 声明与 `Testbed3x`, `Gemm`, `ActivationFunctor`, `testbed`, `check_relative_equality` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2228-2236

```cpp
  int max_alignment = 0;
  // TODO(codeplay): unhardcode max_alignment
#if defined SYCL_INTEL_TARGET
  max_alignment = 32;
#else
  max_alignment = std::max(Gemm::kAlignmentA, Gemm::kAlignmentB);
#endif
  std::vector<int> problem_size_m = {max_alignment, 512 - 3 * max_alignment};
  std::vector<int> problem_size_n = {max_alignment, 512 - 2 * max_alignment};
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2238-2239

```cpp
  constexpr int Stages = Gemm::GemmKernel::DispatchPolicy::Stages;
  constexpr int TileShapeK = cute::size<2>(typename Gemm::GemmKernel::TileShape{});
```
- **EN:** Declares member fields or local variables related to `Stages`, `Gemm`, `GemmKernel`, `DispatchPolicy`, `TileShapeK` for later setup, execution, or verification.
- **CN:** 声明与 `Stages`, `Gemm`, `GemmKernel`, `DispatchPolicy`, `TileShapeK` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2241-2241

```cpp
  std::vector<int> problem_size_k = {max_alignment, TileShapeK * (Stages + 1) - max_alignment};
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `problem_size_k`, `max_alignment`, `TileShapeK` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `problem_size_k`, `max_alignment`, `TileShapeK` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2243-2243

```cpp
  int batches[] = {5, 10};
```
- **EN:** Declares member fields or local variables related to `batches` for later setup, execution, or verification.
- **CN:** 声明与 `batches` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2245-2245

```cpp
  bool passed = true;
```
- **EN:** Declares member fields or local variables related to `passed` for later setup, execution, or verification.
- **CN:** 声明与 `passed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2247-2250

```cpp
  for (int batch : batches) {
    for (int m : problem_size_m) {
      for (int n : problem_size_n) {
        for (int k : problem_size_k) {
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2252-2254

```cpp
          if constexpr (Testbed3x<Gemm, ActivationFunctor>::IsGroupGemm) {
            std::vector<typename ProblemShapeType::UnderlyingProblemShape> problem_sizes_host;
            cutlass::DeviceAllocation<typename ProblemShapeType::UnderlyingProblemShape> problem_sizes_device;
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2256-2258

```cpp
            for (int i = 0; i < batch; ++i) {
              problem_sizes_host.push_back({m * ((i % 3) + 1), n * ((i % 4) + 1), k * ((i % 5) + 1)});
            }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 2260-2261

```cpp
            problem_sizes_device.reset(problem_sizes_host.size());
            problem_sizes_device.copy_from_host(problem_sizes_host.data());
```
- **EN:** Declares member fields or local variables related to `problem_sizes_device`, `reset`, `problem_sizes_host`, `size`, `copy_from_host` for later setup, execution, or verification.
- **CN:** 声明与 `problem_sizes_device`, `reset`, `problem_sizes_host`, `size`, `copy_from_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2263-2270

```cpp
            passed = testbed.run(
              ProblemShapeType{static_cast<int>(problem_sizes_host.size()), problem_sizes_device.get(), problem_sizes_host.data()},
              cutlass::from_real<ElementScalar>(alpha),
              cutlass::from_real<ElementScalar>(beta)
            );
          }
          else {
            ProblemShapeType problem_size{{m, n, k, batch}};
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2272-2277

```cpp
            passed = testbed.run(
              problem_size,
              cutlass::from_real<ElementScalar>(alpha),
              cutlass::from_real<ElementScalar>(beta)
            );
          }
```
- **EN:** Implements or wires together logic around `passed`, `testbed`, `run`, `problem_size`, `from_real` for the current test scenario.
- **CN:** 围绕 `passed`, `testbed`, `run`, `problem_size`, `from_real` 实现或连接当前测试场景所需的逻辑。

### Lines 2279-2286

```cpp
          if (!passed) {
            std::cout << __FILE__ << ':' << __LINE__ << " : GEMM MNKL " << m << " " << n << " " << k << " " << batch << " FAILED.\n";
            return false;
          }
        } // k
      } // n
    } // m
  } // batch
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2288-2289

```cpp
  return passed;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2291-2291

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 2293-2303

```cpp
template <typename Gemm, bool force_legacy_epilogue = false, bool apply_alignment_offset = false>
bool TestSmall(double alpha = 1.0, double beta = 1.0,
  CheckEquality check_relative_equality = CheckEquality::RELATIVE,
  ScalarLoc use_device_scalars = ScalarLoc::ON_DEVICE,
  VectorScale vector_scale_mode = VectorScale::ENABLED,
  std::vector<int> override_problem_size_k = {}) {
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using ElementScalar = typename Gemm::EpilogueOutputOp::ElementScalar;
  using ElementA = typename Gemm::GemmKernel::ElementA;
  using ElementB = typename Gemm::GemmKernel::ElementB;
  using TiledMma = typename Gemm::GemmKernel::TiledMma;
```
- **EN:** Introduces type aliases like `ProblemShapeType`, `ElementScalar`, `ElementA`, `ElementB`, `TiledMma` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ProblemShapeType`, `ElementScalar`, `ElementA`, `ElementB`, `TiledMma`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 2305-2308

```cpp
  static constexpr bool IsF8F6F4 = cutlass::gemm::collective::detail::is_sm100_mma_f8f6f4<TiledMma, ElementA, ElementB>();
  // For fp4 and fp6 kernels, the min alignment_input is 128 elements, so we don't need to add alignment_input in test problem sizes.  
  int alignment_bits_a = cutlass::detail::get_input_alignment_bits<ElementA, IsF8F6F4>();
  int alignment_input_a = (alignment_bits_a / cute::sizeof_bits<ElementA>::value == 128) ? 0 : (alignment_bits_a / cute::sizeof_bits<ElementA>::value);
```
- **EN:** Implements or wires together logic around `IsF8F6F4`, `gemm`, `collective`, `detail`, `is_sm100_mma_f8f6f4` for the current test scenario.
- **CN:** 围绕 `IsF8F6F4`, `gemm`, `collective`, `detail`, `is_sm100_mma_f8f6f4` 实现或连接当前测试场景所需的逻辑。

### Lines 2310-2311

```cpp
  int alignment_bits_b = cutlass::detail::get_input_alignment_bits<ElementB, IsF8F6F4>();
  int alignment_input_b = (alignment_bits_b / cute::sizeof_bits<ElementB>::value == 128) ? 0 : (alignment_bits_b / cute::sizeof_bits<ElementB>::value);
```
- **EN:** Declares member fields or local variables related to `alignment_bits_b`, `detail`, `get_input_alignment_bits`, `ElementB`, `IsF8F6F4` for later setup, execution, or verification.
- **CN:** 声明与 `alignment_bits_b`, `detail`, `get_input_alignment_bits`, `ElementB`, `IsF8F6F4` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2313-2313

```cpp
  int alignment_input = (alignment_input_a == 0 || alignment_input_b == 0) ? 0 : std::max(alignment_input_a, alignment_input_b);
```
- **EN:** Declares member fields or local variables related to `alignment_input`, `alignment_input_a`, `alignment_input_b`, `std`, `max` for later setup, execution, or verification.
- **CN:** 声明与 `alignment_input`, `alignment_input_a`, `alignment_input_b`, `std`, `max` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2315-2322

```cpp
  if constexpr (apply_alignment_offset) {
    // If BlockScaled, then min alignment is SFVecSize
    static constexpr bool IsBlockScaleSupported = Gemm::EpilogueOutputOp::IsBlockScaleSupported;
    static constexpr int SFVecSize = Gemm::GemmKernel::CollectiveMainloop::SFVecSize;
    if constexpr (IsBlockScaleSupported) {
      alignment_input = cutlass::round_up(alignment_input, SFVecSize);
    }
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2325-2330

```cpp
  using CtaShape_MNK = typename Gemm::GemmKernel::CollectiveMainloop::CtaShape_MNK;
  using DispatchPolicy = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy;
  CtaShape_MNK cta_shape;
  Testbed3x<Gemm, cutlass::epilogue::thread::Identity, force_legacy_epilogue> testbed(check_relative_equality, use_device_scalars, vector_scale_mode);
  // For Ptr-Array and Grouped GEMM ideally we need to know SM count at runtime
  static constexpr int SmCount = 16;
```
- **EN:** Introduces type aliases like `CtaShape_MNK`, `DispatchPolicy` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `CtaShape_MNK`, `DispatchPolicy`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 2332-2335

```cpp
  float waves[] = {0.5, 2.5};
  int batches[] = {3};
  int cluster_m = 1;
  int cluster_n = 1;
```
- **EN:** Declares member fields or local variables related to `waves`, `batches`, `cluster_m`, `cluster_n` for later setup, execution, or verification.
- **CN:** 声明与 `waves`, `batches`, `cluster_m`, `cluster_n` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2337-2344

```cpp
  std::vector<int> problem_size_k;
  if (override_problem_size_k.empty()) {
    // this is to test with min alignment
    problem_size_k = {256 - alignment_input, 512 + alignment_input};
  }
  else {
    problem_size_k = override_problem_size_k;
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2346-2350

```cpp
  if constexpr(DispatchPolicy::ArchTag::kMinComputeCapability >= 90) {
    typename DispatchPolicy::ClusterShape cluster_shape;
    cluster_m = cute::size<0>(cluster_shape);
    cluster_n = cute::size<1>(cluster_shape);
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2352-2352

```cpp
  bool passed = true;
```
- **EN:** Declares member fields or local variables related to `passed` for later setup, execution, or verification.
- **CN:** 声明与 `passed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2354-2358

```cpp
  for (int batch : batches) {
    for (float wave : waves) {
      for (int k : problem_size_k) {
        int grid_m, grid_n = 0;
        float num_grid = wave * SmCount;
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2360-2371

```cpp
        if (cluster_m >= cluster_n) {
          grid_m = cluster_m;
          grid_n = static_cast<int>(num_grid) / grid_m;
          // Align grid_n to cluster_n
          grid_n = std::max((grid_n + cluster_n - 1 ) / cluster_n * cluster_n, 1);
        }
        else {
          grid_n = cluster_n;
          grid_m = static_cast<int>(num_grid) / grid_n;
          // Align grid_m to cluster_m
          grid_m = std::max((grid_m + cluster_m - 1 ) / cluster_m * cluster_m, 1);
        }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2373-2374

```cpp
        int m = grid_m * cute::size<0>(cta_shape) - alignment_input; // this is just to test with unusual problem shapes
        int n = grid_n * cute::size<1>(cta_shape) + alignment_input;
```
- **EN:** Implements or wires together logic around `grid_m`, `size`, `cta_shape`, `alignment_input`, `just` for the current test scenario.
- **CN:** 围绕 `grid_m`, `size`, `cta_shape`, `alignment_input`, `just` 实现或连接当前测试场景所需的逻辑。

### Lines 2376-2383

```cpp
        if constexpr (Testbed3x<Gemm, cutlass::epilogue::thread::Identity, force_legacy_epilogue>::IsGroupGemm) {
          std::vector<typename ProblemShapeType::UnderlyingProblemShape> problem_sizes_host;
          cutlass::DeviceAllocation<typename ProblemShapeType::UnderlyingProblemShape> problem_sizes_device;
          for (int i = 0; i < batch; ++i) {
            problem_sizes_host.push_back({m * ((i % 2) + 1), n * ((i % 3) + 1), k * ((i % 2) + 1)});
          }
          problem_sizes_device.reset(problem_sizes_host.size());
          problem_sizes_device.copy_from_host(problem_sizes_host.data());
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2385-2385

```cpp
          ProblemShapeType problem_shapes{batch, problem_sizes_device.get(), problem_sizes_host.data()};
```
- **EN:** Declares member fields or local variables related to `ProblemShapeType`, `problem_shapes`, `batch`, `problem_sizes_device`, `get` for later setup, execution, or verification.
- **CN:** 声明与 `ProblemShapeType`, `problem_shapes`, `batch`, `problem_sizes_device`, `get` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2387-2404

```cpp
          if (CUTLASS_DEBUG_TRACE_LEVEL > 0) {
            for (int i = 0; i < batch; ++i) {
              std::cout << "problem_shapes : "  << problem_shapes.get_host_problem_shape(i) << " \n";
            }
          }
          passed = testbed.run(
            problem_shapes,
            cutlass::from_real<ElementScalar>(alpha),
            cutlass::from_real<ElementScalar>(beta)
          );
        }
        else {
          ProblemShapeType problem_shapes{{m, n, k, batch}};
          if (CUTLASS_DEBUG_TRACE_LEVEL > 0) {
            std::cout << "problem_shapes : "  << problem_shapes.get_host_problem_shape() << " \n";
          }
          passed = testbed.run(
            problem_shapes,
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2405-2408

```cpp
            cutlass::from_real<ElementScalar>(alpha),
            cutlass::from_real<ElementScalar>(beta)
          );
        }
```
- **EN:** Implements or wires together logic around `from_real`, `ElementScalar`, `alpha`, `beta` for the current test scenario.
- **CN:** 围绕 `from_real`, `ElementScalar`, `alpha`, `beta` 实现或连接当前测试场景所需的逻辑。

### Lines 2410-2416

```cpp
        if (!passed) {
          std::cout << __FILE__ << ':' << __LINE__ << " : GEMM MNK " << m << " " << n << " " << k << " FAILED.\n";
          return false;
        }
      } // k
    } // waves
  } // batches
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2418-2419

```cpp
  return passed;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2421-2428

```cpp
template <typename Gemm, bool force_legacy_epilogue = false, bool apply_alignment_offset = true>
bool TestSmallFusion(double alpha = 1.0, double beta = 0.0,
    CheckEquality check_relative_equality = CheckEquality::RELATIVE,
    ScalarLoc use_device_scalars = ScalarLoc::ON_DEVICE,
    VectorScale vector_scale_mode = VectorScale::ENABLED) {
  return TestSmall<Gemm, force_legacy_epilogue, apply_alignment_offset>(
    alpha, beta, check_relative_equality, use_device_scalars, vector_scale_mode);
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2430-2439

```cpp
/// Test for Group GEMM with heterogeneous problem shapes
template <typename Gemm>
bool TestXeGrouped(
    const std::vector<cutlass::gemm::GemmCoord>& problem_sizes, 
    double alpha = 1.0,
    double beta = 0.0
) {
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using UnderlyingProblemShape = typename ProblemShapeType::UnderlyingProblemShape;
  using ElementScalar = typename detail::ElementScalarType<Gemm, float>::Type;
```
- **EN:** Introduces type aliases like `ProblemShapeType`, `UnderlyingProblemShape`, `ElementScalar` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ProblemShapeType`, `UnderlyingProblemShape`, `ElementScalar`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 2441-2441

```cpp
  Testbed3x<Gemm> testbed(CheckEquality::RELATIVE, ScalarLoc::ON_DEVICE, VectorScale::DISABLED);
```
- **EN:** Declares member fields or local variables related to `Testbed3x`, `Gemm`, `testbed`, `CheckEquality`, `RELATIVE` for later setup, execution, or verification.
- **CN:** 声明与 `Testbed3x`, `Gemm`, `testbed`, `CheckEquality`, `RELATIVE` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2443-2449

```cpp
  bool passed = true;
  try {
    // Create host and device arrays from vector of problem sizes
    std::vector<UnderlyingProblemShape> problem_sizes_host;
    for (const auto& coord : problem_sizes) {
      problem_sizes_host.push_back(UnderlyingProblemShape{coord.m(), coord.n(), coord.k()});
    }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 2451-2454

```cpp
    // Allocate device memory and copy
    cutlass::DeviceAllocation<UnderlyingProblemShape> problem_sizes_device;
    problem_sizes_device.reset(problem_sizes_host.size());
    problem_sizes_device.copy_from_host(problem_sizes_host.data(), problem_sizes_host.size());
```
- **EN:** Implements or wires together logic around `Allocate`, `device`, `memory`, `and`, `copy` for the current test scenario.
- **CN:** 围绕 `Allocate`, `device`, `memory`, `and`, `copy` 实现或连接当前测试场景所需的逻辑。

### Lines 2456-2460

```cpp
    // Create GroupProblemShape
    ProblemShapeType group_problem_shape;
    group_problem_shape.num_groups = (int32_t)problem_sizes_host.size();
    group_problem_shape.problem_shapes = problem_sizes_device.get();
    group_problem_shape.host_problem_shapes = problem_sizes_host.data();
```
- **EN:** Implements or wires together logic around `Create`, `GroupProblemShape`, `ProblemShapeType`, `group_problem_shape`, `num_groups` for the current test scenario.
- **CN:** 围绕 `Create`, `GroupProblemShape`, `ProblemShapeType`, `group_problem_shape`, `num_groups` 实现或连接当前测试场景所需的逻辑。

### Lines 2462-2475

```cpp
    passed = testbed.run(
        group_problem_shape,
        ElementScalar(alpha),
        ElementScalar(beta)
    );
  }
  catch (std::exception const& e) {
    EXPECT_TRUE(false) << "TestXeGrouped: testbed.run threw an exception: " << e.what();
    return false;
  }
  catch (...) {
    EXPECT_TRUE(false) << "TestXeGrouped: testbed.run threw an unknown exception";
    return false;
  }
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 2477-2479

```cpp
  EXPECT_TRUE(passed) << "TestXeGrouped: testbed.run failed for " 
                      << problem_sizes.size() << " grouped problems"
                      << ", alpha: " << alpha << ", beta: " << beta;
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 2481-2482

```cpp
  return passed;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2484-2484

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 2486-2491

```cpp
// TestAll template function overload for grouped GEMM testing with explicit problem sizes
template <typename Gemm, template <class T> class ActivationFunctor = cutlass::epilogue::thread::Identity>
bool TestAll(const std::vector<cutlass::gemm::GemmCoord>& problem_sizes,
             double alpha = 1.0, double beta = 0.0) {
  using ElementScalar = typename Gemm::EpilogueOutputOp::ElementScalar;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 2493-2496

```cpp
  if (problem_sizes.empty()) {
    std::cerr << "Error: problem_sizes vector cannot be empty.\n";
    return false;
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2498-2502

```cpp
  Testbed3x<Gemm, ActivationFunctor> testbed(
    CheckEquality::RELATIVE,
    ScalarLoc::ON_DEVICE,
    VectorScale::DISABLED
  );
```
- **EN:** Implements or wires together logic around `Testbed3x`, `Gemm`, `ActivationFunctor`, `testbed`, `CheckEquality` for the current test scenario.
- **CN:** 围绕 `Testbed3x`, `Gemm`, `ActivationFunctor`, `testbed`, `CheckEquality` 实现或连接当前测试场景所需的逻辑。

### Lines 2504-2508

```cpp
  // Convert vector of GemmCoord to the format needed by grouped GEMM testbed
  std::vector<typename ProblemShapeType::UnderlyingProblemShape> problem_sizes_host;
  for (const auto& coord : problem_sizes) {
    problem_sizes_host.push_back({coord.m(), coord.n(), coord.k()});
  }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 2510-2512

```cpp
  cutlass::DeviceAllocation<typename ProblemShapeType::UnderlyingProblemShape> problem_sizes_device;
  problem_sizes_device.reset(problem_sizes_host.size());
  problem_sizes_device.copy_from_host(problem_sizes_host.data());
```
- **EN:** Declares member fields or local variables related to `DeviceAllocation`, `ProblemShapeType`, `UnderlyingProblemShape`, `problem_sizes_device`, `reset` for later setup, execution, or verification.
- **CN:** 声明与 `DeviceAllocation`, `ProblemShapeType`, `UnderlyingProblemShape`, `problem_sizes_device`, `reset` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2515-2523

```cpp
  bool passed = testbed.run(
    ProblemShapeType{
      static_cast<int>(problem_sizes_host.size()),
      problem_sizes_device.get(),
      problem_sizes_host.data()
    },
    cutlass::from_real<ElementScalar>(alpha),
    cutlass::from_real<ElementScalar>(beta)
  );
```
- **EN:** Implements or wires together logic around `passed`, `testbed`, `run`, `ProblemShapeType`, `static_cast` for the current test scenario.
- **CN:** 围绕 `passed`, `testbed`, `run`, `ProblemShapeType`, `static_cast` 实现或连接当前测试场景所需的逻辑。

### Lines 2525-2526

```cpp
  return passed;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2528-2530

```cpp
} // namespace device
} // namespace gemm
} // namespace test
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 2532-2532

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
- **EN:** Grouped or batched problem scheduling.  
  **CN:** 分组或批量问题调度。
- **EN:** Pointer-array based grouped inputs.  
  **CN:** 基于指针数组的 grouped 输入。
- **EN:** Reduction-aware epilogues or verification paths.  
  **CN:** 支持归约的 epilogue 或验证路径。
- **EN:** Epilogue fusion, callbacks, and post-processing composition.  
  **CN:** Epilogue 融合、回调与后处理组合。
- **EN:** CUTLASS device, collective, epilogue, and reference utilities.  
  **CN:** CUTLASS 的 device、collective、epilogue 与参考实现工具。
- **EN:** CUTE shapes, layouts, tiles, and atom abstractions.  
  **CN:** CUTE 的 shape、layout、tile 与 atom 抽象。

## Dependencies / 依赖关系

- `iostream`
- `fstream`
- `sstream`
- `algorithm`
- `random`
- `../../common/cutlass_unit_test.h`
- `cutlass/util/host_tensor.h`
- `cutlass/util/tensor_view_io.h`
- `cutlass/util/distribution.h`
- `cutlass/util/packed_stride.hpp`
- `cutlass/util/reference/host/tensor_fill.h`
- `cutlass/util/reference/host/tensor_copy.h`
- ... and 14 more direct includes / 以及另外 14 个直接依赖头文件
- CUTLASS template APIs and namespaces form the main external programming model / CUTLASS 模板 API 与命名空间构成主要外部编程模型
- CUTE supplies shape/layout metaprogramming primitives / CUTE 提供 shape/layout 元编程原语
