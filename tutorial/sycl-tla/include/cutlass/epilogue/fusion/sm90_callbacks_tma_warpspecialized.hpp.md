# sm90_callbacks_tma_warpspecialized.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`

- **Purpose (EN):** Fusion callbacks specializations for the sm90 TMA warp-specialized (ws) epilogue.

- **作用 (CN):** 实现 `SM90 callbacks TMA warpspecialized` 相关的融合回调接口与回调连接逻辑。


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
  \brief Fusion callbacks specializations for the sm90 TMA warp-specialized (ws) epilogue
*/
```

**EN:** This documentation block explains the intent of the next declaration: ! Fusion callbacks specializations for the sm90 TMA warp-specialized (ws) epilogue.

**CN:** 这一段文档注释说明了紧随其后的声明意图，帮助理解后续模板或实现要解决的问题。


### Line 36

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Line 38

```cpp
#include "cutlass/cutlass.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 40

```cpp
#include "cute/tensor.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cute/tensor.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cute/tensor.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 42-47

```cpp
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/epilogue/fusion/callbacks.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_store_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_compute_tma_warpspecialized.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp`, and 2 more headers. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/epilogue/dispatch_policy.hpp`，`cutlass/epilogue/fusion/callbacks.hpp`，`cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`，`cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp`，以及另外 2 个头文件。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 49

