# dispatch_policy.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/dispatch_policy.hpp`

- **Purpose (EN):** Defines the `dispatch policy` policy object that fixes layout and iteration rules for this epilogue path.

- **作用 (CN):** 定义 `dispatch policy` 策略对象，固定该 epilogue 路径的布局和迭代规则。


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
#include "cutlass/numeric_conversion.h"
#include "cutlass/epilogue/thread/scale_type.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/numeric_conversion.h`, `cutlass/epilogue/thread/scale_type.h`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/numeric_conversion.h`，`cutlass/epilogue/thread/scale_type.h`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 38

```cpp
namespace cutlass::epilogue {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 47-86

```cpp
// Pre-Hopper schedules
struct PtrArrayDefault {};
struct EpilogueSimtVectorized {};
struct EpiloguePtrArraySimtVectorized {};
// Hopper direct store schedules
struct NoSmemWarpSpecialized {};
struct PtrArrayNoSmemWarpSpecialized {};
struct PtrArrayNoSmemWarpSpecializedTransposed {};
// Hopper TMA schedules
struct TmaWarpSpecialized {};
struct TmaWarpSpecializedCooperative {};
struct PtrArrayTmaWarpSpecialized { static constexpr int NumEpilogueWarpGroups = 1; };
struct PtrArrayTmaWarpSpecializedPingpong { static constexpr int NumEpilogueWarpGroups = 2; };
struct PtrArrayTmaWarpSpecializedCooperative { static constexpr int NumEpilogueWarpGroups = 2; };
// Blackwell direct store schedules
struct NoSmemWarpSpecialized1Sm {};
struct NoSmemWarpSpecialized2Sm {};
struct FastF32NoSmemWarpSpecialized1Sm : NoSmemWarpSpecialized1Sm {};
struct FastF32NoSmemWarpSpecialized2Sm : NoSmemWarpSpecialized2Sm {};
struct BlockwiseNoSmemWarpSpecialized1Sm : NoSmemWarpSpecialized1Sm {};
struct BlockwiseNoSmemWarpSpecialized2Sm : NoSmemWarpSpecialized2Sm {};
struct PtrArrayNoSmemWarpSpecialized1Sm : NoSmemWarpSpecialized1Sm {};
struct PtrArrayNoSmemWarpSpecialized2Sm : NoSmemWarpSpecialized2Sm {};
struct PtrArrayFastF32NoSmemWarpSpecialized1Sm : PtrArrayNoSmemWarpSpecialized1Sm {};
struct PtrArrayFastF32NoSmemWarpSpecialized2Sm : PtrArrayNoSmemWarpSpecialized2Sm {};
struct PtrArrayBlockwiseNoSmemWarpSpecialized1Sm : PtrArrayNoSmemWarpSpecialized1Sm {};
struct PtrArrayBlockwiseNoSmemWarpSpecialized2Sm : PtrArrayNoSmemWarpSpecialized2Sm {};
// Blackwell TMA schedules
struct TmaWarpSpecialized1Sm {};
struct TmaWarpSpecialized2Sm {};
struct PtrArrayTmaWarpSpecialized1Sm : TmaWarpSpecialized1Sm {};
struct PtrArrayTmaWarpSpecialized2Sm : TmaWarpSpecialized2Sm {};
struct TmaWarpSpecialized1SmNvf4     final : TmaWarpSpecialized1Sm {};
struct TmaWarpSpecialized2SmNvf4     final : TmaWarpSpecialized2Sm {};
struct TmaWarpSpecialized1SmMxf4     final : TmaWarpSpecialized1Sm {};
struct TmaWarpSpecialized2SmMxf4     final : TmaWarpSpecialized2Sm {};
struct TmaWarpSpecialized1SmMxf8f6f4 final : TmaWarpSpecialized1Sm {};
struct TmaWarpSpecialized2SmMxf8f6f4 final : TmaWarpSpecialized2Sm {};
// Cooperative epilogue schedule for sm120 sparse kernels
struct SparseTmaWarpSpecializedCooperativeSm120 : public TmaWarpSpecializedCooperative {};
```

**EN:** Defines `PtrArrayDefault`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Pre-Hopper schedules.

**CN:** 定义 `PtrArrayDefault`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 88-102

```cpp
// DEPRECATED schedules, will be removed in next release
struct TmaWarpSpecializedElementwiseBase : public TmaWarpSpecialized {};
struct TmaWarpSpecializedCooperativeElementwiseBase : public TmaWarpSpecializedCooperative {};
template <
  template <class T> class ActivationFunctor_,
  thread::ScaleType::Kind Scale_ = thread::ScaleType::Default,
  FloatRoundStyle Round_ = FloatRoundStyle::round_to_nearest
