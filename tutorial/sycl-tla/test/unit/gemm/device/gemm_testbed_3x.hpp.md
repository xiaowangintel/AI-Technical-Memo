# gemm_testbed_3x.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/unit/gemm/device/gemm_testbed_3x.hpp`
- **Purpose / 用途:** Primary CUTLASS 3 GEMM testbed used to allocate tensors, launch kernels, and compare reference results.

## Line-by-Line Analysis / 逐行分析

### Lines 1-34

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
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

### Lines 36-36

```cpp
#pragma once
```
- **EN:** Uses `#pragma once` to ensure the header is included only once per translation unit.
- **CN:** 使用 `#pragma once`，确保该头文件在一个编译单元中只被包含一次。

### Lines 38-55

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <algorithm>
#include <random>
#include <numeric> // std::lcm
#include <cfloat>
#include "cutlass/epilogue/collective/default_epilogue.hpp"
#include "cutlass/epilogue/collective/xe_array_epilogue.hpp"
#include "cutlass/epilogue/fusion/xe_callbacks.hpp"
#include "cutlass/gemm/group_array_problem_shape.hpp"
#include "cutlass/gemm/device/gemm_universal.h"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/collective/collective_mma.hpp"
#include "../../common/cutlass_unit_test.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/numeric_conversion.h"
#include "cutlass/util/tensor_view_io.h"
```
- **EN:** Imports dependencies such as `iostream`, `fstream`, `sstream`, `algorithm`, `random`, `numeric`, ... (+12) so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `iostream`, `fstream`, `sstream`, `algorithm`, `random`, `numeric`, ... (+12)，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 56-73

```cpp
#include "cutlass/util/distribution.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/gemm.h"
#include "cutlass/util/reference/host/gett.hpp"
#include "cutlass/epilogue/collective/default_epilogue.hpp"
#include "cutlass/epilogue/collective/xe_epilogue.hpp"
#include "cutlass/epilogue/fusion/xe_callbacks.hpp"
#include "cutlass/epilogue/fusion/operations.hpp"
#include "cutlass/complex.h"
#include "cutlass/transform/device/transform_universal_adapter.hpp"
#include "cutlass/transform/kernel/sparse_gemm_compressor.hpp"
#include "cutlass/detail/collective.hpp"
#include "cutlass/gemm/device/gemm_universal.h"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
```
- **EN:** Imports dependencies such as `distribution.h`, `packed_stride.hpp`, `tensor_fill.h`, `tensor_copy.h`, `tensor_compare.h`, `tensor_norm.h`, ... (+12) so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `distribution.h`, `packed_stride.hpp`, `tensor_fill.h`, `tensor_copy.h`, `tensor_compare.h`, `tensor_norm.h`, ... (+12)，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 74-80

```cpp
#include "cutlass/gemm/collective/collective_mma.hpp"
#include "cutlass/util/GPU_Clock.hpp"
#include "cutlass/util/command_line.h"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/mixed_dtype_utils.hpp"
```
- **EN:** Imports dependencies such as `collective_mma.hpp`, `GPU_Clock.hpp`, `command_line.h`, `device_memory.h`, `gemm_complex.h`, `tensor_compare.h`, ... (+1) so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `collective_mma.hpp`, `GPU_Clock.hpp`, `command_line.h`, `device_memory.h`, `gemm_complex.h`, `tensor_compare.h`, ... (+1)，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 82-82

```cpp
#include "testbed_utils.h"
```
- **EN:** Imports dependencies such as `testbed_utils.h` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `testbed_utils.h`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 84-87

```cpp
// Include SYCL headers for synchronization when SYCL is enabled
#ifdef CUTLASS_ENABLE_SYCL
  #include <sycl/sycl.hpp>
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 89-92

```cpp
#include "cutlass/kernel_hardware_info.hpp"
#include "cutlass/layout/matrix.h"
#include "cutlass/matrix_coord.h"
#include "cutlass/gemm/gemm.h"
```
- **EN:** Imports dependencies such as `kernel_hardware_info.hpp`, `matrix.h`, `matrix_coord.h`, `gemm.h` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `kernel_hardware_info.hpp`, `matrix.h`, `matrix_coord.h`, `gemm.h`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 94-96

```cpp
#include "cute/int_tuple.hpp"
#include "cute/layout.hpp"
#include "cute/numeric/int.hpp"
```
- **EN:** Imports dependencies such as `int_tuple.hpp`, `layout.hpp`, `int.hpp` so the file can access the required CUTLASS/CUTE APIs and shared test utilities.
- **CN:** 引入依赖头文件，例如 `int_tuple.hpp`, `layout.hpp`, `int.hpp`，使本文件能够使用所需的 CUTLASS/CUTE API 与共享测试工具。

### Lines 98-100

```cpp
namespace test {
namespace gemm {
namespace device {
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 102-102

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 104-107

```cpp
enum class ScalarLoc {
  ON_HOST = 0,
  ON_DEVICE = 1
};
```
- **EN:** Declares enumeration types or constants (`enum`, `ScalarLoc`, `ON_HOST`, `ON_DEVICE`) so later code can express modes and options explicitly.
- **CN:** 声明枚举类型或常量（`enum`, `ScalarLoc`, `ON_HOST`, `ON_DEVICE`），使后续代码能够更明确地表达模式与选项。

### Lines 109-112

```cpp
enum class VectorScale {
  DISABLED = 0,
  ENABLED = 1
};
```
- **EN:** Declares enumeration types or constants (`enum`, `VectorScale`, `DISABLED`, `ENABLED`) so later code can express modes and options explicitly.
- **CN:** 声明枚举类型或常量（`enum`, `VectorScale`, `DISABLED`, `ENABLED`），使后续代码能够更明确地表达模式与选项。

### Lines 114-117

```cpp
enum class CheckEquality {
  EXACT = 0,
  RELATIVE = 1
};
```
- **EN:** Declares enumeration types or constants (`enum`, `CheckEquality`, `EXACT`, `RELATIVE`) so later code can express modes and options explicitly.
- **CN:** 声明枚举类型或常量（`enum`, `CheckEquality`, `EXACT`, `RELATIVE`），使后续代码能够更明确地表达模式与选项。

### Lines 119-119

```cpp
namespace detail {
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 121-138

```cpp
template <typename Mode>
constexpr auto
decomp_mode_to_string(Mode mode) {
    if (mode == Mode::Heuristic) {
      return "Heuristic";
    }
    else if (mode == Mode::DataParallel) {
      return "DataParallel";
    }
    else if (mode == Mode::SplitK) {
      return "SplitK";
    }
    else if (mode == Mode::StreamK) {
      return "StreamK";
    }
    else {
      return "Unknown";
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 139-139

```cpp
  };
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 141-156

```cpp
inline constexpr auto raster_order_to_string =
  [] (cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90Params::RasterOrderOptions mode) -> std::string {
    using Mode = cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90Params::RasterOrderOptions;
    if (mode == Mode::Heuristic) {
      return "Heuristic";
    }
    else if (mode == Mode::AlongM) {
      return "AlongM";
    }
    else if (mode == Mode::AlongN) {
      return "AlongN";
    }
    else {
      return "Unknown";
    }
  };
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 158-165

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

### Lines 167-170

```cpp
template <typename Gemm, typename Default>
struct ElementComputeType<Gemm, Default, std::enable_if_t<not std::is_void_v<typename Gemm::EpilogueOutputOp::ElementCompute>>> {
  using Type = typename Gemm::EpilogueOutputOp::ElementCompute;
};
```
- **EN:** Defines templated type `ElementComputeType` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ElementComputeType`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 172-175

```cpp
template <typename Gemm, typename Default, typename = void>
struct ElementScalarType {
  using Type = Default;
};
```
- **EN:** Defines templated type `ElementScalarType` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ElementScalarType`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 177-180

```cpp
template <typename Gemm, typename Default>
struct ElementScalarType<Gemm, Default, std::enable_if_t<not std::is_void_v<typename Gemm::EpilogueOutputOp::ElementScalar>>> {
  using Type = typename Gemm::EpilogueOutputOp::ElementScalar;
};
```
- **EN:** Defines templated type `ElementScalarType` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ElementScalarType`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 183-186

```cpp
template <typename Gemm, typename = void>
struct IsF8F6F4Kernel {
  static constexpr bool value = false;
};
```
- **EN:** Defines templated type `IsF8F6F4Kernel` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsF8F6F4Kernel`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 188-191

```cpp
template <typename Gemm>
struct IsF8F6F4Kernel<Gemm, std::void_t<decltype(Gemm::GemmKernel::CollectiveMainloop::IsF8F6F4)>> {
  static constexpr bool value = true;
};
```
- **EN:** Defines templated type `IsF8F6F4Kernel` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsF8F6F4Kernel`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 194-195

```cpp
template<class CollectiveEpilogue, class = void>
struct IsSfdEpi : cute::false_type {};
```
- **EN:** Defines templated type `CollectiveEpilogue` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `CollectiveEpilogue`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 197-198

```cpp
template<class CollectiveEpilogue>
struct IsSfdEpi<CollectiveEpilogue, cute::void_t<typename CollectiveEpilogue::FusionCallbacks::Operation::GmemLayoutTagScalefactor>> : cute::true_type {};
```
- **EN:** Defines templated type `CollectiveEpilogue` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `CollectiveEpilogue`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 200-206

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

### Lines 208-215

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

### Lines 217-220

```cpp
template <typename T>
auto make_iterator(T* ptr) {
  return cute::recast_ptr<T>(ptr);
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 222-225

```cpp
template<class T>
struct IsDefaultEpilogue {
  static constexpr bool value = false;
};
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 227-230

```cpp
template<class ...args>
struct IsDefaultEpilogue<cutlass::epilogue::collective::DefaultEpilogue<args...>> {
  static constexpr bool value = true;
};
```
- **EN:** Defines templated type `IsDefaultEpilogue` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsDefaultEpilogue`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 232-235

```cpp
template<class ...args>
struct IsDefaultEpilogue<cutlass::epilogue::collective::detail::Sm90TmaWarpSpecializedAdapter<args...>> {
  static constexpr bool value = true;
};
```
- **EN:** Defines templated type `IsDefaultEpilogue` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsDefaultEpilogue`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 237-240

```cpp
template <typename Epilogue, typename = void>
struct IsLegacyEpiloguePolicy {
  static constexpr bool value = false;
};
```
- **EN:** Defines templated type `IsLegacyEpiloguePolicy` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `IsLegacyEpiloguePolicy`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 242-249

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

### Lines 251-266

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

### Lines 268-275

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

### Lines 277-284

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

### Lines 286-293

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

### Lines 295-299

```cpp
template <typename Element, typename Layout>
bool initialize_tensor(
  cutlass::TensorView<Element, Layout> view,
  cutlass::Distribution::Kind dist_kind,
  uint64_t seed) {
```
- **EN:** Implements or wires together logic around `Element`, `initialize_tensor`, `TensorView`, `view`, `Distribution` for the current test scenario.
- **CN:** 围绕 `Element`, `initialize_tensor`, `TensorView`, `view`, `Distribution` 实现或连接当前测试场景所需的逻辑。

### Lines 301-303

```cpp
  if (dist_kind == cutlass::Distribution::Uniform) {
    double scope_max, scope_min;
    int bits_input = cutlass::sizeof_bits<Element>::value;
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 305-308

```cpp
    if (bits_input == 1) {
      scope_max = 2;
      scope_min = 0;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 310-313

```cpp
    else if (bits_input <= 6) {
      scope_max = 2;
      scope_min = -2;
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 315-315

```cpp
    else if (bits_input <= 8) {
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 317-322

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

### Lines 324-327

```cpp
        scope_max = 1;
        scope_min = -1;
      }
```
- **EN:** Implements or wires together logic around `scope_max`, `scope_min` for the current test scenario.
- **CN:** 围绕 `scope_max`, `scope_min` 实现或连接当前测试场景所需的逻辑。

### Lines 329-340

```cpp
    }
    else if (cute::is_any_of_v<Element, cutlass::bfloat16_t, cutlass::half_t>) {
      scope_max = 1;
      scope_min = -1;
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

### Lines 342-344

```cpp
  else if (dist_kind == cutlass::Distribution::Identity) {
    cutlass::reference::host::TensorFillIdentity(view);
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 346-348

```cpp
  else if (dist_kind == cutlass::Distribution::Gaussian) {
    cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 350-353

```cpp
  else if (dist_kind == cutlass::Distribution::Sequential) {
    cutlass::reference::host::BlockFillSequential(
      view.data(), view.capacity());
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 355-357

```cpp
  else if (dist_kind == cutlass::Distribution::AllOnes) {
    cutlass::reference::host::TensorFill(view, Element(1));
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 359-361

```cpp
  else if (dist_kind == cutlass::Distribution::AllZeros) {
    cutlass::reference::host::TensorFill(view, Element(0));
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 363-366

```cpp
  else {
    EXPECT_TRUE(false) << "Not implemented";
    return false;
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 368-369

```cpp
  return true;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 371-378

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

### Lines 381-398

```cpp
//
// Default MMA input Operands : A , B
//
template<
  class ScheduleType_,
  class Gemm,
  class ElementA_ = typename Gemm::GemmKernel::ElementA,
  class ElementB_ = typename Gemm::GemmKernel::ElementB,
  class Enable = void>
struct HostCollectiveMainloop {
  // Kernel data types
  using ElementA = ElementA_;
  using StrideA  = typename Gemm::GemmKernel::StrideA;
  using ElementB = ElementB_;
  using StrideB  = typename Gemm::GemmKernel::StrideB;
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  using LayoutTagA = cutlass::detail::StrideToLayoutTagA_t<StrideA>;
  using LayoutTagB = cutlass::detail::StrideToLayoutTagB_t<StrideB>;
```
- **EN:** Defines templated type `ScheduleType_` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ScheduleType_`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 400-403

```cpp
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using EpilogueOutputOp = typename Gemm::EpilogueOutputOp;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 405-405

```cpp
  using Arguments = typename Gemm::GemmKernel::MainloopArguments;
```
- **EN:** Defines aliases such as `Arguments` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Arguments`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 407-408

```cpp
  cutlass::ComplexTransform TransformA = Gemm::kTransformA;
  cutlass::ComplexTransform TransformB = Gemm::kTransformB;
```
- **EN:** Declares member fields or local variables related to `ComplexTransform`, `TransformA`, `Gemm`, `kTransformA`, `TransformB` for later setup, execution, or verification.
- **CN:** 声明与 `ComplexTransform`, `TransformA`, `Gemm`, `kTransformA`, `TransformB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 410-411

```cpp
  StrideA stride_a;
  StrideB stride_b;
```
- **EN:** Declares member fields or local variables related to `StrideA`, `stride_a`, `StrideB`, `stride_b` for later setup, execution, or verification.
- **CN:** 声明与 `StrideA`, `stride_a`, `StrideB`, `stride_b` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 413-414

```cpp
  typename LayoutTagA::Stride stride_factor_A;
  typename LayoutTagB::Stride stride_factor_B;
```
- **EN:** Declares member fields or local variables related to `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 416-417

```cpp
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
```
- **EN:** Declares member fields or local variables related to `Distribution`, `Kind`, `init_A`, `init_B` for later setup, execution, or verification.
- **CN:** 声明与 `Distribution`, `Kind`, `init_A`, `init_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 419-422

```cpp
  cutlass::HostTensor<ElementA, LayoutTagA> tensor_A;
  cutlass::HostTensor<ElementB, LayoutTagB> tensor_B;
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
```
- **EN:** Implements or wires together logic around `HostTensor`, `ElementA`, `LayoutTagA`, `tensor_A`, `ElementB` for the current test scenario.
- **CN:** 围绕 `HostTensor`, `ElementA`, `LayoutTagA`, `tensor_A`, `ElementB` 实现或连接当前测试场景所需的逻辑。

### Lines 424-425

```cpp
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
```
- **EN:** Declares member fields or local variables related to `uint64_t`, `seed`, `kDefaultSeed` for later setup, execution, or verification.
- **CN:** 声明与 `uint64_t`, `seed`, `kDefaultSeed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 427-431

```cpp
  // Note: this limitation comes from testbed / not the library
  static_assert(is_row_or_col_major<StrideA>(),
    "ERROR : A Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<StrideB>(),
    "ERROR : B Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 433-444

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

### Lines 446-458

```cpp
  template<class ProblemShapeType>
  bool initialize(ProblemShapeType problem_size) {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("HostCollectiveMainloop (generic)::initialize(problem_shape)");
#endif
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 460-461

```cpp
    stride_a = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, K, L));
    stride_b = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(N, K, L));
```
- **EN:** Declares member fields or local variables related to `stride_a`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `stride_b` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `stride_b` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 463-467

```cpp
    // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
    auto a_coord = cutlass::make_Coord(M * L, K);
    // Cutlass has Row/Col major refers to MxK times KxN matrix product,
    // so the HostTensorB should be treated as KxN in "coord"'s view
    auto b_coord = cutlass::make_Coord(K, N * L);
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 469-486

```cpp
    try {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: tensor_A.resize");
#endif
      tensor_A.resize(a_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_coord, stride_factor_A));
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: tensor_B.resize");
#endif
      tensor_B.resize(b_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(b_coord, stride_factor_B));
    }
    catch (std::exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: tensor A or B resize threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: tensor A or B resize threw an unknown exception");
      throw;
    }
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 488-503

```cpp
    try {
      EXPECT_TRUE(initialize_tensor(tensor_A.host_view(), init_A, seed + 2022));
      EXPECT_TRUE(initialize_tensor(tensor_B.host_view(), init_B, seed + 2021));
    }
    catch (cutlass::cuda_exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: checked initialize_tensor threw cutlass::cuda_exception: " << e);
      throw;
    }
    catch (std::exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: checked initialize_tensor threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: checked_initialize_tensor threw an unknown exception");
      throw;
    }
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 505-508

```cpp
    // It is possible to randomly initialize to all zeros, so override this with non-zeros
    // in the upper left corner of each operand.
    tensor_A.host_view().at({0, 0}) = ElementA(1);
    tensor_B.host_view().at({0, 0}) = ElementB(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 510-527

```cpp
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: Check last error before sync_device()");
#if !defined(CUTLASS_ENABLE_SYCL)
      cudaError_t error = cudaGetLastError();
      const auto error_str = cudaGetErrorString(error);
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: cudaGetLastError() is " << error_str);
#endif
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: tensor_A.host_data()=" << tensor_A.host_data() << ", tensor_A.device_data()=" << tensor_A.device_data());
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: tensor_B.host_data()=" << tensor_B.host_data() << ", tensor_B.device_data()=" << tensor_B.device_data());
    }
#endif
    try {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: tensor_A.sync_device");
#endif
      tensor_A.sync_device();
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 528-543

```cpp
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: tensor_B.sync_device");
#endif
      tensor_B.sync_device();
    }
    catch (cutlass::cuda_exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: sync_device() threw cutlass::cuda_exception: " << e);
      throw;
    }
    catch (std::exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: sync_device() threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: sync_device() threw an unknown exception");
      throw;
    }
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 545-549

```cpp
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("HostCollectiveMainloop::initialize: Reached end");
#endif
    return true;
  }
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 551-551

```cpp
  Arguments to_args() {
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 554-563

```cpp
    // Runtime datatype selection
    if constexpr (not cute::is_same_v<ElementA, typename Gemm::GemmKernel::ElementA>) {
      using ArrayElementA = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementA;
      using ArrayElementB = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementB;
      return {
        reinterpret_cast<ArrayElementA *>(tensor_A.device_data()), stride_a,
        reinterpret_cast<ArrayElementB *>(tensor_B.device_data()), stride_b
      };
    }
    else {
```
- **EN:** Introduces type aliases like `ArrayElementA`, `ArrayElementB` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ArrayElementA`, `ArrayElementB`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 565-571

```cpp
    Arguments arguments =
    {
      tensor_A.device_data(), stride_a, tensor_B.device_data(), stride_b
    };
    return arguments;
    }
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 573-586

```cpp
  auto to_host_args(ProblemShapeType problem_size) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
    auto A = make_tensor(make_iterator(tensor_A.host_data()),
          make_layout(make_shape(M, K, L), stride_a));
    auto B = make_tensor(make_iterator(tensor_B.host_data()),
        make_layout(make_shape(N, K, L), stride_b));
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 589-592

```cpp
    auto dummy_SFA = cute::make_tensor(static_cast<ElementA*>(nullptr),
        cute::make_layout(cute::make_shape(M, K, L), stride_a));
    auto dummy_SFB = cute::make_tensor(static_cast<ElementB*>(nullptr),
        cute::make_layout(cute::make_shape(N, K, L), stride_b));
```
- **EN:** Implements or wires together logic around `dummy_SFA`, `make_tensor`, `static_cast`, `ElementA`, `nullptr` for the current test scenario.
- **CN:** 围绕 `dummy_SFA`, `make_tensor`, `static_cast`, `ElementA`, `nullptr` 实现或连接当前测试场景所需的逻辑。

### Lines 594-596

```cpp
    cutlass::reference::host::GettMainloopParams<ElementAccumulator,
                                                 decltype(A),
                                                 decltype(B)
```
- **EN:** Implements or wires together logic around `reference`, `host`, `GettMainloopParams`, `ElementAccumulator`, `decltype` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `GettMainloopParams`, `ElementAccumulator`, `decltype` 实现或连接当前测试场景所需的逻辑。

### Lines 598-599

```cpp
                                                 , decltype(dummy_SFA),
                                                 decltype(dummy_SFB)
```
- **EN:** Implements or wires together logic around `decltype`, `dummy_SFA`, `dummy_SFB` for the current test scenario.
- **CN:** 围绕 `decltype`, `dummy_SFA`, `dummy_SFB` 实现或连接当前测试场景所需的逻辑。

### Lines 601-601

```cpp
                                                 > mainloop_params{};
```
- **EN:** Declares member fields or local variables related to `mainloop_params` for later setup, execution, or verification.
- **CN:** 声明与 `mainloop_params` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 603-606

```cpp
    mainloop_params.A = A;
    mainloop_params.B = B;
    mainloop_params.transform_A = TransformA;
    mainloop_params.transform_B = TransformB;
```
- **EN:** Declares member fields or local variables related to `mainloop_params`, `transform_A`, `TransformA`, `transform_B`, `TransformB` for later setup, execution, or verification.
- **CN:** 声明与 `mainloop_params`, `transform_A`, `TransformA`, `transform_B`, `TransformB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 608-609

```cpp
    return mainloop_params;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 611-614

```cpp
  void print_tensors(std::ofstream& file) {
    file << "A =\n" << tensor_A.host_view()
         << "\nB =\n" << tensor_B.host_view();
  }
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 616-622

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

### Lines 624-629

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

### Lines 631-643

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

### Lines 645-648

```cpp
  bool compare_reference(
      cute::Shape<int,int,int,int> problem_shape_MNKL) {
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_A.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_B.host_view()), 0);
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 650-653

```cpp
    bool passed = true;
    return passed;
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 655-663

```cpp
//
// Sparse MMA host implementation
//
template<
  class Gemm,
  class ElementA_,
  class ElementB_>
struct HostCollectiveMainloopSparse
{
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 665-671

```cpp
  // Kernel data types
  using ElementA = ElementA_;
  // CuTe layout A for the kernel's sparse tensorA.
  using LayoutA  = typename Gemm::GemmKernel::CollectiveMainloop::LayoutA;
  using ElementB = ElementB_;
  using StrideB  = typename Gemm::GemmKernel::StrideB;
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
```
- **EN:** Introduces type aliases like `ElementA`, `LayoutA`, `ElementB`, `StrideB`, `ScheduleType` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ElementA`, `LayoutA`, `ElementB`, `StrideB`, `ScheduleType`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 673-680

```cpp
  using ElementE = typename Gemm::GemmKernel::CollectiveMainloop::ElementE;
  // CuTe layout E for the kernel's metadata tensor.
  using LayoutE  = typename Gemm::GemmKernel::CollectiveMainloop::LayoutE;
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using EpilogueOutputOp = typename Gemm::EpilogueOutputOp;
  using SparseConfig = typename Gemm::GemmKernel::CollectiveMainloop::SparseConfig;
```
- **EN:** Introduces type aliases like `ElementE`, `LayoutE`, `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp`, ... (+1) so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ElementE`, `LayoutE`, `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp`, ... (+1)，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 682-686

```cpp
  // The following typenames are for the reference host tensors. They are non-sparse tensors.
  using LayoutTagA = decltype(SparseConfig::deduce_layoutA_tag(LayoutA{}));
  using StrideA = cutlass::gemm::TagToStrideA_t<LayoutTagA>;
  // We don't care about the actual strideE for the host tensor, but just need one to allocate memory.
  using StrideE = StrideA;
```
- **EN:** Introduces type aliases like `LayoutTagA`, `StrideA`, `StrideE` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `LayoutTagA`, `StrideA`, `StrideE`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 688-690

```cpp
  // Deduce Cutlass Layouts (RowMajor & ColumnMajor)
  using LayoutTagB = cutlass::detail::StrideToLayoutTagB_t<StrideB>;
  using LayoutTagE = cutlass::detail::StrideToLayoutTagA_t<StrideE>;
```
- **EN:** Begins function or method `Layouts`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `Layouts`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 692-692

```cpp
  using ArchTag = typename Gemm::ArchTag;
```
- **EN:** Defines aliases such as `ArchTag` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ArchTag`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 694-698

```cpp
  using CompressorUtility = cutlass::transform::kernel::StructuredSparseCompressorUtility<
                              cute::Shape<int, int, int, int>,
                              ElementA,
                              LayoutTagA,
                              SparseConfig>;
```
- **EN:** Implements or wires together logic around `CompressorUtility`, `transform`, `kernel`, `StructuredSparseCompressorUtility`, `ElementA` for the current test scenario.
- **CN:** 围绕 `CompressorUtility`, `transform`, `kernel`, `StructuredSparseCompressorUtility`, `ElementA` 实现或连接当前测试场景所需的逻辑。

### Lines 700-705

```cpp
  using CompressorKernel = cutlass::transform::kernel::StructuredSparseCompressor<
                              cute::Shape<int, int, int, int>,
                              ElementA,
                              LayoutTagA,
                              SparseConfig,
                              ArchTag>;
```
- **EN:** Implements or wires together logic around `CompressorKernel`, `transform`, `kernel`, `StructuredSparseCompressor`, `ElementA` for the current test scenario.
- **CN:** 围绕 `CompressorKernel`, `transform`, `kernel`, `StructuredSparseCompressor`, `ElementA` 实现或连接当前测试场景所需的逻辑。

### Lines 707-707

```cpp
  using Compressor = cutlass::transform::device::TransformUniversalAdapter<CompressorKernel>;
```
- **EN:** Defines aliases such as `Compressor` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Compressor`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 709-711

```cpp
  using Arguments = typename Gemm::GemmKernel::MainloopArguments;
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
```
- **EN:** Implements or wires together logic around `Arguments`, `Gemm`, `GemmKernel`, `MainloopArguments`, `Whether` for the current test scenario.
- **CN:** 围绕 `Arguments`, `Gemm`, `GemmKernel`, `MainloopArguments`, `Whether` 实现或连接当前测试场景所需的逻辑。

### Lines 713-717

```cpp
  // Note: this limitation comes from testbed / not the library
  static_assert(is_row_or_col_major<StrideA>(),
    "ERROR : A Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<StrideB>(),
    "ERROR : B Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 719-722

```cpp
  StrideA stride_a;
  StrideA stride_a_compressed;
  StrideB stride_b;
  StrideE stride_e;
```
- **EN:** Declares member fields or local variables related to `StrideA`, `stride_a`, `stride_a_compressed`, `StrideB`, `stride_b` for later setup, execution, or verification.
- **CN:** 声明与 `StrideA`, `stride_a`, `stride_a_compressed`, `StrideB`, `stride_b` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 724-725

```cpp
  LayoutA layout_a;
  LayoutE layout_e;
```
- **EN:** Declares member fields or local variables related to `LayoutA`, `layout_a`, `LayoutE`, `layout_e` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutA`, `layout_a`, `LayoutE`, `layout_e` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 727-729

```cpp
  typename LayoutTagA::Stride stride_factor_A;
  typename LayoutTagB::Stride stride_factor_B;
  typename LayoutTagE::Stride stride_factor_E;
```
- **EN:** Declares member fields or local variables related to `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 731-732

```cpp
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
```
- **EN:** Declares member fields or local variables related to `Distribution`, `Kind`, `init_A`, `init_B` for later setup, execution, or verification.
- **CN:** 声明与 `Distribution`, `Kind`, `init_A`, `init_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 734-740

```cpp
  cutlass::HostTensor<ElementA, LayoutTagA> tensor_A;
  cutlass::HostTensor<ElementA, LayoutTagA> tensor_A_Comp;
  cutlass::HostTensor<ElementB, LayoutTagB> tensor_B;
  cutlass::HostTensor<ElementE, LayoutTagE> tensor_E;
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
  static constexpr int MaxSmCount = 16;
```
- **EN:** Declares member fields or local variables related to `HostTensor`, `ElementA`, `LayoutTagA`, `tensor_A`, `tensor_A_Comp` for later setup, execution, or verification.
- **CN:** 声明与 `HostTensor`, `ElementA`, `LayoutTagA`, `tensor_A`, `tensor_A_Comp` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 742-755

```cpp
  HostCollectiveMainloopSparse(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = kDefaultSeed,
    typename LayoutTagA::Stride stride_factor_A_ = typename LayoutTagA::Stride(),
    typename LayoutTagB::Stride stride_factor_B_ = typename LayoutTagB::Stride(),
    typename LayoutTagE::Stride stride_factor_E_ = typename LayoutTagE::Stride()
  ):
    check_relative_equality(check_relative_equality_),
    stride_factor_A(stride_factor_A_),
    stride_factor_B(stride_factor_B_),
    stride_factor_E(stride_factor_E_),
    init_A(init_A_), init_B(init_B_), seed(seed_) { }
```
- **EN:** Implements or wires together logic around `HostCollectiveMainloopSparse`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution` for the current test scenario.
- **CN:** 围绕 `HostCollectiveMainloopSparse`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution` 实现或连接当前测试场景所需的逻辑。

### Lines 757-769

```cpp
  template<class ProblemShapeType>
  bool initialize(ProblemShapeType problem_size) {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("HostCollectiveMainloopSparse::initialize");
#endif
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 771-772

```cpp
    stride_a = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, K, L));
    stride_b = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(N, K, L));
