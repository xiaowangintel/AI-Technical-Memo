# xe_callbacks.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/fusion/xe_callbacks.hpp`

- **Purpose (EN):** Fusion callbacks specializations for the Intel Xe epilogue.

- **作用 (CN):** 实现 `Xe callbacks` 相关的融合回调接口与回调连接逻辑。


## Line-by-Line Analysis / 逐行分析

### Lines 1-31

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
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
```

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Lines 33-35

```cpp
/*! \file
  \brief Fusion callbacks specializations for the Intel Xe epilogue
*/
```

**EN:** This documentation block explains the intent of the next declaration: ! Fusion callbacks specializations for the Intel Xe epilogue.

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


### Lines 43-53

```cpp
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/epilogue/fusion/callbacks.hpp"
#include "cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/xe_visitor.hpp"
#include "cutlass/epilogue/fusion/xe_visitor_legacy.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_store_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_compute_tma_warpspecialized.hpp"
#include "cutlass/epilogue/fusion/xe_visitor_softmax.hpp"
#include "cutlass/epilogue/fusion/xe_visitor_splitk.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`, and 7 more headers. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/epilogue/dispatch_policy.hpp`，`cutlass/epilogue/fusion/callbacks.hpp`，`cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`，`cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`，以及另外 7 个头文件。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 57

```cpp
namespace cutlass::epilogue::fusion {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 62-76

```cpp
template <
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinearCombination<ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : Sm90LinearCombination<typename cutlass::detail::get_unpacked_element_type<ElementOutput_>::type, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_> {
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 78-83

```cpp
  using Impl = Sm90LinearCombination<typename cutlass::detail::get_unpacked_element_type<ElementOutput_>::type, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>;
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  using Operation = fusion::LinearCombination<ElementOutput, ElementCompute, ElementSource_, ElementScalar, RoundStyle_>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 85-89

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 91-94

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 96-109

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


### Lines 111-122

```cpp
  // Check if the epilogue is an identity operation (D = acc).
  // True when alpha == 1 and beta == 0.
  CUTLASS_DEVICE bool is_identity() const {
    using OuterVisitorImpl = typename Impl::Impl;
    auto const& outer_ops = static_cast<OuterVisitorImpl const&>(*this).ops;
    if (!get<0>(outer_ops).is_zero()) return false;
    auto const& inner_tree = get<2>(outer_ops);
    using InnerTreeType = cute::remove_cvref_t<decltype(inner_tree)>;
    using InnerVisitorImpl = typename InnerTreeType::Impl;
    auto const& alpha_op = get<0>(static_cast<InnerVisitorImpl const&>(inner_tree).ops);
    return alpha_op.scalar == ElementScalar(1);
  }
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 124-126

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 128-143

```cpp
template <
  template <class> class ActivationFn_,
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombEltAct<ActivationFn_, ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : Sm90LinCombEltAct<ActivationFn_, ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_> {
```

**EN:** Declares the templated `ActivationFn_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 145-150

```cpp
  using Impl = Sm90LinCombEltAct<ActivationFn_, typename cutlass::detail::get_unpacked_element_type<ElementOutput_>::type, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>;
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  using Operation = fusion::LinCombEltAct<ActivationFn_, ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 152-156

```cpp
  struct Arguments {
    ElementScalar_ alpha = ElementScalar_(1);
    ElementScalar_ beta = ElementScalar_(0);
    ElementScalar_ const* alpha_ptr = nullptr;
    ElementScalar_ const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 158-161

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 163-164

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn_, ElementOutput_, ElementCompute_, RoundStyle_>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 166-182

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


### Lines 184-186

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 188-203

```cpp
// D = splitk(alpha * acc + beta * C)
template<
  // int FragmentSize,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class CopyOpR2G,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using XeLinCombSplitK =
  Sm90EVT<XeSplitK<CtaTileShapeMNK, EpilogueTile, ElementOutput, ElementCompute, CopyOpR2G, RoundStyle>, // splitk(beta * C + (alpha * acc))
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = splitk(alpha * acc + beta * C).

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 205-222

```cpp
// TODO: update split-k for new epilogues
template <
  // int FragmentSize,
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  class CopyOpR2G_,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinCombSplitK<ElementOutput_, ElementCompute_, CopyOpR2G_, ElementSource_, ElementScalar_, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : XeLinCombSplitK<CtaTileShapeMNK, EpilogueTile, ElementOutput_, ElementCompute_, CopyOpR2G_, ElementSource_, ElementScalar_, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: TODO: update split-k for new epilogues.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 224-229

```cpp
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  using Impl = XeLinCombSplitK<CtaTileShapeMNK, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, CopyOpR2G_, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombSplitK<ElementOutput_, ElementCompute, CopyOpR2G_, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 231-258

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementOutput* output_ptr = nullptr;
    ElementOutput *output_ptr1 = nullptr;
    ElementOutput *output_ptr2 = nullptr;
    size_t NUM_HEAD = 0;
    size_t NOPE_DIM = 0;
    size_t ROPE_DIM = 0;
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
          {output_ptr, output_ptr1, output_ptr2, NUM_HEAD, NOPE_DIM, ROPE_DIM} // unary args: activation
        };   // end unary op
    }
  };
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 260-262

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 264-280

