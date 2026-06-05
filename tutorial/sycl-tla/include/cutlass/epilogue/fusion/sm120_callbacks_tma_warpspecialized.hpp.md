# sm120_callbacks_tma_warpspecialized.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/fusion/sm120_callbacks_tma_warpspecialized.hpp`

- **Purpose (EN):** Fusion callbacks specializations for the SM120 TMA warp-specialized (ws) epilogue.

- **作用 (CN):** 实现 `SM120 callbacks TMA warpspecialized` 相关的融合回调接口与回调连接逻辑。


## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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


### Lines 33-35

```cpp
/*! \file
  \brief Fusion callbacks specializations for the SM120 TMA warp-specialized (ws) epilogue
*/
```

**EN:** This documentation block explains the intent of the next declaration: ! Fusion callbacks specializations for the SM120 TMA warp-specialized (ws) epilogue.

**CN:** 这一段文档注释说明了紧随其后的声明意图，帮助理解后续模板或实现要解决的问题。


### Line 37

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Line 39

```cpp
#include "cutlass/cutlass.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 41

```cpp
#include "cute/tensor.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cute/tensor.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cute/tensor.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 43-47

```cpp
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/epilogue/fusion/callbacks.hpp"
#include "cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/sm100_callbacks_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/sm120_visitor_store_tma_warpspecialized.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm100_callbacks_tma_warpspecialized.hpp`, and 1 more headers. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/epilogue/dispatch_policy.hpp`，`cutlass/epilogue/fusion/callbacks.hpp`，`cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`，`cutlass/epilogue/fusion/sm100_callbacks_tma_warpspecialized.hpp`，以及另外 1 个头文件。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 51

```cpp
namespace cutlass::epilogue::fusion {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 55-86

```cpp
// Sm120 Tma warp specialized callbacks just alias to their sm90 counterpart
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class Operation,
  class CtaTile_MNK,
  class EpilogueTile_MN,
  class... Args
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    Operation,
    CtaTile_MNK,
    EpilogueTile_MN,
    Args...
> : FusionCallbacks<
      epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...
    > {
  using FusionCallbacks<
      epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...>::FusionCallbacks;
};
```

**EN:** Declares the templated `Operation` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Sm120 Tma warp specialized callbacks just alias to their sm90 counterpart.

**CN:** 声明模板类型 `Operation`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 88-110

```cpp
// D = alpha * acc + beta * C
// With BlockScaleFactor Generation.
// 1. Find max of 32 F32 elements
// 2. Convert the max to UE8 (or UE4M3) and store the result.
// 3. Convert the UE8 (or UE4M3) back to F32 scale.
// 4. Reciprocal of F32 scale with MUFU.
// 5. Multiply each F32 element with the above reciprocal, then convert to ElementD
template<
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinearCombRowBlockScaleFactor =
  Sm90EVT<Sm120BlockScaleFactorRowStore<SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,ElementCompute, ElementBlockScaleFactor, RoundStyle>, // gen scalefactor
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C With BlockScaleFactor Generation. 1. Find max of 32 F32 elements 2. Convert the max to UE8 (or UE4M3) and store the result. 3. Convert the UE8 (or UE4M3) back to F32 scale. 4. Reciprocal of F32 scale with MUFU. 5. Multiply each F32 element with the above reciprocal, then convert to ElementD.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 112-133

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute,ElementBlockScaleFactor, cutlass::layout::RowMajor, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinearCombRowBlockScaleFactor<SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Line 135

```cpp
  using Impl = Sm120LinearCombRowBlockScaleFactor<SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 137-143

```cpp
  using Sm100Fusion = FusionCallbacks<
        epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
        fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute,ElementBlockScaleFactor, cutlass::layout::RowMajor, ElementSource, ElementScalar, RoundStyle>,
        CtaTileShapeMNK,
        EpilogueTile
  >;
  using Operation = typename Sm100Fusion::Operation;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 145-155

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 157-160

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 162-179

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {
            // ternary op : beta * C + (alpha * acc)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // binary op : alpha * acc
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                 // leaf args : acc
              {}                  // binary args : multiplies
            },                    // end binary op
            {}                    // ternary args : multiply_add
          },
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 181-183

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 185-212

```cpp
// D = alpha * acc + beta * C + per-row bias
//   with row blockScaled generation
template<
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombPerRowBiasRowBlockScaleFactor =
  Sm90EVT<
    Sm120BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >, // gen scalefactor
    Sm90LinCombPerRowBias<
      CtaTileShapeMNK, ElementCompute, ElementCompute,
      ElementBias, ElementSource, ElementScalar,
      AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per-row bias with row blockScaled generation.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 214-247

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombPerRowBiasRowBlockScaleFactor<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > 
{
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 249-255

```cpp
  using Impl = 
    Sm120LinCombPerRowBiasRowBlockScaleFactor<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 257-262

```cpp
  using Operation = 
    fusion::LinCombPerRowBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor, 
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 264-274

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 276-279

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 281-283

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 285-302

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {  // ternary op : beta * C + (alpha * acc + bias)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // ternary op : alpha * acc + bias
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                 // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {}                  // ternary args : multiply_add
            },                    // end ternary op
            {} // ternary args : multiply_add
          },  // end ternary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 304-306

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 308-336

```cpp
// D = activation(alpha * acc + beta * C + per-row bias) 
//   with row blockScaled generation
template<
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor, 
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombPerRowBiasEltActRowBlockScaleFactor =
  Sm90EVT<
    Sm120BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >, // gen scalefactor
    Sm90LinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, 
      ElementCompute, ElementCompute, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Declares the templated `EpilogueTile` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-row bias) with row blockScaled generation.

**CN:** 声明模板类型 `EpilogueTile`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 338-371

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor, 
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombPerRowBiasEltActRowBlockScaleFactor<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 373-379

```cpp
  using Impl = 
    Sm120LinCombPerRowBiasEltActRowBlockScaleFactor<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 381-386

```cpp
  using Operation = 
    fusion::LinCombPerRowBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor, 
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 388-398

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 400-403

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 405-407

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 409-410

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 412-432

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {    // unary op : activation(beta * C + (alpha * acc + bias))
            {    // ternary op : beta * C + (alpha * acc + bias)
              {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // ternary op : alpha * acc + bias
                {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
                {},                 // leaf args : acc
                {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
                {}                  // ternary args : multiply_add
              },                    // end ternary op
              {} // ternary args : multiply_add
            },   // end ternary op
            activation // unary args : activation
          },   // end unary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 434-436

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 438-466

```cpp
// D = alpha * acc + beta * C + per_col bias
//   with row blockScaled generation
template<
  int StagesC,
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombPerColBiasRowBlockScaleFactor =
  Sm90EVT<
    Sm120BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >, // gen scalefactor
    Sm90LinCombPerColBias<
      StagesC, CtaTileShapeMNK, EpilogueTile, ElementCompute, ElementCompute, 
      ElementBias, ElementSource, ElementScalar,
      AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per_col bias with row blockScaled generation.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 468-502

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementBias, ElementSource, 
      ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombPerColBiasRowBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > 
{
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 504-510

```cpp
  using Impl = 
    Sm120LinCombPerColBiasRowBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 512-518

```cpp
  using Operation = 
    fusion::LinCombPerColBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementBias, ElementSource, 
      ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 520-530

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 532-535

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 538-540

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 542-559

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {  // ternary op : beta * C + (alpha * acc + bias)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // ternary op : alpha * acc + bias
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                 // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {}                  // ternary args : multiply_add
            },                    // end ternary op
            {} // ternary args : multiply_add
          },  // end ternary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 561-563

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 565-594

```cpp
// D = activation(alpha * acc + beta * C + per_col bias) 
//   with row blockScaled generation
template<
  int StagesC,
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor, 
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombPerColBiasEltActRowBlockScaleFactor =
  Sm90EVT<
    Sm120BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >, // gen scalefactor
    Sm90LinCombPerColBiasEltAct<
      StagesC, CtaTileShapeMNK, EpilogueTile, ActivationFn, 
      ElementCompute, ElementCompute, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Declares the templated `EpilogueTile` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per_col bias) with row blockScaled generation.

**CN:** 声明模板类型 `EpilogueTile`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 596-630

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementBias, ElementSource, 
      ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombPerColBiasEltActRowBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 632-638

```cpp
  using Impl =
    Sm120LinCombPerColBiasEltActRowBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 640-646

```cpp
  using Operation =
    fusion::LinCombPerColBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementBias, ElementSource, 
      ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 648-658

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 660-663

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 665-667

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 669-670

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 672-692

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {    // unary op : activation(beta * C + (alpha * acc + bias))
            {    // ternary op : beta * C + (alpha * acc + bias)
              {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // ternary op : alpha * acc + bias
                {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
                {},                 // leaf args : acc
                {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
                {}                  // ternary args : multiply_add
              },                    // end ternary op
              {} // ternary args : multiply_add
            },   // end ternary op
            activation // unary args : activation
          },   // end unary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 694-696

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 700-725

```cpp
// D = alpha * acc + beta * C
//   with per column blockScaled generation
// 1. Find max of 32 F32 elements
// 2. Convert the max to UE8 (or UE4M3) and store the result.
// 3. Convert the UE8 (or UE4M3) back to F32 scale.
// 4. Reciprocal of F32 scale with MUFU.
// 5. Multiply each F32 element with the above reciprocal, then convert to ElementD
template<
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinearCombColBlockScaleFactor = Sm90EVT<
    Sm120BlockScaleFactorColStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle>,
    Sm90LinearCombination<
      ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle>
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C with per column blockScaled generation 1. Find max of 32 F32 elements 2. Convert the max to UE8 (or UE4M3) and store the result. 3. Convert the UE8 (or UE4M3) back to F32 scale. 4. Reciprocal of F32 scale with MUFU. 5. Multiply each F32 element with the above reciprocal, then convert to ElementD.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 727-755

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
  epilogue::Sm120TmaWarpSpecialized<
    StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
  fusion::LinCombBlockScaleFactor<
    SFVecSize, ElementOutput, ElementCompute,ElementBlockScaleFactor, 
    cutlass::layout::ColumnMajor, ElementSource, ElementScalar, RoundStyle>,
  CtaTileShapeMNK,
  EpilogueTile
> : Sm120LinearCombColBlockScaleFactor<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle
    > {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Line 757

```cpp
  using Impl = Sm120LinearCombColBlockScaleFactor<SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 759-765

```cpp
  using Sm100Fusion = FusionCallbacks<
        epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
        fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute,ElementBlockScaleFactor, cutlass::layout::ColumnMajor,ElementSource, ElementScalar, RoundStyle>,
        CtaTileShapeMNK,
        EpilogueTile
  >;
  using Operation = typename Sm100Fusion::Operation;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 767-777

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 779-782

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 784-801

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {
            // ternary op : beta * C + (alpha * acc)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // binary op : alpha * acc
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                 // leaf args : acc
              {}                  // binary args : multiplies
            },                    // end binary op
            {}                    // ternary args : multiply_add
          },
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 803-805

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 807-835

```cpp
// D = alpha * acc + beta * C + per-Col bias
//   with per column blockScaled generation
template<
  int StagesC,
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombPerColBiasColBlockScaleFactor =
  Sm90EVT<
    Sm120BlockScaleFactorColStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerColBias<
      StagesC, CtaTileShapeMNK, EpilogueTile, ElementCompute, ElementCompute, 
      ElementBias, ElementSource, ElementScalar,
      AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per-Col bias with per column blockScaled generation.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 837-870

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombPerColBiasColBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >
{
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 872-878

```cpp
  using Impl =
    Sm120LinCombPerColBiasColBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 880-885

```cpp
  using Operation =
    fusion::LinCombPerColBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 887-897

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 899-902

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 904-906

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 908-925

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {  // ternary op : beta * C + (alpha * acc + bias)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // ternary op : alpha * acc + bias
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                 // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {}                  // ternary args : multiply_add
            },                    // end ternary op
            {} // ternary args : multiply_add
          },  // end ternary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 927-929

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 931-960

```cpp
// D = activation(alpha * acc + beta * C + per_col bias)
//   with per column blockScaled generation
template<
  int StagesC,
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombPerColBiasEltActColBlockScaleFactor =
  Sm90EVT<
    Sm120BlockScaleFactorColStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerColBiasEltAct<
      StagesC, CtaTileShapeMNK, EpilogueTile, ActivationFn,
      ElementCompute, ElementCompute, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Declares the templated `EpilogueTile` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per_col bias) with per column blockScaled generation.