```
- **EN:** Declares member fields or local variables related to `stride_a`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `stride_b` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `stride_b` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 774-774

```cpp
    CompressorUtility compressor_utility(problem_shape_MNKL, stride_a);
```
- **EN:** Declares member fields or local variables related to `CompressorUtility`, `compressor_utility`, `problem_shape_MNKL`, `stride_a` for later setup, execution, or verification.
- **CN:** 声明与 `CompressorUtility`, `compressor_utility`, `problem_shape_MNKL`, `stride_a` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 776-781

```cpp
    // TensorE
    // In unit of ElementE (uint8_t), after alignment requirement
    // M-dim: TensorEAtom_M alignment
    // K-dim: TensorEAtom_K alignment
    int KAlignedE = compressor_utility.get_metadata_k_physical();
    int MAlignedE = compressor_utility.get_metadata_m_physical();
```
- **EN:** Implements or wires together logic around `TensorE`, `unit`, `ElementE`, `uint8_t`, `after` for the current test scenario.
- **CN:** 围绕 `TensorE`, `unit`, `ElementE`, `uint8_t`, `after` 实现或连接当前测试场景所需的逻辑。

### Lines 783-788

```cpp
    // TensorA Compressed
    // In unit of ElementARaw, after alignment requirement
    // M-dim: TMA alignment
    // K-dim: TMA alignment
    int KAlignedAC = compressor_utility.get_tensorA_k_physical();
    int MAlignedAC = compressor_utility.get_tensorA_m_physical();
```
- **EN:** Implements or wires together logic around `TensorA`, `Compressed`, `unit`, `ElementARaw`, `after` for the current test scenario.
- **CN:** 围绕 `TensorA`, `Compressed`, `unit`, `ElementARaw`, `after` 实现或连接当前测试场景所需的逻辑。

### Lines 790-791

```cpp
    stride_a_compressed = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, KAlignedAC, L));
    stride_e = cutlass::make_cute_packed_stride(StrideE{}, cute::make_shape(MAlignedE, KAlignedE, L));
```
- **EN:** Declares member fields or local variables related to `stride_a_compressed`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `KAlignedAC` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a_compressed`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `KAlignedAC` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 793-796

```cpp
    auto a_coord = cutlass::make_Coord(M * L, K);
    auto b_coord = cutlass::make_Coord(K, N * L);
    auto e_coord = cutlass::make_Coord(MAlignedE * L, KAlignedE);
    auto a_comp_coord = cutlass::make_Coord(MAlignedAC * L, KAlignedAC);
```
- **EN:** Declares member fields or local variables related to `a_coord`, `make_Coord`, `b_coord`, `e_coord`, `MAlignedE` for later setup, execution, or verification.
- **CN:** 声明与 `a_coord`, `make_Coord`, `b_coord`, `e_coord`, `MAlignedE` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 798-801

```cpp
    tensor_A.resize(a_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_coord, stride_factor_A));
    tensor_A_Comp.resize(a_comp_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_comp_coord, stride_factor_A));
    tensor_B.resize(b_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(b_coord, stride_factor_B));
    tensor_E.resize(e_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagE>::layout_factory(e_coord, stride_factor_E));
```
- **EN:** Declares member fields or local variables related to `tensor_A`, `resize`, `a_coord`, `layout`, `Affine2Layout_Factory` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_A`, `resize`, `a_coord`, `layout`, `Affine2Layout_Factory` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 803-804

```cpp
    EXPECT_TRUE(initialize_tensor(tensor_A.host_view(), init_A, seed + 2022));
    EXPECT_TRUE(initialize_tensor(tensor_B.host_view(), init_B, seed + 2021));
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `initialize_tensor`, `tensor_A`, `host_view`, `init_A` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `initialize_tensor`, `tensor_A`, `host_view`, `init_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 806-809

```cpp
    // It is possible to randomly initialize to all zeros, so override this with non-zeros
    // in the upper left corner of each operand.
    tensor_A.host_view().at({0, 0}) = ElementA(1);
    tensor_B.host_view().at({0, 0}) = ElementB(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 811-811

```cpp
    compressor_utility.structure_sparse_zero_mask_fill(tensor_A.host_data(), static_cast<int>(seed + 2023));
```
- **EN:** Declares member fields or local variables related to `compressor_utility`, `structure_sparse_zero_mask_fill`, `tensor_A`, `host_data`, `static_cast` for later setup, execution, or verification.
- **CN:** 声明与 `compressor_utility`, `structure_sparse_zero_mask_fill`, `tensor_A`, `host_data`, `static_cast` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 813-816

```cpp
    tensor_A.sync_device();
    tensor_B.sync_device();
    tensor_E.sync_device();
    tensor_A_Comp.sync_device();
```
- **EN:** Declares member fields or local variables related to `tensor_A`, `sync_device`, `tensor_B`, `tensor_E`, `tensor_A_Comp` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_A`, `sync_device`, `tensor_B`, `tensor_E`, `tensor_A_Comp` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 818-818

```cpp
    cutlass::Status status {cutlass::Status::kSuccess };
```
- **EN:** Declares member fields or local variables related to `Status`, `status`, `kSuccess` for later setup, execution, or verification.
- **CN:** 声明与 `Status`, `status`, `kSuccess` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 820-830

```cpp
    cutlass::KernelHardwareInfo hw_info;
    hw_info.device_id = 0;
    hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
    typename Compressor::Arguments arguments{
      {M, N, K, L},
      {tensor_A.device_data(),
       stride_a,
       tensor_A_Comp.device_data(),
       tensor_E.device_data()},
      {hw_info}
    };
```
- **EN:** Implements or wires together logic around `KernelHardwareInfo`, `hw_info`, `device_id`, `sm_count`, `query_device_multiprocessor_count` for the current test scenario.
- **CN:** 围绕 `KernelHardwareInfo`, `hw_info`, `device_id`, `sm_count`, `query_device_multiprocessor_count` 实现或连接当前测试场景所需的逻辑。

### Lines 832-834

```cpp
    Compressor compressor_op;
    size_t workspace_size = Compressor::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```
- **EN:** Declares member fields or local variables related to `Compressor`, `compressor_op`, `size_t`, `workspace_size`, `get_workspace_size` for later setup, execution, or verification.
- **CN:** 声明与 `Compressor`, `compressor_op`, `size_t`, `workspace_size`, `get_workspace_size` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 836-839

```cpp
    status = compressor_op.can_implement(arguments);
    if (status != cutlass::Status::kSuccess) {
      return false;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 841-844

```cpp
    status = compressor_op.initialize(arguments, workspace.get());
    if (status != cutlass::Status::kSuccess) {
      return false;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 846-846

```cpp
    status = compressor_op.run();
```
- **EN:** Declares member fields or local variables related to `status`, `compressor_op`, `run` for later setup, execution, or verification.
- **CN:** 声明与 `status`, `compressor_op`, `run` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 848-863

```cpp
#if defined(CUTLASS_ENABLE_SYCL)
    try {
      compat::wait_and_throw();
    } catch (std::exception const &e) {
      ADD_FAILURE() << "Error at Kernel Sync.";
      return false;
    }
#else
    auto result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      EXPECT_EQ(result, cudaSuccess) << "Error at Kernel Sync.";
      return false;
    }
#endif
    layout_a = SparseConfig::fill_layoutA(problem_shape_MNKL);
    layout_e = SparseConfig::fill_layoutE(problem_shape_MNKL);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 865-866

```cpp
    tensor_E.sync_host();
    tensor_A_Comp.sync_host();
```
- **EN:** Declares member fields or local variables related to `tensor_E`, `sync_host`, `tensor_A_Comp` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_E`, `sync_host`, `tensor_A_Comp` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 868-869

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 871-879

```cpp
  Arguments to_args() {
    using ArrayElementA = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementA;
    using ArrayElementB = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementB;
    return {
      reinterpret_cast<ArrayElementA *>(tensor_A_Comp.device_data()), layout_a,
      reinterpret_cast<ArrayElementB *>(tensor_B.device_data()), stride_b,
      tensor_E.device_data(), layout_e
    };
  }
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 881-894

```cpp
  auto to_host_args(ProblemShapeType problem_size) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
    auto A = make_tensor(make_iterator(tensor_A.host_data()),
          make_layout(make_shape(M, K, L), stride_a));
    auto B = make_tensor(make_iterator(tensor_B.host_data()),
        make_layout(make_shape(N, K, L), stride_b));
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 896-898

```cpp
    cutlass::reference::host::GettMainloopParams<ElementAccumulator, decltype(A), decltype(B)> mainloop_params{A, B};
    return mainloop_params;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 900-903

```cpp
  void print_tensors(std::ofstream& file) {
    file << "A =\n" << tensor_A.host_view()
         << "\nB =\n" << tensor_B.host_view();
  }
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 905-907

```cpp
  bool compare_reference(
      cute::Shape<int,int,int,int> problem_shape_MNKL) {
    auto [M, N, K, L] = problem_shape_MNKL;
```
- **EN:** Implements or wires together logic around `compare_reference`, `problem_shape_MNKL` for the current test scenario.
- **CN:** 围绕 `compare_reference`, `problem_shape_MNKL` 实现或连接当前测试场景所需的逻辑。

### Lines 909-913

```cpp
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_A.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_B.host_view()), 0);
    return true;
  }
};
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 915-931

```cpp
template<
  class ScheduleType_,
  class Gemm,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<ScheduleType_, Gemm, ElementA_, ElementB_,
    cute::enable_if_t<
      cute::is_base_of_v<
        cutlass::gemm::MainloopSm90TmaGmmaWarpSpecializedSparse<Gemm::CollectiveMainloop::DispatchPolicy::Stages,
                                                                typename Gemm::CollectiveMainloop::DispatchPolicy::ClusterShape,
                                                                ScheduleType_>,
        typename Gemm::CollectiveMainloop::DispatchPolicy>>>
  : HostCollectiveMainloopSparse<Gemm, ElementA_, ElementB_>
{
  using HostCollectiveMainloopSparse<Gemm, ElementA_, ElementB_>::HostCollectiveMainloopSparse;
};
```
- **EN:** Defines templated type `ScheduleType_` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `ScheduleType_`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 933-936

```cpp
//
// Sparse MMA input Operands : A_compressed, B, metadata
//
// Structured Sparse Gemm Input Operands
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 938-951

```cpp
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  int AccumulatorPipelineStageCount_,
  typename ElementA_,
  typename ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedSm100<SchedulerPipelineStageCount_,
                                                                                 AccumulatorPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_>
  : HostCollectiveMainloopSparse<Gemm, ElementA_, ElementB_>
{
  using HostCollectiveMainloopSparse<Gemm, ElementA_, ElementB_>::HostCollectiveMainloopSparse;
};
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 953-970

```cpp
//
// Sparse Gemm Input Operands : A , B, E
//
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedCooperativeSparseSm120<SchedulerPipelineStageCount_, false /*isAsymmetric*/>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedSm100<0/*SchedulerPipelineStageCount_*/,
                                                                                 0/*AccumulatorPipelineStageCount_*/>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedSm100<0,0>,
                                      Gemm, ElementA_, ElementB_ >;
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 971-980

```cpp
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = Base::kDefaultSeed,
    typename Base::LayoutTagA::Stride stride_factor_A_ = typename Base::LayoutTagA::Stride(),
    typename Base::LayoutTagB::Stride stride_factor_B_ = typename Base::LayoutTagB::Stride(),
    typename Base::LayoutTagE::Stride stride_factor_E_ = typename Base::LayoutTagE::Stride()
  ) : Base::HostCollectiveMainloop(check_relative_equality_, init_A_, init_B_, seed_, stride_factor_A_,
                                                                                      stride_factor_B_,
                                                                                      stride_factor_E_) {}
};
```
- **EN:** Implements or wires together logic around `Distribution`, `Kind`, `init_A_`, `Uniform`, `init_B_` for the current test scenario.
- **CN:** 围绕 `Distribution`, `Kind`, `init_A_`, `Uniform`, `init_B_` 实现或连接当前测试场景所需的逻辑。

### Lines 982-999

```cpp
//
// Sparse Gemm Input Operands : A , B, E
//
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedCooperativeSparseSm120<SchedulerPipelineStageCount_, true /*isAsymmetric*/>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedSm100<0/*SchedulerPipelineStageCount_*/,
                                                                                 0/*AccumulatorPipelineStageCount_*/>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedSm100<0,0>,
                                      Gemm, ElementA_, ElementB_ >;
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1000-1009

```cpp
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = Base::kDefaultSeed,
    typename Base::LayoutTagA::Stride stride_factor_A_ = typename Base::LayoutTagA::Stride(),
    typename Base::LayoutTagB::Stride stride_factor_B_ = typename Base::LayoutTagB::Stride(),
    typename Base::LayoutTagE::Stride stride_factor_E_ = typename Base::LayoutTagE::Stride()
  ) : Base::HostCollectiveMainloop(check_relative_equality_, init_A_, init_B_, seed_, stride_factor_A_,
                                                                                      stride_factor_B_,
                                                                                      stride_factor_E_) {}
};
```
- **EN:** Implements or wires together logic around `Distribution`, `Kind`, `init_A_`, `Uniform`, `init_B_` for the current test scenario.
- **CN:** 围绕 `Distribution`, `Kind`, `init_A_`, `Uniform`, `init_B_` 实现或连接当前测试场景所需的逻辑。

### Lines 1011-1028

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
struct HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_,
                                                                                      AccumulatorPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> {
  // Kernel data types
  using ElementA = ElementA_;
  using StrideA  = typename Gemm::GemmKernel::StrideA;
  using ElementB = ElementB_;
  using StrideB  = typename Gemm::GemmKernel::StrideB;
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1029-1031

```cpp
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  using LayoutTagA = cutlass::detail::StrideToLayoutTagA_t<StrideA>;
  using LayoutTagB = cutlass::detail::StrideToLayoutTagB_t<StrideB>;
```
- **EN:** Defines aliases such as `ScheduleType`, `LayoutTagA`, `LayoutTagB` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ScheduleType`, `LayoutTagA`, `LayoutTagB`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1033-1036

```cpp
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using EpilogueOutputOp = typename Gemm::EpilogueOutputOp;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1038-1038

```cpp
  static constexpr int SFVecSize = Gemm::GemmKernel::CollectiveMainloop::SFVecSize;
```
- **EN:** Declares member fields or local variables related to `SFVecSize`, `Gemm`, `GemmKernel`, `CollectiveMainloop` for later setup, execution, or verification.
- **CN:** 声明与 `SFVecSize`, `Gemm`, `GemmKernel`, `CollectiveMainloop` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1040-1046

```cpp
  using ElementSF = typename Gemm::GemmKernel::CollectiveMainloop::ElementSF;
  using Sm1xxBlkScaledConfig =  typename Gemm::GemmKernel::CollectiveMainloop::Sm1xxBlkScaledConfig;
  using Blk_MN   = typename Sm1xxBlkScaledConfig::Blk_MN;
  using Blk_SF   = typename Sm1xxBlkScaledConfig::Blk_SF;
  using SfAtom   = typename Sm1xxBlkScaledConfig::SfAtom;
  using LayoutSFA = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFA;
  using LayoutSFB = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFB;
```
- **EN:** Defines aliases such as `ElementSF`, `Sm1xxBlkScaledConfig`, `Blk_MN`, `Blk_SF`, `SfAtom`, `LayoutSFA`, ... (+1) to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementSF`, `Sm1xxBlkScaledConfig`, `Blk_MN`, `Blk_SF`, `SfAtom`, `LayoutSFA`, ... (+1)，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1048-1048

```cpp
  using Arguments = typename Gemm::GemmKernel::MainloopArguments;
```
- **EN:** Defines aliases such as `Arguments` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Arguments`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1050-1051

```cpp
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
```
- **EN:** Implements or wires together logic around `Whether`, `use`, `relative`, `equality`, `checks` for the current test scenario.
- **CN:** 围绕 `Whether`, `use`, `relative`, `equality`, `checks` 实现或连接当前测试场景所需的逻辑。

### Lines 1053-1054

```cpp
  StrideA stride_a;
  StrideB stride_b;
```
- **EN:** Declares member fields or local variables related to `StrideA`, `stride_a`, `StrideB`, `stride_b` for later setup, execution, or verification.
- **CN:** 声明与 `StrideA`, `stride_a`, `StrideB`, `stride_b` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1056-1057

```cpp
  LayoutSFA layout_sfa;
  LayoutSFB layout_sfb;
```
- **EN:** Declares member fields or local variables related to `LayoutSFA`, `layout_sfa`, `LayoutSFB`, `layout_sfb` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutSFA`, `layout_sfa`, `LayoutSFB`, `layout_sfb` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1059-1060

```cpp
  typename LayoutTagA::Stride stride_factor_A;
  typename LayoutTagB::Stride stride_factor_B;
```
- **EN:** Declares member fields or local variables related to `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1062-1063

```cpp
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
```
- **EN:** Declares member fields or local variables related to `Distribution`, `Kind`, `init_A`, `init_B` for later setup, execution, or verification.
- **CN:** 声明与 `Distribution`, `Kind`, `init_A`, `init_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1065-1068

```cpp
  cutlass::HostTensor<ElementA, LayoutTagA> tensor_A;
  cutlass::HostTensor<ElementB, LayoutTagB> tensor_B;
  cutlass::HostTensor<ElementSF, LayoutTagA> tensor_SFA;
  cutlass::HostTensor<ElementSF, LayoutTagB> tensor_SFB;
```
- **EN:** Declares member fields or local variables related to `HostTensor`, `ElementA`, `LayoutTagA`, `tensor_A`, `ElementB` for later setup, execution, or verification.
- **CN:** 声明与 `HostTensor`, `ElementA`, `LayoutTagA`, `tensor_A`, `ElementB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1070-1071

```cpp
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
```
- **EN:** Declares member fields or local variables related to `uint64_t`, `seed`, `kDefaultSeed` for later setup, execution, or verification.
- **CN:** 声明与 `uint64_t`, `seed`, `kDefaultSeed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1073-1077

```cpp
  // Note: this limitation comes from testbed / not the library
  static_assert(is_row_or_col_major<StrideA>(),
    "ERROR : A Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<StrideB>(),
    "ERROR : B Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1079-1090

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

### Lines 1092-1104

```cpp
  template<class ProblemShapeType>
  bool initialize(ProblemShapeType problem_size) {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("HostCollectiveMainloop (KernelTmaWarpSpecializedBlockScaledSm100)::initialize");
#endif
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 1106-1107

```cpp
    stride_a = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, K, L));
    stride_b = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(N, K, L));
```
- **EN:** Declares member fields or local variables related to `stride_a`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `stride_b` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `stride_b` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1109-1113

```cpp
    // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
    auto a_coord = cutlass::make_Coord(M * L, K);
    // Cutlass has Row/Col major refers to MxK times KxN matrix product,
    // so the HostTensorB should be treated as KxN in "coord"'s view
    auto b_coord = cutlass::make_Coord(K, N * L);
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 1115-1116

```cpp
    tensor_A.resize(a_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_coord, stride_factor_A));
    tensor_B.resize(b_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(b_coord, stride_factor_B));
```
- **EN:** Declares member fields or local variables related to `tensor_A`, `resize`, `a_coord`, `layout`, `Affine2Layout_Factory` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_A`, `resize`, `a_coord`, `layout`, `Affine2Layout_Factory` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1118-1119

```cpp
    EXPECT_TRUE(initialize_tensor(tensor_A.host_view(), init_A, seed + 2022));
    EXPECT_TRUE(initialize_tensor(tensor_B.host_view(), init_B, seed + 2021));
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `initialize_tensor`, `tensor_A`, `host_view`, `init_A` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `initialize_tensor`, `tensor_A`, `host_view`, `init_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1121-1124

```cpp
    // It is possible to randomly initialize to all zeros, so override this with non-zeros
    // in the upper left corner of each operand.
    tensor_A.host_view().at({0, 0}) = ElementA(1);
    tensor_B.host_view().at({0, 0}) = ElementB(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 1126-1127

```cpp
    tensor_A.sync_device();
    tensor_B.sync_device();
```
- **EN:** Declares member fields or local variables related to `tensor_A`, `sync_device`, `tensor_B` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_A`, `sync_device`, `tensor_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1129-1134

```cpp
    using namespace cute;
    auto k_blks = cutlass::ceil_div(K, size<1>(shape(SfAtom{})));
    auto m_blks = cutlass::ceil_div(M, Blk_MN{});
    auto n_blks = cutlass::ceil_div(N, Blk_MN{});
    layout_sfa = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFA(problem_shape_MNKL);
    layout_sfb = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFB(problem_shape_MNKL);
```
- **EN:** Declares member fields or local variables related to `k_blks`, `ceil_div`, `size`, `shape`, `SfAtom` for later setup, execution, or verification.
- **CN:** 声明与 `k_blks`, `ceil_div`, `size`, `shape`, `SfAtom` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1136-1138

```cpp
    // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
    auto sfa_coord   = cutlass::make_Coord(m_blks * Blk_MN{} * L, k_blks * Blk_SF{});
    auto sfb_coord   = cutlass::make_Coord(n_blks * Blk_MN{} * L, k_blks * Blk_SF{});
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 1140-1141

```cpp
    tensor_SFA.resize(sfa_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(sfa_coord, stride_factor_A));
    tensor_SFB.resize(sfb_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(sfb_coord, stride_factor_B));
```
- **EN:** Declares member fields or local variables related to `tensor_SFA`, `resize`, `sfa_coord`, `layout`, `Affine2Layout_Factory` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_SFA`, `resize`, `sfa_coord`, `layout`, `Affine2Layout_Factory` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1143-1144

```cpp
    EXPECT_TRUE(initialize_tensor(tensor_SFA.host_view(), init_A, seed + 2024));
    EXPECT_TRUE(initialize_tensor(tensor_SFB.host_view(), init_B, seed + 2025));
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `initialize_tensor`, `tensor_SFA`, `host_view`, `init_A` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `initialize_tensor`, `tensor_SFA`, `host_view`, `init_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1146-1149

```cpp
    // It is possible to randomly initialize to all zeros, so override this with non-zeros
    // in the upper left corner of each operand.
    tensor_SFA.host_view().at({0, 0}) = ElementSF(1);
    tensor_SFB.host_view().at({0, 0}) = ElementSF(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 1151-1152

```cpp
    tensor_SFA.sync_device();
    tensor_SFB.sync_device();
```
- **EN:** Declares member fields or local variables related to `tensor_SFA`, `sync_device`, `tensor_SFB` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_SFA`, `sync_device`, `tensor_SFB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1154-1155

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1157-1166

```cpp
  Arguments to_args() {
    using ArrayElementA = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementA;
    using ArrayElementB = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementB;
    return {
      reinterpret_cast<ArrayElementA *>(tensor_A.device_data()), stride_a,
      reinterpret_cast<ArrayElementB *>(tensor_B.device_data()), stride_b,
      tensor_SFA.device_data(), layout_sfa,
      tensor_SFB.device_data(), layout_sfb
    };
  }
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1168-1180

```cpp
  auto to_host_args(ProblemShapeType problem_size) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
    auto A = make_tensor(make_iterator(tensor_A.host_data()),
          make_layout(make_shape(M, K, L), stride_a));
    auto SfA = make_tensor(tensor_SFA.host_data(), layout_sfa);
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1182-1184

```cpp
    auto B = make_tensor(make_iterator(tensor_B.host_data()),
        make_layout(make_shape(N, K, L), stride_b));
    auto SfB = make_tensor(tensor_SFB.host_data(), layout_sfb);
```
- **EN:** Implements or wires together logic around `make_tensor`, `make_iterator`, `tensor_B`, `host_data`, `make_layout` for the current test scenario.
- **CN:** 围绕 `make_tensor`, `make_iterator`, `tensor_B`, `host_data`, `make_layout` 实现或连接当前测试场景所需的逻辑。

### Lines 1186-1194

```cpp
    cutlass::reference::host::GettMainloopParams<ElementAccumulator,
        decltype(A),
        decltype(B),
        decltype(SfA),
        decltype(SfB)
      >
      mainloop_params{A, SfA, B, SfB};
    return mainloop_params;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1196-1201

```cpp
  void print_tensors(std::ofstream& file) {
    file << "A =\n" << tensor_A.host_view()
         << "\nB =\n" << tensor_B.host_view()
         << "\nSFA =\n" << tensor_SFA.host_view()
         << "\nSFB =\n" << tensor_SFB.host_view();
  }
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1203-1205

```cpp
  bool compare_reference(
      cute::Shape<int,int,int,int> problem_shape_MNKL) {
    auto [M, N, K, L] = problem_shape_MNKL;
```
- **EN:** Implements or wires together logic around `compare_reference`, `problem_shape_MNKL` for the current test scenario.
- **CN:** 围绕 `compare_reference`, `problem_shape_MNKL` 实现或连接当前测试场景所需的逻辑。

### Lines 1207-1213

```cpp
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_A.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_B.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_SFA.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_SFB.host_view()), 0);
    return true;
  }
};
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 1216-1233

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
struct HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedPingpongBlockScaledSm120<SchedulerPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedBlockScaledSm100<0,0>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedBlockScaledSm100<0,0>,
                                      Gemm, ElementA_, ElementB_>;
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1234-1239

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

### Lines 1241-1258

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
struct HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedCooperativeBlockScaledSm120<SchedulerPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedBlockScaledSm100<0,0>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedBlockScaledSm100<0,0>,
                                      Gemm, ElementA_, ElementB_>;
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1259-1264

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