>
struct [[deprecated("Use TmaWarpSpecialized with fusion::LinCombEltAct instead")]]
TmaWarpSpecializedElementwise : public TmaWarpSpecializedElementwiseBase {
  template <class T>
  using ActivationFunctor = ActivationFunctor_<T>;
  static constexpr thread::ScaleType::Kind Scale = Scale_;
  static constexpr FloatRoundStyle Round = Round_;
};
```

**EN:** Declares the templated `TmaWarpSpecializedElementwiseBase` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: DEPRECATED schedules, will be removed in next release.

**CN:** 声明模板类型 `TmaWarpSpecializedElementwiseBase`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 104-115

```cpp
template <
  template <class T> class ActivationFunctor_,
  thread::ScaleType::Kind Scale_ = thread::ScaleType::Default,
  FloatRoundStyle Round_ = FloatRoundStyle::round_to_nearest
>
struct [[deprecated("Use TmaWarpSpecializedCooperative with fusion::LinCombEltAct instead")]]
TmaWarpSpecializedCooperativeElementwise : public TmaWarpSpecializedCooperativeElementwiseBase {
  template <class T>
  using ActivationFunctor = ActivationFunctor_<T>;
  static constexpr thread::ScaleType::Kind Scale = Scale_;
  static constexpr FloatRoundStyle Round = Round_;
};
```

**EN:** Declares the templated `T` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `T`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 117-118

```cpp
struct TmaWarpSpecializedBiasElementwiseBase : public TmaWarpSpecialized{};
struct TmaWarpSpecializedCooperativeBiasElementwiseBase : public TmaWarpSpecializedCooperative {};
```

**EN:** Defines `TmaWarpSpecializedBiasElementwiseBase`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `TmaWarpSpecializedBiasElementwiseBase`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 120-131

```cpp
template <
  template <class T> class ActivationFunctor_,
  class ElementT_,
  template <class T> class BiasOp_,
  bool StoreT_,
  class ElementBias_
