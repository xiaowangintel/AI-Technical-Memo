# sm100_callbacks_tma_warpspecialized.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/fusion/sm100_callbacks_tma_warpspecialized.hpp`

- **Purpose (EN):** Fusion callbacks specializations for the sm100 TMA warp-specialized (ws) epilogue.

- **作用 (CN):** 实现 `SM100 callbacks TMA warpspecialized` 相关的融合回调接口与回调连接逻辑。


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


### Lines 32-34

```cpp
/*! \file
  \brief Fusion callbacks specializations for the sm100 TMA warp-specialized (ws) epilogue
*/
```

**EN:** This documentation block explains the intent of the next declaration: ! Fusion callbacks specializations for the sm100 TMA warp-specialized (ws) epilogue.

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


### Lines 43-45

```cpp
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/epilogue/fusion/callbacks.hpp"
#include "cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/epilogue/dispatch_policy.hpp`，`cutlass/epilogue/fusion/callbacks.hpp`，`cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 47-48

```cpp
#include "cutlass/epilogue/fusion/sm100_visitor_compute_tma_warpspecialized.hpp"  
#include "cutlass/epilogue/fusion/sm100_visitor_store_tma_warpspecialized.hpp" 
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/epilogue/fusion/sm100_visitor_compute_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm100_visitor_store_tma_warpspecialized.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/epilogue/fusion/sm100_visitor_compute_tma_warpspecialized.hpp`，`cutlass/epilogue/fusion/sm100_visitor_store_tma_warpspecialized.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 52

```cpp
namespace cutlass::epilogue::fusion {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 56-87

```cpp
// Sm100 Tma warp specialized callbacks just alias to their sm90 counterpart
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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
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

**EN:** Declares the templated `Operation` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Sm100 Tma warp specialized callbacks just alias to their sm90 counterpart.

**CN:** 声明模板类型 `Operation`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 89-116

```cpp
// Sm100 direct store callbacks alias to sm100 tma callbacks with 0 stages
// Additional copy atom args will be ignored in the 0-stage specializations of aux load/store nodes
template <
  class Operation,
  class CtaTile_MNK,
  class EpilogueTile_MN,
  class... Args
>
struct FusionCallbacks<
    epilogue::Sm100NoSmemWarpSpecialized,
    Operation,
    CtaTile_MNK,
    EpilogueTile_MN,
    Args...
> : FusionCallbacks<
      epilogue::Sm100TmaWarpSpecialized<0, 0, 0, false, false>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...
    > {
  using FusionCallbacks<
      epilogue::Sm100TmaWarpSpecialized<0, 0, 0, false, false>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...>::FusionCallbacks;
};
```

**EN:** Declares the templated `Operation` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Sm100 direct store callbacks alias to sm100 tma callbacks with 0 stages Additional copy atom args will be ignored in the 0-stage specializations of aux load/store nodes.

**CN:** 声明模板类型 `Operation`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 118-149

```cpp
// Sm100 Ptr array tma warp specialized callbacks just alias to their sm90 counterpart
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
    epilogue::Sm100PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    Operation,
    CtaTile_MNK,
    EpilogueTile_MN,
    Args...
> : FusionCallbacks<
      epilogue::Sm90PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore, 1>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...
    > {
  using FusionCallbacks<
      epilogue::Sm90PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore, 1>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...>::FusionCallbacks;
};
```

**EN:** Declares the templated `Operation` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Sm100 Ptr array tma warp specialized callbacks just alias to their sm90 counterpart.

**CN:** 声明模板类型 `Operation`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 153-168

```cpp
// D = alpha * acc + beta * C
// With Row BlockScaleFactor Generation.
template<
  int SFVecsize,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor, 
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm100LinearCombRowBlockScaleFactor =
  Sm90EVT<Sm100BlockScaleFactorRowStore<SFVecsize, EpilogueTile, ElementOutput, ElementCompute, ElementBlockScaleFactor, RoundStyle>, // gen scalefactor
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C With Row BlockScaleFactor Generation.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 170-191

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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute, ElementBlockScaleFactor, cutlass::layout::RowMajor, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinearCombRowBlockScaleFactor<SFVecSize, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 193-194

```cpp
  using Impl =  Sm100LinearCombRowBlockScaleFactor<SFVecSize, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute, ElementBlockScaleFactor, cutlass::layout::RowMajor, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 196-206

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