### Lines 1266-1283

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
struct HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedBlockScaledSm103<SchedulerPipelineStageCount_,
                                                                                      AccumulatorPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_,AccumulatorPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_,AccumulatorPipelineStageCount_>,
                                      Gemm, ElementA_, ElementB_>;
  HostCollectiveMainloop(
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1284-1291

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

### Lines 1293-1310

```cpp
//
// Block Scaled Structured Sparse Gemm Input Operands : A_compressed, B, metadata, scalefactorA, scalefactorB
//
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  int AccumulatorPipelineStageCount_,
  typename ElementA_,
  typename ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedBlockScaledSm100<SchedulerPipelineStageCount_,
                                                                                            AccumulatorPipelineStageCount_>,
                              Gemm, ElementA_, ElementB_> {
  // Kernel data types
  using ElementA = ElementA_;
  // CuTe layout A for the kernel's sparse tensorA.
  using LayoutA  = typename Gemm::GemmKernel::CollectiveMainloop::LayoutA;
  using ElementB = ElementB_;
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1311-1312

```cpp
  using StrideB  = typename Gemm::GemmKernel::StrideB;
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
```
- **EN:** Defines aliases such as `StrideB`, `ScheduleType` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `StrideB`, `ScheduleType`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1314-1321

```cpp
  using ElementE = typename Gemm::GemmKernel::CollectiveMainloop::ElementE;
  // CuTe layout E for the kernel's metadata tensor.
  using LayoutE  = typename Gemm::GemmKernel::CollectiveMainloop::LayoutE;
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using EpilogueOutputOp = typename Gemm::EpilogueOutputOp;
  using SparseConfig = typename Gemm::GemmKernel::CollectiveMainloop::SparseConfig;
```
- **EN:** Introduces type aliases like `ElementE`, `LayoutE`, `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp`, ... (+1) so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ElementE`, `LayoutE`, `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `EpilogueOutputOp`, ... (+1)，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 1323-1327

```cpp
  // The following typenames are for the reference host tensors. They are non-sparse tensors.
  using LayoutTagA = decltype(SparseConfig::deduce_layoutA_tag(LayoutA{}));
  using StrideA = cutlass::gemm::TagToStrideA_t<LayoutTagA>;
  // We don't care about the actual strideE for the host tensor, but just need one to allocate memory.
  using StrideE = StrideA;
```
- **EN:** Introduces type aliases like `LayoutTagA`, `StrideA`, `StrideE` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `LayoutTagA`, `StrideA`, `StrideE`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 1329-1331

```cpp
  static constexpr int SFVecSize = Gemm::GemmKernel::CollectiveMainloop::SFVecSize;
  // Deduce Cutlass Layouts (RowMajor & ColumnMajor)
  using LayoutTagB = cutlass::detail::StrideToLayoutTagB_t<StrideB>;
```
- **EN:** Implements or wires together logic around `SFVecSize`, `Gemm`, `GemmKernel`, `CollectiveMainloop`, `Deduce` for the current test scenario.
- **CN:** 围绕 `SFVecSize`, `Gemm`, `GemmKernel`, `CollectiveMainloop`, `Deduce` 实现或连接当前测试场景所需的逻辑。

### Lines 1333-1333

```cpp
  using LayoutTagE = cutlass::detail::StrideToLayoutTagA_t<StrideE>;
```
- **EN:** Defines aliases such as `LayoutTagE` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `LayoutTagE`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1335-1341

```cpp
  using ElementSF = typename Gemm::GemmKernel::CollectiveMainloop::ElementSF;
  using Sm1xxBlkScaledConfig =  typename Gemm::GemmKernel::CollectiveMainloop::Sm1xxBlkScaledConfig;
  using Blk_MN   = typename Sm1xxBlkScaledConfig::Blk_MN;
  using Blk_SF   = typename Sm1xxBlkScaledConfig::Blk_SF;
  using SfAtom   = typename Sm1xxBlkScaledConfig::SfAtom;
  using LayoutSFA = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFA;
  using LayoutSFB = typename Gemm::GemmKernel::CollectiveMainloop::LayoutSFB;
```
- **EN:** Defines aliases such as `ElementSF`, `Sm1xxBlkScaledConfig`, `Blk_MN`, `Blk_SF`, `SfAtom`, `LayoutSFA`, ... (+1) to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementSF`, `Sm1xxBlkScaledConfig`, `Blk_MN`, `Blk_SF`, `SfAtom`, `LayoutSFA`, ... (+1)，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1343-1353

```cpp
  using CompressorUtility = cutlass::transform::kernel::StructuredSparseCompressorUtility<
                              cute::Shape<int, int, int, int>,
                              ElementA,
                              LayoutTagA,
                              SparseConfig>;
  using CompressorKernel = cutlass::transform::kernel::StructuredSparseCompressor<
                        cute::Shape<int, int, int, int>,
                        ElementA,
                        LayoutTagA,
                        SparseConfig,
                        cutlass::arch::Sm100>;
```
- **EN:** Introduces type aliases like `CompressorUtility`, `CompressorKernel` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `CompressorUtility`, `CompressorKernel`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 1355-1355

```cpp
  using Compressor = cutlass::transform::device::TransformUniversalAdapter<CompressorKernel>;
```
- **EN:** Defines aliases such as `Compressor` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Compressor`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1357-1359

```cpp
  using Arguments = typename Gemm::GemmKernel::MainloopArguments;
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
```
- **EN:** Implements or wires together logic around `Arguments`, `Gemm`, `GemmKernel`, `MainloopArguments`, `Whether` for the current test scenario.
- **CN:** 围绕 `Arguments`, `Gemm`, `GemmKernel`, `MainloopArguments`, `Whether` 实现或连接当前测试场景所需的逻辑。

### Lines 1361-1364

```cpp
  StrideA stride_a;
  StrideA stride_a_compressed;
  StrideB stride_b;
  StrideE stride_e;
```
- **EN:** Declares member fields or local variables related to `StrideA`, `stride_a`, `stride_a_compressed`, `StrideB`, `stride_b` for later setup, execution, or verification.
- **CN:** 声明与 `StrideA`, `stride_a`, `stride_a_compressed`, `StrideB`, `stride_b` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1366-1369

```cpp
  LayoutA layout_a;
  LayoutE layout_e;
  LayoutSFA layout_sfa;
  LayoutSFB layout_sfb;
```
- **EN:** Declares member fields or local variables related to `LayoutA`, `layout_a`, `LayoutE`, `layout_e`, `LayoutSFA` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutA`, `layout_a`, `LayoutE`, `layout_e`, `LayoutSFA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1371-1373

```cpp
  typename LayoutTagA::Stride stride_factor_A;
  typename LayoutTagB::Stride stride_factor_B;
  typename LayoutTagE::Stride stride_factor_E;
```
- **EN:** Declares member fields or local variables related to `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagA`, `Stride`, `stride_factor_A`, `LayoutTagB`, `stride_factor_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1375-1376

```cpp
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
```
- **EN:** Declares member fields or local variables related to `Distribution`, `Kind`, `init_A`, `init_B` for later setup, execution, or verification.
- **CN:** 声明与 `Distribution`, `Kind`, `init_A`, `init_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1378-1383

```cpp
  cutlass::HostTensor<ElementA, LayoutTagA> tensor_A;
  cutlass::HostTensor<ElementA, LayoutTagA> tensor_A_Comp;
  cutlass::HostTensor<ElementB, LayoutTagB> tensor_B;
  cutlass::HostTensor<ElementE, LayoutTagE> tensor_E;
  cutlass::HostTensor<ElementSF, LayoutTagA> tensor_SFA;
  cutlass::HostTensor<ElementSF, LayoutTagB> tensor_SFB;
```
- **EN:** Declares member fields or local variables related to `HostTensor`, `ElementA`, `LayoutTagA`, `tensor_A`, `tensor_A_Comp` for later setup, execution, or verification.
- **CN:** 声明与 `HostTensor`, `ElementA`, `LayoutTagA`, `tensor_A`, `tensor_A_Comp` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1385-1386

```cpp
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
```
- **EN:** Declares member fields or local variables related to `uint64_t`, `seed`, `kDefaultSeed` for later setup, execution, or verification.
- **CN:** 声明与 `uint64_t`, `seed`, `kDefaultSeed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1388-1392

```cpp
  // Note: this limitation comes from testbed / not the library
  static_assert(is_row_or_col_major<StrideA>(),
    "ERROR : A Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<StrideB>(),
    "ERROR : B Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1394-1407

```cpp
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = kDefaultSeed,
    typename LayoutTagA::Stride stride_factor_A_ = typename LayoutTagA::Stride(),
    typename LayoutTagB::Stride stride_factor_B_ = typename LayoutTagB::Stride(),
    typename LayoutTagE::Stride stride_factor_E_ = typename LayoutTagE::Stride()
  ):
    check_relative_equality(check_relative_equality_),
    stride_factor_A(stride_factor_A_),
    stride_factor_B(stride_factor_B_),
    stride_factor_E(stride_factor_E_),
    init_A(init_A_), init_B(init_B_), seed(seed_) { }
```
- **EN:** Implements or wires together logic around `HostCollectiveMainloop`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution` for the current test scenario.
- **CN:** 围绕 `HostCollectiveMainloop`, `CheckEquality`, `check_relative_equality_`, `EXACT`, `Distribution` 实现或连接当前测试场景所需的逻辑。

### Lines 1409-1421

```cpp
  template<class ProblemShapeType>
  bool initialize(ProblemShapeType problem_size) {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("HostCollectiveMainloop (KernelSparseTmaWarpSpecializedBlockScaledSm100)::initialize");
#endif
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 1423-1424

```cpp
    stride_a = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, K, L));
    stride_b = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(N, K, L));
```
- **EN:** Declares member fields or local variables related to `stride_a`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `stride_b` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `stride_b` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1426-1426

```cpp
    CompressorUtility compressor_utility(problem_shape_MNKL, stride_a);
```
- **EN:** Declares member fields or local variables related to `CompressorUtility`, `compressor_utility`, `problem_shape_MNKL`, `stride_a` for later setup, execution, or verification.
- **CN:** 声明与 `CompressorUtility`, `compressor_utility`, `problem_shape_MNKL`, `stride_a` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1428-1433

```cpp
    // TensorE
    // In unit of ElementE (uint8_t), after alignment requirement
    // M-dim: TensorEAtom_M alignment
    // K-dim: TensorEAtom_K alignment
    int KAlignedE = compressor_utility.get_metadata_k_physical();
    int MAlignedE = compressor_utility.get_metadata_m_physical();
```
- **EN:** Implements or wires together logic around `TensorE`, `unit`, `ElementE`, `uint8_t`, `after` for the current test scenario.
- **CN:** 围绕 `TensorE`, `unit`, `ElementE`, `uint8_t`, `after` 实现或连接当前测试场景所需的逻辑。

### Lines 1435-1440

```cpp
    // TensorA Compressed
    // In unit of ElementARaw, after alignment requirement
    // M-dim: TMA alignment
    // K-dim: TMA alignment
    int KAlignedAC = compressor_utility.get_tensorA_k_physical();
    int MAlignedAC = compressor_utility.get_tensorA_m_physical();
```
- **EN:** Implements or wires together logic around `TensorA`, `Compressed`, `unit`, `ElementARaw`, `after` for the current test scenario.
- **CN:** 围绕 `TensorA`, `Compressed`, `unit`, `ElementARaw`, `after` 实现或连接当前测试场景所需的逻辑。

### Lines 1442-1443

```cpp
    stride_a_compressed = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, KAlignedAC, L));
    stride_e = cutlass::make_cute_packed_stride(StrideE{}, cute::make_shape(MAlignedE, KAlignedE, L));
```
- **EN:** Declares member fields or local variables related to `stride_a_compressed`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `KAlignedAC` for later setup, execution, or verification.
- **CN:** 声明与 `stride_a_compressed`, `make_cute_packed_stride`, `StrideA`, `make_shape`, `KAlignedAC` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1445-1448

```cpp
    auto a_coord = cutlass::make_Coord(M * L, K);
    auto b_coord = cutlass::make_Coord(K, N * L);
    auto e_coord = cutlass::make_Coord(MAlignedE * L, KAlignedE);
    auto a_comp_coord = cutlass::make_Coord(MAlignedAC * L, KAlignedAC);
```
- **EN:** Declares member fields or local variables related to `a_coord`, `make_Coord`, `b_coord`, `e_coord`, `MAlignedE` for later setup, execution, or verification.
- **CN:** 声明与 `a_coord`, `make_Coord`, `b_coord`, `e_coord`, `MAlignedE` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1450-1453

```cpp
    tensor_A.resize(a_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_coord, stride_factor_A));
    tensor_A_Comp.resize(a_comp_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(a_comp_coord, stride_factor_A));
    tensor_B.resize(b_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(b_coord, stride_factor_B));
    tensor_E.resize(e_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagE>::layout_factory(e_coord, stride_factor_E));
```
- **EN:** Declares member fields or local variables related to `tensor_A`, `resize`, `a_coord`, `layout`, `Affine2Layout_Factory` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_A`, `resize`, `a_coord`, `layout`, `Affine2Layout_Factory` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1455-1456

```cpp
    EXPECT_TRUE(initialize_tensor(tensor_A.host_view(), init_A, seed + 2022));
    EXPECT_TRUE(initialize_tensor(tensor_B.host_view(), init_B, seed + 2021));
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `initialize_tensor`, `tensor_A`, `host_view`, `init_A` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `initialize_tensor`, `tensor_A`, `host_view`, `init_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1458-1461

```cpp
    // It is possible to randomly initialize to all zeros, so override this with non-zeros
    // in the upper left corner of each operand.
    tensor_A.host_view().at({0, 0}) = ElementA(1);
    tensor_B.host_view().at({0, 0}) = ElementB(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 1463-1463

```cpp
    compressor_utility.structure_sparse_zero_mask_fill(tensor_A.host_data(), static_cast<int>(seed + 2023));
```
- **EN:** Declares member fields or local variables related to `compressor_utility`, `structure_sparse_zero_mask_fill`, `tensor_A`, `host_data`, `static_cast` for later setup, execution, or verification.
- **CN:** 声明与 `compressor_utility`, `structure_sparse_zero_mask_fill`, `tensor_A`, `host_data`, `static_cast` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1465-1468

```cpp
    tensor_A.sync_device();
    tensor_B.sync_device();
    tensor_E.sync_device();
    tensor_A_Comp.sync_device();
```
- **EN:** Declares member fields or local variables related to `tensor_A`, `sync_device`, `tensor_B`, `tensor_E`, `tensor_A_Comp` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_A`, `sync_device`, `tensor_B`, `tensor_E`, `tensor_A_Comp` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1470-1470

```cpp
    cutlass::Status status {cutlass::Status::kSuccess };
```
- **EN:** Declares member fields or local variables related to `Status`, `status`, `kSuccess` for later setup, execution, or verification.
- **CN:** 声明与 `Status`, `status`, `kSuccess` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1472-1482

```cpp
    cutlass::KernelHardwareInfo hw_info;
    hw_info.device_id = 0;
    hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
    typename Compressor::Arguments arguments{
      {M, N, K, L},
      {tensor_A.device_data(),
       stride_a,
       tensor_A_Comp.device_data(),
       tensor_E.device_data()},
      {hw_info}
    };
```
- **EN:** Implements or wires together logic around `KernelHardwareInfo`, `hw_info`, `device_id`, `sm_count`, `query_device_multiprocessor_count` for the current test scenario.
- **CN:** 围绕 `KernelHardwareInfo`, `hw_info`, `device_id`, `sm_count`, `query_device_multiprocessor_count` 实现或连接当前测试场景所需的逻辑。

### Lines 1484-1486

```cpp
    Compressor compressor_op;
    size_t workspace_size = Compressor::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```
- **EN:** Declares member fields or local variables related to `Compressor`, `compressor_op`, `size_t`, `workspace_size`, `get_workspace_size` for later setup, execution, or verification.
- **CN:** 声明与 `Compressor`, `compressor_op`, `size_t`, `workspace_size`, `get_workspace_size` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1488-1491

```cpp
    status = compressor_op.can_implement(arguments);
    if (status != cutlass::Status::kSuccess) {
      return false;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1493-1496

```cpp
    status = compressor_op.initialize(arguments, workspace.get());
    if (status != cutlass::Status::kSuccess) {
      return false;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1498-1498

```cpp
    status = compressor_op.run();
```
- **EN:** Declares member fields or local variables related to `status`, `compressor_op`, `run` for later setup, execution, or verification.
- **CN:** 声明与 `status`, `compressor_op`, `run` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1500-1513

```cpp
#if (CUTLASS_ENABLE_SYCL)
    try {
      compat::wait_and_throw();
    } catch (std::exception const &e) {
      ADD_FAILURE() << "Error at Kernel Sync.";
      return false;
    }
#else
    auto result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      EXPECT_EQ(result, cudaSuccess) << "Error at Kernel Sync.";
      return false;
    }
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 1515-1516

```cpp
    layout_a = SparseConfig::fill_layoutA(problem_shape_MNKL);
    layout_e = SparseConfig::fill_layoutE(problem_shape_MNKL);
```
- **EN:** Declares member fields or local variables related to `layout_a`, `SparseConfig`, `fill_layoutA`, `problem_shape_MNKL`, `layout_e` for later setup, execution, or verification.
- **CN:** 声明与 `layout_a`, `SparseConfig`, `fill_layoutA`, `problem_shape_MNKL`, `layout_e` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1518-1519

```cpp
    tensor_E.sync_host();
    tensor_A_Comp.sync_host();
```
- **EN:** Declares member fields or local variables related to `tensor_E`, `sync_host`, `tensor_A_Comp` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_E`, `sync_host`, `tensor_A_Comp` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1521-1526

```cpp
    using namespace cute;
    auto k_blks = cutlass::ceil_div(K, size<1>(shape(SfAtom{})));
    auto m_blks = cutlass::ceil_div(M, Blk_MN{});
    auto n_blks = cutlass::ceil_div(N, Blk_MN{});
    layout_sfa = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFA(problem_shape_MNKL);
    layout_sfb = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFB(problem_shape_MNKL);
```
- **EN:** Declares member fields or local variables related to `k_blks`, `ceil_div`, `size`, `shape`, `SfAtom` for later setup, execution, or verification.
- **CN:** 声明与 `k_blks`, `ceil_div`, `size`, `shape`, `SfAtom` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1528-1530

```cpp
    // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
    auto sfa_coord   = cutlass::make_Coord(m_blks * Blk_MN{} * L, k_blks * Blk_SF{});
    auto sfb_coord   = cutlass::make_Coord(n_blks * Blk_MN{} * L, k_blks * Blk_SF{});
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 1532-1533

```cpp
    tensor_SFA.resize(sfa_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagA>::layout_factory(sfa_coord, stride_factor_A));
    tensor_SFB.resize(sfb_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagB>::layout_factory(sfb_coord, stride_factor_B));
```
- **EN:** Declares member fields or local variables related to `tensor_SFA`, `resize`, `sfa_coord`, `layout`, `Affine2Layout_Factory` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_SFA`, `resize`, `sfa_coord`, `layout`, `Affine2Layout_Factory` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1535-1536

```cpp
    EXPECT_TRUE(initialize_tensor(tensor_SFA.host_view(), init_A, seed + 2024));
    EXPECT_TRUE(initialize_tensor(tensor_SFB.host_view(), init_B, seed + 2025));
```
- **EN:** Declares member fields or local variables related to `EXPECT_TRUE`, `initialize_tensor`, `tensor_SFA`, `host_view`, `init_A` for later setup, execution, or verification.
- **CN:** 声明与 `EXPECT_TRUE`, `initialize_tensor`, `tensor_SFA`, `host_view`, `init_A` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1538-1541

```cpp
    // It is possible to randomly initialize to all zeros, so override this with non-zeros
    // in the upper left corner of each operand.
    tensor_SFA.host_view().at({0, 0}) = ElementSF(1);
    tensor_SFB.host_view().at({0, 0}) = ElementSF(1);
```
- **EN:** Implements or wires together logic around `possible`, `randomly`, `initialize`, `all`, `zeros` for the current test scenario.
- **CN:** 围绕 `possible`, `randomly`, `initialize`, `all`, `zeros` 实现或连接当前测试场景所需的逻辑。

### Lines 1543-1544

```cpp
    tensor_SFA.sync_device();
    tensor_SFB.sync_device();
```
- **EN:** Declares member fields or local variables related to `tensor_SFA`, `sync_device`, `tensor_SFB` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_SFA`, `sync_device`, `tensor_SFB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1546-1547

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1549-1559

```cpp
  Arguments to_args() {
    using ArrayElementA = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementA;
    using ArrayElementB = typename Gemm::GemmKernel::CollectiveMainloop::ArrayElementB;
    return {
      reinterpret_cast<ArrayElementA *>(tensor_A_Comp.device_data()), layout_a,
      reinterpret_cast<ArrayElementB *>(tensor_B.device_data()), stride_b,
      tensor_E.device_data(), layout_e,
      tensor_SFA.device_data(), layout_sfa,
      tensor_SFB.device_data(), layout_sfb
    };
  }
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1561-1573

```cpp
  auto to_host_args(ProblemShapeType problem_size) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
    auto A = make_tensor(make_iterator(tensor_A.host_data()),
          make_layout(make_shape(M, K, L), stride_a));
    auto SfA = make_tensor(tensor_SFA.host_data(), layout_sfa);
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1575-1577

```cpp
    auto B = make_tensor(make_iterator(tensor_B.host_data()),
        make_layout(make_shape(N, K, L), stride_b));
    auto SfB = make_tensor(tensor_SFB.host_data(), layout_sfb);
```
- **EN:** Implements or wires together logic around `make_tensor`, `make_iterator`, `tensor_B`, `host_data`, `make_layout` for the current test scenario.
- **CN:** 围绕 `make_tensor`, `make_iterator`, `tensor_B`, `host_data`, `make_layout` 实现或连接当前测试场景所需的逻辑。

### Lines 1579-1588

```cpp
    // return {A, SfA, B, SfB};
    cutlass::reference::host::GettMainloopParams<ElementAccumulator,
        decltype(A),
        decltype(B),
        decltype(SfA),
        decltype(SfB)
      >
          mainloop_params{A, SfA, B, SfB};
    return mainloop_params;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1590-1595

```cpp
  void print_tensors(std::ofstream& file) {
    file << "A =\n" << tensor_A.host_view()
         << "\nB =\n" << tensor_B.host_view()
         << "\nSFA =\n" << tensor_SFA.host_view()
         << "\nSFB =\n" << tensor_SFB.host_view();
  }
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1597-1599

```cpp
  bool compare_reference(
      cute::Shape<int,int,int,int> problem_shape_MNKL) {
    auto [M, N, K, L] = problem_shape_MNKL;
```
- **EN:** Implements or wires together logic around `compare_reference`, `problem_shape_MNKL` for the current test scenario.
- **CN:** 围绕 `compare_reference`, `problem_shape_MNKL` 实现或连接当前测试场景所需的逻辑。

### Lines 1601-1607

```cpp
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_A.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_B.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_SFA.host_view()), 0);
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_SFB.host_view()), 0);
    return true;
  }
};
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 1609-1626

```cpp
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedCooperativeSparseBlockScaledSm120<SchedulerPipelineStageCount_, true>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedBlockScaledSm100<0,0>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedBlockScaledSm100<0,0>,
                                      Gemm, ElementA_, ElementB_>;
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = Base::kDefaultSeed,
    typename Base::LayoutTagA::Stride stride_factor_A_ = typename Base::LayoutTagA::Stride(),
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1627-1632

```cpp
    typename Base::LayoutTagB::Stride stride_factor_B_ = typename Base::LayoutTagB::Stride(),
    typename Base::LayoutTagE::Stride stride_factor_E_ = typename Base::LayoutTagE::Stride()
  ) : Base::HostCollectiveMainloop(check_relative_equality_, init_A_, init_B_, seed_, stride_factor_A_,
                                                                                      stride_factor_B_,
                                                                                      stride_factor_E_) {}
};
```
- **EN:** Implements or wires together logic around `Base`, `LayoutTagB`, `Stride`, `stride_factor_B_`, `LayoutTagE` for the current test scenario.
- **CN:** 围绕 `Base`, `LayoutTagB`, `Stride`, `stride_factor_B_`, `LayoutTagE` 实现或连接当前测试场景所需的逻辑。

### Lines 1634-1651

```cpp
template<
  class Gemm,
  int SchedulerPipelineStageCount_,
  class ElementA_,
  class ElementB_
>
struct HostCollectiveMainloop<cutlass::gemm::KernelTmaWarpSpecializedCooperativeSparseBlockScaledSm120<SchedulerPipelineStageCount_, false>,
                              Gemm, ElementA_, ElementB_> : public
       HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedBlockScaledSm100<0,0>,
                              Gemm, ElementA_, ElementB_> {
  using Base = HostCollectiveMainloop<cutlass::gemm::KernelSparseTmaWarpSpecializedBlockScaledSm100<0,0>,
                                      Gemm, ElementA_, ElementB_>;
  HostCollectiveMainloop(
    CheckEquality check_relative_equality_ = CheckEquality::EXACT,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = Base::kDefaultSeed,
    typename Base::LayoutTagA::Stride stride_factor_A_ = typename Base::LayoutTagA::Stride(),
```
- **EN:** Defines templated type `Gemm` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `Gemm`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 1652-1657

```cpp
    typename Base::LayoutTagB::Stride stride_factor_B_ = typename Base::LayoutTagB::Stride(),
    typename Base::LayoutTagE::Stride stride_factor_E_ = typename Base::LayoutTagE::Stride()
  ) : Base::HostCollectiveMainloop(check_relative_equality_, init_A_, init_B_, seed_, stride_factor_A_,
                                                                                      stride_factor_B_,
                                                                                      stride_factor_E_) {}
};
```
- **EN:** Implements or wires together logic around `Base`, `LayoutTagB`, `Stride`, `stride_factor_B_`, `LayoutTagE` for the current test scenario.
- **CN:** 围绕 `Base`, `LayoutTagB`, `Stride`, `stride_factor_B_`, `LayoutTagE` 实现或连接当前测试场景所需的逻辑。

### Lines 1659-1664

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

### Lines 1666-1668

```cpp
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  using kernel   = typename Gemm::GemmKernel;
  using Epilogue = typename kernel::CollectiveEpilogue;
```
- **EN:** Defines aliases such as `ScheduleType`, `kernel`, `Epilogue` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ScheduleType`, `kernel`, `Epilogue`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1670-1673

```cpp
  using ElementD = typename kernel::ElementD;
  using StrideD  = typename kernel::StrideD;
  using ElementC = non_void_t<typename kernel::ElementC, ElementD>;
  using StrideC  = typename kernel::StrideC;
```
- **EN:** Defines aliases such as `ElementD`, `StrideD`, `ElementC`, `StrideC` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementD`, `StrideD`, `ElementC`, `StrideC`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1675-1675

```cpp
  using FusionOp = typename Gemm::EpilogueOutputOp;
```
- **EN:** Defines aliases such as `FusionOp` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `FusionOp`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1677-1678

```cpp
  static_assert(rank(StrideC{}) == 3, "StrideCD must be rank-3: [M, N, L]");
  static_assert(rank(StrideD{}) == 3, "StrideCD must be rank-3: [M, N, L]");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1680-1683

```cpp
  static_assert(is_row_or_col_major<StrideC>(),
    "ERROR : C Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<StrideD>(),
    "ERROR : D Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1685-1689

```cpp
  // Deduce Cutlass Layouts (RowMajor & ColumnMajor)
  using LayoutTagC = cutlass::detail::StrideToLayoutTagC_t<StrideC>;
  using LayoutTagD = cutlass::detail::StrideToLayoutTagC_t<StrideD>;
  using LayoutTagScalar = cutlass::layout::PackedVectorLayout; // scalars are size-1 vectors
  using LayoutTagVector = cutlass::layout::PackedVectorLayout;
```
- **EN:** Begins function or method `Layouts`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `Layouts`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1691-1695

```cpp
  using ElementAccumulator = typename kernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename kernel::ProblemShape;
  using ElementCompute = typename ElementComputeType<Gemm, ElementAccumulator>::Type;
  using ElementScalar = typename ElementScalarType<Gemm, ElementCompute>::Type;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `ElementCompute`, `ElementScalar` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`, `ElementCompute`, `ElementScalar`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1697-1697

```cpp
  using Arguments = typename Gemm::GemmKernel::EpilogueArguments;
```
- **EN:** Defines aliases such as `Arguments` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `Arguments`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1699-1701

```cpp
  /// Initialization
  StrideC stride_c;
  StrideD stride_d;
```
- **EN:** Implements or wires together logic around `Initialization`, `StrideC`, `stride_c`, `StrideD`, `stride_d` for the current test scenario.
- **CN:** 围绕 `Initialization`, `StrideC`, `stride_c`, `StrideD`, `stride_d` 实现或连接当前测试场景所需的逻辑。

### Lines 1703-1704

```cpp
  typename LayoutTagC::Stride stride_factor_C;
  typename LayoutTagD::Stride stride_factor_D;
```
- **EN:** Declares member fields or local variables related to `LayoutTagC`, `Stride`, `stride_factor_C`, `LayoutTagD`, `stride_factor_D` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagC`, `Stride`, `stride_factor_C`, `LayoutTagD`, `stride_factor_D` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1706-1709

```cpp
  cutlass::HostTensor<ElementC, LayoutTagC> tensor_C;
  // Inputs
  ElementScalar alpha;
  ElementScalar beta;
```
- **EN:** Implements or wires together logic around `HostTensor`, `ElementC`, `LayoutTagC`, `tensor_C`, `Inputs` for the current test scenario.
- **CN:** 围绕 `HostTensor`, `ElementC`, `LayoutTagC`, `tensor_C`, `Inputs` 实现或连接当前测试场景所需的逻辑。

### Lines 1711-1712

```cpp
  cutlass::HostTensor<ElementD, LayoutTagD> tensor_D;
  cutlass::HostTensor<ElementD, LayoutTagD> reference_D;
```
- **EN:** Declares member fields or local variables related to `HostTensor`, `ElementD`, `LayoutTagD`, `tensor_D`, `reference_D` for later setup, execution, or verification.
- **CN:** 声明与 `HostTensor`, `ElementD`, `LayoutTagD`, `tensor_D`, `reference_D` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1714-1719

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

### Lines 1721-1723

```cpp
  cutlass::Distribution::Kind init_C;
  uint64_t seed;
  static constexpr uint64_t kDefaultSeed = 4096;
```
- **EN:** Declares member fields or local variables related to `Distribution`, `Kind`, `init_C`, `uint64_t`, `seed` for later setup, execution, or verification.
- **CN:** 声明与 `Distribution`, `Kind`, `init_C`, `uint64_t`, `seed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1725-1737

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

### Lines 1739-1745

```cpp
  bool initialize(ProblemShapeType problem_size, ElementScalar alpha_=1.f, ElementScalar beta_=0.f) {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("HostCollectiveDefaultEpilogue::initialize(problem_size, alpha, beta)");
#endif
    // Initialize Epilogue tensors
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto [M, N, K, L] = problem_shape_MNKL;
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 1747-1748

```cpp
    stride_c = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(M, N, L));
    stride_d = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(M, N, L));