```cpp
// TODO: update softmax for new epilogues
// D = softmax(alpha * acc + beta * C)
template<
  // int FragmentSize,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class CopyOpR2G,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using XeLinCombSoftmaxRow =
  Sm90EVT<XeSoftmaxRowReduction<CtaTileShapeMNK, EpilogueTile, ElementOutput, ElementCompute, CopyOpR2G, RoundStyle>, // softmax(beta * C + (alpha * acc))
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle> // beta * C + (alpha * acc)
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: TODO: update softmax for new epilogues D = softmax(alpha * acc + beta * C).

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 282-298

```cpp
template <
  // int FragmentSize,
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  class CopyOpR2G_,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinCombSoftmaxRow<ElementOutput_, ElementCompute_, CopyOpR2G_, ElementSource_, ElementScalar_, RoundStyle>,
    CtaTileShapeMNK,
    EpilogueTile
> : XeLinCombSoftmaxRow<CtaTileShapeMNK, EpilogueTile, ElementOutput_, ElementCompute_, CopyOpR2G_, ElementSource_, ElementScalar_, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 300-305

```cpp
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  using Impl = XeLinCombSoftmaxRow<CtaTileShapeMNK, EpilogueTile, typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type, ElementCompute, CopyOpR2G_, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombSoftmaxRow<ElementOutput_, ElementCompute, CopyOpR2G_, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 307-312

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
    ElementOutput* output_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 314-330

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
          {output_ptr} // unary args: activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 332-334

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 336-353

```cpp
// XeLinCombDeEltActLegacy - uses old XeAuxLoad for backward compatibility with examples
// EVT tests bypass this and directly instantiate XeAuxLoad via Python code generation
template<
  class StrideAux,
  class CopyOpG2R,
  template <class> class ActivationFn,
  class ElementOutput,
  class ElementCompute,
  class ElementAux = ElementOutput,
  class ElementSource = ElementOutput,
  class ElementScalar = ElementCompute,
  FloatRoundStyle RoundStyle = FloatRoundStyle::round_to_nearest
>
using XeLinCombDeEltActLegacy =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>, // activation(beta * C + (alpha * acc), aux)
    Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle>, // beta * C + (alpha * acc)
    XeAuxLoadLegacy<ElementAux, StrideAux, CopyOpG2R> // aux (legacy version with explicit CopyOpG2R)
  >;
```

**EN:** Declares the templated `StrideAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: XeLinCombDeEltActLegacy - uses old XeAuxLoad for backward compatibility with examples EVT tests bypass this and directly instantiate XeAuxLoad via Python code generation.

**CN:** 声明模板类型 `StrideAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 355-386

```cpp
// Z = Aux
// dY = alpha * acc + beta * C
// D = activation(dY, Z)
//
// Specialization with explicit CopyOpG2R - uses legacy XeAuxLoad for backward compatibility
template <
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput_,
  class ElementCompute_,
  class ElementAux,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class CopyOpG2R