### Lines 208-211

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 213-230

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
              {},                     // leaf args : acc
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


### Lines 232-234

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 236-251

```cpp
// D = alpha * acc + beta * C
// With Col BlockScaleFactor Generation.
template<
  int SFVecsize,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor, 
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm100LinearCombColBlockScaleFactor =
  Sm90EVT<Sm100BlockScaleFactorColStore<SFVecsize, EpilogueTile, ElementOutput, ElementCompute, ElementBlockScaleFactor, RoundStyle>, // gen scalefactor
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C With Col BlockScaleFactor Generation.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 253-274

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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute, ElementBlockScaleFactor, cutlass::layout::ColumnMajor, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinearCombColBlockScaleFactor<SFVecSize, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 276-277

```cpp
  using Impl =  Sm100LinearCombColBlockScaleFactor<SFVecSize, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute, ElementBlockScaleFactor, cutlass::layout::ColumnMajor,  ElementSource, ElementScalar, RoundStyle>;  
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 279-289

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


### Lines 291-294

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 296-313

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
              {},                     // leaf args : acc
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


### Lines 315-317

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 321-337

```cpp
// For Ptr-Array and Grouped GEMM
// D = alpha * acc + beta * C, where alpha and beta can be vectors for each batch/group
// With Row BlockScaleFactor Generation, separate tensors per batch/group.
template<
  int SFVecsize,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor, 
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm100LinearCombRowBlockScaleFactorPtrArray =
  Sm90EVT<Sm100BlockScaleFactorRowStore<SFVecsize, EpilogueTile, ElementOutput, ElementCompute, ElementBlockScaleFactor *, RoundStyle>, // gen scalefactor
    Sm90LinearCombinationPtrArray<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Defines `EpilogueTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: For Ptr-Array and Grouped GEMM D = alpha * acc + beta * C, where alpha and beta can be vectors for each batch/group With Row BlockScaleFactor Generation, separate tensors per batch/group.

