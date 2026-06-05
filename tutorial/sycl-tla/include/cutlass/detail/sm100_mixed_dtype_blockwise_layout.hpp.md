# sm100_mixed_dtype_blockwise_layout.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/detail/sm100_mixed_dtype_blockwise_layout.hpp`

- **EN:** Block Wise Scale configs specific for SM100 Blockwise/Groupwise MMA

- **CN:** 该头文件主要提供上层 CUTLASS 组件使用的内部辅助工具。文件级摘要：Block Wise Scale configs specific for SM100 Blockwise/Groupwise MMA

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

**EN:** This block records the file copyright, SPDX identifier, and redistribution disclaimer.

**CN:** 该代码块记录了文件的版权信息、SPDX 标识以及再分发免责声明。

### Lines 32-34

```cpp
/*! \file
    \brief Block Wise Scale configs specific for SM100 Blockwise/Groupwise MMA
*/
```

**EN:** This file-level comment explains the purpose of the header and introduces the abstractions defined below.

**CN:** 这个文件级注释说明了头文件的用途，并引出了后续定义的抽象。

### Lines 36-36

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 38-38

```cpp
#include "cutlass/layout/matrix.h"
```

**EN:** This block imports dependencies such as `cutlass/layout/matrix.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/layout/matrix.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 40-41

```cpp
#include "cute/int_tuple.hpp"
#include "cute/atom/mma_traits_sm100.hpp"
```

**EN:** This block imports dependencies such as `cute/int_tuple.hpp`, `cute/atom/mma_traits_sm100.hpp`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cute/int_tuple.hpp`, `cute/atom/mma_traits_sm100.hpp` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 43-43

```cpp
namespace cutlass::detail{
```

**EN:** This block opens the namespace scope `cutlass::detail` for the declarations that follow.

**CN:** 该代码块打开了 `cutlass::detail` 命名空间作用域，以容纳后续声明。

### Lines 45-46

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
using namespace cute;
```

**EN:** The preceding comment documents this block. This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 前面的注释说明了这个代码块。这个语句为周围抽象补充了一个实现细节。

### Lines 48-49

```cpp
template<int SFVecSizeMN, int SFVecSizeK, UMMA::Major majorSFA = UMMA::Major::MN>
struct Sm100MixedInputBlockwiseScaleConfig {
```

**EN:** This block begins the definition of `Sm100MixedInputBlockwiseScaleConfig`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm100MixedInputBlockwiseScaleConfig` 这个 `struct`，其成员会在后续代码中展开。

### Lines 51-51

```cpp
  using ShapeScale = Shape<Shape<Int<SFVecSizeMN>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>;
```

**EN:** This alias defines `ShapeScale` as `Shape<Shape<Int<SFVecSizeMN>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>`, shortening later template or member declarations.

**CN:** 这里把 `ShapeScale` 定义为 `Shape<Shape<Int<SFVecSizeMN>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>` 的别名，以简化后续模板或成员声明。

### Lines 53-55

```cpp
  using StrideScale = conditional_t<majorSFA == UMMA::Major::MN, 
      Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, 
      Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>;
```

**EN:** This alias defines `StrideScale` as `conditional_t<majorSFA == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>`, shortening later template or member declarations.

**CN:** 这里把 `StrideScale` 定义为 `conditional_t<majorSFA == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>` 的别名，以简化后续模板或成员声明。

### Lines 57-57

```cpp
  using LayoutScale = Layout<ShapeScale, StrideScale>;
```

**EN:** This alias defines `LayoutScale` as `Layout<ShapeScale, StrideScale>`, shortening later template or member declarations.

**CN:** 这里把 `LayoutScale` 定义为 `Layout<ShapeScale, StrideScale>` 的别名，以简化后续模板或成员声明。

### Lines 59-63

```cpp
  CUTE_HOST_DEVICE
  static constexpr auto
  deduce_layout_scale() {
    return LayoutScale{};
  }
```

**EN:** The function `deduce_layout_scale` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `deduce_layout_scale` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 65-99

```cpp
  template<class CtaShape_MN_K>
  CUTE_HOST_DEVICE
  static constexpr auto
  smem_atom_layout_scale(CtaShape_MN_K cta_shape_mn_k) {
    static_assert(cute::is_static_v<CtaShape_MN_K>, "Expect static CTA shape");

    int constexpr size_MN = cute::get<0>(CtaShape_MN_K{});
    int constexpr size_K = cute::get<1>(CtaShape_MN_K{});

    int constexpr SmemSizeMN = (SFVecSizeMN < size_MN) 
                           ? SFVecSizeMN 
                           : size_MN;

    int constexpr SmemSizeK = (SFVecSizeK < size_K) 
                           ? SFVecSizeK 
                           : size_K;

    int constexpr div_MN = cute::ceil_div(size_MN, SmemSizeMN);
    int constexpr div_K = cute::ceil_div(size_K, SmemSizeK);
    
    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      if constexpr (majorSFA == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, Int<div_MN>{}));
      }
      else {
        return make_stride(make_stride(_0{}, Int<div_K>{}), make_stride(_0{}, _1{}));
      }
    }();

    return make_layout(
      make_shape(make_shape(Int<SmemSizeMN>{}, Int<div_MN>{}),
                 make_shape(Int<SmemSizeK>{}, Int<div_K>{})),
      strides
    );
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 103-128

```cpp
  // The following function is provided for user fill dynamic problem size to the layout_SFA.
  template <class ScaledInputDim>
  CUTE_HOST_DEVICE
  static constexpr auto 
  tile_atom_to_shape_scale(ScaledInputDim scale_input_dims) {
    const auto scale_input_dims_MNKL = append<3>(scale_input_dims, 1);

    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto [MN, K, L] = scale_input_dims_MNKL;
      if constexpr (majorSFA == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, cute::ceil_div(MN, SFVecSizeMN)));
      }
      else {
        return make_stride(make_stride(_0{}, cute::ceil_div(K, SFVecSizeK)), make_stride(_0{}, _1{}));
      }
    }();

    auto [MN, K, L] = scale_input_dims_MNKL;
    auto mk_layout = make_layout(
      make_shape(make_shape(Int<SFVecSizeMN>{}, cute::ceil_div(MN, SFVecSizeMN)),
                 make_shape(Int<SFVecSizeK>{}, cute::ceil_div(K, SFVecSizeK))),
      strides
    );

    return make_layout(append(shape(mk_layout), L), append(stride(mk_layout), size(filter_zeros(mk_layout))));
  }