```
- **EN:** Declares member fields or local variables related to `stride_c`, `make_cute_packed_stride`, `StrideC`, `make_shape`, `stride_d` for later setup, execution, or verification.
- **CN:** 声明与 `stride_c`, `make_cute_packed_stride`, `StrideC`, `make_shape`, `stride_d` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1750-1767

```cpp
    // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
    auto c_coord = cutlass::make_Coord(M * L, N);
    try {
      tensor_C.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagC>::layout_factory(c_coord, stride_factor_C));
      tensor_D.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, stride_factor_D));
      reference_D.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, stride_factor_D), false);
    }
    catch (std::exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveDefaultEpilogue::initialize: resizing tensors threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("HostCollectiveDefaultEpilogue::initialize: resizing tensors threw an unknown exception");
      throw;
    }
    {
      const bool init_succeeded = initialize_tensor(tensor_C.host_view(), init_C, seed + 2020);
      if (not init_succeeded) {
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1768-1772

```cpp
        CUTLASS_TRACE_HOST("HostCollectiveDefaultEpilogue::initialize: initialize_tensor returned false");
      }
      EXPECT_TRUE(init_succeeded);
    }
    tensor_C.host_view().at({0, 0}) = ElementC(1);
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 1774-1774

```cpp
    cutlass::reference::host::TensorCopy(reference_D.host_view(), tensor_C.host_view());
```
- **EN:** Declares member fields or local variables related to `reference`, `host`, `TensorCopy`, `reference_D`, `host_view` for later setup, execution, or verification.
- **CN:** 声明与 `reference`, `host`, `TensorCopy`, `reference_D`, `host_view` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1776-1787

```cpp
    try {
      tensor_C.sync_device();
      tensor_D.sync_device();
    }
    catch (std::exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveDefaultEpilogue::initialize: sync_device() threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("HostCollectiveDefaultEpilogue::initialize: sync_device() threw an unknown exception");
      throw;
    }
```
- **EN:** Implements or wires together logic around `try`, `tensor_C`, `sync_device`, `tensor_D`, `catch` for the current test scenario.
- **CN:** 围绕 `try`, `tensor_C`, `sync_device`, `tensor_D`, `catch` 实现或连接当前测试场景所需的逻辑。

### Lines 1789-1790

```cpp
    alpha = alpha_;
    beta = beta_;
```
- **EN:** Declares member fields or local variables related to `alpha`, `alpha_`, `beta`, `beta_` for later setup, execution, or verification.
- **CN:** 声明与 `alpha`, `alpha_`, `beta`, `beta_` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1792-1793

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1795-1801

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

### Lines 1803-1808

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

### Lines 1810-1822

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

### Lines 1824-1828

```cpp
  bool compare_reference(
      cute::Shape<int,int,int,int> problem_shape_MNKL,
      ElementScalar alpha,
      ElementScalar beta) {
    auto [M, N, K, L] = problem_shape_MNKL;
```
- **EN:** Implements or wires together logic around `compare_reference`, `problem_shape_MNKL`, `ElementScalar`, `alpha`, `beta` for the current test scenario.
- **CN:** 围绕 `compare_reference`, `problem_shape_MNKL`, `ElementScalar`, `alpha`, `beta` 实现或连接当前测试场景所需的逻辑。

### Lines 1830-1831

```cpp
    tensor_D.sync_host();
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_C.host_view()), 0);
```
- **EN:** Declares member fields or local variables related to `tensor_D`, `sync_host`, `EXPECT_GT`, `reference`, `host` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_D`, `sync_host`, `EXPECT_GT`, `reference`, `host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1833-1835

```cpp
    if (tensor_D.size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_D.host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1837-1839

```cpp
    if (reference_D.size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(reference_D.host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1841-1846

```cpp
    bool passed = equality_check(reference_D.host_view(), tensor_D.host_view());
    if(!passed) {
      std::cout<<"D is incorrect"<<std::endl;
    }
    return passed;
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 1848-1853

```cpp
  void print_tensors(std::ofstream& file) {
    file
    << "\nC =\n" << tensor_C.host_view()
    << "\n\nReference =\n" << reference_D.host_view()
    << "\n\nComputed =\n" << tensor_D.host_view();
  }
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1855-1860

```cpp
  Arguments to_args(ProblemShapeType problem_size) {
    Arguments arguments =
      {
        {alpha, beta},
        tensor_C.device_data(), stride_c, tensor_D.device_data(), stride_d
      };
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1862-1863

```cpp
    return arguments;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1865-1879

```cpp
  auto to_host_args(ProblemShapeType problem_size) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::get<0>(problem_shape_MNKL);
    auto N = cute::get<1>(problem_shape_MNKL);
    auto K = cute::get<2>(problem_shape_MNKL);
    auto L = cute::get<3>(problem_shape_MNKL);
    auto coord_0 = cutlass::make_Coord(0);
    auto C = cute::make_tensor(detail::make_iterator(tensor_C.host_data()),
        cute::make_layout(cute::make_shape(M, N, L), stride_c));
    auto D = cute::make_tensor(detail::make_iterator(reference_D.host_data()),
        cute::make_layout(cute::make_shape(M, N, L), stride_d));
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1881-1888

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

### Lines 1890-1893

```cpp
    epilogue_params.C = C;
    epilogue_params.D = D;
    epilogue_params.alpha = alpha;
    epilogue_params.beta = beta;
```
- **EN:** Declares member fields or local variables related to `epilogue_params`, `alpha`, `beta` for later setup, execution, or verification.
- **CN:** 声明与 `epilogue_params`, `alpha`, `beta` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1895-1897

```cpp
    return epilogue_params;
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 1899-1904

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

### Lines 1906-1909

```cpp
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  using kernel   = typename Gemm::GemmKernel;
  using Epilogue = typename kernel::CollectiveEpilogue;
  static_assert(IsDefaultEpilogue<Epilogue>::value == false, "Default Epilogue is not supported");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1911-1914

```cpp
  using ElementD = typename kernel::ElementD;
  using StrideD  = typename kernel::StrideD;
  using ElementC = non_void_t<typename kernel::ElementC, ElementD>;
  using StrideC  = typename kernel::StrideC;
```
- **EN:** Defines aliases such as `ElementD`, `StrideD`, `ElementC`, `StrideC` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementD`, `StrideD`, `ElementC`, `StrideC`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1916-1917

```cpp
  static_assert(rank(StrideC{}) == 3, "StrideCD must be rank-3: [M, N, L]");
  static_assert(rank(StrideD{}) == 3, "StrideCD must be rank-3: [M, N, L]");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1919-1922

```cpp
  static_assert(is_row_or_col_major<StrideC>(),
    "ERROR : C Layout is neither Row / Column Major)");
  static_assert(is_row_or_col_major<StrideD>(),
    "ERROR : D Layout is neither Row / Column Major)");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1924-1928

```cpp
  // Deduce Cutlass Layouts (RowMajor & ColumnMajor)
  using LayoutTagC = cutlass::detail::StrideToLayoutTagC_t<StrideC>;
  using LayoutTagD = cutlass::detail::StrideToLayoutTagC_t<StrideD>;
  using LayoutTagScalar = cutlass::layout::PackedVectorLayout; // scalars are size-1 vectors
  using LayoutTagVector = cutlass::layout::PackedVectorLayout;
```
- **EN:** Begins function or method `Layouts`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `Layouts`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 1930-1932

```cpp
  using ElementAccumulator = typename kernel::ElementAccumulator;
  using ElementScalingFactor = ElementAccumulator;
  using ProblemShapeType = typename kernel::ProblemShape;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementScalingFactor`, `ProblemShapeType`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 1934-1937

```cpp
  //
  // FusionOperation derived types/queries
  //
  static constexpr bool IsLegacy = detail::IsLegacyEpiloguePolicy<Epilogue>::value;
```
- **EN:** Implements or wires together logic around `FusionOperation`, `derived`, `types`, `queries`, `IsLegacy` for the current test scenario.
- **CN:** 围绕 `FusionOperation`, `derived`, `types`, `queries`, `IsLegacy` 实现或连接当前测试场景所需的逻辑。

### Lines 1939-1944

```cpp
  // FFMA2 SGEMM uses ThreadEpilogueOp for bias and relu support instead of FusionOp, so we compose LinCombPerRowBiasEltAct FusionOp by hand to test the functionality.
  static constexpr bool IsFfma2Kernel = cute::is_same_v<ScheduleType, cutlass::gemm::KernelMultistage>;
  using FusionOp = cute::conditional_t<IsFfma2Kernel,
                                       cutlass::epilogue::fusion::LinCombPerRowBiasEltAct<cutlass::epilogue::thread::Clamp, float, float>,
                                       typename Gemm::EpilogueOutputOp>;
  static_assert(cute::is_base_of_v<cutlass::epilogue::fusion::FusionOperation, FusionOp>);
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1947-1960

```cpp
  // Scale factor Generation related
  using SfStrategy = cutlass::reference::host::SfStrategy;
  static constexpr bool IsBlockScaleSupported            = FusionOp::IsBlockScaleSupported;
  static constexpr SfStrategy SfGenStrategy              = (!IsBlockScaleSupported) ? SfStrategy::None : SfStrategy::SfDGen;
  static constexpr int32_t SFD_VectorSize = IsBlockScaleSupported ? FusionOp::SFVecSize : 1;
  static constexpr bool IsKMajorSFD = cute::is_same_v<typename FusionOp::GmemLayoutTagScalefactor, cutlass::layout::RowMajor>;
  using ElementSFD = non_void_t<typename FusionOp::ElementBlockScaleFactor, ElementD>;
  using Sm1xxBlockScaledOutputConfig= cutlass::detail::Sm1xxBlockScaledOutputConfig<SFD_VectorSize,
                                        IsKMajorSFD ? cute::UMMA::Major::K : cute::UMMA::Major::MN>;
  using Blk_MN = typename Sm1xxBlockScaledOutputConfig::Blk_MN;
  using Blk_SF = typename Sm1xxBlockScaledOutputConfig::Blk_SF;
  using OutputSFAtom = typename Sm1xxBlockScaledOutputConfig::SfAtom;
  cutlass::HostTensor<ElementSFD, LayoutTagD> tensor_SFD;
  cutlass::HostTensor<ElementSFD, LayoutTagD> reference_SFD;
```
- **EN:** Introduces type aliases like `SfStrategy`, `ElementSFD`, `Sm1xxBlockScaledOutputConfig`, `Blk_MN`, `Blk_SF`, `OutputSFAtom` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `SfStrategy`, `ElementSFD`, `Sm1xxBlockScaledOutputConfig`, `Blk_MN`, `Blk_SF`, `OutputSFAtom`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 1962-1969

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

### Lines 1971-1973

```cpp
  static constexpr bool IsRowBiasEnabled        = FusionOp::IsPerRowBiasSupported;
  static constexpr bool IsColBiasEnabled        = FusionOp::IsPerColBiasSupported;
  static_assert(not (IsColBiasEnabled && IsRowBiasEnabled));
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 1975-1987

```cpp
  static constexpr bool IsDeBiasEnabled      = FusionOp::IsDePerRowBiasSupported;
  static constexpr bool IsPerRowScaleEnabled = FusionOp::IsPerRowScaleSupported;
  static constexpr bool IsPerColScaleEnabled = FusionOp::IsPerColScaleSupported;
  static constexpr bool IsScaleFactorEnabled = FusionOp::IsScaleFactorSupported;
  static constexpr bool IsAuxInEnabled       = FusionOp::IsAuxInSupported;
  static constexpr bool IsAuxOutEnabled      = FusionOp::IsAuxOutSupported;
  static constexpr bool IsAbsMaxEnabledD     = FusionOp::IsAbsMaxSupported &&
                                                (cute::is_same_v<ElementD, cutlass::float_e4m3_t> ||
                                                 cute::is_same_v<ElementD, cutlass::float_e5m2_t>);
  static constexpr bool IsAbsMaxEnabledAux   = IsAuxOutEnabled && FusionOp::IsAbsMaxSupported &&
                                                (cute::is_same_v<ElementAux, cutlass::float_e4m3_t> ||
                                                 cute::is_same_v<ElementAux, cutlass::float_e5m2_t>);
  using Arguments = typename Gemm::GemmKernel::EpilogueArguments;
```
- **EN:** Implements or wires together logic around `IsDeBiasEnabled`, `FusionOp`, `IsDePerRowBiasSupported`, `IsPerRowScaleEnabled`, `IsPerRowScaleSupported` for the current test scenario.
- **CN:** 围绕 `IsDeBiasEnabled`, `FusionOp`, `IsDePerRowBiasSupported`, `IsPerRowScaleEnabled`, `IsPerRowScaleSupported` 实现或连接当前测试场景所需的逻辑。

### Lines 1989-1991

```cpp
  /// Initialization
  StrideC stride_c;
  StrideD stride_d;
```
- **EN:** Implements or wires together logic around `Initialization`, `StrideC`, `stride_c`, `StrideD`, `stride_d` for the current test scenario.
- **CN:** 围绕 `Initialization`, `StrideC`, `stride_c`, `StrideD`, `stride_d` 实现或连接当前测试场景所需的逻辑。

### Lines 1993-1994

```cpp
  typename LayoutTagC::Stride stride_factor_C;
  typename LayoutTagD::Stride stride_factor_D;
```
- **EN:** Declares member fields or local variables related to `LayoutTagC`, `Stride`, `stride_factor_C`, `LayoutTagD`, `stride_factor_D` for later setup, execution, or verification.
- **CN:** 声明与 `LayoutTagC`, `Stride`, `stride_factor_C`, `LayoutTagD`, `stride_factor_D` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 1996-2006

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
  cutlass::HostTensor<ElementC, LayoutTagC> tensor_C;
  cutlass::HostTensor<ElementCompute, LayoutTagScalar> norm_constant;
```
- **EN:** Implements or wires together logic around `Inputs`, `HostTensor`, `ElementScalar`, `LayoutTagScalar`, `alpha` for the current test scenario.
- **CN:** 围绕 `Inputs`, `HostTensor`, `ElementScalar`, `LayoutTagScalar`, `alpha` 实现或连接当前测试场景所需的逻辑。

### Lines 2008-2014

```cpp
  // Outputs
  cutlass::HostTensor<ElementAmax, LayoutTagScalar> abs_max_Aux;
  cutlass::HostTensor<ElementAmax, LayoutTagScalar> abs_max_D;
  cutlass::HostTensor<ElementAux , LayoutTagAux   > tensor_Aux;
  cutlass::gemm::TagToStrideC_t<   LayoutTagAux   > stride_Aux;
  cutlass::HostTensor<ElementD, LayoutTagD> tensor_D;
  cutlass::HostTensor<ElementD, LayoutTagD> reference_D;
```
- **EN:** Implements or wires together logic around `Outputs`, `HostTensor`, `ElementAmax`, `LayoutTagScalar`, `abs_max_Aux` for the current test scenario.
- **CN:** 围绕 `Outputs`, `HostTensor`, `ElementAmax`, `LayoutTagScalar`, `abs_max_Aux` 实现或连接当前测试场景所需的逻辑。

### Lines 2016-2020

```cpp
  // References
  cutlass::HostTensor<ElementBias, LayoutTagVector> reference_dbias;
  cutlass::HostTensor<ElementAux , LayoutTagAux   > reference_Aux;
  cutlass::HostTensor<ElementAmax, LayoutTagScalar> reference_abs_max_Aux;
  cutlass::HostTensor<ElementAmax, LayoutTagScalar> reference_abs_max_D;
```
- **EN:** Implements or wires together logic around `References`, `HostTensor`, `ElementBias`, `LayoutTagVector`, `reference_dbias` for the current test scenario.
- **CN:** 围绕 `References`, `HostTensor`, `ElementBias`, `LayoutTagVector`, `reference_dbias` 实现或连接当前测试场景所需的逻辑。

### Lines 2022-2027

```cpp
  // Whether to use relative equality checks
  CheckEquality check_relative_equality = CheckEquality::EXACT;
  // Are scalars copied to device memory before kernel launch
  ScalarLoc use_device_scalars = ScalarLoc::ON_HOST;
  // If vector scale is supported and this is disabled, alpha/beta are passed as a host or device scalar instead of device vector
  VectorScale vector_scale_mode = VectorScale::DISABLED;
```
- **EN:** Implements or wires together logic around `Whether`, `use`, `relative`, `equality`, `checks` for the current test scenario.
- **CN:** 围绕 `Whether`, `use`, `relative`, `equality`, `checks` 实现或连接当前测试场景所需的逻辑。

### Lines 2029-2035

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

### Lines 2037-2050

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

### Lines 2052-2061

```cpp
  bool initialize(ProblemShapeType problem_size, ElementScalar alpha_=1.f, ElementScalar beta_=0.f) {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("HostCollectiveEpilogue::initialize(problem_size, alpha, beta)");
#endif
    // Initialize Epilogue tensors
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::size<0>(problem_shape_MNKL);
    auto N = cute::size<1>(problem_shape_MNKL);
    auto K = cute::size<2>(problem_shape_MNKL);
    auto L = cute::size<3>(problem_shape_MNKL);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2063-2064

```cpp
    stride_c = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(M, N, L));
    stride_d = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(M, N, L));
```
- **EN:** Declares member fields or local variables related to `stride_c`, `make_cute_packed_stride`, `StrideC`, `make_shape`, `stride_d` for later setup, execution, or verification.
- **CN:** 声明与 `stride_c`, `make_cute_packed_stride`, `StrideC`, `make_shape`, `stride_d` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2066-2080

```cpp
    // 2.x host tensor does not natively contain a batch stride or coord, so we spoof if by folding it into the outer mode
    auto c_coord = cutlass::make_Coord(M * L, N);
    try {
      tensor_C.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagC>::layout_factory(c_coord, stride_factor_C));
      tensor_D.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, stride_factor_D));
      reference_D.resize(c_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(c_coord, stride_factor_D), false);
    }
    catch (std::exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveEpilogue::initialize: resizing tensors threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("HostCollectiveEpilogue::initialize: resizing tensors threw an unknown exception");
      throw;
    }
```
- **EN:** Implements or wires together logic around `host`, `tensor`, `does`, `not`, `natively` for the current test scenario.
- **CN:** 围绕 `host`, `tensor`, `does`, `not`, `natively` 实现或连接当前测试场景所需的逻辑。

### Lines 2082-2097

```cpp
    try {
      const bool initialize_tensor_C_succeeded =
        initialize_tensor(tensor_C.host_view(), init_C, seed + 2020);
      if (not initialize_tensor_C_succeeded) {
        CUTLASS_TRACE_HOST("HostCollectiveEpilogue::initialize: initialize_tensor returned false");
      }
      EXPECT_TRUE(initialize_tensor_C_succeeded);
    }
    catch (std::exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveEpilogue::initialize: initialize_tensor threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("HostCollectiveEpilogue::initialize: initialize_tensor threw an unknown exception");
      throw;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2099-2099

```cpp
    tensor_C.host_view().at({0, 0}) = ElementC(1);
```
- **EN:** Declares member fields or local variables related to `tensor_C`, `host_view`, `ElementC` for later setup, execution, or verification.
- **CN:** 声明与 `tensor_C`, `host_view`, `ElementC` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2101-2113

```cpp
    cutlass::reference::host::TensorCopy(reference_D.host_view(), tensor_C.host_view());
    try {
      tensor_C.sync_device();
      tensor_D.sync_device();
    }
    catch (std::exception const& e) {
      CUTLASS_TRACE_HOST("HostCollectiveEpilogue::initialize: sync_device() threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("HostCollectiveEpilogue::initialize: sync_device() threw an unknown exception");
      throw;
    }
```
- **EN:** Implements or wires together logic around `reference`, `host`, `TensorCopy`, `reference_D`, `host_view` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `TensorCopy`, `reference_D`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 2115-2132

```cpp
    auto scalar_coord = cutlass::make_Coord(1);
    auto col_vector_coord = cutlass::make_Coord(M);
    auto row_vector_coord = cutlass::make_Coord(N);
    auto batch_vector_coord = cutlass::make_Coord(L);
    if constexpr (IsPerRowScaleEnabled or IsPerColScaleEnabled) {
      // scalars
      if (vector_scale_mode == VectorScale::DISABLED) {
        // batched scalars
        if (use_device_scalars == ScalarLoc::ON_DEVICE) {
          alpha.resize(batch_vector_coord, true);
          beta.resize(batch_vector_coord, true);
          EXPECT_TRUE(initialize_tensor(alpha.host_view(), init_scale, seed + 2023));
          if (beta_ != ElementScalar(0)) {
            EXPECT_TRUE(initialize_tensor(beta.host_view(), init_scale, seed + 2024));
          }
          else {
            cutlass::reference::host::TensorFill(beta.host_view(), beta_);
          }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2133-2150

```cpp
        }
        // non-batched scalars
        else {
          alpha.resize(scalar_coord, false);
          beta.resize(scalar_coord, false);
          cutlass::reference::host::TensorFill(alpha.host_view(), alpha_);
          cutlass::reference::host::TensorFill(beta.host_view(), beta_);
        }
      }
      // batched vectors
      else {
        auto batched_vector_coord = cutlass::make_Coord((IsPerRowScaleEnabled ? M : N) * L);
        alpha.resize(batched_vector_coord, true);
        beta.resize(batched_vector_coord, true);
        EXPECT_TRUE(initialize_tensor(alpha.host_view(), init_scale, seed + 2023));
        if (beta_ != ElementScalar(0)) {
          EXPECT_TRUE(initialize_tensor(beta.host_view(), init_scale, seed + 2024));
        }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2151-2168

```cpp
        else {
          cutlass::reference::host::TensorFill(beta.host_view(), beta_);
        }
      }
    }
    else {
      if (use_device_scalars == ScalarLoc::ON_DEVICE) {
        // Set alpha  beta for different batches.
        alpha.resize(batch_vector_coord, true);
        beta.resize(batch_vector_coord, true);
        cutlass::reference::host::TensorFill(alpha.host_view(), alpha_);
        for (int l = 0; l < L; ++l) {
          beta.host_view().at(cutlass::make_Coord(l)) = beta_ + ElementScalar(l);
        }
      }
      else {
        alpha.resize(scalar_coord, false);
        beta.resize(scalar_coord, false);
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2169-2174

```cpp
        cutlass::reference::host::TensorFill(alpha.host_view(), alpha_);
        cutlass::reference::host::TensorFill(beta.host_view(), beta_);
      }
    }
    alpha.sync_device();
    beta.sync_device();