```cpp
#include "cutlass/epilogue/fusion/sm90_visitor_topk_softmax.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/epilogue/fusion/sm90_visitor_topk_softmax.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/epilogue/fusion/sm90_visitor_topk_softmax.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 53

```cpp
namespace cutlass::epilogue::fusion {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 57-58

```cpp
template <class NodeOp, class... ChildOps>
using Sm90EVT = Sm90TreeVisitor<NodeOp, ChildOps...>;
```

**EN:** Declares the templated `NodeOp` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `NodeOp`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 60-61

```cpp
template <class NodeOp, class... ChildOps>
using XeEVT = Sm90EVT<NodeOp, ChildOps...>;
```

**EN:** Declares the templated `NodeOp` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `NodeOp`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 63-91

```cpp
// D = alpha * acc
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::ScaledAcc<ElementOutput, ElementCompute, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90EVT<Sm90Compute<multiplies, ElementOutput, ElementCompute, RoundStyle>,
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, 
      Sm90AccFetch
    > {
  using Impl = 
    Sm90EVT<Sm90Compute<multiplies, ElementOutput, ElementCompute, RoundStyle>,
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>,
      Sm90AccFetch
    >;
  using Operation = fusion::ScaledAcc<ElementOutput, ElementCompute, ElementScalar, RoundStyle>;
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = alpha * acc.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 93-98

```cpp
  struct Arguments {
    // Give a name and flat ordering to the fusion callback args
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 100-101

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 103-113

```cpp
    // Conversion to the args expected by the visitor implementation
    // to_underlying_arguments will implicitly call this
    operator typename Impl::Arguments() const {
      return
        {    // binary op : alpha * acc
          {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
          {},                     // leaf args : acc
          {} // binary args : multiplies
        };   // end binary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 115-117

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 121-137

```cpp
// D = alpha * acc + beta * C
template<
  class ElementOutput,
  class ElementCompute,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinearCombination =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc)
    Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // beta
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<multiplies, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // alpha
      Sm90AccFetch // acc
    >
  >;
```

**EN:** Defines `ElementOutput`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C.

**CN:** 定义 `ElementOutput`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 139-158

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinearCombination<ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinearCombination<typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 160-161

```cpp
  using Impl = Sm90LinearCombination<typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinearCombination<ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 163-167

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 169-172

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 174-187

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // ternary op : beta * C + (alpha * acc)
          {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
          {},                   // leaf args : C
          {                     // binary op : alpha * acc
            {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
            {},                     // leaf args : acc
            {}                  // binary args : multiplies
          },                    // end binary op
          {} // ternary args : multiply_add
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 189-191

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 195-211

```cpp
// D = alpha * acc + beta * C, where beta and alpha can be vectors for each batch
template<
  class ElementOutput,
  class ElementCompute,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinearCombinationPtrArray =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc)
    Sm90ScalarBroadcastPtrArray<ElementScalar, Stride<_0,_0,int64_t>>, // beta
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<multiplies, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc
      Sm90ScalarBroadcastPtrArray<ElementScalar, Stride<_0,_0,int64_t>>, // alpha
      Sm90AccFetch // acc
    >
  >;
```

**EN:** Defines `ElementOutput`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C, where beta and alpha can be vectors for each batch.

**CN:** 定义 `ElementOutput`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 213-239

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
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90PtrArrayTmaWarpSpecialized<StagesC, 
                                             StagesD, 
                                             FragmentSize, 
                                             ReuseSmemC, 
                                             DelayTmaStore, 
                                             NumEpilogueWarpGroups
                                            >,
    fusion::LinearCombination<ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinearCombinationPtrArray<typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 241-242

```cpp
  using Impl = Sm90LinearCombinationPtrArray<typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinearCombination<ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 244-250

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementScalar const* const* alpha_ptr_array = nullptr;
    ElementScalar const* const* beta_ptr_array = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 252-255

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 257-270

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // ternary op : beta * C + (alpha * acc)
          {{beta}, {beta_ptr}, {beta_ptr_array}, {dBeta}}, // leaf args : beta
          {},                   // leaf args : C
          {                     // binary op : alpha * acc
            {{alpha}, {alpha_ptr}, {alpha_ptr_array}, {dAlpha}}, // leaf args : alpha
            {},                     // leaf args : acc
            {}                  // binary args : multiplies
          },                    // end binary op
          {} // ternary args : multiply_add
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 272-274

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 278-290

```cpp
// D = activation(alpha * acc + beta * C)
template<
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombEltAct =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>, // activation(beta * C + (alpha * acc))
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C).

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 292-312

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
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombEltAct<ActivationFn, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinCombEltAct<ActivationFn, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 314-315

```cpp
  using Impl = Sm90LinCombEltAct<ActivationFn, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombEltAct<ActivationFn, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 317-321

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 323-326

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 328-329

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 331-347

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // unary op: activation(beta * C + (alpha * acc))
          {    // ternary op : beta * C + (alpha * acc)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // binary op : alpha * acc
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                     // leaf args : acc
              {}                  // binary args : multiplies
            },                    // end binary op
            {} // ternary args : multiply_add
          },   // end ternary op
          activation // unary args: activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 349-351

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 355-367

```cpp
// D = activation(alpha * acc + beta * C), where beta and alpha can be vectors for each batch
template<
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombEltActPtrArray =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>, // activation(beta * C + (alpha * acc))
    Sm90LinearCombinationPtrArray<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C), where beta and alpha can be vectors for each batch.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 369-396

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
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90PtrArrayTmaWarpSpecialized<StagesC, 
                                             StagesD, 
                                             FragmentSize, 
                                             ReuseSmemC, 
                                             DelayTmaStore, 
                                             NumEpilogueWarpGroups
                                            >,
    fusion::LinCombEltAct<ActivationFn, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinCombEltActPtrArray<ActivationFn, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 398-399

```cpp
  using Impl = Sm90LinCombEltActPtrArray<ActivationFn, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombEltAct<ActivationFn, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 401-407

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementScalar const* const* alpha_ptr_array = nullptr;
    ElementScalar const* const* beta_ptr_array = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 409-412

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 414-415

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 417-433

```cpp
    operator typename Impl::Arguments() const {
      return
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
          activation // unary args: activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 435-437

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 441-461

```cpp
// D = alpha * acc + beta * C + per-row bias
template<
  class CtaTileShapeMNK,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombPerRowBias =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc + bias)
    Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // beta
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc + bias
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // alpha
      Sm90AccFetch, // acc
      Sm90ColBroadcast<0, CtaTileShapeMNK, ElementBias, ElementCompute, Stride<_1,_0,int64_t>, AlignmentBias> // bias
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per-row bias.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 463-489

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBias<ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinCombPerRowBias<
      CtaTileShapeMNK, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle> {
  using Impl = Sm90LinCombPerRowBias<
    CtaTileShapeMNK, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>;
  using Operation = fusion::LinCombPerRowBias<
    ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>;
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 491-495

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 497-500

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 502-504

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 506-520

```cpp
    operator typename Impl::Arguments() const {
      return
        {     // ternary op : beta * C + (alpha * acc + bias)
          {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
          {},                   // leaf args : C
          {                     // ternary op : alpha * acc + bias
            {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
            {},                     // leaf args : acc
            {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
            {}                  // ternary args : multiply_add
          },                    // end ternary op
          {} // ternary args : multiply_add
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 522-524

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 528-550

```cpp
// D = alpha * acc + beta * C + per-column bias
template<
  int StagesC,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombPerColBias =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc + bias)
    Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // beta
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc + bias
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // alpha
      Sm90AccFetch, // acc
      Sm90RowBroadcast<0, CtaTileShapeMNK, ElementBias, ElementCompute, Stride<_0,_1,int64_t>, AlignmentBias> // bias
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per-column bias.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 552-578

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBias<ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinCombPerColBias<
      StagesC, CtaTileShapeMNK, EpilogueTile, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle> {
  using Impl = Sm90LinCombPerColBias<
    StagesC, CtaTileShapeMNK, EpilogueTile, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>;
  using Operation = fusion::LinCombPerColBias<
    ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>;
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 580-584

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 586-589

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 591-593

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 595-609

```cpp
    operator typename Impl::Arguments() const {
      return
        {     // ternary op : beta * C + (alpha * acc + bias)
          {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
          {},                   // leaf args : C
          {                     // ternary op : alpha * acc + bias
            {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
            {},                     // leaf args : acc
            {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
            {}                  // ternary args : multiply_add
          },                    // end ternary op
          {} // ternary args : multiply_add
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 611-613

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 617-632

```cpp
// D = activation(alpha * acc + beta * C + per-row bias)
template<
  class CtaTileShapeMNK,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombPerRowBiasEltAct =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
    Sm90LinCombPerRowBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-row bias).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 634-660

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
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 662-669

```cpp
  using Impl =
    Sm90LinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::LinCombPerRowBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 671-675

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 677-680

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 682-684

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 686-687

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 689-706

```cpp
    operator typename Impl::Arguments() const {
      return
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
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 708-710

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 714-731

```cpp
// D = activation(alpha * acc + beta * C + per-column bias)
template<
  int StagesC,
  class CtaTileShapeMNK,
  class EpilogueTile,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombPerColBiasEltAct =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
    Sm90LinCombPerColBias<StagesC, CtaTileShapeMNK, EpilogueTile, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-column bias).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 733-759

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
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinCombPerColBiasEltAct<
      StagesC, CtaTileShapeMNK, EpilogueTile, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 761-768

```cpp
  using Impl =
    Sm90LinCombPerColBiasEltAct<
      StagesC, CtaTileShapeMNK, EpilogueTile, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::LinCombPerColBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 770-774

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 776-779

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 781-783

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 785-786

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 788-805

```cpp
    operator typename Impl::Arguments() const {
      return
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
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 807-809

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 813-838

```cpp
// D = activation(alpha * acc + beta * C + per-row bias)
// Aux = alpha * acc + beta * C + per-row bias)
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int Stages,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpR2S,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombPerRowBiasEltActAux =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
    Sm90EVT<Sm90AuxStore<Stages, EpilogueTile, ElementAux, RoundStyle, StrideAux, SmemLayoutAtom, CopyOpR2S, AlignmentAux>,
      Sm90LinCombPerRowBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-row bias) Aux = alpha * acc + beta * C + per-row bias).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 840-875

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class SmemLayoutAtom,
  class CopyOpR2S
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerRowBiasEltActAux<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    SmemLayoutAtom,
    CopyOpR2S
> : Sm90LinCombPerRowBiasEltActAux<
      CtaTileShapeMNK, EpilogueTile, StagesD, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, SmemLayoutAtom, CopyOpR2S, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 877-886

```cpp
  using Impl =
    Sm90LinCombPerRowBiasEltActAux<
      CtaTileShapeMNK, EpilogueTile, StagesD, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, SmemLayoutAtom, CopyOpR2S, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::LinCombPerRowBiasEltActAux<
      GmemLayoutTagAux, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 888-892

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 894-897

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 899-901

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 903-904

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 906-908

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 910-930

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // unary op : activation(store(beta * C + (alpha * acc + bias)))
          {                 // unary op : store(beta * C + (alpha * acc + bias))
            {                  // ternary op : beta * C + (alpha * acc + bias)
              {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // ternary op : alpha * acc + bias
                {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
                {},                     // leaf args : acc
                {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
                {}                  // ternary args : multiply_add
              },                    // end ternary op
              {}               // ternary args : multiply_add
            },                 // end ternary op
            {aux_ptr, dAux} // unary args : store
          },                // end unary op
          activation // unary args : activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 932-934

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 937-963

```cpp
// D = activation(alpha * acc + beta * C + per_col bias)
// Aux = alpha * acc + beta * C + per_col bias)
template<
  int StagesC,
  class CtaTileShapeMNK,
  class EpilogueTile,
  int Stages,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpR2S,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombPerColBiasEltActAux =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
    Sm90EVT<Sm90AuxStore<Stages, EpilogueTile, ElementAux, RoundStyle, StrideAux, SmemLayoutAtom, CopyOpR2S, AlignmentAux>,
      Sm90LinCombPerColBias<StagesC, CtaTileShapeMNK, EpilogueTile, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per_col bias) Aux = alpha * acc + beta * C + per_col bias).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 965-1000

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class SmemLayoutAtom,
  class CopyOpR2S
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombPerColBiasEltActAux<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    SmemLayoutAtom,
    CopyOpR2S
> : Sm90LinCombPerColBiasEltActAux<
      StagesC, CtaTileShapeMNK, EpilogueTile, StagesD, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, SmemLayoutAtom, CopyOpR2S, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1002-1011

```cpp
  using Impl =
    Sm90LinCombPerColBiasEltActAux<
      StagesC, CtaTileShapeMNK, EpilogueTile, StagesD, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, SmemLayoutAtom, CopyOpR2S, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::LinCombPerColBiasEltActAux<
      GmemLayoutTagAux, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1013-1017

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1019-1022

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1024-1026

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1028-1029

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1031-1033

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 1035-1055

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // unary op : activation(store(beta * C + (alpha * acc + bias)))
          {                 // unary op : store(beta * C + (alpha * acc + bias))
            {                  // ternary op : beta * C + (alpha * acc + bias)
              {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // ternary op : alpha * acc + bias
                {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
                {},                     // leaf args : acc
                {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
                {}                  // ternary args : multiply_add
              },                    // end ternary op
              {}               // ternary args : multiply_add
            },                 // end ternary op
            {aux_ptr, dAux} // unary args : store
          },                // end unary op
          activation // unary args : activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1057-1059

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1063-1084

```cpp
// D = per-row alpha * acc + per-row beta * C + per-row bias
template<
  class CtaTileShapeMNK,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  int AlignmentScalar = 128 / sizeof_bits_v<ElementScalar>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90PerRowLinCombPerRowBias =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc + bias)
    Sm90ColBroadcast<0, CtaTileShapeMNK, ElementScalar, ElementCompute, Stride<bool,_0,int64_t>, AlignmentScalar>, // beta, dynamic scalar/vector broadcast
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc + bias
      Sm90ColBroadcast<0, CtaTileShapeMNK, ElementScalar, ElementCompute, Stride<bool,_0,int64_t>, AlignmentScalar>, // alpha, dynamic scalar/vector broadcast
      Sm90AccFetch, // acc
      Sm90ColBroadcast<0, CtaTileShapeMNK, ElementBias, ElementCompute, Stride<_1,_0,int64_t>, AlignmentBias> // bias
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = per-row alpha * acc + per-row beta * C + per-row bias.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1086-1103

```cpp
// D = activation(per-row alpha * acc + per-row beta * C + per-row bias)
template<
  class CtaTileShapeMNK,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  int AlignmentScalar = 128 / sizeof_bits_v<ElementScalar>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90PerRowLinCombPerRowBiasEltAct =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
    Sm90PerRowLinCombPerRowBias<CtaTileShapeMNK, ElementCompute, ElementCompute,
                                ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle>
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(per-row alpha * acc + per-row beta * C + per-row bias).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1105-1132

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
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  int AlignmentScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::PerRowLinCombPerRowBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90PerRowLinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1134-1141

```cpp
  using Impl =
    Sm90PerRowLinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >;
  using Operation =
    fusion::PerRowLinCombPerRowBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1143-1151

```cpp
  struct Arguments {
    using StrideAlpha = Stride<bool,_0,int64_t>;
    using StrideBeta  = Stride<bool,_0,int64_t>;
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    StrideAlpha dAlpha = {bool(1), _0{}, 0};
    StrideBeta  dBeta  = {bool(1), _0{}, 0};
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1153-1155

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1157-1158

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1160-1177

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // unary op : activation(beta * C + (alpha * acc + bias))
          {    // ternary op : beta * C + (alpha * acc + bias)
            {beta_ptr, beta, dBeta}, // leaf args : beta
            {},                      // leaf args : C
            {                        // ternary op : alpha * acc + bias
              {alpha_ptr, alpha, dAlpha}, // leaf args : alpha
              {},                         // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {}                     // ternary args : multiply_add
            },                       // end ternary op
            {} // ternary args : multiply_add
          },   // end ternary op
          activation // unary args : activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1179-1181

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1185-1208

```cpp
// D = per-col alpha * acc + per-col beta * C + per-column bias
template<
  int StagesC,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  int AlignmentScalar = 128 / sizeof_bits_v<ElementScalar>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90PerColLinCombPerColBias =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc + bias)
    Sm90RowBroadcast<0, CtaTileShapeMNK, ElementScalar, ElementCompute, Stride<_0,bool,int64_t>, AlignmentScalar>, // beta, dynamic scalar/vector broadcast
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc + bias
      Sm90RowBroadcast<0, CtaTileShapeMNK, ElementScalar, ElementCompute, Stride<_0,bool,int64_t>, AlignmentScalar>, // alpha, dynamic scalar/vector broadcast
      Sm90AccFetch, // acc
      Sm90RowBroadcast<0, CtaTileShapeMNK, ElementBias, ElementCompute, Stride<_0,_1,int64_t>, AlignmentBias> // bias
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = per-col alpha * acc + per-col beta * C + per-column bias.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1210-1229

```cpp
// D = activation(per-col alpha * acc + per-col beta * C + per-column bias)
template<
  int StagesC,
  class CtaTileShapeMNK,
  class EpilogueTile,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  int AlignmentScalar = 128 / sizeof_bits_v<ElementScalar>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90PerColLinCombPerColBiasEltAct =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
    Sm90PerColLinCombPerColBias<StagesC, CtaTileShapeMNK, EpilogueTile, ElementCompute, ElementCompute,
                                ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle>
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(per-col alpha * acc + per-col beta * C + per-column bias).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1231-1258

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
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  int AlignmentScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::PerColLinCombPerColBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90PerColLinCombPerColBiasEltAct<
      StagesC, CtaTileShapeMNK, EpilogueTile, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1260-1267

```cpp
  using Impl =
    Sm90PerColLinCombPerColBiasEltAct<
      StagesC, CtaTileShapeMNK, EpilogueTile, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >;
  using Operation =
    fusion::PerColLinCombPerColBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1269-1273

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1275-1278

```cpp
    using StrideAlpha = Stride<_0,bool,int64_t>;
    using StrideBeta  = Stride<_0,bool,int64_t>;
    StrideAlpha dAlpha = {_0{}, bool(1), 0};
    StrideBeta  dBeta  = {_0{}, bool(1), 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1280-1282

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1284-1285

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1287-1304

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // unary op : activation(beta * C + (alpha * acc + bias))
          {    // ternary op : beta * C + (alpha * acc + bias)
            {beta_ptr, beta, dBeta}, // leaf args : beta
            {},               // leaf args : C
            {                 // ternary op : alpha * acc + bias
              {alpha_ptr, alpha, dAlpha},   // leaf args : alpha
              {},                   // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {}              // ternary args : multiply_add
            },                // end ternary op
            {} // ternary args : multiply_add
          },   // end ternary op
          activation // unary args : activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1306-1308

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1312-1337

```cpp
// D = activation(per-col alpha * acc + per-column bias) + per-col beta * C
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  int AlignmentScalar = 128 / sizeof_bits_v<ElementScalar>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90PerColResAddPerColBiasEltAct =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + activation(alpha * acc + bias)
    Sm90RowBroadcast<0, CtaTileShapeMNK, ElementScalar, ElementCompute, Stride<_0,bool,int64_t>, AlignmentScalar>, // beta, dynamic scalar/vector broadcast
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<ActivationFn, ElementCompute, ElementCompute, RoundStyle>, // activation(alpha * acc + bias)
      Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc + bias
        Sm90RowBroadcast<0, CtaTileShapeMNK, ElementScalar, ElementCompute, Stride<_0,bool,int64_t>, AlignmentScalar>, // alpha, dynamic scalar/vector broadcast
        Sm90AccFetch, // acc
        Sm90RowBroadcast<0, CtaTileShapeMNK, ElementBias, ElementCompute, Stride<_0,_1,int64_t>, AlignmentBias> // bias
      >
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(per-col alpha * acc + per-column bias) + per-col beta * C.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1339-1366

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
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  int AlignmentScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::PerColResAddPerColBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90PerColResAddPerColBiasEltAct<
      CtaTileShapeMNK, EpilogueTile, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1368-1375

```cpp
  using Impl =
    Sm90PerColResAddPerColBiasEltAct<
      CtaTileShapeMNK, EpilogueTile, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >;
  using Operation =
    fusion::PerColResAddPerColBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, AlignmentScalar, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1377-1381

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1383-1386

```cpp
    using StrideAlpha = Stride<_0,bool,int64_t>;
    using StrideBeta  = Stride<_0,bool,int64_t>;
    StrideAlpha dAlpha = {_0{}, bool(1), 0};
    StrideBeta  dBeta  = {_0{}, bool(1), 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1388-1390

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1392-1393

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1395-1412

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // ternary op : beta * C + activation(alpha * acc + bias)
          {beta_ptr, beta, dBeta}, // leaf args : beta
          {},                      // leaf args : C
          {                        // unary op : activation(alpha * acc + bias)
            {                          // ternary op : alpha * acc + bias
              {alpha_ptr, alpha, dAlpha},        // leaf args : alpha
              {},                                // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {}                       // ternary args : multiply_add
            },                         // end ternary op
            activation             // unary args : activation
          },                       // end unary op
          {} // ternary args : multiply_add
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1414-1416

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Line 1420

```cpp
namespace detail {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 1422-1423

```cpp
template <typename T>
constexpr bool is_fp8_v = cute::is_same_v<T,float_e4m3_t> || cute::is_same_v<T,float_e5m2_t>;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 1425-1431

```cpp
// We only apply the scaling factor if output is fp8
template <typename ElementOutput>
struct ScaleOutOp { template <typename T> using Op = cutlass::first<T>; };
template <>
struct ScaleOutOp<float_e4m3_t> { template <typename T> using Op = cutlass::multiplies<T>; };
template <>
struct ScaleOutOp<float_e5m2_t> { template <typename T> using Op = cutlass::multiplies<T>; };
```

**EN:** Declares the templated `ScaleOutOp` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: We only apply the scaling factor if output is fp8.

**CN:** 声明模板类型 `ScaleOutOp`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1433-1434

```cpp
template <typename T>
using amax = cutlass::maximum_absolute_value_reduction<T, true>; // propogate nans
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1438-1458

```cpp
// D = scale_a * scale_b * alpha * acc + scale_c * beta * C + per-row bias
template<
  class CtaTileShapeMNK,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerRowBias =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc + bias)
    Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>, 2>, // scale_c * beta
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc + bias
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>, 3>, // scale_a * scale_b * alpha
      Sm90AccFetch, // acc
      Sm90ColBroadcast<0, CtaTileShapeMNK, ElementBias, ElementCompute, Stride<_1,_0,int64_t>, AlignmentBias> // bias
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = scale_a * scale_b * alpha * acc + scale_c * beta * C + per-row bias.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1460-1483

```cpp
// Z = scale_a * scale_b * alpha * acc + beta * scale_c * C + per-row bias
// if D is fp8 
//   D = scale_d * activation(Z)
// else
//   D = activation(Z)
template<
  class CtaTileShapeMNK,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerRowBiasEltAct =
  Sm90EVT<Sm90Compute<detail::ScaleOutOp<ElementOutput>::template Op, ElementOutput, ElementCompute, RoundStyle>, // activation(Z) * scale_d
    Sm90EVT<Sm90Compute<ActivationFn, ElementCompute, ElementCompute, RoundStyle>, // activation(Z)
      // Z = scale_a * scale_b * alpha * acc + beta * scale_c * C + per-row bias
      Sm90ScaledLinCombPerRowBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
    >,
    Sm90ScalarBroadcast<ElementScalar> // scale_d
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Z = scale_a * scale_b * alpha * acc + beta * scale_c * C + per-row bias if D is fp8 D = scale_d * activation(Z) else D = activation(Z).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1485-1511

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
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::ScaledLinCombPerRowBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90ScaledLinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1513-1520

```cpp
  using Impl =
    Sm90ScaledLinCombPerRowBiasEltAct<
      CtaTileShapeMNK, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::ScaledLinCombPerRowBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1522-1526

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1528-1535

```cpp
    ElementScalar scale_a = ElementScalar(1);
    ElementScalar scale_b = ElementScalar(1);
    ElementScalar scale_c = ElementScalar(1);
    ElementScalar scale_d = ElementScalar(1);
    ElementScalar const* scale_a_ptr = nullptr;
    ElementScalar const* scale_b_ptr = nullptr;
    ElementScalar const* scale_c_ptr = nullptr;
    ElementScalar const* scale_d_ptr = nullptr;
```

**EN:** This method block implements `ElementScalar`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementScalar`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 1537-1540

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1542-1544

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1546-1547

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1549-1578

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // binary op : activation((scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias)) * scale_d
          {    // unary op : activation((scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias))
            {    // ternary op : (scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias)
              {{beta, scale_c},
               {beta_ptr, scale_c_ptr},
               {dBeta, {_0{}, _0{}, 0}}
               },  // leaf args : (scale_c * beta)
              {},  // leaf args : C
              {    // ternary op : (scale_a * scale_b * alpha) * acc + bias
                {{alpha, scale_a, scale_b}, 
                 {alpha_ptr, scale_a_ptr, scale_b_ptr},
                 {dAlpha, {_0{}, _0{}, 0}, {_0{}, _0{}, 0}}
                 },                   // leaf args : (scale_a * scale_b * alpha)
                {},                   // leaf args : acc
                {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
                {} // ternary args : multiply_add
              },   // end ternary op
              {} // ternary args : multiply_add
            },   // end ternary op
            activation // unary args : activation
          },   // end unary op
          {{scale_d},
           {scale_d_ptr}
           },   // leaf args : scale_d
          {} // binary args : multiplies or first
        };   // end binary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1580-1582

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1586-1606

```cpp
// D = scale_a * scale_b * alpha * acc + scale_c * beta * C + per-col bias
template<
  class CtaTileShapeMNK,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerColBias =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc + bias)
    Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>, 2>, // scale_c * beta
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc + bias
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>, 3>, // scale_a * scale_b * alpha
      Sm90AccFetch, // acc
      Sm90RowBroadcast<0, CtaTileShapeMNK, ElementBias, ElementCompute, Stride<_0,_1,int64_t>, AlignmentBias> // bias
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = scale_a * scale_b * alpha * acc + scale_c * beta * C + per-col bias.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1608-1631

```cpp
// Z = scale_a * scale_b * alpha * acc + beta * scale_c * C + per-col bias
// if D is fp8 
//   D = scale_d * activation(Z)
// else
//   D = activation(Z)
template<
  class CtaTileShapeMNK,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerColBiasEltAct =
  Sm90EVT<Sm90Compute<detail::ScaleOutOp<ElementOutput>::template Op, ElementOutput, ElementCompute, RoundStyle>, // activation(Z) * scale_d
    Sm90EVT<Sm90Compute<ActivationFn, ElementCompute, ElementCompute, RoundStyle>, // activation(Z)
      // Z = scale_a * scale_b * alpha * acc + beta * scale_c * C + per-row bias
      Sm90ScaledLinCombPerColBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
    >,
    Sm90ScalarBroadcast<ElementScalar> // scale_d
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Z = scale_a * scale_b * alpha * acc + beta * scale_c * C + per-col bias if D is fp8 D = scale_d * activation(Z) else D = activation(Z).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1633-1659

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
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::ScaledLinCombPerColBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90ScaledLinCombPerColBiasEltAct<
      CtaTileShapeMNK, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `ActivationFn` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1661-1668

```cpp
  using Impl =
    Sm90ScaledLinCombPerColBiasEltAct<
      CtaTileShapeMNK, ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::ScaledLinCombPerColBiasEltAct<
      ActivationFn, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1670-1674

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1676-1683

```cpp
    ElementScalar scale_a = ElementScalar(1);
    ElementScalar scale_b = ElementScalar(1);
    ElementScalar scale_c = ElementScalar(1);
    ElementScalar scale_d = ElementScalar(1);
    ElementScalar const* scale_a_ptr = nullptr;
    ElementScalar const* scale_b_ptr = nullptr;
    ElementScalar const* scale_c_ptr = nullptr;
    ElementScalar const* scale_d_ptr = nullptr;
```

**EN:** This method block implements `ElementScalar`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementScalar`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 1685-1688

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1690-1692

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1694-1695

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1697-1726

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // binary op : activation((scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias)) * scale_d
          {    // unary op : activation((scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias))
            {    // ternary op : (scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias)
              {{beta, scale_c},
               {beta_ptr, scale_c_ptr},
               {dBeta, {_0{}, _0{}, 0}}
               },  // leaf args : (scale_c * beta)
              {},  // leaf args : C
              {    // ternary op : (scale_a * scale_b * alpha) * acc + bias
                {{alpha, scale_a, scale_b}, 
                 {alpha_ptr, scale_a_ptr, scale_b_ptr},
                 {dAlpha, {_0{}, _0{}, 0}, {_0{}, _0{}, 0}}
                 },                   // leaf args : (scale_a * scale_b * alpha)
                {},                   // leaf args : acc
                {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
                {} // ternary args : multiply_add
              },   // end ternary op
              {} // ternary args : multiply_add
            },   // end ternary op
            activation // unary args : activation
          },   // end unary op
          {{scale_d},
           {scale_d_ptr}
           },   // leaf args : scale_d
          {} // binary args : multiplies or first
        };   // end binary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1728-1730

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1746-1788

```cpp
// fp8 aux specialization
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int StagesD,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpR2S,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementAmax = ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerRowBiasEltActAmaxAuxFp8 =
  Sm90SplitTreeVisitor<
    // Z = scale_a * scale_b * alpha * acc + scale_c * beta * C + per-row bias
    Sm90ScaledLinCombPerRowBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>,
    // D = activation(Z) * scale_d, amax_d = max(abs(elements in D))
    Sm90EVT<Sm90Compute<detail::ScaleOutOp<ElementOutput>::template Op, ElementOutput, ElementCompute, RoundStyle>, // activation(Z) * scale_d
      Sm90EVT<Sm90ScalarReduction<detail::amax, atomic_maximum, ElementAmax, ElementCompute, RoundStyle>, // amax_d
        Sm90EVT<Sm90Compute<ActivationFn, ElementCompute, ElementCompute, RoundStyle>, // activation(Z)
          Sm90SplitTreeFetch // Z
        >
      >,
      Sm90ScalarBroadcast<ElementScalar> // scale_d
    >,
    // Aux = Z * scale_aux, amax_aux = max(abs(elements in Aux))
    Sm90EVT<Sm90AuxStore<StagesD, EpilogueTile, ElementAux, RoundStyle, StrideAux, SmemLayoutAtom, CopyOpR2S, AlignmentAux>, // store(Aux)
      Sm90EVT<Sm90Compute<cutlass::multiplies, ElementCompute, ElementCompute, RoundStyle>, // Z * scale_aux
        Sm90EVT<Sm90ScalarReduction<detail::amax, atomic_maximum, ElementAmax, ElementCompute, RoundStyle>, // amax_aux
          Sm90SplitTreeFetch // Z
        >,
        Sm90ScalarBroadcast<ElementScalar> // scale_aux
      >
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: fp8 aux specialization.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1790-1823

```cpp
// non-fp8 aux specialization
// lets us use some EVT specializations such as relu + uint1b_t aux
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int StagesD,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpR2S,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementAmax = ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerRowBiasEltActAmaxAuxNotFp8 =
  // D = activation(Z) * scale_d, amax_d = max(abs(elements in D))
  Sm90EVT<Sm90Compute<detail::ScaleOutOp<ElementOutput>::template Op, ElementOutput, ElementCompute, RoundStyle>, // activation(Z) * scale_d
    Sm90EVT<Sm90ScalarReduction<detail::amax, atomic_maximum, ElementAmax, ElementCompute, RoundStyle>, // amax_d
      Sm90EVT<Sm90Compute<ActivationFn, ElementCompute, ElementCompute, RoundStyle>, // activation(Z)
        Sm90EVT<Sm90AuxStore<StagesD, EpilogueTile, ElementAux, RoundStyle, StrideAux, SmemLayoutAtom, CopyOpR2S, AlignmentAux>, // Aux = Z
          // Z = scale_a * scale_b * alpha * acc + scale_c * beta * C + per-row bias
          Sm90ScaledLinCombPerRowBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
        >
      >
    >,
    Sm90ScalarBroadcast<ElementScalar> // scale_d
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: non-fp8 aux specialization lets us use some EVT specializations such as relu + uint1b_t aux.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1825-1854

```cpp
// dispatcher
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int StagesD,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpR2S,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementAmax = ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerRowBiasEltActAmaxAux = conditional_t<detail::is_fp8_v<ElementAux>,
  Sm90ScaledLinCombPerRowBiasEltActAmaxAuxFp8<
    CtaTileShapeMNK, EpilogueTile, StagesD, StrideAux, SmemLayoutAtom, CopyOpR2S, ActivationFn,
    ElementOutput, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar,AlignmentAux, AlignmentBias, RoundStyle
  >,
  Sm90ScaledLinCombPerRowBiasEltActAmaxAuxNotFp8<
    CtaTileShapeMNK, EpilogueTile, StagesD, StrideAux, SmemLayoutAtom, CopyOpR2S, ActivationFn,
    ElementOutput, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
  >
>;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: dispatcher.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1857-1894

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux,
  class ElementAmax,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class SmemLayoutAtom,
  class CopyOpR2S
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::ScaledLinCombPerRowBiasEltActAmaxAux<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    SmemLayoutAtom,
    CopyOpR2S
> : Sm90ScaledLinCombPerRowBiasEltActAmaxAux<
      CtaTileShapeMNK, EpilogueTile, StagesD, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>,
      SmemLayoutAtom, CopyOpR2S, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1896-1906

```cpp
  using Impl =
    Sm90ScaledLinCombPerRowBiasEltActAmaxAux<
      CtaTileShapeMNK, EpilogueTile, StagesD, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>,
      SmemLayoutAtom, CopyOpR2S, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::ScaledLinCombPerRowBiasEltActAmaxAux<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1908-1912

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1914-1921

```cpp
    ElementScalar scale_a = ElementScalar(1);
    ElementScalar scale_b = ElementScalar(1);
    ElementScalar scale_c = ElementScalar(1);
    ElementScalar scale_d = ElementScalar(1);
    ElementScalar const* scale_a_ptr = nullptr;
    ElementScalar const* scale_b_ptr = nullptr;
    ElementScalar const* scale_c_ptr = nullptr;
    ElementScalar const* scale_d_ptr = nullptr;
```

**EN:** This method block implements `ElementScalar`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementScalar`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 1923-1924

```cpp
    ElementScalar scale_aux = ElementScalar(1);
    ElementScalar const* scale_aux_ptr = nullptr;
```

**EN:** This method block implements `ElementScalar`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementScalar`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 1926-1929

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1931-1933

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1935-1936

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1938-1939

```cpp
    ElementAmax* amax_D_ptr = nullptr;
    ElementAmax* amax_aux_ptr = nullptr;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 1941-1943

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 1945-1950

```cpp
    operator typename Impl::Arguments() const {
      // Only compute amax_d if D is fp8
      ElementAmax* amax_D_ptr_ = nullptr;
      if constexpr (detail::is_fp8_v<ElementOutput>) {
        amax_D_ptr_ = amax_D_ptr;
      }
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1952-1956

```cpp
      // Aux is fp8 -> DAG arguments
      if constexpr (detail::is_fp8_v<ElementAux>) {
        typename Impl::Arguments args;
        // always use structured binding to unpack DAG args since it may or may not be a tuple
        auto& [Z_args, aux_args, D_args] = args;
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1958-1975

```cpp
        Z_args =
          {    // ternary op : (scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias)
            {{beta, scale_c},
             {beta_ptr, scale_c_ptr},
             {dBeta, {_0{}, _0{}, 0}}
             },  // leaf args : (scale_c * beta)
            {},  // leaf args : C
            {    // ternary op : (scale_a * scale_b * alpha) * acc + bias
              {{alpha, scale_a, scale_b}, 
               {alpha_ptr, scale_a_ptr, scale_b_ptr},
               {dAlpha ,{_0{}, _0{}, 0}, {_0{}, _0{}, 0}}
               },                   // leaf args : (scale_a * scale_b * alpha)
              {},                   // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {} // ternary args : multiply_add
            },   // end ternary op
            {} // ternary args : multiply_add
          };   // end ternary op
```

**EN:** This method block implements `ElementBias`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementBias`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 1977-1990

```cpp
        D_args =
          {    // binary op : activation(Z) * scale_d or activation(Z)
            {    // unary op : reduce(activation(Z))
              {             // unary op : activation(Z)
                {},             // leaf args : Z
                activation      // unary args : activation
              },                // end unary op
              {amax_D_ptr_} // unary args : reduce
            },              // end unary op
            {{scale_d},
             {scale_d_ptr}
             },  // leaf args : scale_d
            {} // binary args : multiplies or first
          };   // end binary op
```

**EN:** This method block implements `activation`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `activation`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 1992-2005

```cpp
        aux_args =
          {    // unary op : store(Aux)
            {    // binary op : Z * scale_d or Z
              {    // unary op : reduce(Z)
                {},            // leaf args : Z
                {amax_aux_ptr} // unary args : reduce
              },   // end unary op
              {{scale_aux},
               {scale_aux_ptr}
               },  // leaf args : scale_d
              {} // binary args : multiplies
            },   // end binary op
            {aux_ptr, dAux} // unary args : store
          };   // end unary op
```

**EN:** Implements a store path that writes a fragment back to memory in the arrangement expected by the epilogue pipeline.

**CN:** 实现存储路径，以 epilogue 流水线期望的排列方式把片段写回内存。


### Lines 2007-2008

```cpp
        return args;
      }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 2010-2046

```cpp
      // Aux is not fp8 -> Tree arguments
      else {
        return
          {  // binary op : activation(Z) * scale_d or activation(Z)
            {  // unary op : reduce(activation(Z))
              {  // unary op : activation(Z)
                {  // unary op : store(Z)
                  {  // ternary op : (scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias)
                    {{beta, scale_c},
                     {beta_ptr, scale_c_ptr},
                     {dBeta, {_0{}, _0{}, 0}}
                    },                // leaf args : (scale_c * beta)
                    {},               // leaf args : C
                    {                 // ternary op : (scale_a * scale_b * alpha) * acc + bias
                      {{alpha, scale_a, scale_b}, 
                       {alpha_ptr, scale_a_ptr, scale_b_ptr},
                       {dAlpha, {_0{}, _0{}, 0}}
                      },                // leaf args : (scale_a * scale_b * alpha)
                      {},               // leaf args : acc
                      {bias_ptr, ElementBias(0), dBias
                      },                // leaf args : bias
                      {}              // ternary args : multiply_add
                    },                // end ternary op
                    {}              // ternary args : multiply_add
                  },                // end ternary op
                  {aux_ptr, dAux} // unary args : store
                },                // end unary op
                activation     // unary args : activation
              },               // end unary op
              {amax_D_ptr_} // unary args : reduce
            },              // end unary op
            {{scale_d},{scale_d_ptr}}, // leaf args : scale_d
            {} // binary args : multiplies or first
          };   // end binary op
      }
    }
  };
```

**EN:** This method block implements `activation`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `activation`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 2048-2050

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2066-2108

```cpp
// fp8 aux specialization
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int StagesD,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpR2S,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementAmax = ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerColBiasEltActAmaxAuxFp8 =
  Sm90SplitTreeVisitor<
    // Z = scale_a * scale_b * alpha * acc + scale_c * beta * C + per-col bias
    Sm90ScaledLinCombPerColBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>,
    // D = activation(Z) * scale_d, amax_d = max(abs(elements in D))
    Sm90EVT<Sm90Compute<detail::ScaleOutOp<ElementOutput>::template Op, ElementOutput, ElementCompute, RoundStyle>, // activation(Z) * scale_d
      Sm90EVT<Sm90ScalarReduction<detail::amax, atomic_maximum, ElementAmax, ElementCompute, RoundStyle>, // amax_d
        Sm90EVT<Sm90Compute<ActivationFn, ElementCompute, ElementCompute, RoundStyle>, // activation(Z)
          Sm90SplitTreeFetch // Z
        >
      >,
      Sm90ScalarBroadcast<ElementScalar> // scale_d
    >,
    // Aux = Z * scale_aux, amax_aux = max(abs(elements in Aux))
    Sm90EVT<Sm90AuxStore<StagesD, EpilogueTile, ElementAux, RoundStyle, StrideAux, SmemLayoutAtom, CopyOpR2S, AlignmentAux>, // store(Aux)
      Sm90EVT<Sm90Compute<cutlass::multiplies, ElementCompute, ElementCompute, RoundStyle>, // Z * scale_aux
        Sm90EVT<Sm90ScalarReduction<detail::amax, atomic_maximum, ElementAmax, ElementCompute, RoundStyle>, // amax_aux
          Sm90SplitTreeFetch // Z
        >,
        Sm90ScalarBroadcast<ElementScalar> // scale_aux
      >
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: fp8 aux specialization.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2110-2143

```cpp
// non-fp8 aux specialization
// lets us use some EVT specializations such as relu + uint1b_t aux
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int StagesD,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpR2S,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementAmax = ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerColBiasEltActAmaxAuxNotFp8 =
  // D = activation(Z) * scale_d, amax_d = max(abs(elements in D))
  Sm90EVT<Sm90Compute<detail::ScaleOutOp<ElementOutput>::template Op, ElementOutput, ElementCompute, RoundStyle>, // activation(Z) * scale_d
    Sm90EVT<Sm90ScalarReduction<detail::amax, atomic_maximum, ElementAmax, ElementCompute, RoundStyle>, // amax_d
      Sm90EVT<Sm90Compute<ActivationFn, ElementCompute, ElementCompute, RoundStyle>, // activation(Z)
        Sm90EVT<Sm90AuxStore<StagesD, EpilogueTile, ElementAux, RoundStyle, StrideAux, SmemLayoutAtom, CopyOpR2S, AlignmentAux>, // Aux = Z
          // Z = scale_a * scale_b * alpha * acc + scale_c * beta * C + per-row bias
          Sm90ScaledLinCombPerColBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
        >
      >
    >,
    Sm90ScalarBroadcast<ElementScalar> // scale_d
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: non-fp8 aux specialization lets us use some EVT specializations such as relu + uint1b_t aux.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2145-2174

```cpp
// dispatcher
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int StagesD,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpR2S,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementAmax = ElementCompute,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90ScaledLinCombPerColBiasEltActAmaxAux = conditional_t<detail::is_fp8_v<ElementAux>,
  Sm90ScaledLinCombPerColBiasEltActAmaxAuxFp8<
    CtaTileShapeMNK, EpilogueTile, StagesD, StrideAux, SmemLayoutAtom, CopyOpR2S, ActivationFn,
    ElementOutput, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar,AlignmentAux, AlignmentBias, RoundStyle
  >,
  Sm90ScaledLinCombPerColBiasEltActAmaxAuxNotFp8<
    CtaTileShapeMNK, EpilogueTile, StagesD, StrideAux, SmemLayoutAtom, CopyOpR2S, ActivationFn,
    ElementOutput, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
  >
>;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: dispatcher.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2177-2214

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux,
  class ElementAmax,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class SmemLayoutAtom,
  class CopyOpR2S
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::ScaledLinCombPerColBiasEltActAmaxAux<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    SmemLayoutAtom,
    CopyOpR2S
> : Sm90ScaledLinCombPerColBiasEltActAmaxAux<
      CtaTileShapeMNK, EpilogueTile, StagesD, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>,
      SmemLayoutAtom, CopyOpR2S, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2216-2226

```cpp
  using Impl =
    Sm90ScaledLinCombPerColBiasEltActAmaxAux<
      CtaTileShapeMNK, EpilogueTile, StagesD, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>,
      SmemLayoutAtom, CopyOpR2S, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::ScaledLinCombPerColBiasEltActAmaxAux<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementAmax, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 2228-2232

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 2234-2241

```cpp
    ElementScalar scale_a = ElementScalar(1);
    ElementScalar scale_b = ElementScalar(1);
    ElementScalar scale_c = ElementScalar(1);
    ElementScalar scale_d = ElementScalar(1);
    ElementScalar const* scale_a_ptr = nullptr;
    ElementScalar const* scale_b_ptr = nullptr;
    ElementScalar const* scale_c_ptr = nullptr;
    ElementScalar const* scale_d_ptr = nullptr;
```

**EN:** This method block implements `ElementScalar`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementScalar`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 2243-2244

```cpp
    ElementScalar scale_aux = ElementScalar(1);
    ElementScalar const* scale_aux_ptr = nullptr;
```

**EN:** This method block implements `ElementScalar`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementScalar`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 2246-2249

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2251-2253

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2255-2256

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 2258-2259

```cpp
    ElementAmax* amax_D_ptr = nullptr;
    ElementAmax* amax_aux_ptr = nullptr;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 2261-2263

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 2265-2270

```cpp
    operator typename Impl::Arguments() const {
      // Only compute amax_d if D is fp8
      ElementAmax* amax_D_ptr_ = nullptr;
      if constexpr (detail::is_fp8_v<ElementOutput>) {
        amax_D_ptr_ = amax_D_ptr;
      }
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 2272-2276

```cpp
      // Aux is fp8 -> DAG arguments
      if constexpr (detail::is_fp8_v<ElementAux>) {
        typename Impl::Arguments args;
        // always use structured binding to unpack DAG args since it may or may not be a tuple
        auto& [Z_args, aux_args, D_args] = args;
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 2278-2295

```cpp
        Z_args =
          {    // ternary op : (scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias)
            {{beta, scale_c},
             {beta_ptr, scale_c_ptr},
             {dBeta, {_0{}, _0{}, 0}}
             },  // leaf args : (scale_c * beta)
            {},  // leaf args : C
            {    // ternary op : (scale_a * scale_b * alpha) * acc + bias
              {{alpha, scale_a, scale_b}, 
               {alpha_ptr, scale_a_ptr, scale_b_ptr},
               {dAlpha, {_0{}, _0{}, 0}, {_0{}, _0{}, 0}}
               },                   // leaf args : (scale_a * scale_b * alpha)
              {},                   // leaf args : acc
              {bias_ptr, ElementBias(0), dBias}, // leaf args : bias
              {} // ternary args : multiply_add
            },   // end ternary op
            {} // ternary args : multiply_add
          };   // end ternary op
```

**EN:** This method block implements `ElementBias`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementBias`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 2297-2310

```cpp
        D_args =
          {    // binary op : activation(Z) * scale_d or activation(Z)
            {    // unary op : reduce(activation(Z))
              {             // unary op : activation(Z)
                {},             // leaf args : Z
                activation      // unary args : activation
              },                // end unary op
              {amax_D_ptr_} // unary args : reduce
            },              // end unary op
            {{scale_d},
             {scale_d_ptr}
             },  // leaf args : scale_d
            {} // binary args : multiplies or first
          };   // end binary op
```

**EN:** This method block implements `activation`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `activation`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 2312-2325

```cpp
        aux_args =
          {    // unary op : store(Aux)
            {    // binary op : Z * scale_d or Z
              {    // unary op : reduce(Z)
                {},            // leaf args : Z
                {amax_aux_ptr} // unary args : reduce
              },   // end unary op
              {{scale_aux},
               {scale_aux_ptr}
               },  // leaf args : scale_d
              {} // binary args : multiplies
            },   // end binary op
            {aux_ptr, dAux} // unary args : store
          };   // end unary op
```

**EN:** Implements a store path that writes a fragment back to memory in the arrangement expected by the epilogue pipeline.

**CN:** 实现存储路径，以 epilogue 流水线期望的排列方式把片段写回内存。


### Lines 2327-2328

```cpp
        return args;
      }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 2330-2366

```cpp
      // Aux is not fp8 -> Tree arguments
      else {
        return
          {  // binary op : activation(Z) * scale_d or activation(Z)
            {  // unary op : reduce(activation(Z))
              {  // unary op : activation(Z)
                {  // unary op : store(Z)
                  {  // ternary op : (scale_c * beta) * C + ((scale_a * scale_b * alpha) * acc + bias)
                    {{beta, scale_c},
                    {beta_ptr, scale_c_ptr},
                    {dBeta, {_0{}, _0{}, 0}}
                    },  // leaf args : (scale_c * beta)
                    {},               // leaf args : C
                    {                 // ternary op : (scale_a * scale_b * alpha) * acc + bias
                      {{alpha, scale_a, scale_b}, 
                       {alpha_ptr, scale_a_ptr, scale_b_ptr},
                       {dAlpha, {_0{}, _0{}, 0}, {_0{}, _0{}, 0}}
                      },                // leaf args : (scale_a * scale_b * alpha)
                      {},               // leaf args : acc
                      {bias_ptr, ElementBias(0), dBias
                      },                // leaf args : bias
                      {}              // ternary args : multiply_add
                    },                // end ternary op
                    {}              // ternary args : multiply_add
                  },                // end ternary op
                  {aux_ptr, dAux} // unary args : store
                },                // end unary op
                activation     // unary args : activation
              },               // end unary op
              {amax_D_ptr_} // unary args : reduce
            },              // end unary op
            {{scale_d},{scale_d_ptr}}, // leaf args : scale_d
            {} // binary args : multiplies or first
          };   // end binary op
      }
    }
  };
```

**EN:** This method block implements `activation`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `activation`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 2368-2370

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2374-2394

```cpp
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int Stages,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpS2R,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombDeEltAct =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>, // activation(beta * C + (alpha * acc), aux)
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle>, // beta * C + (alpha * acc)
    Sm90AuxLoad<Stages, EpilogueTile, ElementAux, StrideAux, SmemLayoutAtom, CopyOpS2R, AlignmentAux> // aux
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2396-2429

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class SmemLayoutAtom,
  class CopyOpS2R
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    SmemLayoutAtom,
    CopyOpS2R
> : Sm90LinCombDeEltAct<
      CtaTileShapeMNK, EpilogueTile, StagesC, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, SmemLayoutAtom, CopyOpS2R, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2431-2440

```cpp
  using Impl =
    Sm90LinCombDeEltAct<
      CtaTileShapeMNK, EpilogueTile, StagesC, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, SmemLayoutAtom, CopyOpS2R, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >;
  using Operation =
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 2442-2446

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 2448-2451

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2453-2454

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 2456-2458

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux const* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 2460-2477

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // binary op : activation(beta * C + (alpha * acc), aux)
          {                  // ternary op : beta * C + (alpha * acc)
            {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // binary op : alpha * acc
              {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
              {},                     // leaf args : acc
              {}                  // binary args : multiplies
            },                    // end binary op
            {}               // ternary args : multiply_add
          },                 // end ternary op
          {aux_ptr, ElementAux(0), dAux}, // leaf args : aux
          activation // binary args : activation
        };   // end binary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 2479-2481

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2485-2510

```cpp
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  int Stages,
  class StrideAux,
  class SmemLayoutAtom,
  class CopyOpS2R,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementBias = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  int AlignmentAux = 128 / sizeof_bits_v<ElementAux>,
  int AlignmentBias = 128 / sizeof_bits_v<ElementBias>,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombDeEltActDePerRowBias =
  Sm90EVT<Sm90Compute<cutlass::epilogue::thread::Identity, ElementOutput, ElementCompute, RoundStyle>, // Identity for final conversion
    Sm90EVT<Sm90ColReduction<plus, plus, plus, 0, CtaTileShapeMNK,
                             ElementBias, ElementCompute, RoundStyle, Stride<_1,_0,int64_t>, AlignmentBias>,
      Sm90LinCombDeEltAct<CtaTileShapeMNK, EpilogueTile, Stages, StrideAux, SmemLayoutAtom, CopyOpS2R, ActivationFn,
                          ElementCompute, ElementCompute, ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle>
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2512-2547

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux,
  class ElementBias,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  int AlignmentBias,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class SmemLayoutAtom,
  class CopyOpS2R
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombDeEltActDePerRowBias<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    SmemLayoutAtom,
    CopyOpS2R
> : Sm90LinCombDeEltActDePerRowBias<
      CtaTileShapeMNK, EpilogueTile, StagesC, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, SmemLayoutAtom, CopyOpS2R, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2549-2558

```cpp
  using Impl =
    Sm90LinCombDeEltActDePerRowBias<
      CtaTileShapeMNK, EpilogueTile, StagesC, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, SmemLayoutAtom, CopyOpS2R, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
  using Operation =
    fusion::LinCombDeEltActDePerRowBias<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 2560-2564

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 2566-2569

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2571-2572

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 2574-2576

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux const* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 2578-2580

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias* dbias_ptr = nullptr;
    StrideBias dDbias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2582-2605

```cpp
    operator typename Impl::Arguments() const {
      return
      {   // unary op : identity/convert
        {    // unary op : reduce(activation(beta * C + (alpha * acc), aux))
          {    // binary op : activation(beta * C + (alpha * acc), aux)
            {                  // ternary op : beta * C + (alpha * acc)
              {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
              {},                   // leaf args : C
              {                     // binary op : alpha * acc
                {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
                {},                     // leaf args : acc
                {}                  // binary args : multiplies
              },                    // end binary op
              {}               // ternary args : multiply_add
            },                 // end ternary op
            {aux_ptr, ElementAux(0), dAux}, // leaf args : aux
            activation // binary args : activation
          },   // end binary op
          {dbias_ptr, ElementCompute(0), dDbias} // unary args : reduce
        },   // end unary op
        {} // unary args : identity/convert
      };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 2607-2609

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2613-2628

```cpp
// D = softmax(top_k(alpha * acc + beta * C))
template<
  int TopK,
  int FragmentSize,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinCombTopKSoftmaxCol =
  Sm90EVT<Sm90TopKSoftmaxColReduction<TopK, FragmentSize, CtaTileShapeMNK, EpilogueTile, ElementOutput, ElementCompute, RoundStyle>, // softmax(top_k(beta * C + (alpha * acc)))
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = softmax(top_k(alpha * acc + beta * C)).

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 2630-2650

```cpp
template <
  int TopK,
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinCombTopKSoftmaxCol<TopK, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinCombTopKSoftmaxCol<TopK, FragmentSize, CtaTileShapeMNK, EpilogueTile, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2652-2653

```cpp
  using Impl = Sm90LinCombTopKSoftmaxCol<TopK, FragmentSize, CtaTileShapeMNK, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombTopKSoftmaxCol<TopK, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 2655-2659

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 2661-2677

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // unary op: activation(beta * C + (alpha * acc))
          {    // ternary op : beta * C + (alpha * acc)
            {{beta}, {beta_ptr}}, // leaf args : beta
            {},                   // leaf args : C
            {                     // binary op : alpha * acc
              {{alpha}, {alpha_ptr}}, // leaf args : alpha
              {},                     // leaf args : acc
              {}                  // binary args : multiplies
            },                    // end binary op
            {} // ternary args : multiply_add
          },   // end ternary op
          {} // unary args: activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 2679-2681

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2685-2702

```cpp
// Grouped Wgrad Conv
template<
  class GroupsPerTile,
  class ElementOutput,
  class ElementCompute,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using Sm90LinearCombinationGroupedWgrad =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc)
    Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // beta
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<multiplies, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // alpha
      Sm90AccFetchGroupedWgrad<GroupsPerTile> // acc
    >
  >;
```

**EN:** Defines `GroupsPerTile`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Grouped Wgrad Conv.

**CN:** 定义 `GroupsPerTile`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 2704-2724

```cpp
template <
  int StagesC,
  int StagesD,
  int FragmentSize,
  bool ReuseSmemC,
  bool DelayTmaStore,
  class ElementOutput,
  class ElementCompute,
  class ElementSource,
  class ElementScalar,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class GroupsPerTile
>
struct FusionCallbacks<
    epilogue::Sm90TmaWarpSpecialized<StagesC, StagesD, FragmentSize, ReuseSmemC, DelayTmaStore>,
    fusion::LinearCombinationGroupedWgrad<GroupsPerTile, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90LinearCombinationGroupedWgrad<GroupsPerTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2726-2727

```cpp
  using Impl = Sm90LinearCombinationGroupedWgrad<GroupsPerTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinearCombinationGroupedWgrad<GroupsPerTile, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 2729-2734

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    //ElementScalar groups = ElementScalar(1);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 2736-2739

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2741-2754

```cpp
    operator typename Impl::Arguments() const {
      return
        {    // ternary op : beta * C + (alpha * acc)
          {{beta}, {beta_ptr}, {dBeta}}, // leaf args : beta
          {},                   // leaf args : C
          {                     // binary op : alpha * acc
            {{alpha}, {alpha_ptr}, {dAlpha}}, // leaf args : alpha
            {},                     // leaf args : acc
            {}                  // binary args : multiplies
          },                    // end binary op
          {} // ternary args : multiply_add
        };   // end ternary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 2756-2758

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 2762-2766

```cpp
namespace detail {
template <class FusionOpOrCallbacks, class = cute::void_t<>>
struct get_element_aux {
  using type = void;
};
```

**EN:** Declares the templated `detail` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `detail`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2768-2771

```cpp
template <class FusionOpOrCallbacks>
struct get_element_aux<FusionOpOrCallbacks, cute::void_t<typename FusionOpOrCallbacks::ElementAux>> {
  using type = typename FusionOpOrCallbacks::ElementAux;
};
```

**EN:** Declares the templated `FusionOpOrCallbacks` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `FusionOpOrCallbacks`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2773-2776

```cpp
template <class NodeOp, class... ChildOps>
struct get_element_aux<Sm90TreeVisitor<NodeOp, ChildOps...>, cute::void_t<>> {
  using type = typename get_element_aux<NodeOp>::type;
};
```

**EN:** Declares the templated `NodeOp` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `NodeOp`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2778-2785

```cpp
template <class... Ts>
struct get_element_aux<FusionCallbacks<Ts...>, cute::void_t<typename FusionCallbacks<Ts...>::Operation>> {
 private:
  using Operation = typename FusionCallbacks<Ts...>::Operation;
 public:
  using type = typename get_element_aux<Operation>::type;
};
} // namespace cutlass:epilogue::fusion::detail
```

**EN:** Declares the templated `get_element_aux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `get_element_aux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 2787-2788

```cpp
template <class Callbacks>
using get_element_aux_t = typename detail::get_element_aux<Callbacks>::type;
```

**EN:** Declares the templated `Callbacks` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `Callbacks`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


## Key Concepts / 关键概念

- **Fusion framework / 融合框架:** Uses callbacks, visitors, or operation tags to compose multiple post-processing steps into the epilogue. / 通过回调、访问者或操作标签把多个后处理步骤组合进 epilogue。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。

- **Scaling semantics / 缩放语义:** Tracks how alpha/beta parameters mix accumulator values with source tensors during output generation. / 描述 alpha/beta 参数如何在生成输出时混合累加器值与源张量。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cute/tensor.hpp`, `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_store_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_compute_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_topk_softmax.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::fusion`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_store_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_compute_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_topk_softmax.hpp`

- **Cute/CuTe dependencies / Cute 依赖:** `cute/tensor.hpp`