>
struct FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput_, ElementCompute_,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    CopyOpG2R
> : XeLinCombDeEltActLegacy<
      cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, CopyOpG2R, ActivationFn, ElementOutput_,
      ElementCompute_, ElementAux, ElementSource, ElementScalar, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Z = Aux dY = alpha * acc + beta * C D = activation(dY, Z) Specialization with explicit CopyOpG2R - uses legacy XeAuxLoad for backward compatibility.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 388-389

```cpp
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 391-400

```cpp
  using Impl =
    XeLinCombDeEltActLegacy<
      cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, CopyOpG2R, ActivationFn, ElementOutput,
      ElementCompute, ElementAux, ElementSource, ElementScalar, RoundStyle
    >;
  using Operation =
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 402-406

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 408-411

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 413-414

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 416-418

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux const* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 420-437

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


### Lines 439-441

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 443-446

```cpp
// Temporary: provide default G2R operation for LinCombDeEltAct, for CollectiveBuilder.
// Used by examples and non-EVT tests that don't specify CopyOpG2R template parameter
template <typename ElementSource>
using XeAuxLoadDefaultCopyOpG2R = conditional_t<sizeof_bits_v<ElementSource> == 32, XE_2D_U32x8x16_LD_N, XE_2D_U16x8x16_LD_N>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 448-479

```cpp
template <
  class GmemLayoutTagAux, template <class> class ActivationFn,
  class ElementOutput_, class ElementCompute_, class ElementAux, class ElementSource, class ElementScalar,
  int AlignmentAux, FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK, class EpilogueTile
>
struct FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput_, ElementCompute_,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput_, ElementCompute_,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    XeAuxLoadDefaultCopyOpG2R<ElementSource>
> {
  using FusionCallbacks<
    epilogue::IntelXeGeneric, fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput_, ElementCompute_,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK, EpilogueTile, XeAuxLoadDefaultCopyOpG2R<ElementSource>
  >::FusionCallbacks;
};
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 485-511

```cpp
// FusionCallbacks specialization for XeLinCombDeEltAct using new XeAuxLoad
template <
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput_,
  class ElementCompute_,
  class ElementAux,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::XeLinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput_, ElementCompute_,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile
> : Sm90EVT<
      Sm90Compute<ActivationFn, ElementOutput_, ElementCompute_, RoundStyle>,
      Sm90LinearCombination<ElementCompute_, ElementCompute_, ElementSource, ElementScalar, RoundStyle>,
      XeAuxLoad<ElementAux, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>>
    > {
```

**EN:** Declares the templated `new` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: FusionCallbacks specialization for XeLinCombDeEltAct using new XeAuxLoad.

**CN:** 声明模板类型 `new`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 513-514

```cpp
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 516-526

```cpp
  using Impl =
    Sm90EVT<
      Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
      Sm90LinearCombination<ElementCompute, ElementCompute, ElementSource, ElementScalar, RoundStyle>,
      XeAuxLoad<ElementAux, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>>
    >;
  using Operation =
    fusion::XeLinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 528-532

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 534-537

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 539-540

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 542-544

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux const* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 546-563

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


### Lines 565-567

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 570-587

```cpp
// D = alpha * acc + beta * C + per-row bias
template <
  class ElementOutput_,
  class ElementCompute_,
  class ElementBias_,
  class ElementSource_,
  class ElementScalar_,
  int AlignmentBias_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerRowBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : Sm90LinCombPerRowBias<CtaTileShapeMNK_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_> {
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = alpha * acc + beta * C + per-row bias.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 589-600

```cpp
  using Impl = Sm90LinCombPerRowBias<
      CtaTileShapeMNK_,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput_>::type,
      ElementCompute_, ElementBias_, ElementSource_, ElementScalar_,
      AlignmentBias_, RoundStyle_>;
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementBias = ElementBias_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  static constexpr int AlignmentBias = AlignmentBias_;
  using Operation = fusion::LinCombPerRowBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 602-606