```

**EN:** The preceding comment documents this block. The function `scale_input_dims_MNKL` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`scale_input_dims_MNKL` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 132-133

```cpp
template<UMMA::Major majorScale = UMMA::Major::MN>
struct RuntimeMixedInputBlockwiseScaleConfig {
```

**EN:** This block begins the definition of `RuntimeMixedInputBlockwiseScaleConfig`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `RuntimeMixedInputBlockwiseScaleConfig` 这个 `struct`，其成员会在后续代码中展开。

### Lines 135-135

```cpp
  using ShapeScale = Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>;
```

**EN:** This alias defines `ShapeScale` as `Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>`, shortening later template or member declarations.

**CN:** 这里把 `ShapeScale` 定义为 `Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>` 的别名，以简化后续模板或成员声明。

### Lines 137-139

```cpp
  using StrideScale = conditional_t<majorScale == UMMA::Major::MN, 
      Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, 
      Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>;
```

**EN:** This alias defines `StrideScale` as `conditional_t<majorScale == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>`, shortening later template or member declarations.

**CN:** 这里把 `StrideScale` 定义为 `conditional_t<majorScale == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>` 的别名，以简化后续模板或成员声明。

### Lines 141-141

```cpp
  using LayoutScale = Layout<ShapeScale, StrideScale>;
```

**EN:** This alias defines `LayoutScale` as `Layout<ShapeScale, StrideScale>`, shortening later template or member declarations.

**CN:** 这里把 `LayoutScale` 定义为 `Layout<ShapeScale, StrideScale>` 的别名，以简化后续模板或成员声明。

### Lines 143-147

```cpp
  CUTE_HOST_DEVICE
  static constexpr auto
  deduce_layout_scale() {
    return LayoutScale{};
  }
```

**EN:** The function `deduce_layout_scale` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `deduce_layout_scale` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 149-176

```cpp
  // The following function is provided for user fill dynamic problem size to the layout_S.
  template <class ProblemShape, class SFVecShape>
  CUTE_HOST_DEVICE
  static constexpr auto 
  tile_atom_to_shape_scale(ProblemShape problem_shape, SFVecShape sf_vec_shape) {
    auto problem_shape_MNKL = append<3>(problem_shape, 1);

    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto [MN, K, L] = problem_shape_MNKL;
      auto [sfmn, sfk] = sf_vec_shape;
      if constexpr (majorScale == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, cute::ceil_div(MN, sfmn)));
      }
      else {
        return make_stride(make_stride(_0{}, cute::ceil_div(K, sfk)), make_stride(_0{}, _1{}));
      }
    }();

    auto [MN, K, L] = problem_shape_MNKL;
    auto [sfmn, sfk] = sf_vec_shape;
    auto mk_layout = make_layout(
      make_shape(make_shape(sfmn, cute::ceil_div(MN, sfmn)),
                 make_shape(sfk, cute::ceil_div(K, sfk))),
      strides
    );

    return make_layout(append(shape(mk_layout), L), append(stride(mk_layout), size(filter_zeros(mk_layout))));
  }
```

**EN:** The preceding comment documents this block. The function `problem_shape_MNKL` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`problem_shape_MNKL` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Type traits and compile-time checks constrain valid instantiations.
  **CN:** 类型萃取与编译期检查用于约束合法的模板实例化。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/layout/matrix.h`, `cute/int_tuple.hpp`, `cute/atom/mma_traits_sm100.hpp`.
  **CN:** 直接包含：`cutlass/layout/matrix.h`, `cute/int_tuple.hpp`, `cute/atom/mma_traits_sm100.hpp`。

- **EN:** Primary namespaces: `cutlass::detail`, `cute;`.
  **CN:** 主要命名空间：`cutlass::detail`, `cute;`。

- **EN:** Important macros or compile flags: `CUTLASS_LAMBDA_FUNC_INLINE`.
  **CN:** 重要宏或编译开关：`CUTLASS_LAMBDA_FUNC_INLINE`。