**CN:** 定义 `EpilogueTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 339-360

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
    epilogue::Sm100PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute, ElementBlockScaleFactor, cutlass::layout::RowMajor, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinearCombRowBlockScaleFactorPtrArray<SFVecSize, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 362-363

```cpp
  using Impl =  Sm100LinearCombRowBlockScaleFactorPtrArray<SFVecSize, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombBlockScaleFactor<SFVecSize, ElementOutput, ElementCompute, ElementBlockScaleFactor, cutlass::layout::RowMajor, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 365-378

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementScalar const* const* alpha_ptr_array = nullptr;
    ElementScalar const* const* beta_ptr_array = nullptr;
    ElementBlockScaleFactor ** block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    // NormConst is a single device-side constant value, its not per-batch or per-group
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 380-383

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 385-402

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {
            // ternary op : beta * C + (alpha * acc)
            {{beta}, {beta_ptr}, {beta_ptr_array}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // binary op : alpha * acc
              {{alpha}, {alpha_ptr}, {alpha_ptr_array}, {dAlpha}}, // leaf args : alpha
              {},                     // leaf args : acc
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


### Lines 404-406

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 410-427

```cpp
// For Ptr-Array and Grouped GEMM
// D = activation(alpha * acc + beta * C), where alpha and beta can be vectors for each batch/group
// With Row BlockScaleFactor Generation, separate tensors per batch/group.
template<
  int SFVecsize,
  class EpilogueTile,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor, 
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm100LinCombEltActRowBlockScaleFactorPtrArray =
  Sm90EVT<Sm100BlockScaleFactorRowStore<SFVecsize, EpilogueTile, ElementOutput, ElementCompute, ElementBlockScaleFactor *, RoundStyle>, // gen scalefactor
    Sm90LinCombEltActPtrArray<ActivationFn, ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // activation(beta * C + (alpha * acc))
  >;
```

**EN:** Declares the templated `EpilogueTile` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: For Ptr-Array and Grouped GEMM D = activation(alpha * acc + beta * C), where alpha and beta can be vectors for each batch/group With Row BlockScaleFactor Generation, separate tensors per batch/group.

**CN:** 声明模板类型 `EpilogueTile`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 429-451

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
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm100PtrArrayTmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombEltActBlockScaleFactor<ActivationFn, SFVecSize, ElementOutput, ElementCompute, ElementBlockScaleFactor, cutlass::layout::RowMajor, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinCombEltActRowBlockScaleFactorPtrArray<SFVecSize, EpilogueTile, ActivationFn, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 453-454

```cpp
  using Impl =  Sm100LinCombEltActRowBlockScaleFactorPtrArray<SFVecSize, EpilogueTile, ActivationFn, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementBlockScaleFactor, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombEltActBlockScaleFactor<ActivationFn, SFVecSize, ElementOutput, ElementCompute, ElementBlockScaleFactor, cutlass::layout::RowMajor, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 456-468

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementScalar const* const* alpha_ptr_array = nullptr;
    ElementScalar const* const* beta_ptr_array = nullptr;
    ElementBlockScaleFactor ** block_scale_factor_ptr = nullptr;
    // A matrix wide constant value to scale the output matrix
    // Avoids generating small FP4 values.
    using StrideNormConst = Stride<_0,_0,int64_t>;
    ElementCompute const* norm_constant_ptr = nullptr;
    StrideNormConst dNormConst = {_0{}, _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 470-473

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 475-476

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 478-497

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {    // unary op: activation(beta * C + (alpha * acc))
            {    // ternary op : beta * C + (alpha * acc)
              {{beta}, {beta_ptr}, {beta_ptr_array}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // binary op : alpha * acc
                {{alpha}, {alpha_ptr}, {alpha_ptr_array}, {dAlpha}}, // leaf args : alpha
                {},                     // leaf args : acc
                {}                  // binary args : multiplies
              },                    // end binary op
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


### Lines 499-501

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 505-531

```cpp
// D = alpha * acc + beta * C + per-row bias
//   with row blockScaled generation
template<
  int SFVecsize,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm100LinCombPerRowBiasRowBlockScaleFactor =
  Sm90EVT<
    Sm100BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, ElementOutput, 
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerRowBias<
      CtaTileShapeMNK, ElementCompute, ElementCompute, 
      ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per-row bias with row blockScaled generation.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 533-569

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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinCombPerRowBiasRowBlockScaleFactor<
      SFVecSize, CtaTileShapeMNK, EpilogueTile, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, 
      ElementSource, 
      ElementScalar, 
      AlignmentBias,
       RoundStyle
    > 
{
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 571-577

```cpp
  using Impl = 
    Sm100LinCombPerRowBiasRowBlockScaleFactor<
      SFVecSize, CtaTileShapeMNK, EpilogueTile, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 579-584

```cpp
  using Operation = 
    fusion::LinCombPerRowBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::RowMajor, 
      ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 586-596

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


### Lines 598-601

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 603-605

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 607-624

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {  // ternary op : beta * C + (alpha * acc + bias)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // ternary op : alpha * acc + bias
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                     // leaf args : acc
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


### Lines 626-628

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 631-657

```cpp
// D = alpha * acc + beta * C + per-row bias
//   with col blockScaled generation
template<
  int SFVecsize,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm100LinCombPerRowBiasColBlockScaleFactor =
  Sm90EVT<
    Sm100BlockScaleFactorColStore<
      SFVecsize, EpilogueTile, ElementOutput, 
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerRowBias<
      CtaTileShapeMNK, ElementCompute, ElementCompute, 
      ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per-row bias with col blockScaled generation.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 659-693

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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor,
      ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinCombPerRowBiasColBlockScaleFactor<
      SFVecSize, CtaTileShapeMNK, EpilogueTile, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > 
{
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 695-701

```cpp
  using Impl = 
    Sm100LinCombPerRowBiasColBlockScaleFactor<
      SFVecSize, CtaTileShapeMNK, EpilogueTile, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 703-708

```cpp
  using Operation = 
    fusion::LinCombPerRowBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor, 
      ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 710-720

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


### Lines 722-725

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 727-729

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 731-748

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {  // ternary op : beta * C + (alpha * acc + bias)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // ternary op : alpha * acc + bias
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                     // leaf args : acc
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


### Lines 750-752

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 756-783

```cpp
// D = alpha * acc + beta * C + per_col bias
//   with row blockScaled generation
template<
  int StagesC,
  int SFVecsize,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementBlockScaleFactor,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm100LinCombPerColBiasRowBlockScaleFactor =
  Sm90EVT<
    Sm100BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, ElementOutput, 
      ElementCompute, ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerColBias<
      StagesC, CtaTileShapeMNK, EpilogueTile, ElementCompute, ElementCompute, 
      ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per_col bias with row blockScaled generation.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 785-819

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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBiasBlockScaleFactor<
      SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementBias, ElementSource, 
      ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinCombPerColBiasRowBlockScaleFactor<
      StagesC, SFVecSize, CtaTileShapeMNK, EpilogueTile, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > 
{
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 821-827

```cpp
  using Impl = 
    Sm100LinCombPerColBiasRowBlockScaleFactor<
      StagesC, SFVecSize, CtaTileShapeMNK, EpilogueTile, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 829-835

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


### Lines 837-847

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


### Lines 849-852

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 855-857

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 859-876

```cpp
    operator typename Impl::Arguments() const {
      return
        {
          {  // ternary op : beta * C + (alpha * acc + bias)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // ternary op : alpha * acc + bias
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                     // leaf args : acc
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


### Lines 878-880

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 884-912

```cpp
// D = activation(alpha * acc + beta * C + per-row bias) 
//   with row blockScaled generation
template<
  int SFVecsize,
  class CtaTileShapeMNK,
  class EpilogueTile,
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
using Sm100LinCombPerRowBiasEltActRowBlockScaleFactor =
  Sm90EVT<
    Sm100BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, 
      ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, 
      ElementCompute, ElementCompute, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-row bias) with row blockScaled generation.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 914-947

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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::RowMajor, 
      ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinCombPerRowBiasEltActRowBlockScaleFactor<
      SFVecSize, CtaTileShapeMNK, EpilogueTile, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 949-955

```cpp
  using Impl = 
    Sm100LinCombPerRowBiasEltActRowBlockScaleFactor<
      SFVecSize, CtaTileShapeMNK, EpilogueTile, ActivationFn, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 957-962

```cpp
  using Operation = 
    fusion::LinCombPerRowBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::RowMajor, 
      ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 964-974

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


### Lines 976-979

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 981-983

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 985-986

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 988-1008

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
                {},                     // leaf args : acc
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


### Lines 1010-1012

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1016-1044

```cpp
// D = activation(alpha * acc + beta * C + per-row bias) 
//   with col blockScaled generation
template<
  int SFVecsize,
  class CtaTileShapeMNK,
  class EpilogueTile,
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
using Sm100LinCombPerRowBiasEltActColBlockScaleFactor =
  Sm90EVT<
    Sm100BlockScaleFactorColStore<
      SFVecsize, EpilogueTile, 
      ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, 
      ElementCompute, ElementCompute, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-row bias) with col blockScaled generation.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1046-1079

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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor, 
      ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinCombPerRowBiasEltActColBlockScaleFactor<
      SFVecSize, CtaTileShapeMNK, EpilogueTile, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1081-1087

```cpp
  using Impl = 
    Sm100LinCombPerRowBiasEltActColBlockScaleFactor<
      SFVecSize, CtaTileShapeMNK, EpilogueTile, ActivationFn, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1089-1094

```cpp
  using Operation = 
    fusion::LinCombPerRowBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::ColumnMajor, 
      ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1096-1106

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


### Lines 1108-1111

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1114-1116

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1118-1119

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1121-1141

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
                {},                     // leaf args : acc
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


### Lines 1143-1145

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1149-1178

```cpp
// D = activation(alpha * acc + beta * C + per_col bias) 
//   with row blockScaled generation
template<
  int StagesC,
  int SFVecsize,
  class CtaTileShapeMNK,
  class EpilogueTile,
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
using Sm100LinCombPerColBiasEltActRowBlockScaleFactor =
  Sm90EVT<
    Sm100BlockScaleFactorRowStore<
      SFVecsize, EpilogueTile, 
      ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, RoundStyle
    >,
    Sm90LinCombPerColBiasEltAct<
      StagesC, CtaTileShapeMNK, EpilogueTile, ActivationFn, 
      ElementCompute, ElementCompute, ElementBias, 
      ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per_col bias) with row blockScaled generation.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1180-1214

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
    epilogue::Sm100TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBiasEltActBlockScaleFactor<
      ActivationFn, SFVecSize, ElementOutput, ElementCompute, 
      ElementBlockScaleFactor, cutlass::layout::RowMajor,
      ElementBias, ElementSource, 
      ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm100LinCombPerColBiasEltActRowBlockScaleFactor<
      StagesC, SFVecSize, CtaTileShapeMNK, EpilogueTile, ActivationFn,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1216-1222

```cpp
  using Impl = 
    Sm100LinCombPerColBiasEltActRowBlockScaleFactor<
      StagesC, SFVecSize, CtaTileShapeMNK, EpilogueTile, ActivationFn, 
      typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, 
      ElementCompute, ElementBlockScaleFactor, ElementBias, ElementSource, ElementScalar, 
      AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1224-1230

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


### Lines 1232-1242

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


### Lines 1244-1247

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1249-1251

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1253-1254

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1256-1276

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
                {},                     // leaf args : acc
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


### Lines 1278-1280

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1283-1304

```cpp
// --------------------------------------------------------------------
//  Sm100PtrArrayNoSmemWarpSpecialized  (direct-store, grouped GEMM)
// --------------------------------------------------------------------
template <
    class Operation,
    class CtaTile_MNK,
    class EpilogueTile_MN,
    class... Args
>
struct FusionCallbacks<
        epilogue::Sm100PtrArrayNoSmemWarpSpecialized,
        Operation,
        CtaTile_MNK,
        EpilogueTile_MN,
        Args...>
  : FusionCallbacks<
        // reuse the ptr-array *TMA* callbacks with 0 stages
        epilogue::Sm100PtrArrayTmaWarpSpecialized<0,0,0,false,false>,
        Operation,
        CtaTile_MNK,
        EpilogueTile_MN,
        Args...> {
```

**EN:** Declares the templated `Operation` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: -------------------------------------------------------------------- Sm100PtrArrayNoSmemWarpSpecialized (direct-store, grouped GEMM) --------------------------------------------------------------------.

**CN:** 声明模板类型 `Operation`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1306-1311

```cpp
  using Base = FusionCallbacks<
      epilogue::Sm100PtrArrayTmaWarpSpecialized<0,0,0,false,false>,
      Operation,
      CtaTile_MNK,
      EpilogueTile_MN,
      Args...>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1313-1315

```cpp
  // bring ctors into scope
  using Base::Base;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


## Key Concepts / 关键概念

- **Fusion framework / 融合框架:** Uses callbacks, visitors, or operation tags to compose multiple post-processing steps into the epilogue. / 通过回调、访问者或操作标签把多个后处理步骤组合进 epilogue。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。

- **Scaling semantics / 缩放语义:** Tracks how alpha/beta parameters mix accumulator values with source tensors during output generation. / 描述 alpha/beta 参数如何在生成输出时混合累加器值与源张量。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cute/tensor.hpp`, `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm100_visitor_compute_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm100_visitor_store_tma_warpspecialized.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::fusion`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm100_visitor_compute_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm100_visitor_store_tma_warpspecialized.hpp`

- **Cute/CuTe dependencies / Cute 依赖:** `cute/tensor.hpp`