```cpp
  struct Arguments {
    ElementScalar_ alpha = ElementScalar_(1);
    ElementScalar_ beta = ElementScalar_(0);
    ElementScalar_ const* alpha_ptr = nullptr;
    ElementScalar_ const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 608-611

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 613-615

```cpp
    using StrideBias = Stride<_1, _0, int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 617-631

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


### Lines 633-635

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 637-661

```cpp
// D = alpha * acc + beta * C + per-column bias
// XeLinCombPerColBiasLegacy - uses XeRowBroadcastLegacy for backward compatibility with examples
// EVT tests bypass this and directly instantiate XeRowBroadcast via Python code generation
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
using XeLinCombPerColBiasLegacy =
  Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementOutput, ElementCompute, RoundStyle>, // beta * C + (alpha * acc + bias)
    Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // beta
    Sm90SrcFetch<ElementSource>, // C
    Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle>, // alpha * acc + bias
      Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>, // alpha
      Sm90AccFetch, // acc
      XeRowBroadcastLegacy<0, CtaTileShapeMNK, ElementBias, ElementCompute, Stride<_0,_1,int64_t>, AlignmentBias> // bias (legacy version)
    >
  >;
```

**EN:** Defines `CtaTileShapeMNK`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: D = alpha * acc + beta * C + per-column bias XeLinCombPerColBiasLegacy - uses XeRowBroadcastLegacy for backward compatibility with examples EVT tests bypass this and directly instantiate XeRowBroadcast via Python code generation.

**CN:** 定义 `CtaTileShapeMNK`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 663-679

```cpp
template <
  class ElementOutput_,
  class ElementCompute_,
  class ElementBias_,
  class ElementSource_,
  class ElementScalar_,
  int AlignmentBias_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerColBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : XeLinCombPerColBiasLegacy<_1{} /* Stages */, CtaTileShapeMNK_, EpilogueTile_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_> {
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 681-694

```cpp
  using Impl = XeLinCombPerColBiasLegacy<
      _1{},
      CtaTileShapeMNK_,
      EpilogueTile_,
      typename cutlass::detail::get_unpacked_element_type<ElementOutput_>::type,
      ElementCompute_, ElementBias_, ElementSource_, ElementScalar_,
      AlignmentBias_, RoundStyle_>;
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementBias = ElementBias_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  static constexpr int AlignmentBias = AlignmentBias_;
  using Operation = fusion::LinCombPerColBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 696-700

```cpp
  struct Arguments {
    ElementScalar_ alpha = ElementScalar_(1);
    ElementScalar_ beta = ElementScalar_(0);
    ElementScalar_ const* alpha_ptr = nullptr;
    ElementScalar_ const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 702-705

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 707-709

```cpp
    using StrideBias = Stride<_0, _1, int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 711-725

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


### Lines 727-729

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 731-747

```cpp
// TODO: move to new epilogue
template <
  int TopK,
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile
>
struct FusionCallbacks<
  epilogue::IntelXeXMX16,
  fusion::LinCombTopKSoftmaxCol<TopK, ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle>,
  CtaTileShapeMNK,
  EpilogueTile
> : Sm90LinCombTopKSoftmaxCol<TopK, 8 /*FragmentSize*/, CtaTileShapeMNK, EpilogueTile, ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle> {
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: TODO: move to new epilogue.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 749-757

```cpp
  static constexpr int FragmentSize = 8;
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  using Impl = Sm90LinCombTopKSoftmaxCol<TopK, FragmentSize, CtaTileShapeMNK, EpilogueTile,
                                        typename cutlass::detail::get_unpacked_element_type<ElementOutput>::type,
                                        ElementCompute, ElementSource, ElementScalar, RoundStyle>;
  using Operation = fusion::LinCombTopKSoftmaxCol<TopK, ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 759-763

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 765-781

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


### Lines 783-785

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 791-815