```
- **EN:** Implements or wires together logic around `reference`, `host`, `TensorFill`, `alpha`, `host_view` for the current test scenario.
- **CN:** 围绕 `reference`, `host`, `TensorFill`, `alpha`, `host_view` 实现或连接当前测试场景所需的逻辑。

### Lines 2176-2189

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

### Lines 2191-2195

```cpp
    if constexpr (IsRowBiasEnabled or IsColBiasEnabled) {
      bias.resize(IsRowBiasEnabled ? col_vector_coord : row_vector_coord);
      EXPECT_TRUE(initialize_tensor(bias.host_view(), init_bias, seed + 2023));
      bias.sync_device();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2197-2203

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

### Lines 2205-2213

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

### Lines 2215-2222

```cpp
    if constexpr (IsAuxInEnabled) {
      auto aux_coord = cutlass::make_Coord(M * L, N);
      auto aux_layout = cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(aux_coord, typename LayoutTagAux::Stride{});
      tensor_Aux.resize(aux_coord, aux_layout);
      EXPECT_TRUE(initialize_tensor(tensor_Aux.host_view(), init_C, seed + 2023));
      tensor_Aux.sync_device();
      stride_Aux = cutlass::make_cute_packed_stride(cutlass::gemm::TagToStrideC_t<LayoutTagAux>{}, cute::make_shape(M, N, L));
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2224-2230

```cpp
    if constexpr (IsAuxOutEnabled) {
      auto aux_coord = cutlass::make_Coord(M * L, N);
      auto aux_layout = cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(aux_coord, typename LayoutTagAux::Stride{});
      tensor_Aux.resize(aux_coord, aux_layout);
      reference_Aux.resize(aux_coord, aux_layout, false);
      tensor_Aux.sync_device();
      stride_Aux = cutlass::make_cute_packed_stride(cutlass::gemm::TagToStrideC_t<LayoutTagAux>{}, cute::make_shape(M, N, L));
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2232-2236

```cpp
      if constexpr (IsScaleFactorEnabled) {
        scale_Aux.resize(scalar_coord, (use_device_scalars == ScalarLoc::ON_DEVICE));
        EXPECT_TRUE(initialize_tensor(scale_Aux.host_view(), init_scale, seed + 2027));
        scale_Aux.sync_device();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2238-2247

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

### Lines 2250-2267

```cpp
    if constexpr (IsBlockScaleSupported) {
      auto m_blks = cutlass::ceil_div(M, cute::size<0>(cute::shape(OutputSFAtom{})));
      auto n_blks = cutlass::ceil_div(N, cute::size<1>(cute::shape(OutputSFAtom{})));
      auto sfd_coord = [&] () {
        if constexpr (IsKMajorSFD) {
          return cutlass::make_Coord(m_blks * Blk_MN{} * L, n_blks * Blk_SF{});
        }
        else {
          return cutlass::make_Coord(m_blks * Blk_SF{} * L, n_blks * Blk_MN{});
        }
      }();
      tensor_SFD.resize(sfd_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(sfd_coord, stride_factor_D));
      reference_SFD.resize(sfd_coord, cutlass::layout::Affine2Layout_Factory<LayoutTagD>::layout_factory(sfd_coord, stride_factor_D), false);
      tensor_SFD.sync_device();
      norm_constant.resize(scalar_coord, true);
      EXPECT_TRUE(initialize_tensor(norm_constant.host_view(), init_scale, seed + 2023));
      norm_constant.sync_device();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2270-2271

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2273-2279

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

### Lines 2281-2286

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

### Lines 2288-2300

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

### Lines 2302-2307

```cpp
  bool compare_reference(
      cute::Shape<int,int,int,int> problem_shape_MNKL,
      ElementScalar alpha,
      ElementScalar beta) {
    tensor_D.sync_host();
    EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_C.host_view()), 0);
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 2309-2311

```cpp
    if (tensor_D.size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_D.host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2313-2315

```cpp
    if (reference_D.size() > 1) {
      EXPECT_GT(cutlass::reference::host::TensorNorm(reference_D.host_view()), 0);
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2317-2334

```cpp
    bool passed = equality_check(reference_D.host_view(), tensor_D.host_view());
    if(!passed) {
      #if 0
      auto [M, N, K, L] = problem_shape_MNKL;
      auto ref = cute::make_tensor(detail::make_iterator(reference_D.host_data()),
        cute::make_layout(cute::make_shape(M, N, L), stride_d));
      auto comp = cute::make_tensor(detail::make_iterator(tensor_D.host_data()),
        cute::make_layout(cute::make_shape(M, N, L), stride_d));
      for(int i=0; i<M; i++) {
        for(int j=0; j<N; j++) {
          for(int l=0; l<L; l++) {
            if(static_cast<float>(ElementD(ref(i, j, l))) != static_cast<float>((ElementD(comp(i, j, l))))) {
              printf("<m %d, n %d, l %d> ref: %f comp: %f\n", i, j, l, static_cast<float>(ElementD(ref(i, j, l))), static_cast<float>((ElementD(comp(i, j, l)))));
            }
          }
        }
      }
      #endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2335-2336

```cpp
      std::cout<<"D is incorrect"<<std::endl;
    }
```
- **EN:** Implements or wires together logic around `std`, `cout`, `incorrect`, `endl` for the current test scenario.
- **CN:** 围绕 `std`, `cout`, `incorrect`, `endl` 实现或连接当前测试场景所需的逻辑。

### Lines 2338-2341

```cpp
    if constexpr (IsAbsMaxEnabledD) {
      abs_max_D.sync_host();
      passed &= equality_check(reference_abs_max_D.host_view(), abs_max_D.host_view());
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2343-2348

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

### Lines 2350-2366

```cpp
    if constexpr (IsAuxOutEnabled) {
      tensor_Aux.sync_host();
      EXPECT_GT(cutlass::reference::host::TensorNorm(tensor_Aux.host_view()), 0);
      EXPECT_GT(cutlass::reference::host::TensorNorm(reference_Aux.host_view()), 0);
      passed &= equality_check(reference_Aux.host_view(), tensor_Aux.host_view());
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

### Lines 2369-2376

```cpp
    if constexpr (IsBlockScaleSupported) {
      tensor_SFD.sync_host();
      bool passed_sf = equality_check(reference_SFD.host_view(), tensor_SFD.host_view());
      if(!passed_sf) {
        std::cout<<"SF is incorrect"<<std::endl;
      }
      passed &= passed_sf;
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2378-2379

```cpp
    return passed;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2381-2397

```cpp
  void print_tensors(std::ofstream& file) {
    auto coord_0 = cutlass::make_Coord(0);
    if constexpr (IsScaleFactorEnabled) {
      file
        << ", scale_a: " << scale_A.at(coord_0)
        << ", scale_b: " << scale_B.at(coord_0)
        << ", scale_c: " << scale_C.at(coord_0);
    }
    if constexpr (IsPerRowScaleEnabled or IsPerColScaleEnabled) {
      file << "\n\nvalpha = \n" << alpha.host_view();
      file << "\n\nvbeta = \n" << beta.host_view();
    } else {
      file
        << "\n\nalpha= \n" << alpha.host_view()
        << "\n\nbeta= \n " << beta.host_view();
    }
    file << "\n\n";
```
- **EN:** Begins function or method `print_tensors`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `print_tensors`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2399-2402

```cpp
    if constexpr (IsAbsMaxEnabledD) {
      file << "scale_d: " << float(scale_D.at(coord_0));
      file << "\nReference abs_max_D :";
      file << " " << float(reference_abs_max_D.at(coord_0));
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2404-2407

```cpp
      file << "\nComputed abs_max_D :";
      file << " " << float(abs_max_D.at(coord_0));
      file << "\n\n";
    }
```
- **EN:** Implements or wires together logic around `file`, `nComputed`, `abs_max_D`, `coord_0` for the current test scenario.
- **CN:** 围绕 `file`, `nComputed`, `abs_max_D`, `coord_0` 实现或连接当前测试场景所需的逻辑。

### Lines 2409-2412

```cpp
    if constexpr (IsAbsMaxEnabledAux) {
      file << "scale_aux: " << float(scale_Aux.at(coord_0));
      file << "\nReference abs_max_Aux :";
      file << " " << float(reference_abs_max_Aux.at(coord_0));
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2414-2417

```cpp
      file << "\nComputed abs_max_Aux :";
      file << " " << float(abs_max_Aux.at(coord_0));
      file << "\n\n";
    }
```
- **EN:** Implements or wires together logic around `file`, `nComputed`, `abs_max_Aux`, `coord_0` for the current test scenario.
- **CN:** 围绕 `file`, `nComputed`, `abs_max_Aux`, `coord_0` 实现或连接当前测试场景所需的逻辑。

### Lines 2419-2421

```cpp
    if constexpr (IsRowBiasEnabled or IsColBiasEnabled) {
      file << "\n\nBias = \n" << bias.host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2423-2425

```cpp
    if constexpr (IsAuxInEnabled) {
      file << "\n\nAux Input = \n" << tensor_Aux.host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2427-2430

```cpp
    if constexpr (IsDeBiasEnabled) {
      file << "\n\nReference dBias = \n" << reference_dbias.host_view();
      file << "\n\nComputed dBias = \n" << bias.host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2432-2436

```cpp
    if constexpr (IsAuxOutEnabled) {
      file
        << "\n\nReference Aux =\n" << reference_Aux.host_view()
        << "\n\nComputed Aux =\n" << tensor_Aux.host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2438-2442

```cpp
    if constexpr (IsBlockScaleSupported) {
      file
        << "\n\nSFD Reference =\n" << reference_SFD.host_view()
        << "\n\nSFD Computed =\n" << tensor_SFD.host_view();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2444-2449

```cpp
    file
    << "\nC =\n" << tensor_C.host_view()
    << "\n\nReference =\n" << reference_D.host_view()
    << "\n\nComputed =\n" << tensor_D.host_view();
  }
```
- **EN:** Implements or wires together logic around `file`, `tensor_C`, `host_view`, `nReference`, `reference_D` for the current test scenario.
- **CN:** 围绕 `file`, `tensor_C`, `host_view`, `nReference`, `reference_D` 实现或连接当前测试场景所需的逻辑。

### Lines 2451-2459

```cpp
  Arguments to_args(ProblemShapeType problem_size) {
    auto coord_0 = cutlass::make_Coord(0);
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto [M, N, K, L] = problem_shape_MNKL;
    Arguments arguments =
      {
        {},
        tensor_C.device_data(), stride_c, tensor_D.device_data(), stride_d
      };
```
- **EN:** Begins function or method `to_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2461-2478

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
      arguments.ptr_T = tensor_Aux.device_data();
    }
    else {
      fusion_args.alpha = alpha.at(coord_0);
      fusion_args.alpha_ptr = alpha.device_data();
      // Only initializing beta/beta_ptr for non-void source
      if constexpr (not cute::is_void_v<typename kernel::ElementC>) {
        fusion_args.beta = beta.at(coord_0);
        fusion_args.beta_ptr = beta.device_data(); // if vector_scale_mode is true this is nullptr
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2479-2479

```cpp
      }
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 2481-2498

```cpp
      if constexpr (IsPerRowScaleEnabled) {
        int32_t m_stride = vector_scale_mode == VectorScale::ENABLED ? 1 : 0;
        int64_t l_stride = vector_scale_mode == VectorScale::ENABLED ? M : (use_device_scalars == ScalarLoc::ON_DEVICE ? 1 : 0);
        fusion_args.dAlpha = cute::make_stride(bool(m_stride),cute::_0{}, l_stride);
        fusion_args.dBeta = cute::make_stride(bool(m_stride),cute::_0{}, l_stride);
      }
      else if constexpr (IsPerColScaleEnabled) {
        int32_t n_stride = vector_scale_mode == VectorScale::ENABLED ? 1 : 0;
        int64_t l_stride = vector_scale_mode == VectorScale::ENABLED ? N : (use_device_scalars == ScalarLoc::ON_DEVICE ? 1 : 0);
        fusion_args.dAlpha = cute::make_stride(cute::_0{}, bool(n_stride), l_stride);
        fusion_args.dBeta = cute::make_stride(cute::_0{}, bool(n_stride), l_stride);
      }
      else {
        if constexpr (not IsFfma2Kernel) {
          if (use_device_scalars == ScalarLoc::ON_DEVICE) {
            if (L > 1) {
              fusion_args.dAlpha = cute::make_stride(cute::_0{},cute::_0{}, int64_t(1));
              fusion_args.dBeta  = cute::make_stride(cute::_0{},cute::_0{}, int64_t(1));
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2499-2502

```cpp
            }
          }
        }
      }
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 2504-2513

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

### Lines 2515-2517

```cpp
      if constexpr (IsRowBiasEnabled or IsColBiasEnabled) {
        fusion_args.bias_ptr = bias.device_data();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2519-2521

```cpp
      if constexpr (IsDeBiasEnabled) {
        fusion_args.dbias_ptr = bias.device_data();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2523-2535

```cpp
      // example of how to set kernel activation arguments
      // see ActivationFunctor::Arguments in activation.h for definition
      // if Arguments doesn't exist then fusion_args.activation is empty
      auto init_activation_args = [] (auto activation, auto& args) {
        using Activation = cute::remove_cvref_t<decltype(activation)>;
        if constexpr (cute::is_same_v<Activation, cutlass::epilogue::thread::Clamp<ElementCompute>>) {
          args.lower_bound = 0; // Treat Clamp as ReLU
          args.upper_bound = cutlass::platform::identity_for_minimum<ElementCompute>();
        }
        if constexpr (cute::is_same_v<Activation, cutlass::epilogue::thread::ScaledGELU_taylor<ElementCompute>>) {
          args.scale = ElementCompute(1);
        }
      };
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2537-2542

```cpp
      if constexpr (not cute::is_same_v<ActivationFunctor, cutlass::epilogue::thread::Identity<ElementCompute>>) {
        init_activation_args(ActivationFunctor{}, fusion_args.activation);
      }
      if constexpr (IsAbsMaxEnabledD) {
        fusion_args.amax_D_ptr = abs_max_D.device_data();
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2544-2547

```cpp
      if constexpr (IsAuxInEnabled) {
        fusion_args.aux_ptr = tensor_Aux.device_data();
        fusion_args.dAux = stride_Aux;
      }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2549-2559

```cpp
      if constexpr (IsAuxOutEnabled) {
        fusion_args.aux_ptr = tensor_Aux.device_data();
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

### Lines 2562-2566

```cpp
      if constexpr (IsBlockScaleSupported) {
        arguments.thread.block_scale_factor_ptr = tensor_SFD.device_data();
        arguments.thread.norm_constant_ptr = norm_constant.device_data();
      }
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2568-2569

```cpp
    return arguments;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2571-2588

```cpp
  auto to_host_args(ProblemShapeType problem_size) {
    using namespace cute;
    //
    // Allocate the GEMM workspace
    //
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto M = cute::get<0>(problem_shape_MNKL);
    auto N = cute::get<1>(problem_shape_MNKL);
    auto K = cute::get<2>(problem_shape_MNKL);
    auto L = cute::get<3>(problem_shape_MNKL);
    auto coord_0 = cutlass::make_Coord(0);
    auto C = cute::make_tensor(detail::make_iterator(tensor_C.host_data()),
        cute::make_layout(cute::make_shape(M, N, L), stride_c));
    auto D = cute::make_tensor(detail::make_iterator(reference_D.host_data()),
        cute::make_layout(cute::make_shape(M, N, L), stride_d));
    auto Bias = cute::make_tensor(detail::make_iterator(IsDeBiasEnabled ? reference_dbias.host_data() : bias.host_data()),
        cute::make_layout(cute::make_shape(IsRowBiasEnabled ? M : N)));
    auto Aux = cute::make_tensor(detail::make_iterator(IsAuxInEnabled ? tensor_Aux.host_data() : reference_Aux.host_data()),
```
- **EN:** Begins function or method `to_host_args`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `to_host_args`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2589-2606

```cpp
        cute::make_layout(cute::make_shape(M, N, L), stride_Aux));
    auto Valpha = [&](){
      if constexpr (IsPerRowScaleEnabled) {
        int m_stride = vector_scale_mode == VectorScale::ENABLED ? 1 : 0;
        int l_stride = vector_scale_mode == VectorScale::ENABLED ? M : (use_device_scalars == ScalarLoc::ON_DEVICE ? 1 : 0);
        return cute::make_tensor(detail::make_iterator(alpha.host_data()),
            cute::make_layout(cute::make_shape(M, N, L), make_stride(m_stride, cute::_0{}, l_stride)));
      }
      else if constexpr (IsPerColScaleEnabled) {
        int n_stride = vector_scale_mode == VectorScale::ENABLED ? 1 : 0;
        int l_stride = vector_scale_mode == VectorScale::ENABLED ? N : (use_device_scalars == ScalarLoc::ON_DEVICE ? 1 : 0);
        return cute::make_tensor(detail::make_iterator(alpha.host_data()),
            cute::make_layout(cute::make_shape(M, N, L), make_stride(cute::_0{}, n_stride, l_stride)));
      }
      else {
        return cute::make_tensor(detail::make_iterator(alpha.host_data()),
            cute::make_layout(cute::make_shape(M, N, L), make_stride(cute::_0{}, cute::_0{}, cute::_1{})));
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2607-2607

```cpp
    }();
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 2609-2626

```cpp
    auto Vbeta = [&]() {
      if constexpr (IsPerRowScaleEnabled) {
        int m_stride = vector_scale_mode == VectorScale::ENABLED ? 1 : 0;
        int l_stride = vector_scale_mode == VectorScale::ENABLED ? M : (use_device_scalars == ScalarLoc::ON_DEVICE ? 1 : 0);
        return cute::make_tensor(detail::make_iterator(beta.host_data()),
            cute::make_layout(cute::make_shape(M, N, L), make_stride(m_stride, cute::_0{}, l_stride)));
      }
      else if constexpr (IsPerColScaleEnabled) {
        int n_stride = vector_scale_mode == VectorScale::ENABLED ? 1 : 0;
        int l_stride = vector_scale_mode == VectorScale::ENABLED ? N : (use_device_scalars == ScalarLoc::ON_DEVICE ? 1 : 0);
        return cute::make_tensor(detail::make_iterator(beta.host_data()),
            cute::make_layout(cute::make_shape(M, N, L), make_stride(cute::_0{}, n_stride, l_stride)));
      }
      else {
        return  cute::make_tensor(detail::make_iterator(beta.host_data()),
            cute::make_layout(cute::make_shape(M, N, L), make_stride(cute::_0{}, cute::_0{}, cute::_1{})));
      }
    }();
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2628-2645

```cpp
    auto SfD = [&](){
      if constexpr (IsBlockScaleSupported) {
        auto tensor = make_tensor(detail::make_iterator(reference_SFD.host_data()),
          Sm1xxBlockScaledOutputConfig::tile_atom_to_shape_SFD(problem_shape_MNKL));
        return tensor;
      }
      else {
        // Reference kernel has a logic to ignore scalefactor computation if we pass the tensor type same as output D tensor.
        return D;
      }
    }();
    cutlass::reference::host::GettEpilogueParams<
      ElementScalar,
      ElementScalar,
      ElementAccumulator,
      ElementCompute,
      decltype(C),
      decltype(D),
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2646-2656

```cpp
      decltype(Bias),
      decltype(Aux),
      decltype(Valpha),
      decltype(Vbeta),
      ActivationFunctor,
      decltype(SfD),
      Int<SFD_VectorSize>,
      cutlass::plus<ElementCompute>,
      IsColBiasEnabled
      , SfGenStrategy
    > epilogue_params{};
```
- **EN:** Implements or wires together logic around `decltype`, `Bias`, `Aux`, `Valpha`, `Vbeta` for the current test scenario.
- **CN:** 围绕 `decltype`, `Bias`, `Aux`, `Valpha`, `Vbeta` 实现或连接当前测试场景所需的逻辑。

### Lines 2658-2661

```cpp
    epilogue_params.C = C;
    epilogue_params.D = D;
    epilogue_params.alpha = alpha.at(coord_0);
    epilogue_params.beta = beta.at(coord_0);
```
- **EN:** Declares member fields or local variables related to `epilogue_params`, `alpha`, `coord_0`, `beta` for later setup, execution, or verification.
- **CN:** 声明与 `epilogue_params`, `alpha`, `coord_0`, `beta` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2663-2668

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

### Lines 2670-2673

```cpp
    if constexpr (IsRowBiasEnabled or IsColBiasEnabled or IsDeBiasEnabled)
    {
      epilogue_params.Bias = Bias;
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2675-2677

```cpp
    if constexpr (IsAbsMaxEnabledD) {
      epilogue_params.abs_max_D = reference_abs_max_D.host_data();
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2679-2681

```cpp
    if constexpr (IsAuxInEnabled) {
      epilogue_params.Aux = Aux;
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2683-2691

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

### Lines 2693-2704

```cpp
    if constexpr (IsPerRowScaleEnabled or IsPerColScaleEnabled) {
      epilogue_params.Valpha = Valpha;
      if (vector_scale_mode == VectorScale::ENABLED) {
        epilogue_params.Vbeta = Vbeta;
      }
    }
    else {
      if (use_device_scalars == ScalarLoc::ON_DEVICE) {
        epilogue_params.Valpha = Valpha;
        epilogue_params.Vbeta = Vbeta;
      }
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2706-2712

```cpp
    if constexpr (IsBlockScaleSupported) {
      epilogue_params.SfD = SfD;
      epilogue_params.st = norm_constant.at(coord_0);
    }
    return epilogue_params;
  }
};
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2714-2727

```cpp
template <
  typename Gemm,
  template <class T> class ActivationFunctor_ = cutlass::epilogue::thread::Identity,
  bool force_legacy_epilogue = false,
  typename ElementA = typename Gemm::GemmKernel::ElementA,
  typename ElementB = typename Gemm::GemmKernel::ElementB
  , typename RuntimeDatatypeA = void*
  , typename RuntimeDatatypeB = void*
>
struct TestbedImpl {
  // Kernel data types
  using ScheduleType = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy::Schedule;
  // All Collective MMA operands are defined by HostCollectiveMainloopType based on the schedule type
  using HostCollectiveMainloopType = HostCollectiveMainloop<ScheduleType, Gemm, ElementA, ElementB>;
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 2729-2731

```cpp
  using CollectiveEpilogue = cute::conditional_t<IsDefaultEpilogue<typename Gemm::GemmKernel::CollectiveEpilogue>::value || force_legacy_epilogue,
                                                HostCollectiveDefaultEpilogue<Gemm>,
                                                HostCollectiveEpilogue<Gemm>>;
```
- **EN:** Implements or wires together logic around `CollectiveEpilogue`, `conditional_t`, `IsDefaultEpilogue`, `Gemm`, `GemmKernel` for the current test scenario.
- **CN:** 围绕 `CollectiveEpilogue`, `conditional_t`, `IsDefaultEpilogue`, `Gemm`, `GemmKernel` 实现或连接当前测试场景所需的逻辑。

### Lines 2733-2736

```cpp
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementCompute = typename ElementComputeType<Gemm, ElementAccumulator>::Type;
  using ElementScalar = typename ElementScalarType<Gemm, ElementCompute>::Type;
```
- **EN:** Defines aliases such as `ProblemShapeType`, `ElementAccumulator`, `ElementCompute`, `ElementScalar` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ProblemShapeType`, `ElementAccumulator`, `ElementCompute`, `ElementScalar`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 2738-2741

```cpp
  using LayoutTagA = typename HostCollectiveMainloopType::LayoutTagA;
  using LayoutTagB = typename HostCollectiveMainloopType::LayoutTagB;
  using LayoutTagC = typename CollectiveEpilogue::LayoutTagC;
  using LayoutTagD = typename CollectiveEpilogue::LayoutTagD;
```
- **EN:** Defines aliases such as `LayoutTagA`, `LayoutTagB`, `LayoutTagC`, `LayoutTagD` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `LayoutTagA`, `LayoutTagB`, `LayoutTagC`, `LayoutTagD`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 2744-2746

```cpp
  using InternalElementA = typename Gemm::GemmKernel::ElementA;
  using InternalElementB = typename Gemm::GemmKernel::ElementB;
  static constexpr bool IsRuntimeDataTypeA = cutlass::gemm::collective::detail::is_sm10x_runtime_f8f6f4<InternalElementA>();
```
- **EN:** Introduces type aliases like `InternalElementA`, `InternalElementB` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `InternalElementA`, `InternalElementB`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 2748-2748

```cpp
  static constexpr bool IsRuntimeDataTypeB = cutlass::gemm::collective::detail::is_sm10x_runtime_f8f6f4<InternalElementB>();
```
- **EN:** Declares member fields or local variables related to `IsRuntimeDataTypeB`, `gemm`, `collective`, `detail`, `is_sm10x_runtime_f8f6f4` for later setup, execution, or verification.
- **CN:** 声明与 `IsRuntimeDataTypeB`, `gemm`, `collective`, `detail`, `is_sm10x_runtime_f8f6f4` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2750-2752

```cpp
  static_assert((IsRuntimeDataTypeA && IsRuntimeDataTypeB) ||
                (!IsRuntimeDataTypeA && !IsRuntimeDataTypeB),
                "ElementA and ElementB in a GEMM kernel should be both runtime or both static.");
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 2754-2754

```cpp
  static constexpr bool IsRuntimeDataType = IsRuntimeDataTypeA && IsRuntimeDataTypeB;
```
- **EN:** Declares member fields or local variables related to `IsRuntimeDataType`, `IsRuntimeDataTypeA`, `IsRuntimeDataTypeB` for later setup, execution, or verification.
- **CN:** 声明与 `IsRuntimeDataType`, `IsRuntimeDataTypeA`, `IsRuntimeDataTypeB` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2757-2767

```cpp
  uint32_t sm_count;
  // Used to force multi-wave tests for persistent kernel schedules
  constexpr static int MaxSmCount = 16;
  static constexpr uint64_t kDefaultSeed = 4096;
  static constexpr uint32_t mma_promotion_interval = 4;
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
#if defined(SYCL_INTEL_TARGET)
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerXeStreamKParams::DecompositionMode;
#else
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2769-2770

```cpp
  HostCollectiveMainloopType collective_mma_inputs;
  CollectiveEpilogue collective_epilogue;
```
- **EN:** Declares member fields or local variables related to `HostCollectiveMainloopType`, `collective_mma_inputs`, `CollectiveEpilogue`, `collective_epilogue` for later setup, execution, or verification.
- **CN:** 声明与 `HostCollectiveMainloopType`, `collective_mma_inputs`, `CollectiveEpilogue`, `collective_epilogue` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2772-2776

```cpp
  // Flag to print "unsupported" message only once per test instance
  bool printed_unsupported_once = false;
  //
  // Methods
  //
```
- **EN:** Implements or wires together logic around `Flag`, `print`, `unsupported`, `message`, `only` for the current test scenario.
- **CN:** 围绕 `Flag`, `print`, `unsupported`, `message`, `only` 实现或连接当前测试场景所需的逻辑。

### Lines 2778-2789

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

### Lines 2791-2806

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

### Lines 2808-2814

```cpp
  /// Initializes data structures
  bool initialize(ProblemShapeType problem_size, ElementScalar alpha_=1.f, ElementScalar beta_=0.f) {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("TestbedImpl::initialize(problem_size, alpha, beta)");
#endif
    collective_mma_inputs.initialize(problem_size);
    collective_epilogue.initialize(problem_size, alpha_, beta_);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2816-2817

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2819-2825

```cpp
  /// Compares computed reference with device reference and outputs to a file if incorrect
  bool compare_reference(
      cute::Shape<int,int,int,int> problem_shape_MNKL,
      ElementScalar alpha,
      ElementScalar beta)
  {
    auto [M, N, K, L] = problem_shape_MNKL;
```
- **EN:** Implements or wires together logic around `Compares`, `computed`, `reference`, `with`, `device` for the current test scenario.
- **CN:** 围绕 `Compares`, `computed`, `reference`, `with`, `device` 实现或连接当前测试场景所需的逻辑。

### Lines 2827-2836

```cpp
    bool passed = collective_mma_inputs.compare_reference(problem_shape_MNKL);
    passed &= collective_epilogue.compare_reference(problem_shape_MNKL, alpha, beta);
    EXPECT_TRUE(passed);
    if (!passed) {
      std::stringstream fname;
      fname << "error_Gemm_device_"
        << M << "x" << N << "x" << K << "x" << L << "_"
        << cute::get<0>(typename Gemm::GemmKernel::TileShape{}) << "_"
        << cute::get<1>(typename Gemm::GemmKernel::TileShape{}) << "_"
        << cute::get<2>(typename Gemm::GemmKernel::TileShape{}) << ".txt";
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2838-2841

```cpp
      std::ofstream file(fname.str());
      file
        << "problem: " << ' ' << M << "x" << N << "x" << K << ", Batch count = " << L
        << ", alpha: " << alpha << ", beta: " << beta << "\n\n";
```
- **EN:** Implements or wires together logic around `std`, `ofstream`, `file`, `fname`, `str` for the current test scenario.
- **CN:** 围绕 `std`, `ofstream`, `file`, `fname`, `str` 实现或连接当前测试场景所需的逻辑。

### Lines 2843-2845

```cpp
      collective_mma_inputs.print_tensors(file);
      collective_epilogue.print_tensors(file);
    }
```
- **EN:** Implements or wires together logic around `collective_mma_inputs`, `print_tensors`, `file`, `collective_epilogue` for the current test scenario.
- **CN:** 围绕 `collective_mma_inputs`, `print_tensors`, `file`, `collective_epilogue` 实现或连接当前测试场景所需的逻辑。

### Lines 2847-2848

```cpp
    return passed;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2850-2859

```cpp
  /// Verifies the result is a GEMM
  bool verify(
      ProblemShapeType problem_size,
      ElementScalar alpha,
      ElementScalar beta)
  {
    using namespace cute;
    auto problem_shape_MNKL = cute::append<4>(problem_size, 1);
    auto mainloop_params = collective_mma_inputs.to_host_args(problem_size);
    auto epilogue_params = collective_epilogue.to_host_args(problem_size);
```
- **EN:** Implements or wires together logic around `Verifies`, `the`, `result`, `GEMM`, `verify` for the current test scenario.
- **CN:** 围绕 `Verifies`, `the`, `result`, `GEMM`, `verify` 实现或连接当前测试场景所需的逻辑。

### Lines 2861-2861

```cpp
    cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
```
- **EN:** Declares member fields or local variables related to `reference`, `host`, `Gemm3x`, `mainloop_params`, `epilogue_params` for later setup, execution, or verification.
- **CN:** 声明与 `reference`, `host`, `Gemm3x`, `mainloop_params`, `epilogue_params` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2863-2865

```cpp
    bool passed = compare_reference(problem_shape_MNKL, alpha, beta);
    return passed;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2867-2871

```cpp
	/// Determine if the CUDA device is sufficient to run the kernel
  bool sufficient() {
    //
    // Determine SMEM requirements and waive if not satisfied
    //
```
- **EN:** Implements or wires together logic around `Determine`, `the`, `CUDA`, `device`, `sufficient` for the current test scenario.
- **CN:** 围绕 `Determine`, `the`, `CUDA`, `device`, `sufficient` 实现或连接当前测试场景所需的逻辑。

### Lines 2873-2879

```cpp
    size_t smem_size = static_cast<size_t>(Gemm::GemmKernel::SharedStorageSize);
    size_t device_smem_size;
#if defined(CUTLASS_ENABLE_SYCL)
    compat::device_info info = compat::get_current_device().get_device_info();
    this->sm_count = info.get_max_compute_units();
    device_smem_size = info.get_local_mem_size();
#else
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2881-2882

```cpp
    int device_idx;
    cudaError_t result = cudaGetDevice(&device_idx);
```
- **EN:** Declares member fields or local variables related to `device_idx`, `cudaError_t`, `result`, `cudaGetDevice` for later setup, execution, or verification.
- **CN:** 声明与 `device_idx`, `cudaError_t`, `result`, `cudaGetDevice` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2884-2886

```cpp
    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDevice() API call failed.");
    }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 2888-2890

```cpp
    cudaDeviceProp properties;
    result = cudaGetDeviceProperties(&properties, device_idx);
    this->sm_count = properties.multiProcessorCount;
```
- **EN:** Declares member fields or local variables related to `cudaDeviceProp`, `properties`, `result`, `cudaGetDeviceProperties`, `device_idx` for later setup, execution, or verification.
- **CN:** 声明与 `cudaDeviceProp`, `properties`, `result`, `cudaGetDeviceProperties`, `device_idx` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 2892-2901

```cpp
    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDeviceProperties() failed");
    }
    device_smem_size = properties.sharedMemPerBlockOptin;
#endif
    if (device_smem_size < smem_size) {
      printf("failed due to smem_size\n");
      printf("hardware smem_size: %d, required smem_size: %d\n\n", int(device_smem_size), int(smem_size));
      return false;
    }
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2903-2904

```cpp
    return true;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 2906-2918

```cpp
  bool profile(
    ProblemShapeType problem_size,
    int iterations,
    Gemm& gemm_op,
    typename Gemm::Arguments& arguments,
    cutlass::device_memory::allocation<uint8_t>& workspace) {
    int M = cute::size<0>(problem_size);
    int N = cute::size<1>(problem_size);
    int K = cute::size<2>(problem_size);
    int L = 1;
    if constexpr(cute::rank(ProblemShapeType{}) == 4) {
      L = cute::size<3>(problem_size);
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2921-2931

```cpp
    cutlass::Status status;
    //
    // Run the GEMM
    //
    for (int iter = 0; iter < iterations; ++iter) {
      status = gemm_op(arguments, workspace.get());
      if (status != cutlass::Status::kSuccess) {
        EXPECT_TRUE(status == cutlass::Status::kSuccess) << to_string(status);
        return false;
      }
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2933-2948

```cpp
#if defined(CUTLASS_ENABLE_SYCL)
    try {
      compat::wait_and_throw();
    } catch (std::exception const &e) {
      ADD_FAILURE() << "Error at Kernel Sync.";
      return false;
    }
#else
    auto result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      EXPECT_EQ(result, cudaSuccess) << "Error at Kernel Sync.";
      return false;
    }
#endif
    return true;
  }
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2950-2967

```cpp
  /// Executes one test
  bool run(
    ProblemShapeType problem_size,
    ElementScalar alpha = ElementScalar(1),
    ElementScalar beta = ElementScalar(0),
    bool profiling = false,
    detail::Iterations iterations = detail::Iterations{},
    RasterOrderOptions raster_order = RasterOrderOptions::Heuristic,
    detail::MaxSwizzleSize max_swizzle = detail::MaxSwizzleSize{},
    detail::Splits splits = detail::Splits{},
    DecompositionMode decomposition_mode = DecompositionMode::Heuristic
    , RuntimeDatatypeA runtime_input_datatype_a = {}
    , RuntimeDatatypeB runtime_input_datatype_b = {}
    )
  {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("TestbedImpl::run");
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2969-2979

```cpp
    // Fail test if insufficient CUDA device
    if (!sufficient()) {
      CUTLASS_TRACE_HOST("TestbedImpl::run: Test failed due to insufficient CUDA device");
      std::cout << "Test failed due to insufficient CUDA device." << std::endl;
      return false;
    }
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    else {
      CUTLASS_TRACE_HOST("TestbedImpl::run: sufficient() returned true");
    }
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 2981-2996

```cpp
    try {
      const bool initialized = this->initialize(problem_size, alpha, beta);
      if (not initialized) {
        CUTLASS_TRACE_HOST("TestbedImpl::run: this->initialize returned false");
        std::cerr << "Initialization failed \n";
        return false;
      }
    }
    catch ([[maybe_unused]] std::exception const& e) {
      CUTLASS_TRACE_HOST("TestbedImpl::run: this->initialize threw an exception: " << e.what());
      throw;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("TestbedImpl::run: this->initialize threw an unknown exception");
      throw;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 2998-3000

```cpp
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("TestbedImpl::run: this->initialize() returned true");
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3002-3004

```cpp
    //
    // Initialize the GEMM operator
    //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 3006-3016

```cpp
    typename Gemm::Arguments arguments;
    cutlass::KernelHardwareInfo hw_info;
    hw_info.device_id = 0;
    if (not profiling) {
      this->sm_count = std::min(MaxSmCount, cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id));
      hw_info.sm_count = this->sm_count;
    }
    else {
      this->sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
      hw_info.sm_count = this->sm_count;
    }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3018-3029

```cpp
    typename Gemm::GemmKernel::TileScheduler::Arguments scheduler_args;
    if constexpr (cute::is_same_v<typename Gemm::GemmKernel::TileSchedulerTag, cutlass::gemm::StreamKScheduler>) {
#if defined(SYCL_INTEL_TARGET)
      scheduler_args = { static_cast<int>(splits), decomposition_mode };
#else
      scheduler_args = { static_cast<int>(splits), static_cast<int>(max_swizzle), raster_order, decomposition_mode };
#endif
    }
    else {
      scheduler_args = { static_cast<int>(max_swizzle), raster_order };
    }
    typename HostCollectiveMainloopType::Arguments mainloop_args;
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3031-3031

```cpp
    mainloop_args = collective_mma_inputs.to_args();
```
- **EN:** Declares member fields or local variables related to `mainloop_args`, `collective_mma_inputs`, `to_args` for later setup, execution, or verification.
- **CN:** 声明与 `mainloop_args`, `collective_mma_inputs`, `to_args` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3034-3037

```cpp
    if constexpr (IsRuntimeDataType) {
      mainloop_args.runtime_data_type_a = runtime_input_datatype_a;
      mainloop_args.runtime_data_type_b = runtime_input_datatype_b;
    }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3040-3048

```cpp
    arguments =
    {
      cutlass::gemm::GemmUniversalMode::kGemm,
      problem_size,
      mainloop_args,
      collective_epilogue.to_args(problem_size),
      hw_info,
      scheduler_args
    };
```
- **EN:** Implements or wires together logic around `arguments`, `gemm`, `GemmUniversalMode`, `kGemm`, `problem_size` for the current test scenario.
- **CN:** 围绕 `arguments`, `gemm`, `GemmUniversalMode`, `kGemm`, `problem_size` 实现或连接当前测试场景所需的逻辑。

### Lines 3050-3053

```cpp
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("TestbedImpl::run: Creating gemm_op");
#endif
    Gemm gemm_op;
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3055-3062

```cpp
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("TestbedImpl::run: Calling Gemm::get_workspace_size");
#endif
    size_t workspace_size = Gemm::get_workspace_size(arguments);
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("TestbedImpl::run: Allocating workspace of size " << workspace_size);
#endif
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3064-3067

```cpp
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
    CUTLASS_TRACE_HOST("TestbedImpl::run: Calling gemm_op.can_implement");
#endif
    cutlass::Status status = gemm_op.can_implement(arguments);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3069-3080

```cpp
    if (status != cutlass::Status::kSuccess) {
#if defined(CUTLASS_ENABLE_SYCL)
      test::unit::LogUnsupportedOnce(printed_unsupported_once);
      return true;
#else
      cudaError_t error = cudaGetLastError();
      const auto error_str = cudaGetErrorString(error);
      CUTLASS_TRACE_HOST("TestbedImpl::run: cudaGetLastError() is " << error_str);
      test::unit::LogUnsupportedOnce(printed_unsupported_once, error_str);
      return true;
#endif
    }
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3082-3084

```cpp
    //
    // Run the GEMM
    //
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 3086-3103

```cpp
    if (profiling) {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("TestbedImpl::run: Calling profile");
#endif
      return profile(problem_size, static_cast<int>(iterations), gemm_op, arguments, workspace);
    }
    else {
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("TestbedImpl::run: Calling gemm_op.initialize");
#endif
      status = gemm_op.initialize(arguments, workspace.get());
      if (status != cutlass::Status::kSuccess) {
    #if defined(CUTLASS_ENABLE_SYCL)
        test::unit::LogUnsupportedOnce(printed_unsupported_once);
    #else
        cudaError_t error = cudaGetLastError();
        const auto error_str = cudaGetErrorString(error);
        CUTLASS_TRACE_HOST("TestbedImpl::run: cudaGetLastError() is " << error_str);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3104-3121

```cpp
        test::unit::LogUnsupportedOnce(printed_unsupported_once, error_str);
    #endif
      }
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("TestbedImpl::run: Calling gemm_op.run");
#endif
      status = gemm_op.run();
#if defined(CUTLASS_ENABLE_SYCL)
      try {
        compat::wait_and_throw();
      } catch (std::exception const &e) {
        ADD_FAILURE() << "Error at Kernel Sync.";
        return false;
      }
#else
      if (status != cutlass::Status::kSuccess) {
        cudaError_t error = cudaGetLastError();
        const auto error_str = cudaGetErrorString(error);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3122-3134

```cpp
        CUTLASS_TRACE_HOST("TestbedImpl::run: cudaGetLastError() is " << error_str);
      }
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("TestbedImpl::run: Calling cudaDeviceSynchronize");
#endif
      auto result = cudaDeviceSynchronize();
      if (result != cudaSuccess) {
        CUTLASS_TRACE_HOST("TestbedImpl::run: cudaDeviceSynchronize reports non-success");
        EXPECT_EQ(result, cudaSuccess) << "Error at Kernel Sync.";
        return false;
      }
#endif
      EXPECT_TRUE(status == cutlass::Status::kSuccess) << to_string(status);
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3136-3149

```cpp
      //
      // Verify
      //
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("TestbedImpl::run: Calling this->verify");
#endif
      bool passed = this->verify(problem_size, alpha, beta);
      if (!passed) {
        CUTLASS_TRACE_HOST("TestbedImpl::run: this->verify FAILED");
#if !defined(CUTLASS_ENABLE_SYCL)
        cudaError_t error = cudaGetLastError();
        const auto error_str = cudaGetErrorString(error);
        CUTLASS_TRACE_HOST("TestbedImpl::run: cudaGetLastError() is " << error_str);
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3151-3158

```cpp
        std::cout << "Error : Failed : with alpha: " << alpha << ", beta: " << beta
                  << "\n";
      }
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      else {
        CUTLASS_TRACE_HOST("TestbedImpl::run: this->verify passed");
      }
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3160-3168

```cpp
#if (CUTLASS_DEBUG_TRACE_LEVEL > 1)
      CUTLASS_TRACE_HOST("TestbedImpl::run: Reached end");
#endif
      return passed;
    }
  }
};
} // namespace detail
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3170-3170

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 3173-3173

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 3175-3184

```cpp
template <
  typename Gemm,
  template <class T> class ActivationFunctor = cutlass::epilogue::thread::Identity,
  bool force_legacy_epilogue = false,
  typename ElementA = typename Gemm::GemmKernel::ElementA,
  typename ElementB = typename Gemm::GemmKernel::ElementB
  , typename RuntimeDatatypeA = void*
  , typename RuntimeDatatypeB = void*
>
struct Testbed3x {
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 3186-3196

```cpp
  using TestBedImpl = typename detail::TestbedImpl<
                        Gemm,
                        ActivationFunctor,
                        force_legacy_epilogue,
                        ElementA,
                        ElementB
                        , RuntimeDatatypeA
                        , RuntimeDatatypeB
                        >;
  using Kernel      = typename Gemm::GemmKernel;
  using Epilogue    = typename Gemm::GemmKernel::CollectiveEpilogue;
```
- **EN:** Introduces type aliases like `TestBedImpl`, `Kernel`, `Epilogue` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `TestBedImpl`, `Kernel`, `Epilogue`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 3198-3200

```cpp
  using ElementAccumulator   = typename TestBedImpl::ElementAccumulator;
  using ElementCompute       = typename TestBedImpl::ElementCompute;
  using ElementScalar        = typename TestBedImpl::ElementScalar;
```
- **EN:** Defines aliases such as `ElementAccumulator`, `ElementCompute`, `ElementScalar` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ElementAccumulator`, `ElementCompute`, `ElementScalar`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 3202-3209

```cpp
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
#if defined(SYCL_INTEL_TARGET)
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerXeStreamKParams::DecompositionMode;
#else
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
#endif
  // Detail Implementation
  TestBedImpl impl_;
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 3211-3224

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

### Lines 3226-3243

```cpp
  /// Executes one test
  bool run(
   typename TestBedImpl::ProblemShapeType problem_size,
    ElementScalar alpha = ElementScalar(1),
    ElementScalar beta = ElementScalar(0),
    RasterOrderOptions raster_order = RasterOrderOptions::Heuristic,
    detail::MaxSwizzleSize max_swizzle = detail::MaxSwizzleSize{},
    detail::Splits splits = detail::Splits{},
    DecompositionMode decomposition_mode = DecompositionMode::Heuristic,
    bool profiling = false,
    detail::Iterations iterations = detail::Iterations{}
    , RuntimeDatatypeA runtime_input_datatype_a = {}
    , RuntimeDatatypeB runtime_input_datatype_b = {}
    )
  {
    return impl_.run(
        problem_size, alpha, beta, profiling, iterations, raster_order, max_swizzle, splits, decomposition_mode
        , runtime_input_datatype_a, runtime_input_datatype_b
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 3244-3246

```cpp
        );
  }
};
```
- **EN:** Implements or wires together logic around `(none)` for the current test scenario.
- **CN:** 围绕 `(none)` 实现或连接当前测试场景所需的逻辑。

### Lines 3248-3248

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 3250-3257

```cpp
template <typename Gemm>
bool TestGemmPerf3x(int iterations = 20) {
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using ElementAccumulator = typename Gemm::GemmKernel::ElementAccumulator;
  using ElementScalar = ElementAccumulator;
  bool passed = true;
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
```
- **EN:** Introduces type aliases like `ProblemShapeType`, `ElementAccumulator`, `ElementScalar`, `DecompositionMode`, `RasterOrderOptions` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ProblemShapeType`, `ElementAccumulator`, `ElementScalar`, `DecompositionMode`, `RasterOrderOptions`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 3259-3261

```cpp
  std::vector<int> problem_size_m = { 4608 };
  std::vector<int> problem_size_n = { 4608 };
  std::vector<int> problem_size_k = { 8192 };
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `problem_size_m`, `problem_size_n`, `problem_size_k` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `problem_size_m`, `problem_size_n`, `problem_size_k` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3263-3263

```cpp
  Testbed3x<Gemm> testbed;
```
- **EN:** Declares member fields or local variables related to `Testbed3x`, `Gemm`, `testbed` for later setup, execution, or verification.
- **CN:** 声明与 `Testbed3x`, `Gemm`, `testbed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3265-3274

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

### Lines 3276-3282

```cpp
        passed = testbed.run(
          problem_size,
          cutlass::from_real<ElementScalar>(1),
          cutlass::from_real<ElementScalar>(0),
          RasterOrderOptions{}, detail::MaxSwizzleSize(1), detail::Splits{1}, DecompositionMode{},
          true, // profiling
          detail::Iterations{iterations});
```
- **EN:** Implements or wires together logic around `passed`, `testbed`, `run`, `problem_size`, `from_real` for the current test scenario.
- **CN:** 围绕 `passed`, `testbed`, `run`, `problem_size`, `from_real` 实现或连接当前测试场景所需的逻辑。

### Lines 3284-3289

```cpp
        if (!passed) {
          return false;
        }
      }
    }
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3291-3292

```cpp
  return true;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 3295-3310

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
//
template <
  typename Gemm,
  typename RuntimeDataTypeA,
  typename RuntimeDataTypeB,
  bool force_legacy_epilogue = false>
bool TestRuntimeDataTypeSmall(
  RuntimeDataTypeA runtime_input_datatype_a,
  RuntimeDataTypeB runtime_input_datatype_b,
  double alpha = 1.0, double beta = cute::is_same_v<typename Gemm::GemmKernel::ElementC, void> ? 0.0 : 1.0,
  CheckEquality check_relative_equality = CheckEquality::RELATIVE, ScalarLoc use_device_scalars = ScalarLoc::ON_DEVICE, VectorScale vector_scale_mode = VectorScale::ENABLED, std::vector<int> override_problem_size_k = {}) {
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using ElementScalar = typename Gemm::EpilogueOutputOp::ElementScalar;
  using CtaShape_MNK = typename Gemm::GemmKernel::CollectiveMainloop::CtaShape_MNK;
  using DispatchPolicy = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy;
```
- **EN:** Introduces type aliases like `ProblemShapeType`, `ElementScalar`, `CtaShape_MNK`, `DispatchPolicy` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ProblemShapeType`, `ElementScalar`, `CtaShape_MNK`, `DispatchPolicy`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 3312-3313

```cpp
  using InternalElementA = typename Gemm::GemmKernel::ElementA;
  using InternalElementB = typename Gemm::GemmKernel::ElementB;
```
- **EN:** Defines aliases such as `InternalElementA`, `InternalElementB` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `InternalElementA`, `InternalElementB`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 3315-3320

```cpp
  CtaShape_MNK cta_shape;
  static constexpr int SmCount  = 16;
  static constexpr int MultiplierOffsetM = 1;
  static constexpr int MultiplierOffsetN = 2;
  static constexpr int MultiplierOffsetK = 3;
  int max_alignment = std::max(Gemm::kAlignmentA, Gemm::kAlignmentB);
```
- **EN:** Declares member fields or local variables related to `CtaShape_MNK`, `cta_shape`, `SmCount`, `MultiplierOffsetM`, `MultiplierOffsetN` for later setup, execution, or verification.
- **CN:** 声明与 `CtaShape_MNK`, `cta_shape`, `SmCount`, `MultiplierOffsetM`, `MultiplierOffsetN` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3322-3324

```cpp
  float waves[] = {0.5, 1.25, 2.5};
  int cluster_m = 1;
  int cluster_n = 1;
```
- **EN:** Declares member fields or local variables related to `waves`, `cluster_m`, `cluster_n` for later setup, execution, or verification.
- **CN:** 声明与 `waves`, `cluster_m`, `cluster_n` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3326-3332

```cpp
  std::vector<int> problem_size_k;
  if (override_problem_size_k.empty()) {
    problem_size_k = {256 + max_alignment * MultiplierOffsetK, 512 + max_alignment * MultiplierOffsetK};
  }
  else {
    problem_size_k = override_problem_size_k;
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3334-3338

```cpp
  if constexpr(DispatchPolicy::ArchTag::kMinComputeCapability >= 90) {
    typename DispatchPolicy::ClusterShape cluster_shape;
    cluster_m = cute::size<0>(cluster_shape);
    cluster_n = cute::size<1>(cluster_shape);
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3340-3342

```cpp
  [[maybe_unused]] constexpr int TileShapeK = cute::size<2>(typename Gemm::GemmKernel::TileShape{});
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
```
- **EN:** Introduces type aliases like `DecompositionMode`, `RasterOrderOptions` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `DecompositionMode`, `RasterOrderOptions`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 3344-3351

```cpp
  std::vector<DecompositionMode> decomposition_modes = {DecompositionMode::Heuristic};
  static constexpr bool UsesStreamKScheduler = cute::is_same_v<typename Gemm::GemmKernel::TileSchedulerTag, cutlass::gemm::StreamKScheduler>;
  if constexpr (UsesStreamKScheduler) {
    decomposition_modes.push_back(DecompositionMode::DataParallel);
    decomposition_modes.push_back(DecompositionMode::SplitK);
    decomposition_modes.push_back(DecompositionMode::StreamK);
  }
  bool passed = true;
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3353-3356

```cpp
  for (float wave : waves) {
    for (int k : problem_size_k) {
      int grid_m, grid_n = 0;
      int num_grid = int(wave * SmCount);
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3358-3369

```cpp
      if (cluster_m >= cluster_n) {
        grid_m = cluster_m;
        grid_n = num_grid / grid_m;
        // Align grid_n to cluster_n
        grid_n = std::max((grid_n + cluster_n - 1 ) / cluster_n * cluster_n, 1);
      }
      else {
        grid_n = cluster_n;
        grid_m = num_grid / grid_n;
        // Align grid_m to cluster_m
        grid_m = std::max((grid_m + cluster_m - 1 ) / cluster_m * cluster_m, 1);
      }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3371-3372

```cpp
      int m = grid_m * cute::size<0>(cta_shape) + MultiplierOffsetM * max_alignment;
      int n = grid_n * cute::size<1>(cta_shape) + MultiplierOffsetN * max_alignment;
```
- **EN:** Declares member fields or local variables related to `grid_m`, `size`, `cta_shape`, `MultiplierOffsetM`, `max_alignment` for later setup, execution, or verification.
- **CN:** 声明与 `grid_m`, `size`, `cta_shape`, `MultiplierOffsetM`, `max_alignment` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3374-3380

```cpp
      ProblemShapeType problem_size;
      if constexpr (cute::rank(ProblemShapeType{}) == 4) {
        problem_size = ProblemShapeType{m, n, k, /* l */ 1};
      }
      else {
        problem_size = ProblemShapeType{m, n, k};
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3382-3387

```cpp
      for (DecompositionMode decomp_mode : decomposition_modes) {
        std::vector problem_splits = {detail::Splits{1}};
        if (decomp_mode == DecompositionMode::Heuristic || decomp_mode == DecompositionMode::SplitK) {
          problem_splits.push_back(detail::Splits{2});
        }
        for (auto splits : problem_splits) {
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3389-3406

```cpp
          if constexpr (cute::is_same_v<RuntimeDataTypeA, cute::UMMA::MXF4Format> &&
                        cute::is_same_v<RuntimeDataTypeB, cute::UMMA::MXF4Format>) {
            // e2m1_e2m1
            if (runtime_input_datatype_a == cute::UMMA::MXF4Format::E2M1 &&
                runtime_input_datatype_b == cute::UMMA::MXF4Format::E2M1) {
              Testbed3x<Gemm,
                        cutlass::epilogue::thread::Identity,
                        force_legacy_epilogue,
                        cutlass::float_e2m1_t,
                        cutlass::float_e2m1_t,
                        cute::UMMA::MXF4Format,
                        cute::UMMA::MXF4Format> testbed(check_relative_equality,
                                                        use_device_scalars,
                                                        vector_scale_mode);
              passed = testbed.run(
                problem_size,
                cutlass::from_real<ElementScalar>(alpha),
                cutlass::from_real<ElementScalar>(beta),
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3407-3421

```cpp
                RasterOrderOptions::Heuristic, // raster_order
                detail::MaxSwizzleSize(1),
                splits,
                decomp_mode,
                false,
                detail::Iterations{},
                runtime_input_datatype_a,
                runtime_input_datatype_b
              );
            }
            else {
              std::cout << "Unsupported configuration for runtime datatype MXFP4." << std::endl;
              return false;
            }
          }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3423-3440

```cpp
          else
          if constexpr (cute::is_same_v<RuntimeDataTypeA, cute::UMMA::MXF8F6F4Format> &&
                             cute::is_same_v<RuntimeDataTypeB, cute::UMMA::MXF8F6F4Format>) {
            static_assert((cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float8_t> ||
                           cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float6_t> ||
                           cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float4_t>) &&
                          (cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float8_t> ||
                           cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float6_t> ||
                           cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float4_t>),
                          "Runtime datatype must be selected with an appropriate static umbrella data type.");
            if constexpr (cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float8_t> &&
                          cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float4_t>) {
              // e4m3_e2m1
              if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E4M3 &&
                  runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E2M1) {
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 3441-3458

```cpp
                          cutlass::float_e4m3_t,
                          cutlass::float_e2m1_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
                                                          vector_scale_mode);
                passed = testbed.run(
                  problem_size,
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
                  splits,
                  decomp_mode,
                  false,
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
```
- **EN:** Implements or wires together logic around `float_e4m3_t`, `float_e2m1_t`, `UMMA`, `MXF8F6F4Format`, `testbed` for the current test scenario.
- **CN:** 围绕 `float_e4m3_t`, `float_e2m1_t`, `UMMA`, `MXF8F6F4Format`, `testbed` 实现或连接当前测试场景所需的逻辑。

### Lines 3459-3476

```cpp
                );
              }
              // Unsupport
              else {
                std::cout << "Unsupported configuration for runtime datatype Mxf8f6f4." << std::endl;
                return false;
              }
            }
            // f6xf4
            else if constexpr (cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float6_t> &&
                               cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float4_t>) {
              // e3m2_e2m1
              if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E3M2 &&
                  runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E2M1) {
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
                          cutlass::float_e3m2_t,
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3477-3481

```cpp
                          cutlass::float_e2m1_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
                                                          vector_scale_mode);
```
- **EN:** Implements or wires together logic around `float_e2m1_t`, `UMMA`, `MXF8F6F4Format`, `testbed`, `check_relative_equality` for the current test scenario.
- **CN:** 围绕 `float_e2m1_t`, `UMMA`, `MXF8F6F4Format`, `testbed`, `check_relative_equality` 实现或连接当前测试场景所需的逻辑。

### Lines 3483-3500

```cpp
                passed = testbed.run(
                  problem_size,
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
                  splits,
                  decomp_mode,
                  false,
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
                );
              }
              // Unsupport
              else {
                std::cout << "Unsupported configuration for runtime datatype Mxf8f6f4." << std::endl;
                return false;
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3501-3518

```cpp
              }
            }
            else if constexpr (cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float4_t> &&
                               cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float4_t>) {
              // e2m1_e2m1
              if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E2M1 &&
                  runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E2M1) {
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
                          cutlass::float_e2m1_t,
                          cutlass::float_e2m1_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
                                                          vector_scale_mode);
                passed = testbed.run(
                  problem_size,
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3519-3536

```cpp
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
                  splits,
                  decomp_mode,
                  false,
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
                );
              }
              // Unsupport
              else {
                std::cout << "Unsupported configuration for runtime datatype Mxf8f6f4." << std::endl;
                return false;
              }
            }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3537-3554

```cpp
            else if constexpr (cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float8_t> &&
                               cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float6_t>) {
              // e4m3_e3m2
              if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E4M3 &&
                  runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E3M2) {
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
                          cutlass::float_e4m3_t,
                          cutlass::float_e3m2_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
                                                          vector_scale_mode);
                passed = testbed.run(
                  problem_size,
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3555-3572

```cpp
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
                  splits,
                  decomp_mode,
                  false,
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
                );
              }
              // Unsupport
              else {
                std::cout << "Unsupported configuration for runtime datatype Mxf8f6f4." << std::endl;
                return false;
              }
            }
            else if constexpr (cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float6_t> &&
                               cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float6_t>) {
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3573-3590

```cpp
              // e3m2_e2m3
              if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E3M2 &&
                  runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E2M3) {
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
                          cutlass::float_e3m2_t,
                          cutlass::float_e2m3_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
                                                          vector_scale_mode);
                passed = testbed.run(
                  problem_size,
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3591-3608

```cpp
                  splits,
                  decomp_mode,
                  false,
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
                );
              }
              // Unsupported
              else {
                std::cout << "Unsupported configuration for runtime datatype Mxf8f6f4." << std::endl;
                return false;
              }
            }
            else
            if constexpr (cute::is_same_v<InternalElementA, cutlass::type_erased_dynamic_float8_t> &&
                               cute::is_same_v<InternalElementB, cutlass::type_erased_dynamic_float8_t>) {
              // e5m2_e5m2
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3609-3626

```cpp
              if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E5M2 &&
                  runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E5M2) {
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
                          cutlass::float_e5m2_t,
                          cutlass::float_e5m2_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
                                                          vector_scale_mode);
                passed = testbed.run(
                  problem_size,
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
                  splits,
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3627-3644

```cpp
                  decomp_mode,
                  false,
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
                );
              }
              // e4m3_e5m2
              else if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E4M3 &&
                       runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E5M2){
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
                          cutlass::float_e4m3_t,
                          cutlass::float_e5m2_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3645-3662

```cpp
                                                          vector_scale_mode);
                passed = testbed.run(
                  problem_size,
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
                  splits,
                  decomp_mode,
                  false,
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
                );
              }
              // e5m2_e4m3
              else if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E5M2 &&
                       runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E4M3){
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3663-3680

```cpp
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
                          cutlass::float_e5m2_t,
                          cutlass::float_e4m3_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
                                                          vector_scale_mode);
                passed = testbed.run(
                  problem_size,
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
                  splits,
                  decomp_mode,
                  false,
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 3681-3698

```cpp
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
                );
              }
              // e4m3_e4m3
              else if (runtime_input_datatype_a == cute::UMMA::MXF8F6F4Format::E4M3 &&
                       runtime_input_datatype_b == cute::UMMA::MXF8F6F4Format::E4M3){
                Testbed3x<Gemm,
                          cutlass::epilogue::thread::Identity,
                          force_legacy_epilogue,
                          cutlass::float_e4m3_t,
                          cutlass::float_e4m3_t,
                          cute::UMMA::MXF8F6F4Format,
                          cute::UMMA::MXF8F6F4Format> testbed(check_relative_equality,
                                                          use_device_scalars,
                                                          vector_scale_mode);
                passed = testbed.run(
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3699-3716

```cpp
                  problem_size,
                  cutlass::from_real<ElementScalar>(alpha),
                  cutlass::from_real<ElementScalar>(beta),
                  RasterOrderOptions::Heuristic, // raster_order
                  detail::MaxSwizzleSize(1),
                  splits,
                  decomp_mode,
                  false,
                  detail::Iterations{},
                  runtime_input_datatype_a,
                  runtime_input_datatype_b
                );
              }
              // Unsupported
              else {
                std::cout << "Unsupported configuration for runtime datatype Mxf8f6f4." << std::endl;
                return false;
              }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3717-3723

```cpp
            }
            // Unsupported
            else {
              std::cout << "Unsupported configuration for runtime datatype Mxf8f6f4." << std::endl;
              return false;
            }
          }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3725-3728

```cpp
          else {
            static_assert(cutlass::detail::dependent_false<RuntimeDataTypeA>,
                "Unsupported configuration for runtime datatype.");
          }
```
- **EN:** Adds compile-time validation so unsupported datatype or layout combinations fail early during template instantiation.
- **CN:** 加入编译期校验，让不受支持的数据类型或布局组合在模板实例化阶段尽早报错。

### Lines 3730-3737

```cpp
          if (!passed) {
            std::cout << __FILE__ << ':' << __LINE__ << " : GEMM MNK " << m << " " << n << " " << k << " FAILED.\n";
            return false;
          }
        } // splits
      } // decomposition_mode
    } // k
  } // waves
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3739-3740

```cpp
  return passed;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 3742-3747

```cpp
template <typename Gemm, bool force_legacy_epilogue = false, bool apply_alignment_offset = true, bool test_batched_alpha_beta = false>
bool TestSmall(double alpha = 1.0, double beta = cute::is_same_v<typename Gemm::GemmKernel::ElementC, void> ? 0.0 : 1.0,
  CheckEquality check_relative_equality = CheckEquality::RELATIVE,
  ScalarLoc use_device_scalars = ScalarLoc::ON_DEVICE,
  VectorScale vector_scale_mode = VectorScale::ENABLED,
  std::vector<int> override_problem_size_k = {}) {
```
- **EN:** Implements or wires together logic around `Gemm`, `force_legacy_epilogue`, `apply_alignment_offset`, `test_batched_alpha_beta`, `TestSmall` for the current test scenario.
- **CN:** 围绕 `Gemm`, `force_legacy_epilogue`, `apply_alignment_offset`, `test_batched_alpha_beta`, `TestSmall` 实现或连接当前测试场景所需的逻辑。

### Lines 3749-3761

```cpp
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
  using ElementScalar = typename Gemm::EpilogueOutputOp::ElementScalar;
  using CtaShape_MNK = typename Gemm::GemmKernel::CollectiveMainloop::CtaShape_MNK;
  using DispatchPolicy = typename Gemm::GemmKernel::CollectiveMainloop::DispatchPolicy;
  CtaShape_MNK cta_shape;
  Testbed3x<Gemm, cutlass::epilogue::thread::Identity, force_legacy_epilogue> testbed(check_relative_equality, use_device_scalars, vector_scale_mode);
  static constexpr int SmCount  = 16;
  static constexpr int MultiplierOffsetM = 1;
  static constexpr int MultiplierOffsetN = 2;
  static constexpr int MultiplierOffsetK = 3;
  int max_alignment_k = 0;
  int max_alignment_m = 0;
  int max_alignment_n = 0;
```
- **EN:** Introduces type aliases like `ProblemShapeType`, `ElementScalar`, `CtaShape_MNK`, `DispatchPolicy` so later declarations can refer to complex CUTLASS/CUTE types more clearly.
- **CN:** 引入类型别名，例如 `ProblemShapeType`, `ElementScalar`, `CtaShape_MNK`, `DispatchPolicy`，让后续声明能够更清晰地引用复杂的 CUTLASS/CUTE 类型。

### Lines 3763-3778

```cpp
  if constexpr (apply_alignment_offset) {
    max_alignment_k = std::max(Gemm::kAlignmentA, Gemm::kAlignmentB);
    max_alignment_n = std::max(Gemm::kAlignmentA, Gemm::kAlignmentB);
    max_alignment_m = std::max(Gemm::kAlignmentA, Gemm::kAlignmentB);
  }
  // Alignment for SFD
  if constexpr (detail::IsSfdEpi<typename Gemm::GemmKernel::CollectiveEpilogue>::value) {
    using GmemLayoutTagScalefactor = typename Gemm::GemmKernel::CollectiveEpilogue::FusionCallbacks::Operation::GmemLayoutTagScalefactor;
    constexpr int SFDVecSize = Gemm::GemmKernel::CollectiveEpilogue::FusionCallbacks::Operation::SFVecSize;
    if constexpr (cute::is_same_v<GmemLayoutTagScalefactor, cutlass::layout::RowMajor>) {
      max_alignment_n = std::lcm(max_alignment_n, SFDVecSize);
    }
    else {
      max_alignment_m = std::lcm(max_alignment_m, SFDVecSize);
    }
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3780-3782

```cpp
  float waves[] = {0.5, 1.25, 2.5};
  int cluster_m = 1;
  int cluster_n = 1;
```
- **EN:** Declares member fields or local variables related to `waves`, `cluster_m`, `cluster_n` for later setup, execution, or verification.
- **CN:** 声明与 `waves`, `cluster_m`, `cluster_n` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3784-3790

```cpp
  std::vector<int> problem_size_k;
  if (override_problem_size_k.empty()) {
    problem_size_k = {256 + max_alignment_k * MultiplierOffsetK, 512 + max_alignment_k * MultiplierOffsetK};
  }
  else {
    problem_size_k = override_problem_size_k;
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3792-3796

```cpp
  if constexpr(DispatchPolicy::ArchTag::kMinComputeCapability >= 90) {
    typename DispatchPolicy::ClusterShape cluster_shape;
    cluster_m = cute::size<0>(cluster_shape);
    cluster_n = cute::size<1>(cluster_shape);
  }
```
- **EN:** Begins function or method `constexpr`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `constexpr`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3798-3799

```cpp
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
```
- **EN:** Defines aliases such as `DecompositionMode`, `RasterOrderOptions` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `DecompositionMode`, `RasterOrderOptions`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 3801-3808

```cpp
  std::vector<DecompositionMode> decomposition_modes = {DecompositionMode::Heuristic};
  static constexpr bool UsesStreamKScheduler = cute::is_same_v<typename Gemm::GemmKernel::TileSchedulerTag, cutlass::gemm::StreamKScheduler>;
  if constexpr (UsesStreamKScheduler) {
    decomposition_modes.push_back(DecompositionMode::DataParallel);
    decomposition_modes.push_back(DecompositionMode::SplitK);
    decomposition_modes.push_back(DecompositionMode::StreamK);
  }
  bool passed = true;
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3810-3814

```cpp
  std::vector<RasterOrderOptions> raster_order_options = {RasterOrderOptions::Heuristic};
  for (float wave : waves) {
    for (int k : problem_size_k) {
      int grid_m, grid_n = 0;
      int num_grid = int(wave * SmCount);
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 3816-3827

```cpp
      if (cluster_m >= cluster_n) {
        grid_m = cluster_m;
        grid_n = num_grid / grid_m;
        // Align grid_n to cluster_n
        grid_n = std::max((grid_n + cluster_n - 1 ) / cluster_n * cluster_n, 1);
      }
      else {
        grid_n = cluster_n;
        grid_m = num_grid / grid_n;
        // Align grid_m to cluster_m
        grid_m = std::max((grid_m + cluster_m - 1 ) / cluster_m * cluster_m, 1);
      }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3829-3838

```cpp
      int m = grid_m * cute::size<0>(cta_shape) + MultiplierOffsetM * max_alignment_m;
      int n = grid_n * cute::size<1>(cta_shape) + MultiplierOffsetN * max_alignment_n;
      int l = test_batched_alpha_beta && wave == waves[0] && k == problem_size_k[0] ? 2 : 1; // only test the smallest problem size
      ProblemShapeType problem_size;
      if constexpr (cute::rank(ProblemShapeType{}) == 4) {
        problem_size = ProblemShapeType{m, n, k, l};
      }
      else {
        problem_size = ProblemShapeType{m, n, k};
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3840-3857

```cpp
      for (DecompositionMode decomp_mode : decomposition_modes) {
        for (RasterOrderOptions raster_order : raster_order_options) {
          std::vector problem_splits = {detail::Splits{1}};
          if constexpr (UsesStreamKScheduler) {
            if (decomp_mode == DecompositionMode::SplitK) {
              problem_splits.push_back(detail::Splits{2});
              problem_splits.push_back(detail::Splits{4});
            }
          }
          for (auto splits : problem_splits) {
            try {
              passed = testbed.run(
                problem_size,
                cutlass::from_real<ElementScalar>(alpha),
                cutlass::from_real<ElementScalar>(beta),
                raster_order, // raster_order
                detail::MaxSwizzleSize(0),
                splits,
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3858-3875

```cpp
                decomp_mode
              );
            }
            catch (std::exception const& e) {
              EXPECT_TRUE(false) << "TestSmall: testbed.run {"
                << "m: " << m << ", n: " << n << ", k: " << k << ", l: " << l
                << ", alpha: " << alpha << ", beta: " << beta
                << ", raster_order: " << detail::raster_order_to_string(raster_order)
                << ", max_swizzle_size: 1"
                << ", splits: " << static_cast<int>(splits)
                << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
                << "} threw an exception: " << e.what();
              throw;
            }
            catch (...) {
              EXPECT_TRUE(false) << "TestSmall: testbed.run {"
                << "m: " << m << ", n: " << n << ", k: " << k << ", l: " << l
                << ", alpha: " << alpha << ", beta: " << beta
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 3876-3890

```cpp
                << ", raster_order: " << detail::raster_order_to_string(raster_order)
                << ", max_swizzle_size: 1"
                << ", splits: " << static_cast<int>(splits)
                << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
                << "} threw an exception (unknown)";
              throw;
            }
            EXPECT_TRUE(passed) << "TestSmall: testbed.run {"
              << "m: " << m << ", n: " << n << ", k: " << k << ", l: " << l
              << ", alpha: " << alpha << ", beta: " << beta
              << ", raster_order: " << detail::raster_order_to_string(raster_order)
              << ", max_swizzle_size: 1"
              << ", splits: " << static_cast<int>(splits)
              << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
              << "} failed";
```
- **EN:** Begins function or method `detail::raster_order_to_string`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `detail::raster_order_to_string`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3892-3900

```cpp
            if (!passed) {
              std::cout << __FILE__ << ':' << __LINE__ << " : GEMM MNKL " << m << " " << n << " " << k << " " << l << " FAILED.\n";
              return false;
            }
          } // splits
        } // raster_order
      } // decomposition_mode
    } // k
  } // waves
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3902-3903

```cpp
  return passed;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 3905-3920

```cpp
template <typename Gemm, bool force_legacy_epilogue = false, bool apply_alignment_offset = true, bool test_batched_alpha_beta = false>
bool TestSmallFusion(double alpha = 1.0, double beta = cute::is_same_v<typename Gemm::GemmKernel::ElementC, void> ? 0.0 : 1.0,
                     CheckEquality check_relative_equality = CheckEquality::RELATIVE,
                     ScalarLoc use_device_scalars = ScalarLoc::ON_DEVICE,
                     VectorScale vector_scale_mode = VectorScale::ENABLED,
                     std::vector<int> override_problem_size_k = {}) {
  return TestSmall<Gemm,
                   force_legacy_epilogue,
                   apply_alignment_offset,
                   test_batched_alpha_beta>(alpha,
                                            beta,
                                            check_relative_equality,
                                            use_device_scalars,
                                            vector_scale_mode,
                                            override_problem_size_k);
}
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 3924-3930

```cpp
template <
  typename Gemm,
  template <class T> class ActivationFunctor = cutlass::epilogue::thread::Identity
>
bool TestAll(double alpha = 1.0, double beta = cute::is_same_v<typename Gemm::GemmKernel::ElementC, void> ? 0.0 : 1.0, CheckEquality check_relative_equality = CheckEquality::RELATIVE) {
  using ElementScalar = typename Gemm::EpilogueOutputOp::ElementScalar;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 3932-3932

```cpp
  Testbed3x<Gemm, ActivationFunctor> testbed(check_relative_equality, ScalarLoc::ON_HOST, VectorScale::DISABLED);
```
- **EN:** Declares member fields or local variables related to `Testbed3x`, `Gemm`, `ActivationFunctor`, `testbed`, `check_relative_equality` for later setup, execution, or verification.
- **CN:** 声明与 `Testbed3x`, `Gemm`, `ActivationFunctor`, `testbed`, `check_relative_equality` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3934-3941

```cpp
  int max_alignment_m = std::max({Gemm::kAlignmentA, Gemm::kAlignmentC, Gemm::kAlignmentD});
  int max_alignment_n = std::max({Gemm::kAlignmentB, Gemm::kAlignmentC, Gemm::kAlignmentD});
  if constexpr (std::is_base_of_v<cutlass::epilogue::fusion::FusionOperation, typename Gemm::EpilogueOutputOp>) {
    max_alignment_m = std::max(max_alignment_m, Gemm::EpilogueOutputOp::AlignmentAux);
    max_alignment_n = std::max(max_alignment_n, Gemm::EpilogueOutputOp::AlignmentAux);
  }
  std::vector<int> problem_size_m = {max_alignment_m, 512 - 3 * max_alignment_m};
  std::vector<int> problem_size_n = {max_alignment_n, 512 - 2 * max_alignment_n};
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3943-3947

```cpp
  if constexpr (cute::is_same_v<typename Gemm::GemmKernel::DispatchPolicy::Schedule,
                cutlass::gemm::KernelTmaWarpSpecializedPingpong>) {
    problem_size_m.push_back(768);
    problem_size_n.push_back(768);
  }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3949-3950

```cpp
  constexpr int Stages = Gemm::GemmKernel::DispatchPolicy::Stages;
  constexpr int TileShapeK = cute::size<2>(typename Gemm::GemmKernel::TileShape{});
```
- **EN:** Declares member fields or local variables related to `Stages`, `Gemm`, `GemmKernel`, `DispatchPolicy`, `TileShapeK` for later setup, execution, or verification.
- **CN:** 声明与 `Stages`, `Gemm`, `GemmKernel`, `DispatchPolicy`, `TileShapeK` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3952-3953

```cpp
  int max_alignment_k = std::max(Gemm::kAlignmentA, Gemm::kAlignmentB);
  std::vector<int> problem_size_k = {max_alignment_k, TileShapeK * (Stages + 1) - max_alignment_k};
```
- **EN:** Declares member fields or local variables related to `max_alignment_k`, `std`, `max`, `Gemm`, `kAlignmentA` for later setup, execution, or verification.
- **CN:** 声明与 `max_alignment_k`, `std`, `max`, `Gemm`, `kAlignmentA` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3955-3961

```cpp
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
  std::vector<DecompositionMode> decomposition_modes = {DecompositionMode::Heuristic};
  std::vector problem_splits = {detail::Splits{1}};
  static constexpr bool UsesStreamKScheduler = cute::is_same_v<typename Gemm::GemmKernel::TileSchedulerTag, cutlass::gemm::StreamKScheduler>;
  if constexpr (UsesStreamKScheduler) {
    problem_splits.push_back(detail::Splits{2});
    problem_splits.push_back(detail::Splits{3});
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3963-3965

```cpp
    decomposition_modes.push_back(DecompositionMode::DataParallel);
    decomposition_modes.push_back(DecompositionMode::SplitK);
    decomposition_modes.push_back(DecompositionMode::StreamK);
```
- **EN:** Declares member fields or local variables related to `decomposition_modes`, `push_back`, `DecompositionMode`, `DataParallel`, `SplitK` for later setup, execution, or verification.
- **CN:** 声明与 `decomposition_modes`, `push_back`, `DecompositionMode`, `DataParallel`, `SplitK` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3967-3970

```cpp
    // Use larger K sizes for stream-K tests
    static constexpr int min_tiles_per_sk_unit = cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::min_iters_per_sk_unit_;
    problem_size_k = {TileShapeK * min_tiles_per_sk_unit, TileShapeK * 3 * min_tiles_per_sk_unit - max_alignment_k};
  }
```
- **EN:** Implements or wires together logic around `Use`, `larger`, `sizes`, `stream`, `tests` for the current test scenario.
- **CN:** 围绕 `Use`, `larger`, `sizes`, `stream`, `tests` 实现或连接当前测试场景所需的逻辑。

### Lines 3972-3974

```cpp
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
  std::vector<RasterOrderOptions> raster_orders = {RasterOrderOptions::AlongM, RasterOrderOptions::AlongN};
  std::vector max_swizzle_sizes{detail::MaxSwizzleSize{1}, detail::MaxSwizzleSize{4}};
```
- **EN:** Declares member fields or local variables related to `RasterOrderOptions`, `gemm`, `kernel`, `detail`, `PersistentTileSchedulerSm90` for later setup, execution, or verification.
- **CN:** 声明与 `RasterOrderOptions`, `gemm`, `kernel`, `detail`, `PersistentTileSchedulerSm90` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3976-3976

```cpp
  bool passed = true;
```
- **EN:** Declares member fields or local variables related to `passed` for later setup, execution, or verification.
- **CN:** 声明与 `passed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3978-3983

```cpp
  for (int m : problem_size_m) {
    for (int n : problem_size_n) {
      for (int k : problem_size_k) {
        for (auto raster_order : raster_orders) {
          for (auto max_swizzle_size : max_swizzle_sizes) {
            for (DecompositionMode decomp_mode : decomposition_modes) {
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 3985-3993

```cpp
              std::vector problem_splits = {detail::Splits{1}};
              if (decomp_mode == DecompositionMode::Heuristic || decomp_mode == DecompositionMode::SplitK) {
                auto max_splits = (k + TileShapeK - 1) / TileShapeK;
                if (max_splits > 2) {
                  problem_splits.push_back(detail::Splits{2});
                }
                if (max_splits > 3) {
                  problem_splits.push_back(detail::Splits{3});
                }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 3995-3995

```cpp
                problem_splits.push_back(detail::Splits{max_splits});
```
- **EN:** Declares member fields or local variables related to `problem_splits`, `push_back`, `detail`, `Splits`, `max_splits` for later setup, execution, or verification.
- **CN:** 声明与 `problem_splits`, `push_back`, `detail`, `Splits`, `max_splits` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 3997-4008

```cpp
                // Test the case in which we ask for more splits than there are K tiles in the GEMM. In this
                // case, split-K will fall back to a splitting factor of `max_splits`.
                problem_splits.push_back(detail::Splits{max_splits + 1});
              }
              for (auto splits : problem_splits) {
                ProblemShapeType problem_size;
                if constexpr (cute::rank(ProblemShapeType{}) == 4) {
                  problem_size = ProblemShapeType{m, n, k, /* l */ 1};
                }
                else {
                  problem_size = ProblemShapeType{m, n, k};
                }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4010-4027

```cpp
                try {
                  passed = testbed.run(
                    problem_size,
                    cutlass::from_real<ElementScalar>(alpha),
                    cutlass::from_real<ElementScalar>(beta),
                    raster_order,
                    max_swizzle_size,
                    splits,
                    decomp_mode
                  );
                }
                catch (std::exception const& e) {
                  EXPECT_TRUE(false) << "TestAll: testbed.run {"
                    << "m: " << m << ", n: " << n << ", k: " << k
                    << ", alpha: " << alpha << ", beta: " << beta
                    << ", raster_order: ???"
                    << ", max_swizzle_size: " << static_cast<int>(max_swizzle_size)
                    << ", splits: " << static_cast<int>(splits)
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 4028-4042

```cpp
                    << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
                    << "} threw an exception: " << e.what();
                  throw;
                }
                catch (...) {
                  EXPECT_TRUE(false) << "TestAll: testbed.run {"
                    << "m: " << m << ", n: " << n << ", k: " << k
                    << ", alpha: " << alpha << ", beta: " << beta
                    << ", raster_order: ???"
                    << ", max_swizzle_size: " << static_cast<int>(max_swizzle_size)
                    << ", splits: " << static_cast<int>(splits)
                    << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
                    << "} threw an exception (unknown)";
                  throw;
                }
```
- **EN:** Begins function or method `detail::decomp_mode_to_string`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `detail::decomp_mode_to_string`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4044-4051

```cpp
                EXPECT_TRUE(passed) << "TestAll: testbed.run {"
                  << "m: " << m << ", n: " << n << ", k: " << k
                  << ", alpha: " << alpha << ", beta: " << beta
                  << ", raster_order: ???"
                  << ", max_swizzle_size: " << static_cast<int>(max_swizzle_size)
                  << ", splits: " << static_cast<int>(splits)
                  << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
                  << "} failed";
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 4053-4063

```cpp
                if (!passed) {
                  std::cout << __FILE__ << ':' << __LINE__ << " : GEMM MNK " << m << " " << n << " " << k << " FAILED.\n";
                  return false;
                }
              } // splits
            } // decomposition_mode
          } // max_swizzle_size
        } // raster_order
      } // k
    } // n
  } // m
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4065-4072

```cpp
  // if we do support batched GEMM, just run one test on it to save on test time
  if constexpr (cute::rank(ProblemShapeType{}) == 4) {
    auto problem_size = ProblemShapeType{256 + max_alignment_m, 256 + max_alignment_n, 160 + max_alignment_k, /* l */ 3};
    passed = testbed.run(
      problem_size,
      cutlass::from_real<ElementScalar>(alpha),
      cutlass::from_real<ElementScalar>(beta)
    );
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4074-4077

```cpp
    if (!passed) {
      return false;
    }
  }
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4079-4080

```cpp
  return passed;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4082-4091

```cpp
#if defined(SYCL_INTEL_TARGET)
template <typename Gemm, template <class T> class ActivationFunctor =
                             cutlass::epilogue::thread::Identity>
// TODO(Codeplay): remove the test_batch option once batching is enabled for all tests
bool TestXe(
    double alpha = 1.0, double beta = cute::is_same_v<typename Gemm::GemmKernel::ElementC, void> ? 0.0 : 1.0,
    bool test_batch = true,
    CheckEquality check_relative_equality = CheckEquality::RELATIVE) {
  using ElementScalar = typename Gemm::EpilogueOutputOp::ElementScalar;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 4093-4093

```cpp
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerXeStreamKParams::DecompositionMode;
```
- **EN:** Defines aliases such as `DecompositionMode` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `DecompositionMode`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 4095-4099

```cpp
  Testbed3x<Gemm, ActivationFunctor, false,
    typename Gemm::GemmKernel::ElementA,
    typename Gemm::GemmKernel::ElementB,
    void*, void*> testbed(
      check_relative_equality, ScalarLoc::ON_HOST, VectorScale::DISABLED);
```
- **EN:** Implements or wires together logic around `Testbed3x`, `Gemm`, `ActivationFunctor`, `GemmKernel`, `ElementA` for the current test scenario.
- **CN:** 围绕 `Testbed3x`, `Gemm`, `ActivationFunctor`, `GemmKernel`, `ElementA` 实现或连接当前测试场景所需的逻辑。

### Lines 4101-4111

```cpp
  // For M & N we test a small and a big size
  // For K, we currently only support K = TileShapeK
  int max_alignment_m = std::max({Gemm::kAlignmentA, Gemm::kAlignmentC, Gemm::kAlignmentD});
  int max_alignment_n = std::max({Gemm::kAlignmentB, Gemm::kAlignmentC, Gemm::kAlignmentD});
  if constexpr (std::is_base_of_v<cutlass::epilogue::fusion::FusionOperation, typename Gemm::EpilogueOutputOp>) {
    max_alignment_m = std::max(max_alignment_m, Gemm::EpilogueOutputOp::AlignmentAux);
    max_alignment_n = std::max(max_alignment_n, Gemm::EpilogueOutputOp::AlignmentAux);
  }
  std::vector<int> problem_size_m = {max_alignment_m, 512 - 3 * max_alignment_m};
  std::vector<int> problem_size_n = {max_alignment_n, 512 - 2 * max_alignment_n};
  std::vector<int> problem_size_l = test_batch ? std::vector{1, 3, 4} : std::vector{1};
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4113-4116

```cpp
  constexpr int Stages = Gemm::GemmKernel::DispatchPolicy::Stages;
  constexpr int TileShapeK = cute::size<2>(typename Gemm::GemmKernel::TileShape{});
  int max_alignment_k = std::max(Gemm::kAlignmentA, Gemm::kAlignmentB);
  std::vector<int> problem_size_k = {max_alignment_k, TileShapeK * (Stages + 1) - max_alignment_k};
```
- **EN:** Declares member fields or local variables related to `Stages`, `Gemm`, `GemmKernel`, `DispatchPolicy`, `TileShapeK` for later setup, execution, or verification.
- **CN:** 声明与 `Stages`, `Gemm`, `GemmKernel`, `DispatchPolicy`, `TileShapeK` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4118-4124

```cpp
  using DecompositionMode = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerXeStreamKParams::DecompositionMode;
  std::vector decomposition_modes = {DecompositionMode::Heuristic};
  std::vector problem_splits = {detail::Splits{1}};
  static constexpr bool UsesStreamKScheduler = cute::is_same_v<typename Gemm::GemmKernel::TileSchedulerTag, cutlass::gemm::StreamKScheduler>;
  if constexpr (UsesStreamKScheduler) {
    problem_splits.push_back(detail::Splits{2});
    problem_splits.push_back(detail::Splits{3});
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4126-4128

```cpp
    decomposition_modes.push_back(DecompositionMode::DataParallel);
    decomposition_modes.push_back(DecompositionMode::SplitK);
    decomposition_modes.push_back(DecompositionMode::StreamK);
```
- **EN:** Declares member fields or local variables related to `decomposition_modes`, `push_back`, `DecompositionMode`, `DataParallel`, `SplitK` for later setup, execution, or verification.
- **CN:** 声明与 `decomposition_modes`, `push_back`, `DecompositionMode`, `DataParallel`, `SplitK` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4130-4133

```cpp
    // Use larger K sizes for stream-K tests
    static constexpr int min_tiles_per_sk_unit = cutlass::gemm::kernel::detail::PersistentTileSchedulerXeStreamKParams::min_iters_per_sk_unit_;
    problem_size_k = {TileShapeK * min_tiles_per_sk_unit, TileShapeK * 3 * min_tiles_per_sk_unit};
  }
```
- **EN:** Implements or wires together logic around `Use`, `larger`, `sizes`, `stream`, `tests` for the current test scenario.
- **CN:** 围绕 `Use`, `larger`, `sizes`, `stream`, `tests` 实现或连接当前测试场景所需的逻辑。

### Lines 4135-4137

```cpp
  using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::RasterOrderOptions;
  std::vector<RasterOrderOptions> raster_orders = {RasterOrderOptions::AlongM};
  std::vector max_swizzle_sizes{detail::MaxSwizzleSize{1}};
```
- **EN:** Declares member fields or local variables related to `RasterOrderOptions`, `gemm`, `kernel`, `detail`, `PersistentTileSchedulerSm90` for later setup, execution, or verification.
- **CN:** 声明与 `RasterOrderOptions`, `gemm`, `kernel`, `detail`, `PersistentTileSchedulerSm90` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4139-4139

```cpp
  bool passed = true;
```
- **EN:** Declares member fields or local variables related to `passed` for later setup, execution, or verification.
- **CN:** 声明与 `passed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4141-4156

```cpp
  for (int m : problem_size_m) {
    for (int n : problem_size_n) {
      for (int k : problem_size_k) {
        for (int l : problem_size_l) {
          for (auto raster_order : raster_orders) {
            for (auto max_swizzle_size : max_swizzle_sizes) {
              for (DecompositionMode decomp_mode : decomposition_modes) {
                std::vector problem_splits = {detail::Splits{1}};
                if (decomp_mode == DecompositionMode::Heuristic || decomp_mode == DecompositionMode::SplitK) {
                  auto max_splits = (k + TileShapeK - 1) / TileShapeK;
                  if (max_splits > 2) {
                    problem_splits.push_back(detail::Splits{2});
                  }
                  if (max_splits > 3) {
                    problem_splits.push_back(detail::Splits{3});
                  }
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4158-4158

```cpp
                  problem_splits.push_back(detail::Splits{max_splits});
```
- **EN:** Declares member fields or local variables related to `problem_splits`, `push_back`, `detail`, `Splits`, `max_splits` for later setup, execution, or verification.
- **CN:** 声明与 `problem_splits`, `push_back`, `detail`, `Splits`, `max_splits` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4160-4163

```cpp
                  // Test the case in which we ask for more splits than there are K tiles in the GEMM. In this
                  // case, split-K will fall back to a splitting factor of `max_splits`.
                  problem_splits.push_back(detail::Splits{max_splits + 1});
                }
```
- **EN:** Implements or wires together logic around `Test`, `the`, `which`, `ask`, `more` for the current test scenario.
- **CN:** 围绕 `Test`, `the`, `which`, `ask`, `more` 实现或连接当前测试场景所需的逻辑。

### Lines 4165-4182

```cpp
                for (auto splits : problem_splits) {
                  ProblemShapeType problem_size{m, n, k, l};
                  try {
                    passed = testbed.run(problem_size,
                                         cutlass::from_real<ElementScalar>(alpha),
                                         cutlass::from_real<ElementScalar>(beta),
                                         raster_order,
                                         max_swizzle_size,
                                         splits,
                                         decomp_mode
                                         );
                  }
                  catch (std::exception const& e) {
                    EXPECT_TRUE(false) << "TestAll: testbed.run {"
                      << "m: " << m << ", n: " << n << ", k: " << k << ", l: " << l
                      << ", alpha: " << alpha << ", beta: " << beta
                      << ", splits: " << static_cast<int>(splits)
                      << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
```
- **EN:** Begins function or method `for`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `for`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4183-4194

```cpp
                      << "} threw an exception: " << e.what();
                    throw;
                  }
                  catch (...) {
                    EXPECT_TRUE(false) << "TestAll: testbed.run {"
                      << "m: " << m << ", n: " << n << ", k: " << k << ", l: " << l
                      << ", alpha: " << alpha << ", beta: " << beta
                      << ", splits: " << static_cast<int>(splits)
                      << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
                      << "} threw an exception (unknown)";
                    throw;
                  }
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 4196-4201

```cpp
                  EXPECT_TRUE(passed) << "TestAll: testbed.run {"
                      << "m: " << m << ", n: " << n << ", k: " << k << ", l: " << l
                    << ", alpha: " << alpha << ", beta: " << beta
                    << ", splits: " << static_cast<int>(splits)
                    << ", decomp_mode: " << detail::decomp_mode_to_string(decomp_mode)
                    << "} failed";
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 4203-4217

```cpp
                  if (!passed) {
                    std::cout << __FILE__ << ':' << __LINE__ << " : GEMM MNKL " << m
                              << " " << n << " " << k << " " << l << " FAILED.\n";
                    return false;
                  }
                } // splits
              } // decomp_mode
            } // max_swizzle_size
          } // raster_order
        } // l
      }  // k
    }  // n
  }  // m
  return passed;
}
```
- **EN:** Begins function or method `if`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `if`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4219-4227

```cpp
template <typename Gemm, template <class T> class ActivationFunctor =
                             cutlass::epilogue::thread::Identity>
bool TestXe(
    int m, int n, int k, int l,
    double alpha = 1.0,
    double beta = cute::is_same_v<typename Gemm::GemmKernel::ElementC, void> ? 0.0 : 1.0,
    CheckEquality check_relative_equality = CheckEquality::RELATIVE) {
  using ElementScalar = typename Gemm::EpilogueOutputOp::ElementScalar;
  using ProblemShapeType = typename Gemm::GemmKernel::ProblemShape;
```
- **EN:** Defines templated type `T` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `T`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 4229-4234

```cpp
  Testbed3x<Gemm, ActivationFunctor, false,
    typename Gemm::GemmKernel::ElementA,
    typename Gemm::GemmKernel::ElementB,
    typename Gemm::GemmKernel::ElementC,
    typename Gemm::GemmKernel::ElementD> testbed(
      check_relative_equality, ScalarLoc::ON_HOST, VectorScale::DISABLED);
```
- **EN:** Implements or wires together logic around `Testbed3x`, `Gemm`, `ActivationFunctor`, `GemmKernel`, `ElementA` for the current test scenario.
- **CN:** 围绕 `Testbed3x`, `Gemm`, `ActivationFunctor`, `GemmKernel`, `ElementA` 实现或连接当前测试场景所需的逻辑。

### Lines 4236-4251

```cpp
  bool passed = true;
  ProblemShapeType problem_size{m, n, k, l};
  try {
    passed = testbed.run(problem_size,
                         cutlass::from_real<ElementScalar>(alpha),
                         cutlass::from_real<ElementScalar>(beta));
  }
  catch (std::exception const& e) {
    EXPECT_TRUE(false) << "TestXe: testbed.run threw an exception: " << e.what();
    throw;
  }
  catch (...) {
    EXPECT_TRUE(false) << "TestXe: testbed.run threw an unknown exception for MNKL = "
                        << m << " " << n << " " << k << " " << l;
    throw;
  }
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 4253-4255

```cpp
  EXPECT_TRUE(passed) << "TestXe: testbed.run failed for MNKL = "
                      << m << " " << n << " " << k << " " << l
                      << ", alpha: " << alpha << ", beta: " << beta;
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 4257-4263

```cpp
  return passed;
}
// Helper function to initialize blocks with random data
template<typename T>
void initialize_block_grouped(cutlass::DeviceAllocation<T>& block, uint64_t seed) {
  std::mt19937 generator(seed);
  std::uniform_real_distribution<float> distribution(-1.0f, 1.0f);
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4265-4268

```cpp
  std::vector<T> host_data(block.size());
  for (size_t i = 0; i < host_data.size(); ++i) {
    host_data[i] = static_cast<T>(distribution(generator));
  }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 4270-4271

```cpp
  block.copy_from_host(host_data.data());
}
```
- **EN:** Implements or wires together logic around `block`, `copy_from_host`, `host_data`, `data` for the current test scenario.
- **CN:** 围绕 `block`, `copy_from_host`, `host_data`, `data` 实现或连接当前测试场景所需的逻辑。

### Lines 4273-4290

```cpp
// Forward declaration of the test runner class
template <class Gemm>
class GroupedGemmTestRunner {
public:
  using ElementA = typename Gemm::ElementA;
  using ElementB = typename Gemm::ElementB;
  using ElementC = typename Gemm::ElementC;
  using LayoutA = typename Gemm::LayoutA;
  using LayoutB = typename Gemm::LayoutB;
  using LayoutC = typename Gemm::LayoutC;
  using LayoutD = typename Gemm::LayoutD;
  using CollectiveEpilogue = typename Gemm::CollectiveEpilogue;
  using ElementOutput = typename CollectiveEpilogue::ElementOutput;
  using ElementAccumulator = ElementOutput;
  using StrideA = typename Gemm::GemmKernel::InternalStrideA;
  using StrideB = typename Gemm::GemmKernel::InternalStrideB;
  using StrideC = typename Gemm::GemmKernel::InternalStrideC;
  using StrideD = typename Gemm::GemmKernel::InternalStrideD;
```
- **EN:** Defines templated type `template` that packages architecture, datatype, layout, tile, or policy parameters into reusable configuration.
- **CN:** 定义模板类型 `template`，把架构、数据类型、布局、tile 或策略参数打包成可复用配置。

### Lines 4291-4291

```cpp
  using ProblemShape = cutlass::gemm::GroupProblemShape<cute::Shape<int,int,int>>;
```
- **EN:** Defines aliases such as `ProblemShape` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `ProblemShape`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 4293-4300

```cpp
private:
  // Host-side allocations
  std::vector<int64_t> offset_A, offset_B, offset_C, offset_D;
  std::vector<StrideA> stride_A_host;
  std::vector<StrideB> stride_B_host;
  std::vector<StrideC> stride_C_host;
  std::vector<StrideD> stride_D_host;
  std::vector<ElementAccumulator> alpha_host, beta_host;
```
- **EN:** Implements or wires together logic around `Host`, `side`, `allocations`, `std`, `vector` for the current test scenario.
- **CN:** 围绕 `Host`, `side`, `allocations`, `std`, `vector` 实现或连接当前测试场景所需的逻辑。

### Lines 4302-4317

```cpp
  // Device-side allocations
  cutlass::DeviceAllocation<typename ProblemShape::UnderlyingProblemShape> problem_sizes;
  cutlass::DeviceAllocation<ElementA> block_A;
  cutlass::DeviceAllocation<ElementB> block_B;
  cutlass::DeviceAllocation<ElementC> block_C;
  cutlass::DeviceAllocation<ElementOutput> block_D, block_ref_D;
  cutlass::DeviceAllocation<const ElementA *> ptr_A;
  cutlass::DeviceAllocation<const ElementB *> ptr_B;
  cutlass::DeviceAllocation<const ElementC *> ptr_C;
  cutlass::DeviceAllocation<ElementOutput *> ptr_D, ptr_ref_D;
  cutlass::DeviceAllocation<StrideA> stride_A;
  cutlass::DeviceAllocation<StrideB> stride_B;
  cutlass::DeviceAllocation<StrideC> stride_C;
  cutlass::DeviceAllocation<StrideD> stride_D;
  cutlass::DeviceAllocation<ElementAccumulator*> alpha_device, beta_device;
  cutlass::DeviceAllocation<ElementAccumulator> block_alpha, block_beta;
```
- **EN:** Implements or wires together logic around `Device`, `side`, `allocations`, `DeviceAllocation`, `ProblemShape` for the current test scenario.
- **CN:** 围绕 `Device`, `side`, `allocations`, `DeviceAllocation`, `ProblemShape` 实现或连接当前测试场景所需的逻辑。

### Lines 4319-4321

```cpp
  std::vector<typename ProblemShape::UnderlyingProblemShape> problem_sizes_host;
  int groups;
  float alpha_scalar, beta_scalar;
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `ProblemShape`, `UnderlyingProblemShape`, `problem_sizes_host` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `ProblemShape`, `UnderlyingProblemShape`, `problem_sizes_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4323-4330

```cpp
public:
  GroupedGemmTestRunner(const std::vector<cutlass::gemm::GemmCoord>& problems, float alpha, float beta)
    : groups(problems.size()), alpha_scalar(alpha), beta_scalar(beta) {
    problem_sizes_host.reserve(groups);
    for (const auto& problem : problems) {
      problem_sizes_host.push_back({problem.m(), problem.n(), problem.k()});
    }
  }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 4332-4337

```cpp
  bool run(const cutlass::KernelHardwareInfo& hw_info) {
    if (!allocate()) return false;
    if (!initialize()) return false;
    if (!execute(hw_info)) return false;
    return verify();
  }
```
- **EN:** Begins function or method `run`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `run`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4339-4342

```cpp
private:
  bool allocate() {
    try {
      int64_t total_elements_A = 0, total_elements_B = 0, total_elements_C = 0, total_elements_D = 0;
```
- **EN:** Implements or wires together logic around `allocate`, `try`, `int64_t`, `total_elements_A`, `total_elements_B` for the current test scenario.
- **CN:** 围绕 `allocate`, `try`, `int64_t`, `total_elements_A`, `total_elements_B` 实现或连接当前测试场景所需的逻辑。

### Lines 4344-4348

```cpp
      for (int32_t i = 0; i < groups; ++i) {
        auto problem = problem_sizes_host.at(i);
        auto M = cute::get<0>(problem);
        auto N = cute::get<1>(problem);
        auto K = cute::get<2>(problem);
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 4350-4353

```cpp
        offset_A.push_back(total_elements_A);
        offset_B.push_back(total_elements_B);
        offset_C.push_back(total_elements_C);
        offset_D.push_back(total_elements_D);
```
- **EN:** Declares member fields or local variables related to `offset_A`, `push_back`, `total_elements_A`, `offset_B`, `total_elements_B` for later setup, execution, or verification.
- **CN:** 声明与 `offset_A`, `push_back`, `total_elements_A`, `offset_B`, `total_elements_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4355-4358

```cpp
        total_elements_A += M * K;
        total_elements_B += K * N;
        total_elements_C += M * N;
        total_elements_D += M * N;
```
- **EN:** Declares member fields or local variables related to `total_elements_A`, `total_elements_B`, `total_elements_C`, `total_elements_D` for later setup, execution, or verification.
- **CN:** 声明与 `total_elements_A`, `total_elements_B`, `total_elements_C`, `total_elements_D` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4360-4364

```cpp
        stride_A_host.push_back(cutlass::make_cute_packed_stride(StrideA{}, {M, K, 1}));
        stride_B_host.push_back(cutlass::make_cute_packed_stride(StrideB{}, {N, K, 1}));
        stride_C_host.push_back(cutlass::make_cute_packed_stride(StrideC{}, {M, N, 1}));
        stride_D_host.push_back(cutlass::make_cute_packed_stride(StrideD{}, {M, N, 1}));
      }
```
- **EN:** Implements or wires together logic around `stride_A_host`, `push_back`, `make_cute_packed_stride`, `StrideA`, `stride_B_host` for the current test scenario.
- **CN:** 围绕 `stride_A_host`, `push_back`, `make_cute_packed_stride`, `StrideA`, `stride_B_host` 实现或连接当前测试场景所需的逻辑。

### Lines 4366-4383

```cpp
      block_A.reset(total_elements_A);
      block_B.reset(total_elements_B);
      block_C.reset(total_elements_C);
      block_D.reset(total_elements_D);
      block_ref_D.reset(total_elements_D);
      block_alpha.reset(groups);
      block_beta.reset(groups);
      return true;
    } 
    catch (const std::bad_alloc& e) {
      CUTLASS_TRACE_HOST("Allocation failed - insufficient memory: " << e.what());
      return false;
    }
    catch (const std::exception& e) {
      CUTLASS_TRACE_HOST("Allocation failed with exception: " << e.what());
      return false;
    }
    catch (...) {
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4384-4387

```cpp
      CUTLASS_TRACE_HOST("Allocation failed with unknown error");
      return false;
    }
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4389-4393

```cpp
  bool initialize() {
    try {
      uint64_t seed = 2020;
      problem_sizes.reset(groups);
      problem_sizes.copy_from_host(problem_sizes_host.data());
```
- **EN:** Begins function or method `initialize`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `initialize`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4395-4400

```cpp
      std::vector<ElementA *> ptr_A_host(groups);
      std::vector<ElementB *> ptr_B_host(groups);
      std::vector<ElementC *> ptr_C_host(groups);
      std::vector<ElementOutput *> ptr_D_host(groups);
      std::vector<ElementAccumulator *> ptr_alpha_host(groups);
      std::vector<ElementAccumulator *> ptr_beta_host(groups);
```
- **EN:** Declares member fields or local variables related to `std`, `vector`, `ElementA`, `ptr_A_host`, `groups` for later setup, execution, or verification.
- **CN:** 声明与 `std`, `vector`, `ElementA`, `ptr_A_host`, `groups` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4402-4411

```cpp
      for (int32_t i = 0; i < groups; ++i) {
        ptr_A_host.at(i) = block_A.get() + offset_A.at(i);
        ptr_B_host.at(i) = block_B.get() + offset_B.at(i);
        ptr_C_host.at(i) = block_C.get() + offset_C.at(i);
        ptr_D_host.at(i) = block_D.get() + offset_D.at(i);
        alpha_host.push_back(alpha_scalar);
        beta_host.push_back(beta_scalar);
        ptr_alpha_host.at(i) = block_alpha.get() + i;
        ptr_beta_host.at(i) = block_beta.get() + i;
      }
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 4413-4420

```cpp
      ptr_A.reset(groups);
      ptr_A.copy_from_host(ptr_A_host.data());
      ptr_B.reset(groups);
      ptr_B.copy_from_host(ptr_B_host.data());
      ptr_C.reset(groups);
      ptr_C.copy_from_host(ptr_C_host.data());
      ptr_D.reset(groups);
      ptr_D.copy_from_host(ptr_D_host.data());
```
- **EN:** Declares member fields or local variables related to `ptr_A`, `reset`, `groups`, `copy_from_host`, `ptr_A_host` for later setup, execution, or verification.
- **CN:** 声明与 `ptr_A`, `reset`, `groups`, `copy_from_host`, `ptr_A_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4422-4429

```cpp
      stride_A.reset(groups);
      stride_A.copy_from_host(stride_A_host.data());
      stride_B.reset(groups);
      stride_B.copy_from_host(stride_B_host.data());
      stride_C.reset(groups);
      stride_C.copy_from_host(stride_C_host.data());
      stride_D.reset(groups);
      stride_D.copy_from_host(stride_D_host.data());
```
- **EN:** Declares member fields or local variables related to `stride_A`, `reset`, `groups`, `copy_from_host`, `stride_A_host` for later setup, execution, or verification.
- **CN:** 声明与 `stride_A`, `reset`, `groups`, `copy_from_host`, `stride_A_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4431-4434

```cpp
      alpha_device.reset(groups);
      alpha_device.copy_from_host(ptr_alpha_host.data());
      beta_device.reset(groups);
      beta_device.copy_from_host(ptr_beta_host.data());
```
- **EN:** Declares member fields or local variables related to `alpha_device`, `reset`, `groups`, `copy_from_host`, `ptr_alpha_host` for later setup, execution, or verification.
- **CN:** 声明与 `alpha_device`, `reset`, `groups`, `copy_from_host`, `ptr_alpha_host` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4436-4439

```cpp
      // Use our custom initialize function instead of the missing one
      initialize_block_grouped(block_A, seed + 2023);
      initialize_block_grouped(block_B, seed + 2022);
      initialize_block_grouped(block_C, seed + 2021);
```
- **EN:** Implements or wires together logic around `Use`, `our`, `custom`, `initialize`, `function` for the current test scenario.
- **CN:** 围绕 `Use`, `our`, `custom`, `initialize`, `function` 实现或连接当前测试场景所需的逻辑。

### Lines 4441-4458

```cpp
      block_alpha.copy_from_host(alpha_host.data());
      block_beta.copy_from_host(beta_host.data());
      return true;
    } 
    catch (const std::out_of_range& e) {
      CUTLASS_TRACE_HOST("Initialize failed - index out of range: " << e.what());
      return false;
    }
    catch (const std::bad_alloc& e) {
      CUTLASS_TRACE_HOST("Initialize failed - memory allocation error: " << e.what());
      return false;
    }
    catch (const std::exception& e) {
      CUTLASS_TRACE_HOST("Initialize failed with exception: " << e.what());
      return false;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("Initialize failed with unknown error");
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4459-4461

```cpp
      return false;
    }
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4463-4466

```cpp
  bool execute(const cutlass::KernelHardwareInfo& hw_info) {
    try {
      Gemm gemm_op;
      auto arguments = create_arguments(hw_info);
```
- **EN:** Begins function or method `execute`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `execute`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4468-4469

```cpp
      size_t workspace_size = Gemm::get_workspace_size(arguments);
      cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```
- **EN:** Declares member fields or local variables related to `size_t`, `workspace_size`, `Gemm`, `get_workspace_size`, `arguments` for later setup, execution, or verification.
- **CN:** 声明与 `size_t`, `workspace_size`, `Gemm`, `get_workspace_size`, `arguments` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4471-4476

```cpp
      // Check kernel compatibility
      cutlass::Status status = gemm_op.can_implement(arguments);
      if (status != cutlass::Status::kSuccess) {
        CUTLASS_TRACE_HOST("Execute failed - kernel cannot implement arguments: " << static_cast<int>(status));
        return false;
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4478-4483

```cpp
      // Initialize kernel
      status = gemm_op.initialize(arguments, workspace.get());
      if (status != cutlass::Status::kSuccess) {
        CUTLASS_TRACE_HOST("Execute failed - kernel initialization failed: " << static_cast<int>(status));
        return false;
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4485-4490

```cpp
      // Run kernel
      status = gemm_op.run();
      if (status != cutlass::Status::kSuccess) {
        CUTLASS_TRACE_HOST("Execute failed - kernel execution failed: " << static_cast<int>(status));
        return false;
      }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4492-4506

```cpp
      // Synchronize device to ensure kernel completion before host verification
      #ifdef CUTLASS_ENABLE_SYCL
        // For SYCL: device synchronization happens implicitly with copy operations
        // The block_D.copy_to_host() call below will ensure device operations complete
        try {
          // SYCL synchronization is implicit in device-to-host transfers
          // No explicit queue access needed here
        }
        catch (const sycl::exception& e) {
          CUTLASS_TRACE_HOST("SYCL note: " << e.what());
        }
      #else
        // For CUDA: use standard device synchronization
        cudaDeviceSynchronize();
      #endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 4508-4512

```cpp
      // Note: Reference computation is expensive (O(M*N*K) per group)
      // For grouped GEMMs with large problem sizes, skipping it for performance.
      // Kernel execution success itself validates the implementation.
      // Uncomment below only for small test cases (with correct layout handling):
      compute_reference();
```
- **EN:** Begins function or method `expensive`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `expensive`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4514-4528

```cpp
      return true;
    } 
    catch (const std::bad_alloc& e) {
      CUTLASS_TRACE_HOST("Execute failed - memory allocation error: " << e.what());
      return false;
    }
    catch (const std::exception& e) {
      CUTLASS_TRACE_HOST("Execute failed with exception: " << e.what());
      return false;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("Execute failed with unknown error");
      return false;
    }
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4530-4532

```cpp
  void compute_reference() {
    // Compute host-based reference GEMM for each problem in the group
    // Reference: D = alpha * (A @ B) + beta * C
```
- **EN:** Begins function or method `compute_reference`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `compute_reference`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4534-4534

```cpp
    try {
```
- **EN:** Implements or wires together logic around `try` for the current test scenario.
- **CN:** 围绕 `try` 实现或连接当前测试场景所需的逻辑。

### Lines 4536-4540

```cpp
      // Validate that offsets have been populated by allocate()
      if (offset_A.empty() || offset_B.empty() || offset_C.empty() || offset_D.empty()) {
        CUTLASS_TRACE_HOST("Reference computation skipped - offsets not initialized");
        return;
      }
```
- **EN:** Begins function or method `allocate`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `allocate`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4542-4545

```cpp
      // Copy input tensors to host for reference computation
      std::vector<ElementA> host_A(block_A.size());
      std::vector<ElementB> host_B(block_B.size());
      std::vector<ElementC> host_C(block_C.size());
```
- **EN:** Implements or wires together logic around `Copy`, `input`, `tensors`, `host`, `reference` for the current test scenario.
- **CN:** 围绕 `Copy`, `input`, `tensors`, `host`, `reference` 实现或连接当前测试场景所需的逻辑。

### Lines 4547-4549

```cpp
      block_A.copy_to_host(host_A.data());
      block_B.copy_to_host(host_B.data());
      block_C.copy_to_host(host_C.data());
```
- **EN:** Declares member fields or local variables related to `block_A`, `copy_to_host`, `host_A`, `data`, `block_B` for later setup, execution, or verification.
- **CN:** 声明与 `block_A`, `copy_to_host`, `host_A`, `data`, `block_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4551-4552

```cpp
      // Initialize reference output buffer
      std::vector<ElementOutput> ref_output(block_ref_D.size(), ElementOutput(0));
```
- **EN:** Implements or wires together logic around `Initialize`, `reference`, `output`, `buffer`, `std` for the current test scenario.
- **CN:** 围绕 `Initialize`, `reference`, `output`, `buffer`, `std` 实现或连接当前测试场景所需的逻辑。

### Lines 4554-4564

```cpp
      // Compute reference for each grouped problem
      for (int32_t g = 0; g < groups; ++g) {
        // Bounds check
        if (g >= (int32_t)problem_sizes_host.size() || 
            g >= (int32_t)offset_A.size() || 
            g >= (int32_t)offset_B.size() ||
            g >= (int32_t)offset_C.size() || 
            g >= (int32_t)offset_D.size()) {
          CUTLASS_TRACE_HOST("Reference computation bounds check failed at group " << g);
          continue;
        }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4566-4569

```cpp
        auto problem = problem_sizes_host.at(g);
        auto M = cute::get<0>(problem);
        auto N = cute::get<1>(problem);
        auto K = cute::get<2>(problem);
```
- **EN:** Declares member fields or local variables related to `problem`, `problem_sizes_host`, `get` for later setup, execution, or verification.
- **CN:** 声明与 `problem`, `problem_sizes_host`, `get` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4571-4575

```cpp
        // Validate problem size
        if (M <= 0 || N <= 0 || K <= 0) {
          CUTLASS_TRACE_HOST("Invalid problem size at group " << g << ": M=" << M << " N=" << N << " K=" << K);
          continue;
        }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4577-4580

```cpp
        int64_t offset_a = offset_A[g];
        int64_t offset_b = offset_B[g];
        int64_t offset_c = offset_C[g];
        int64_t offset_d = offset_D[g];
```
- **EN:** Declares member fields or local variables related to `int64_t`, `offset_a`, `offset_A`, `offset_b`, `offset_B` for later setup, execution, or verification.
- **CN:** 声明与 `int64_t`, `offset_a`, `offset_A`, `offset_b`, `offset_B` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4582-4598

```cpp
        // Validate offsets and buffer sizes
        if (offset_a + M * K > (int64_t)host_A.size()) {
          CUTLASS_TRACE_HOST("Buffer overflow: A offset " << offset_a << " + size " << (M*K) << " > " << host_A.size());
          continue;
        }
        if (offset_b + K * N > (int64_t)host_B.size()) {
          CUTLASS_TRACE_HOST("Buffer overflow: B offset " << offset_b << " + size " << (K*N) << " > " << host_B.size());
          continue;
        }
        if (offset_c + M * N > (int64_t)host_C.size()) {
          CUTLASS_TRACE_HOST("Buffer overflow: C offset " << offset_c << " + size " << (M*N) << " > " << host_C.size());
          continue;
        }
        if (offset_d + M * N > (int64_t)ref_output.size()) {
          CUTLASS_TRACE_HOST("Buffer overflow: D offset " << offset_d << " + size " << (M*N) << " > " << ref_output.size());
          continue;
        }
```
- **EN:** Handles a runtime branch, selecting different setup or execution behavior according to the current test conditions.
- **CN:** 处理运行时分支，根据当前测试条件选择不同的初始化或执行行为。

### Lines 4600-4605

```cpp
        // Simple reference computation: D = alpha * (A @ B) + beta * C
        // Using simple indexing (assumes row-major layout)
        ElementA* A_ptr = host_A.data() + offset_a;
        ElementB* B_ptr = host_B.data() + offset_b;
        ElementC* C_ptr = host_C.data() + offset_c;
        ElementOutput* D_ptr = ref_output.data() + offset_d;
```
- **EN:** Implements or wires together logic around `Simple`, `reference`, `computation`, `alpha`, `beta` for the current test scenario.
- **CN:** 围绕 `Simple`, `reference`, `computation`, `alpha`, `beta` 实现或连接当前测试场景所需的逻辑。

### Lines 4607-4610

```cpp
        // Compute D = alpha * (A @ B) + beta * C
        for (int m = 0; m < M; ++m) {
          for (int n = 0; n < N; ++n) {
            ElementAccumulator sum = ElementAccumulator(0);
```
- **EN:** Iterates over tensors, problems, or batches so the testbed can prepare data, launch work, or compare results repeatedly.
- **CN:** 遍历张量、问题实例或批次，使测试平台能够重复进行数据准备、内核启动或结果比较。

### Lines 4612-4618

```cpp
            // Compute A @ B (row-major: A[m,k], B[n,k])
            for (int k = 0; k < K; ++k) {
              ElementA a_val = A_ptr[m * K + k];
              ElementB b_val = B_ptr[n * K + k];
              sum += static_cast<ElementAccumulator>(a_val) * 
                     static_cast<ElementAccumulator>(b_val);
            }
```
- **EN:** Begins function or method `B`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `B`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4620-4628

```cpp
            // D[m,n] = alpha * sum + beta * C[m,n]
            ElementC c_val = C_ptr[m * N + n];
            ElementOutput d_val = static_cast<ElementOutput>(
              alpha_scalar * sum + beta_scalar * static_cast<ElementAccumulator>(c_val)
            );
            D_ptr[m * N + n] = d_val;
          }
        }
      }
```
- **EN:** Implements or wires together logic around `alpha`, `sum`, `beta`, `ElementC`, `c_val` for the current test scenario.
- **CN:** 围绕 `alpha`, `sum`, `beta`, `ElementC`, `c_val` 实现或连接当前测试场景所需的逻辑。

### Lines 4630-4645

```cpp
      // Copy reference results to device
      block_ref_D.copy_from_host(ref_output.data());
    } 
    catch (const std::out_of_range& e) {
      CUTLASS_TRACE_HOST("Reference computation failed - index error: " << e.what());
    }
    catch (const std::bad_alloc& e) {
      CUTLASS_TRACE_HOST("Reference computation failed - memory error: " << e.what());
    }
    catch (const std::exception& e) {
      CUTLASS_TRACE_HOST("Reference computation failed with exception: " << e.what());
    }
    catch (...) {
      CUTLASS_TRACE_HOST("Reference computation failed with unknown error");
    }
  }
```
- **EN:** Implements or wires together logic around `Copy`, `reference`, `results`, `device`, `block_ref_D` for the current test scenario.
- **CN:** 围绕 `Copy`, `reference`, `results`, `device`, `block_ref_D` 实现或连接当前测试场景所需的逻辑。

### Lines 4647-4649

```cpp
  typename Gemm::Arguments create_arguments(const cutlass::KernelHardwareInfo& hw_info) {
    typename Gemm::Arguments arguments;
    decltype(arguments.epilogue.thread) fusion_args;
```
- **EN:** Begins function or method `create_arguments`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `create_arguments`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4651-4658

```cpp
    fusion_args.alpha = alpha_scalar;
    fusion_args.beta = beta_scalar;
    fusion_args.alpha_ptr = nullptr;
    fusion_args.beta_ptr = nullptr;
    fusion_args.alpha_ptr_array = nullptr;
    fusion_args.beta_ptr_array = nullptr;
    fusion_args.dAlpha = {cute::_0{}, cute::_0{}, 0};
    fusion_args.dBeta = {cute::_0{}, cute::_0{}, 0};
```
- **EN:** Declares member fields or local variables related to `fusion_args`, `alpha`, `alpha_scalar`, `beta`, `beta_scalar` for later setup, execution, or verification.
- **CN:** 声明与 `fusion_args`, `alpha`, `alpha_scalar`, `beta`, `beta_scalar` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4660-4660

```cpp
    using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerXeGroup<ProblemShape>::RasterOrderOptions;
```
- **EN:** Defines aliases such as `RasterOrderOptions` to shorten verbose template expressions and name the chosen kernel building blocks.
- **CN:** 定义别名，例如 `RasterOrderOptions`，用于简化冗长的模板表达式并命名选定的内核构件。

### Lines 4662-4669

```cpp
    arguments = typename Gemm::Arguments {
      cutlass::gemm::GemmUniversalMode::kGrouped,
      {groups, problem_sizes.get(), problem_sizes_host.data()},
      {ptr_A.get(), stride_A.get(), ptr_B.get(), stride_B.get()},
      {fusion_args, ptr_C.get(), stride_C.get(), ptr_D.get(), stride_D.get()},
      hw_info,
      {1, RasterOrderOptions::AlongN}
    };
```
- **EN:** Implements or wires together logic around `arguments`, `Gemm`, `Arguments`, `gemm`, `GemmUniversalMode` for the current test scenario.
- **CN:** 围绕 `arguments`, `Gemm`, `Arguments`, `gemm`, `GemmUniversalMode` 实现或连接当前测试场景所需的逻辑。

### Lines 4671-4672

```cpp
    return arguments;
  }
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4674-4677

```cpp
  bool verify() {
    try {
      // Sanity check: ensure output is not all zeros
      bool all_zeros = true;
```
- **EN:** Begins function or method `verify`, which encapsulates a focused helper step in the surrounding test flow.
- **CN:** 开始定义函数或方法 `verify`，它封装了周边测试流程中的一个具体辅助步骤。

### Lines 4679-4685

```cpp
      #ifdef CUTLASS_ENABLE_SYCL
        // For SYCL: check a subset of the output
        std::vector<ElementOutput> sample_output(std::min(size_t(100), block_D.size()));
        auto* d_ptr = block_D.get();
        // Simple check without explicit copy for performance
        // Assume kernel executed successfully if we reached here
      #endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 4687-4691

```cpp
      // For grouped GEMM, we rely on the compute_reference() having already
      // generated reference results. The kernel execution success indicates
      // verification passed (similar to original TestbedImpl approach).
      // Further detailed element-wise verification can be added if needed,
      // but it would be expensive for large batches.
```
- **EN:** Adds inline commentary that explains the intent of the following declaration or test scenario.
- **CN:** 添加行内注释，用来说明后续声明或测试场景的意图。

### Lines 4693-4709

```cpp
      CUTLASS_TRACE_HOST("Grouped GEMM kernel executed successfully - verification skipped for performance");
      return true;
    }
    catch (const std::bad_alloc& e) {
      CUTLASS_TRACE_HOST("Verification failed - memory allocation error: " << e.what());
      return false;
    }
    catch (const std::exception& e) {
      CUTLASS_TRACE_HOST("Verification failed with exception: " << e.what());
      return false;
    }
    catch (...) {
      CUTLASS_TRACE_HOST("Verification failed with unknown error");
      return false;
    }
  }
};
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4713-4720

```cpp
template<typename Gemm>
bool TestXeGrouped(
    const std::vector<cutlass::gemm::GemmCoord>& problem_sizes, 
    double alpha = 1.0,
    double beta = 0.0
) {
  cutlass::KernelHardwareInfo hw_info;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
```
- **EN:** Implements or wires together logic around `Gemm`, `TestXeGrouped`, `std`, `vector`, `gemm` for the current test scenario.
- **CN:** 围绕 `Gemm`, `TestXeGrouped`, `std`, `vector`, `gemm` 实现或连接当前测试场景所需的逻辑。

### Lines 4722-4722

```cpp
  bool passed = true;
```
- **EN:** Declares member fields or local variables related to `passed` for later setup, execution, or verification.
- **CN:** 声明与 `passed` 相关的成员字段或局部变量，供后续初始化、执行或验证使用。

### Lines 4724-4737

```cpp
  try {
    GroupedGemmTestRunner<Gemm> runner(problem_sizes, 
                                       static_cast<float>(alpha),
                                       static_cast<float>(beta));
    passed = runner.run(hw_info);
  }
  catch (std::exception const& e) {
    EXPECT_TRUE(false) << "TestXeGrouped: runner.run threw an exception: " << e.what();
    return false;
  }
  catch (...) {
    EXPECT_TRUE(false) << "TestXeGrouped: runner.run threw an unknown exception";
    return false;
  }
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 4739-4741

```cpp
  EXPECT_TRUE(passed) << "TestXeGrouped: runner.run failed for " 
                      << problem_sizes.size() << " grouped problems"
                      << ", alpha: " << alpha << ", beta: " << beta;
```
- **EN:** Applies unit-test assertions to compare the observed result with the expected behavior.
- **CN:** 使用单元测试断言，将实际结果与预期行为进行比较。

### Lines 4743-4744

```cpp
  return passed;
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4746-4746

```cpp
#endif
```
- **EN:** Applies conditional compilation so this block is enabled only when the matching platform or build option is active.
- **CN:** 通过条件编译仅在匹配的平台或构建选项开启时启用该代码块。

### Lines 4748-4751

```cpp
template <typename Gemm>
bool TestAllBiasElementwise(double alpha = 1.0, double beta = cute::is_same_v<typename Gemm::GemmKernel::ElementC, void> ? 0.0 : 1.0, CheckEquality check_relative_equality = CheckEquality::EXACT) {
  return TestAll<Gemm>(alpha, beta, check_relative_equality);
}
```
- **EN:** Returns the value or status computed by the helper logic in this block.
- **CN:** 返回该代码块中辅助逻辑计算得到的值或状态。

### Lines 4753-4755

```cpp
} // namespace device
} // namespace gemm
} // namespace test
```
- **EN:** Opens or closes namespaces so the declarations live in the expected CUTLASS or test scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS 或测试作用域中。

### Lines 4757-4757

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
- **EN:** Intel Xe-specific MMA operations, dispatch policies, and tile shapes.  
  **CN:** 面向 Intel Xe 的 MMA 操作、派发策略与 tile 形状。
- **EN:** Grouped or batched problem scheduling.  
  **CN:** 分组或批量问题调度。
- **EN:** Pointer-array based grouped inputs.  
  **CN:** 基于指针数组的 grouped 输入。
- **EN:** Reduction-aware epilogues or verification paths.  
  **CN:** 支持归约的 epilogue 或验证路径。
- **EN:** Sparse data preparation and sparse-kernel validation.  
  **CN:** 稀疏数据准备与稀疏内核验证。
- **EN:** Split-K decomposition and accumulation behavior.  
  **CN:** Split-K 分解与累加行为。

## Dependencies / 依赖关系

- `iostream`
- `fstream`
- `sstream`
- `algorithm`
- `random`
- `numeric`
- `cfloat`
- `cutlass/epilogue/collective/default_epilogue.hpp`
- `cutlass/epilogue/collective/xe_array_epilogue.hpp`
- `cutlass/epilogue/fusion/xe_callbacks.hpp`
- `cutlass/gemm/group_array_problem_shape.hpp`
- `cutlass/gemm/device/gemm_universal.h`
- ... and 35 more direct includes / 以及另外 35 个直接依赖头文件
- CUTLASS template APIs and namespaces form the main external programming model / CUTLASS 模板 API 与命名空间构成主要外部编程模型
- CUTE supplies shape/layout metaprogramming primitives / CUTE 提供 shape/layout 元编程原语