**CN:** 声明模板类型 `EpilogueTile`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 962-996

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, ElementSource,
      ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombPerColBiasEltActColBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,ElementSource, ElementScalar,
      AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 998-1004

```cpp
  using Impl =
    Sm120LinCombPerColBiasEltActColBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,ElementSource, ElementScalar,
      AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1006-1012

```cpp
  using Operation =
    fusion::LinCombPerColBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, ElementSource,
      ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1014-1024

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1026-1029

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1031-1033

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1035-1036

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1038-1058

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {      // unary op : activation(beta * C + (alpha * acc + bias))
            {    // ternary op : beta * C + (alpha * acc + bias)
              {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // ternary op : alpha * acc + bias
                {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
                {},                 // leaf args : acc
                {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
                {}                  // ternary args : multiply_add
              },                    // end ternary op
              {} // ternary args : multiply_add
            },   // end ternary op
            activation // unary args : activation
          },     // end unary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };       // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1060-1062

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1064-1093

```cpp
// D = activation(alpha * acc + beta * C + per-row bias)
//   with per column blockScaled generation
template<
  int StagesC,
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombPerRowBiasEltActColBlockScaleFactor =
  Sm90EVT<
    Sm120BlockScaleFactorColStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn,
      ElementCompute, ElementCompute, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Declares the templated `EpilogueTile` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-row bias) with per column blockScaled generation.

**CN:** 声明模板类型 `EpilogueTile`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1095-1128

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombPerRowBiasEltActColBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,ElementSource, ElementScalar,
      AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1131-1137

```cpp
  using Impl =
    Sm120LinCombPerRowBiasEltActColBlockScaleFactor<
      StagesC, SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,ElementSource, ElementScalar,
      AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1139-1144

```cpp
  using Operation =
    fusion::LinCombPerRowBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1146-1156

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1158-1161

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1163-1165

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1167-1168

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1170-1190

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {    // unary op : activation(beta * C + (alpha * acc + bias))
            {    // ternary op : beta * C + (alpha * acc + bias)
              {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // ternary op : alpha * acc + bias
                {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
                {},                 // leaf args : acc
                {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
                {}                  // ternary args : multiply_add
              },                    // end ternary op
              {}   // ternary args : multiply_add
            },     // end ternary op
            activation // unary args : activation
          },    // end unary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };    // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1192-1194

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1197-1224

```cpp
// D = alpha * acc + beta * C + per-row bias
//   with per column blockScaled generation
template<
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombPerRowBiasColBlockScaleFactor =
  Sm90EVT<
    Sm120BlockScaleFactorColStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >, // gen scalefactor
    Sm90LinCombPerRowBias<
      CtaTileShapeMNK, ElementCompute, ElementCompute,
      ElementBias, ElementSource, ElementScalar,
      AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per-row bias with per column blockScaled generation.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1226-1259

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombPerRowBiasColBlockScaleFactor<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > 
{
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1261-1267

```cpp
  using Impl = 
    Sm120LinCombPerRowBiasColBlockScaleFactor<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementBias,
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1269-1274

```cpp
  using Operation = 
    fusion::LinCombPerRowBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, ElementSource, ElementScalar,AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1276-1286

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementBlockScaleFactor * block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1288-1291

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1293-1295

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1297-1314

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {  // ternary op : beta * C + (alpha * acc + bias)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // ternary op : alpha * acc + bias
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                 // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {}                  // ternary args : multiply_add
            },                    // end ternary op
            {} // ternary args : multiply_add
          },  // end ternary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1316-1318

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1320-1352

```cpp
// Sm120 Ptr array tma warp specialized callbacks just alias to their sm90 counterpart
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  int NumEpilogueWarpGroups,
  class Operation,
  class CtaTile_MNK,
  class EpilogueTile_MN,
  class... Args
>
struct FusionCallbacks<
    epilogue::Sm120PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore, NumEpilogueWarpGroups>,
    Operation,
    CtaTile_MNK,
    EpilogueTile_MN,
    Args...
> : FusionCallbacks<
      epilogue::Sm90PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore, NumEpilogueWarpGroups>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...
    > {
  using FusionCallbacks<
      epilogue::Sm90PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore, NumEpilogueWarpGroups>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...>::FusionCallbacks;
};
```

**EN:** Declares the templated `Operation` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Sm120 Ptr array tma warp specialized callbacks just alias to their sm90 counterpart.

**CN:** 声明模板类型 `Operation`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1354-1378

```cpp
// For Ptr-Array and Grouped GEMM
// D = alpha * acc + beta * C, where alpha and beta can be vectors for each batch/group
// With Row BlockScaleFactor Generation, separate tensors per batch/group.
template<
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor, 
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinearCombRowBlockScaleFactorPtrArray =
  Sm90EVT<
    Sm120BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor *, RoundStyle
    >, // gen scalefactor
    Sm90LinearCombinationPtrArray< ElementCompute, ElementCompute, 
      ElementSource, ElementScalar, RoundStyle
    > // beta * C + (alpha * acc)
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: For Ptr-Array and Grouped GEMM D = alpha * acc + beta * C, where alpha and beta can be vectors for each batch/group With Row BlockScaleFactor Generation, separate tensors per batch/group.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1380-1410

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  int NumEpilogueWarpGroups,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore, NumEpilogueWarpGroups>,
    fusion::LinCombBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementSource, ElementScalar, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinearCombRowBlockScaleFactorPtrArray<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle
    > {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1412-1417

```cpp
  using Impl =
    Sm120LinearCombRowBlockScaleFactorPtrArray<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1419-1424

```cpp
  using Operation =
    fusion::LinCombBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementSource, ElementScalar, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1426-1433

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementScalar const* const* alpha_ptr_array = nullptr;
    ElementScalar const* const* beta_ptr_array = nullptr;
    ElementBlockScaleFactor ** block_scale_factor_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1435-1439

```cpp
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1441-1444

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1447-1463

```cpp
    operator typename Impl::Arguments() const {
      return
        {
            {    // ternary op : beta * C + (alpha * acc + bias)
              {{beta}, {beta_ptr}, {beta_ptr_array}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // ternary op : alpha * acc + bias
                {{alpha}, {alpha_ptr}, {alpha_ptr_array}, {dAlpha}}, // leaf args : alpha
                {},                 // leaf args : acc
                {}                  // ternary args : multiply_add
              },                    // end ternary op
              {} // ternary args : multiply_add
            },   // end ternary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1465-1467

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1470-1495

```cpp
// For Ptr-Array and Grouped GEMM
// D = activation(alpha * acc + beta * C), where alpha and beta can be vectors for each batch/group
// With Row BlockScaleFactor Generation, separate tensors per batch/group.
template<
  int SFVecsize,
  class EpilogueTile,
  class CtaTileShapeMNK,
  int FragmentSize,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor, 
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm120LinCombEltActRowBlockScaleFactorPtrArray =
  Sm90EVT<
    Sm120BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ElementOutput,
      ElementCompute, ElementBlockScaleFactor *, RoundStyle
    >, // gen scalefactor
    Sm90LinCombEltActPtrArray<ActivationFn, ElementCompute, ElementCompute, 
      ElementSource, ElementScalar, RoundStyle
    > // activation(beta * C + (alpha * acc))
  >;
```

**EN:** Declares the templated `EpilogueTile` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: For Ptr-Array and Grouped GEMM D = activation(alpha * acc + beta * C), where alpha and beta can be vectors for each batch/group With Row BlockScaleFactor Generation, separate tensors per batch/group.

**CN:** 声明模板类型 `EpilogueTile`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1497-1528

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  int NumEpilogueWarpGroups,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  int SFVecSize,
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm120PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore, NumEpilogueWarpGroups>,
    fusion::LinCombEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementSource, ElementScalar, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm120LinCombEltActRowBlockScaleFactorPtrArray<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1530-1535

```cpp
  using Impl =
    Sm120LinCombEltActRowBlockScaleFactorPtrArray<
      SFVecSize, EpilogueTile, CtaTileShapeMNK, FragmentSize, ActivationFn, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
      ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1537-1542

```cpp
  using Operation =
    fusion::LinCombEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute,
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementSource, ElementScalar, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1544-1551

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementScalar const* const* alpha_ptr_array = nullptr;
    ElementScalar const* const* beta_ptr_array = nullptr;
    ElementBlockScaleFactor ** block_scale_factor_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1553-1557

```cpp
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1559-1562

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1564-1565

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1567-1586

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {    // unary op : activation(beta * C + (alpha * acc + bias))
            {    // ternary op : beta * C + (alpha * acc + bias)
              {{beta}, {beta_ptr}, {beta_ptr_array}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // ternary op : alpha * acc + bias
                {{alpha}, {alpha_ptr}, {alpha_ptr_array}, {dAlpha}}, // leaf args : alpha
                {},                 // leaf args : acc
                {}                  // ternary args : multiply_add
              },                    // end ternary op
              {} // ternary args : multiply_add
            },   // end ternary op
            activation // unary args : activation
          },   // end unary op
          {block_scale_factor_ptr, norm_constant_ptr, dNormConst} // BlockScaleFactor args
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1588-1591

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
} // namespace cutlass::epilogue::fusion
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


## Key Concepts / 关键概念

- **Fusion framework / 融合框架:** Uses callbacks, visitors, or operation tags to compose multiple post-processing steps into the epilogue. / 通过回调、访问者或操作标签把多个后处理步骤组合进 epilogue。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。

- **Scaling semantics / 缩放语义:** Tracks how alpha/beta parameters mix accumulator values with source tensors during output generation. / 描述 alpha/beta 参数如何在生成输出时混合累加器值与源张量。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cute/tensor.hpp`, `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm100_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm120_visitor_store_tma_warpspecialized.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::fusion`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm100_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm120_visitor_store_tma_warpspecialized.hpp`

- **Cute/CuTe dependencies / Cute 依赖:** `cute/tensor.hpp`