```cpp
template <
 // int FragmentSize,
  //bool ReuseSmemC,
 // bool DelayTmaStore,
  template <class> class ActivationFn_,
  class ElementOutput_,
  class ElementCompute_,
  class ElementBias_,
  class ElementSource_,
  class ElementScalar_,
  int AlignmentBias_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
     epilogue::IntelXeGeneric,
    fusion::LinCombPerRowBiasEltAct<
      ActivationFn_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_
    >,
    CtaTileShapeMNK_,
    EpilogueTile_
> : Sm90LinCombPerRowBiasEltAct<
      CtaTileShapeMNK_, ActivationFn_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_
    > {
```

**EN:** Declares the templated `ActivationFn_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 817-830

```cpp
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementBias = ElementBias_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  static constexpr int AlignmentBias = AlignmentBias_;
  using Impl =
    Sm90LinCombPerRowBiasEltAct<
      CtaTileShapeMNK_, ActivationFn_, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle_
    >;
  using Operation =
    fusion::LinCombPerRowBiasEltAct<
      ActivationFn_, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle_
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 832-836

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 838-841

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 843-845

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 847-848

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn_, ElementOutput, ElementCompute, RoundStyle_>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 850-867

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


### Lines 869-871

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 874-889

```cpp
// D = alpha * acc + beta * C, where beta and alpha can be vectors for each batch
template <
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeGenericGroup,
    fusion::LinearCombination<ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : Sm90LinearCombinationPtrArray<typename cutlass::detail::get_unpacked_element_type<ElementOutput_>::type, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_> {
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = alpha * acc + beta * C, where beta and alpha can be vectors for each batch.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 891-896

```cpp
  using Impl = Sm90LinearCombinationPtrArray<typename cutlass::detail::get_unpacked_element_type<ElementOutput_>::type, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>;
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  using Operation = fusion::LinearCombination<ElementOutput, ElementCompute, ElementSource, ElementScalar, RoundStyle_>;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 898-904

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


### Lines 906-909

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 911-924

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


### Lines 926-928

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 930-953

```cpp
// D = activation(alpha * acc + beta * C + per-row bias)
// Aux = alpha * acc + beta * C + per-row bias)
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  class StrideAux,
  class CopyOpR2G,
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
using XeLinCombPerRowBiasEltActAux =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
    Sm90EVT<XeAuxStore<ElementAux, StrideAux, CopyOpR2G>,
      Sm90LinCombPerRowBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-row bias) Aux = alpha * acc + beta * C + per-row bias).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 955-983

```cpp
template <
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
  class CopyOpR2G
>
struct FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerRowBiasEltActAux<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    CopyOpR2G
> : XeLinCombPerRowBiasEltActAux<
      CtaTileShapeMNK, EpilogueTile, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, CopyOpR2G, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 985-994

```cpp
  using Impl =
    XeLinCombPerRowBiasEltActAux<
      CtaTileShapeMNK, EpilogueTile, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, CopyOpR2G, ActivationFn,
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


### Lines 996-1000

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1002-1005

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1007-1009

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1011-1012

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1014-1016

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 1018-1038

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
            {aux_ptr, ElementAux(0), dAux} // unary args : store
          },                // end unary op
          activation // unary args : activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1040-1042

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1048-1078

```cpp
template <
  template <class> class ActivationFn_,
  class ElementOutput_,
  class ElementCompute_,
  class ElementBias_,
  class ElementSource_,
  class ElementScalar_,
  int AlignmentBias_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
     epilogue::IntelXeGeneric,
    fusion::XeLinCombPerRowBiasEltAct<
      ActivationFn_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_
    >,
    CtaTileShapeMNK_,
    EpilogueTile_
