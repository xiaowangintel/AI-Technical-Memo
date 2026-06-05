# collective_builder.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/collective/collective_builder.hpp`

- **Purpose (EN):** Builds `collective builder` types from compile-time epilogue configuration parameters.

- **作用 (CN):** 根据编译期 epilogue 配置参数构建 `collective builder` 类型。


## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Line 31

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 33-34

```cpp
#include <cute/arch/copy.hpp>         // cute::DefaultCopy
#include <cute/util/type_traits.hpp>  // cute::is_base_of_v
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cute/arch/copy.hpp`, `cute/util/type_traits.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cute/arch/copy.hpp`，`cute/util/type_traits.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 36-37

```cpp
#include "cutlass/detail/dependent_false.hpp"
#include "cutlass/epilogue/fusion/callbacks.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/detail/dependent_false.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/detail/dependent_false.hpp`，`cutlass/epilogue/fusion/callbacks.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 41

```cpp
namespace cutlass::epilogue::collective {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 45-46

```cpp
// Used to specify epilogue subtile shape or dispatch to automatic computation of subtile shape
struct EpilogueTileAuto {};
```

**EN:** Defines `EpilogueTileAuto`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Used to specify epilogue subtile shape or dispatch to automatic computation of subtile shape.

**CN:** 定义 `EpilogueTileAuto`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 48-50

```cpp
// Used to let the builder pick the epilogue schedule automatically.
// Can be overridden with kernel schedule tags in cutlass/gemm/dispatch_policy.hpp
struct EpilogueScheduleAuto {};
```

**EN:** Defines `EpilogueScheduleAuto`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Used to let the builder pick the epilogue schedule automatically. Can be overridden with kernel schedule tags in cutlass/gemm/dispatch_policy.hpp.

**CN:** 定义 `EpilogueScheduleAuto`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 52-73

```cpp
template <
  class ArchTag,
  class OpClass,
  class TileShape_MNK,
  class ClusterShape_MNK,
  class EpilogueTileType,
  class ElementAccumulator,
  class ElementCompute,
  class ElementC,
  class GmemLayoutTagC,
  int AlignmentC,
  class ElementD,
  class GmemLayoutTagD,
  int AlignmentD,
  class EpilogueScheduleType,
  class FusionOpOrCallbacks = cutlass::epilogue::fusion::LinearCombination<ElementD,ElementCompute,ElementC,ElementCompute>,
  class Enable = void
>
struct CollectiveBuilder {
  static_assert(cutlass::detail::dependent_false<ArchTag>,
      "Could not build a collective epilogue for given parameters.");
};
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 75-76

```cpp
// helper sub-builder for epilogue fusion callbacks (for internal use by CollectiveBuilder only)
namespace detail {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 78-90

```cpp
// callbacks builder with operation tag
template<
  class DispatchPolicy,
  class FusionOp,
  class TileShape_MNK,
  class EpilogueTile_MN,
  class ElementAccumulator,
  class AccLoadOp = cute::DefaultCopy,
  class = void
>
struct CallbacksBuilder {
  using Callbacks = fusion::FusionCallbacks<DispatchPolicy, FusionOp, TileShape_MNK, EpilogueTile_MN>;
};
```

**EN:** Defines `DispatchPolicy`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: callbacks builder with operation tag.

**CN:** 定义 `DispatchPolicy`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 92-111

```cpp
// callbacks builder with callbacks passthrough
template <
  class DispatchPolicy,
  class FusionCallbacks,
  class TileShape_MNK,
  class EpilogueTile_MN,
  class AccLoadOp,
  class ElementAccumulator
>
struct CallbacksBuilder<
  DispatchPolicy,
  FusionCallbacks,
  TileShape_MNK,
  EpilogueTile_MN,
  ElementAccumulator,
  AccLoadOp,
  cute::enable_if_t<not cute::is_base_of_v<fusion::FusionOperation, FusionCallbacks>>
> {
  using Callbacks = FusionCallbacks;
};
```

**EN:** Declares the templated `DispatchPolicy` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: callbacks builder with callbacks passthrough.

**CN:** 声明模板类型 `DispatchPolicy`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 121-124

```cpp
#include "builders/sm90_builder.inl"
#include "builders/sm100_builder.inl"  
#include "builders/sm103_builder.inl"
#include "builders/sm120_builder.inl"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `builders/sm90_builder.inl`, `builders/sm100_builder.inl`, `builders/sm103_builder.inl`, `builders/sm120_builder.inl`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `builders/sm90_builder.inl`，`builders/sm100_builder.inl`，`builders/sm103_builder.inl`，`builders/sm120_builder.inl`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 126-128

```cpp
#if defined(SYCL_INTEL_TARGET)
#include "builders/xe_builder.inl"
#endif
```

**EN:** This include block pulls in the direct dependencies for the file, such as `builders/xe_builder.inl`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `builders/xe_builder.inl`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 130-132

```cpp
#if defined(CUTLASS_ENABLE_SYCL)
#include "builders/device_agnostic_builder.inl"
#endif
```

**EN:** This include block pulls in the direct dependencies for the file, such as `builders/device_agnostic_builder.inl`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `builders/device_agnostic_builder.inl`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


## Key Concepts / 关键概念

- **Collective epilogue / Collective epilogue:** Coordinates epilogue work at the collective level, often tying dispatch policy, tile shape, and fusion callbacks together. / 在 collective 层面组织 epilogue 工作，通常把调度策略、tile 形状和融合回调连接在一起。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cute/arch/copy.hpp`, `cute/util/type_traits.hpp`, `cutlass/detail/dependent_false.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `builders/sm90_builder.inl`, `builders/sm100_builder.inl`, `builders/sm103_builder.inl`, `builders/sm120_builder.inl`, `builders/xe_builder.inl`, `builders/device_agnostic_builder.inl`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::collective`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/fusion/callbacks.hpp`

- **Cute/CuTe dependencies / Cute 依赖:** `cute/arch/copy.hpp`, `cute/util/type_traits.hpp`

- **Feature macros / 特性宏:** `CUTLASS_ENABLE_SYCL`, `SYCL_INTEL_TARGET`