>
struct [[deprecated("Use TmaWarpSpecialized with fusion::LinCombPerRowBiasEltActAux instead")]]
TmaWarpSpecializedBiasElementwise : public TmaWarpSpecializedBiasElementwiseBase {
  template <class T>
  using ActivationFunctor = ActivationFunctor_<T>;
  using ElementT = ElementT_;
```

**EN:** Declares the templated `T` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `T`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 133-134

```cpp
  template <class T>
  using BiasOp = BiasOp_<T>;
```

**EN:** Declares the templated `T` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `T`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 136-138

```cpp
  static constexpr bool StoreT = StoreT_;
  using ElementBias = ElementBias_;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 140-150

```cpp
template <
  template <class T> class ActivationFunctor_,
  class ElementT_,
  template <class T> class BiasOp_,
  bool StoreT_,
  class ElementBias_
>
struct [[deprecated("Use TmaWarpSpecializedCooperative with fusion::LinCombPerRowBiasEltActAux instead")]]
TmaWarpSpecializedCooperativeBiasElementwise : public TmaWarpSpecializedCooperativeBiasElementwiseBase {
  template <class T>
  using ActivationFunctor = ActivationFunctor_<T>;
```

**EN:** Declares the templated `T` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `T`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Line 152

```cpp
  using ElementT = ElementT_;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 154-155

```cpp
  template <class T>
  using BiasOp = BiasOp_<T>;
```

**EN:** Declares the templated `T` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `T`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 157-159

```cpp
  static constexpr bool StoreT = StoreT_;
  using ElementBias = ElementBias_;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 167-180

```cpp
template<
  int StagesC_,
  int StagesD_,
  int FragmentSize_,
  bool ReuseSmemC_,
  bool DelayTmaStore_
>
struct Sm90TmaWarpSpecialized {
  constexpr static int StagesC = StagesC_;
  constexpr static int StagesD = StagesD_;
  constexpr static int FragmentSize = FragmentSize_;
  constexpr static bool ReuseSmemC = ReuseSmemC_;
  constexpr static bool DelayTmaStore = DelayTmaStore_;
};
```

**EN:** Defines `Sm90TmaWarpSpecialized`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm90TmaWarpSpecialized`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 182-197

```cpp
template<
  int StagesC_,
  int StagesD_,
  int FragmentSize_,
  bool ReuseSmemC_,
  bool DelayTmaStore_,
  int NumEpilogueWarpGroups_
>
struct Sm90PtrArrayTmaWarpSpecialized {
  constexpr static int StagesC = StagesC_;
  constexpr static int StagesD = StagesD_;
  constexpr static int FragmentSize = FragmentSize_;
  constexpr static bool ReuseSmemC = ReuseSmemC_;
  constexpr static bool DelayTmaStore = DelayTmaStore_;
  constexpr static int NumEpilogueWarpGroups = NumEpilogueWarpGroups_;
};
```

**EN:** Defines `Sm90PtrArrayTmaWarpSpecialized`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm90PtrArrayTmaWarpSpecialized`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 199-209

```cpp
// DEPRECATED policies, will be removed in next release
template<
  int StagesC_,
  int StagesD_,
  int FragmentSize_ = 2
>
struct Sm90TmaWarpSpecializedBiasElementwise {
  constexpr static int StagesC = StagesC_;
  constexpr static int StagesD = StagesD_;
  constexpr static int FragmentSize = FragmentSize_;
};
```

**EN:** Defines `Sm90TmaWarpSpecializedBiasElementwise`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: DEPRECATED policies, will be removed in next release.

**CN:** 定义 `Sm90TmaWarpSpecializedBiasElementwise`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 212-225

```cpp
template<
  int StagesC_,
  int StagesD_,
  int FragmentSize_,
  bool ReuseSmemC_,
  bool DelayTmaStore_
>
struct Sm100TmaWarpSpecialized {
  constexpr static int StagesC = StagesC_;
  constexpr static int StagesD = StagesD_;
  constexpr static int FragmentSize = FragmentSize_;
  constexpr static bool ReuseSmemC = ReuseSmemC_;
  constexpr static bool DelayTmaStore = DelayTmaStore_;
};
```

**EN:** Defines `Sm100TmaWarpSpecialized`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm100TmaWarpSpecialized`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 227-239

```cpp
template<
  int StagesC_,
  int StagesD_,
  int FragmentSize_,
  bool ReuseSmemC_,
  bool DelayTmaStore_
>
struct Sm100PtrArrayTmaWarpSpecialized {
  constexpr static int StagesC = StagesC_;
  constexpr static int StagesD = StagesD_;
  constexpr static int FragmentSize = FragmentSize_;
  constexpr static bool ReuseSmemC = ReuseSmemC_;
  constexpr static bool DelayTmaStore = DelayTmaStore_;
```

**EN:** Defines `Sm100PtrArrayTmaWarpSpecialized`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm100PtrArrayTmaWarpSpecialized`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 241-243

```cpp
  static_assert(StagesC >= 1, "StagesC must be >= 1");
  static_assert(StagesD >= 1, "StagesD must be >= 1");
};
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 245-249

```cpp
struct Sm100NoSmem {
  constexpr static int StagesC = 1;
  constexpr static int StagesD = 1;
  constexpr static int FragmentSize = 1;
};
```

**EN:** Defines `Sm100NoSmem`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm100NoSmem`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 251-255

```cpp
struct Sm100NoSmemWarpSpecialized {
  constexpr static int StagesC = 1;
  constexpr static int StagesD = 1;
  constexpr static int FragmentSize = 1;
};
```

**EN:** Defines `Sm100NoSmemWarpSpecialized`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm100NoSmemWarpSpecialized`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 257-261

```cpp
struct Sm100PtrArrayNoSmem {
  constexpr static int StagesC = 1;
  constexpr static int StagesD = 1;
  constexpr static int FragmentSize = 1;
};
```

**EN:** Defines `Sm100PtrArrayNoSmem`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm100PtrArrayNoSmem`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 263-281

```cpp
struct Sm100PtrArrayNoSmemWarpSpecialized {
  constexpr static int StagesC = 1;
  constexpr static int StagesD = 1;
  constexpr static int FragmentSize = 1;
};
template<
  int StagesC_,
  int StagesD_,
  int FragmentSize_,
  bool ReuseSmemC_,
  bool DelayTmaStore_
>
struct Sm120TmaWarpSpecialized {
  constexpr static int StagesC = StagesC_;
  constexpr static int StagesD = StagesD_;
  constexpr static int FragmentSize = FragmentSize_;
  constexpr static bool ReuseSmemC = ReuseSmemC_;
  constexpr static bool DelayTmaStore = DelayTmaStore_;
};
```

**EN:** Defines `Sm100PtrArrayNoSmemWarpSpecialized`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm100PtrArrayNoSmemWarpSpecialized`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 283-298

```cpp
template<
  int StagesC_,
  int StagesD_,
  int FragmentSize_,
  bool ReuseSmemC_,
  bool DelayTmaStore_,
  int NumEpilogueWarpGroups_
>
struct Sm120PtrArrayTmaWarpSpecialized {
  constexpr static int StagesC = StagesC_;
  constexpr static int StagesD = StagesD_;
  constexpr static int FragmentSize = FragmentSize_;
  constexpr static bool ReuseSmemC = ReuseSmemC_;
  constexpr static bool DelayTmaStore = DelayTmaStore_;
  constexpr static int NumEpilogueWarpGroups = NumEpilogueWarpGroups_;
};
```

**EN:** Defines `Sm120PtrArrayTmaWarpSpecialized`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Sm120PtrArrayTmaWarpSpecialized`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 300-304

```cpp
#if defined (SYCL_INTEL_TARGET)
// Standard Xe epilogue.
struct IntelXeGeneric {
  static constexpr int SubgroupSize = 16;
};
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 306-312

```cpp
struct IntelXeGenericGroup {
  static constexpr int SubgroupSize = 16;
};
// Legacy epilogues.
struct IntelXeXMX16 {
  static constexpr int SubgroupSize = 16;
};
```

**EN:** Defines `IntelXeGenericGroup`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `IntelXeGenericGroup`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 314-317

```cpp
struct IntelXeXMX16Group {
  static constexpr int SubgroupSize = 16;
};
#endif
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


## Key Concepts / 关键概念

- **Template metaprogramming / 模板元编程:** The header relies on compile-time selection to adapt epilogue behavior. / 该头文件依赖编译期选择来适配 epilogue 行为。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/numeric_conversion.h`, `cutlass/epilogue/thread/scale_type.h`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/thread/scale_type.h`

- **Feature macros / 特性宏:** `SYCL_INTEL_TARGET`