> : Sm90EVT<
      Sm90Compute<ActivationFn_, ElementOutput_, ElementCompute_, RoundStyle_>,
      Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute_, ElementCompute_, RoundStyle_>,
        Sm90ScalarBroadcast<ElementScalar_, Stride<_0,_0,int64_t>>,
        Sm90SrcFetch<ElementSource_>,
        Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute_, ElementCompute_, RoundStyle_>,
          Sm90ScalarBroadcast<ElementScalar_, Stride<_0,_0,int64_t>>,
          Sm90AccFetch,
          XeColBroadcast<0, CtaTileShapeMNK_, ElementBias_, ElementCompute_, Stride<_1,_0,int64_t>, AlignmentBias_>
        >
      >
    > {
```

**EN:** Declares the templated `ActivationFn_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1080-1102

```cpp
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementBias = ElementBias_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  static constexpr int AlignmentBias = AlignmentBias_;
  using Impl =
    Sm90EVT<
      Sm90Compute<ActivationFn_, ElementOutput, ElementCompute, RoundStyle_>,
      Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle_>,
        Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>,
        Sm90SrcFetch<ElementSource>,
        Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle_>,
          Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>,
          Sm90AccFetch,
          XeColBroadcast<0, CtaTileShapeMNK_, ElementBias, ElementCompute, Stride<_1,_0,int64_t>, AlignmentBias_>
        >
      >
    >;
  using Operation =
    fusion::XeLinCombPerRowBiasEltAct<
      ActivationFn_, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle_
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1104-1108

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1110-1113

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1115-1117

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1119-1120

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn_, ElementOutput, ElementCompute, RoundStyle_>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1122-1139

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


### Lines 1141-1143

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1149-1179

```cpp
template <
  template <class> class ActivationFn_,
  class ElementOutput_,
  class ElementCompute_,
  class ElementBias_,
  class ElementSource_,
  class ElementScalar_,
  int AlignmentBias_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
     epilogue::IntelXeGeneric,
    fusion::XeLinCombPerColBiasEltAct<
      ActivationFn_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_
    >,
    CtaTileShapeMNK_,
    EpilogueTile_
> : Sm90EVT<
      Sm90Compute<ActivationFn_, ElementOutput_, ElementCompute_, RoundStyle_>,
      Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute_, ElementCompute_, RoundStyle_>,
        Sm90ScalarBroadcast<ElementScalar_, Stride<_0,_0,int64_t>>,
        Sm90SrcFetch<ElementSource_>,
        Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute_, ElementCompute_, RoundStyle_>,
          Sm90ScalarBroadcast<ElementScalar_, Stride<_0,_0,int64_t>>,
          Sm90AccFetch,
          XeRowBroadcast<0, CtaTileShapeMNK_, ElementBias_, ElementCompute_, Stride<_0,_1,int64_t>, AlignmentBias_>
        >
      >
    > {
```

**EN:** Declares the templated `ActivationFn_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1181-1203

```cpp
  using ElementOutput = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementBias = ElementBias_;
  using ElementSource = ElementSource_;
  using ElementScalar = ElementScalar_;
  static constexpr int AlignmentBias = AlignmentBias_;
  using Impl =
    Sm90EVT<
      Sm90Compute<ActivationFn_, ElementOutput, ElementCompute, RoundStyle_>,
      Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle_>,
        Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>,
        Sm90SrcFetch<ElementSource>,
        Sm90EVT<Sm90Compute<homogeneous_multiply_add, ElementCompute, ElementCompute, RoundStyle_>,
          Sm90ScalarBroadcast<ElementScalar, Stride<_0,_0,int64_t>>,
          Sm90AccFetch,
          XeRowBroadcast<0, CtaTileShapeMNK_, ElementBias, ElementCompute, Stride<_0,_1,int64_t>, AlignmentBias_>
        >
      >
    >;
  using Operation =
    fusion::XeLinCombPerColBiasEltAct<
      ActivationFn_, ElementOutput, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle_
    >;
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1205-1209

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1211-1214

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1216-1218

```cpp
    using StrideBias = Stride<_0,_1,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1220-1221

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn_, ElementOutput, ElementCompute, RoundStyle_>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1223-1240

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


### Lines 1242-1244

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1250-1273

```cpp
// D = activation(alpha * acc + beta * C + per-row bias)
// Aux = alpha * acc + beta * C + per-row bias)
template<
  class CtaTileShapeMNK,
  class EpilogueTile,
  class StrideAux,
  class CopyOpR2G,
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
using XeLinCombPerRowBiasEltActAuxLegacy =
  Sm90EVT<Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>,
    Sm90EVT<XeAuxStoreLegacy<ElementAux, StrideAux, CopyOpR2G>,
      Sm90LinCombPerRowBias<CtaTileShapeMNK, ElementCompute, ElementCompute, ElementBias, ElementSource, ElementScalar, AlignmentBias, RoundStyle>
    >
  >;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: D = activation(alpha * acc + beta * C + per-row bias) Aux = alpha * acc + beta * C + per-row bias).

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1275-1303

```cpp
template <
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
  class CopyOpR2G
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinCombPerRowBiasEltActAux<
      GmemLayoutTagAux, ActivationFn, ElementOutput, ElementCompute,
      ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    CopyOpR2G
> : XeLinCombPerRowBiasEltActAuxLegacy<
      CtaTileShapeMNK, EpilogueTile, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, CopyOpR2G, ActivationFn,
      ElementOutput, ElementCompute, ElementAux, ElementBias, ElementSource, ElementScalar, AlignmentAux, AlignmentBias, RoundStyle
    > {
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1305-1314

```cpp
  using Impl =
    XeLinCombPerRowBiasEltActAuxLegacy<
      CtaTileShapeMNK, EpilogueTile, cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>, CopyOpR2G, ActivationFn,
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


### Lines 1316-1320

```cpp
  struct Arguments {
    ElementScalar alpha = ElementScalar(1);
    ElementScalar beta = ElementScalar(0);
    ElementScalar const* alpha_ptr = nullptr;
    ElementScalar const* beta_ptr = nullptr;
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 1322-1325

```cpp
    using StrideAlpha = Stride<_0,_0,int64_t>;
    using StrideBeta  = Stride<_0,_0,int64_t>;
    StrideAlpha dAlpha = {_0{}, _0{}, 0};
    StrideBeta  dBeta  = {_0{}, _0{}, 0};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1327-1329

```cpp
    using StrideBias = Stride<_1,_0,int64_t>;
    ElementBias const* bias_ptr = nullptr;
    StrideBias dBias = {};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1331-1332

```cpp
    using ActivationArguments = typename Sm90Compute<ActivationFn, ElementOutput, ElementCompute, RoundStyle>::Arguments;
    ActivationArguments activation = ActivationArguments();
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 1334-1336

```cpp
    using StrideAux = cutlass::gemm::TagToStrideC_t<GmemLayoutTagAux>;
    ElementAux* aux_ptr = nullptr;
    StrideAux dAux = {};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 1338-1358

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
            {aux_ptr, ElementAux(0), dAux} // unary args : store
          },                // end unary op
          activation // unary args : activation
        };   // end unary op
    }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 1360-1362

```cpp
  // Ctor inheritance
  using Impl::Impl;
};
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 1364-1389

```cpp
template <
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinearCombination<ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinearCombination<ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> {
  using FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinearCombination<ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_>::FusionCallbacks;
};
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1391-1417

```cpp
template <
  template <class> class ActivationFn_,
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinCombEltAct<ActivationFn_, ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombEltAct<ActivationFn_, ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> {
  using FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombEltAct<ActivationFn_, ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_>::FusionCallbacks;
};
```

**EN:** Declares the templated `ActivationFn_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1419-1462

```cpp
template <
  class GmemLayoutTagAux,
  template <class> class ActivationFn,
  class ElementOutput_,
  class ElementCompute_,
  class ElementAux,
  class ElementSource,
  class ElementScalar,
  int AlignmentAux,
  FloatRoundStyle RoundStyle,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class CopyOpG2R
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput_, ElementCompute_,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    CopyOpG2R
> : FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput_, ElementCompute_,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    CopyOpG2R
> {
  using FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombDeEltAct<
      GmemLayoutTagAux, ActivationFn, ElementOutput_, ElementCompute_,
      ElementAux, ElementSource, ElementScalar, AlignmentAux, RoundStyle
    >,
    CtaTileShapeMNK,
    EpilogueTile,
    CopyOpG2R
  >::FusionCallbacks;
};
```

**EN:** Declares the templated `GmemLayoutTagAux` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `GmemLayoutTagAux`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1464-1492

```cpp
template <
  class ElementOutput_,
  class ElementCompute_,
  class ElementBias_,
  class ElementSource_,
  class ElementScalar_,
  int AlignmentBias_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinCombPerRowBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerRowBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> {
  using FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerRowBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
  >::FusionCallbacks;
};
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1494-1522

```cpp
template <
  class ElementOutput_,
  class ElementCompute_,
  class ElementBias_,
  class ElementSource_,
  class ElementScalar_,
  int AlignmentBias_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinCombPerColBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerColBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> {
  using FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerColBias<ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
  >::FusionCallbacks;
};
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1524-1559

```cpp
template <
  template <class> class ActivationFn_,
  class ElementOutput_,
  class ElementCompute_,
  class ElementBias_,
  class ElementSource_,
  class ElementScalar_,
  int AlignmentBias_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16,
    fusion::LinCombPerRowBiasEltAct<
      ActivationFn_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_
    >,
    CtaTileShapeMNK_,
    EpilogueTile_
> : FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerRowBiasEltAct<
      ActivationFn_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_
    >,
    CtaTileShapeMNK_,
    EpilogueTile_
> {
  using FusionCallbacks<
    epilogue::IntelXeGeneric,
    fusion::LinCombPerRowBiasEltAct<
      ActivationFn_, ElementOutput_, ElementCompute_, ElementBias_, ElementSource_, ElementScalar_, AlignmentBias_, RoundStyle_
    >,
    CtaTileShapeMNK_,
    EpilogueTile_
  >::FusionCallbacks;
};
```

**EN:** Declares the templated `ActivationFn_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ActivationFn_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 1561-1587

```cpp
template <
  class ElementOutput_,
  class ElementCompute_,
  class ElementSource_,
  class ElementScalar_,
  FloatRoundStyle RoundStyle_,
  class CtaTileShapeMNK_,
  class EpilogueTile_
>
struct FusionCallbacks<
    epilogue::IntelXeXMX16Group,
    fusion::LinearCombination<ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> : FusionCallbacks<
    epilogue::IntelXeGenericGroup,
    fusion::LinearCombination<ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
> {
  using FusionCallbacks<
    epilogue::IntelXeGenericGroup,
    fusion::LinearCombination<ElementOutput_, ElementCompute_, ElementSource_, ElementScalar_, RoundStyle_>,
    CtaTileShapeMNK_,
    EpilogueTile_
  >::FusionCallbacks;
};
```

**EN:** Declares the templated `ElementOutput_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ElementOutput_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


## Key Concepts / 关键概念

- **Fusion framework / 融合框架:** Uses callbacks, visitors, or operation tags to compose multiple post-processing steps into the epilogue. / 通过回调、访问者或操作标签把多个后处理步骤组合进 epilogue。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。

- **Scaling semantics / 缩放语义:** Tracks how alpha/beta parameters mix accumulator values with source tensors during output generation. / 描述 alpha/beta 参数如何在生成输出时混合累加器值与源张量。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cute/tensor.hpp`, `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/xe_visitor.hpp`, `cutlass/epilogue/fusion/xe_visitor_legacy.hpp`, `cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_store_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_compute_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/xe_visitor_softmax.hpp`, `cutlass/epilogue/fusion/xe_visitor_splitk.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::fusion`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/xe_visitor.hpp`, `cutlass/epilogue/fusion/xe_visitor_legacy.hpp`, `cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_store_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/sm90_visitor_compute_tma_warpspecialized.hpp`, `cutlass/epilogue/fusion/xe_visitor_softmax.hpp`, `cutlass/epilogue/fusion/xe_visitor_splitk.hpp`

- **Cute/CuTe dependencies / Cute 依赖:** `cute/tensor.hpp`

- **Feature macros / 特性宏:** `CUTLASS_DEVICE`
