# blockwise_scale_layout.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/detail/blockwise_scale_layout.hpp`

- **EN:** Blockwise Scale configs specific for Blockwise/Groupwise MMA

- **CN:** 该头文件主要提供上层 CUTLASS 组件使用的内部辅助工具。文件级摘要：Blockwise Scale configs specific for Blockwise/Groupwise MMA

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

### Lines 34-36

```cpp
/*! \file
    \brief Blockwise Scale configs specific for Blockwise/Groupwise MMA
*/
```

**EN:** This file-level comment explains the purpose of the header and introduces the abstractions defined below.

**CN:** 这个文件级注释说明了头文件的用途，并引出了后续定义的抽象。

### Lines 38-38

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 40-40

```cpp
#include "cutlass/layout/matrix.h"
```

**EN:** This block imports dependencies such as `cutlass/layout/matrix.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/layout/matrix.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 42-44

```cpp
#include "cute/int_tuple.hpp"
#include "cute/atom/mma_traits_sm100.hpp"
#include "cute/arch/mma_sm90.hpp"
```

**EN:** This block imports dependencies such as `cute/int_tuple.hpp`, `cute/atom/mma_traits_sm100.hpp`, `cute/arch/mma_sm90.hpp`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cute/int_tuple.hpp`, `cute/atom/mma_traits_sm100.hpp`, `cute/arch/mma_sm90.hpp` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 46-46

```cpp
namespace cutlass::detail{
```

**EN:** This block opens the namespace scope `cutlass::detail` for the declarations that follow.

**CN:** 该代码块打开了 `cutlass::detail` 命名空间作用域，以容纳后续声明。

### Lines 48-49

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
using namespace cute;
```

**EN:** The preceding comment documents this block. This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 前面的注释说明了这个代码块。这个语句为周围抽象补充了一个实现细节。

### Lines 51-52

```cpp
template<int SFVecSizeM, int SFVecSizeN, int SFVecSizeK, UMMA::Major majorSFA = UMMA::Major::MN, UMMA::Major majorSFB = UMMA::Major::MN>
struct Sm1xxBlockwiseScaleConfig {
```

**EN:** This block begins the definition of `Sm1xxBlockwiseScaleConfig`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm1xxBlockwiseScaleConfig` 这个 `struct`，其成员会在后续代码中展开。

### Lines 54-54

```cpp
  using ShapeSFA = Shape<Shape<Int<SFVecSizeM>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>;
```

**EN:** This alias defines `ShapeSFA` as `Shape<Shape<Int<SFVecSizeM>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>`, shortening later template or member declarations.

**CN:** 这里把 `ShapeSFA` 定义为 `Shape<Shape<Int<SFVecSizeM>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>` 的别名，以简化后续模板或成员声明。

### Lines 55-55

```cpp
  using ShapeSFB = Shape<Shape<Int<SFVecSizeN>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>;
```

**EN:** This alias defines `ShapeSFB` as `Shape<Shape<Int<SFVecSizeN>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>`, shortening later template or member declarations.

**CN:** 这里把 `ShapeSFB` 定义为 `Shape<Shape<Int<SFVecSizeN>, int32_t>, Shape<Int<SFVecSizeK>, int32_t>, int32_t>` 的别名，以简化后续模板或成员声明。

### Lines 57-59

```cpp
  using StrideSFA = conditional_t<majorSFA == UMMA::Major::MN, 
      Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, 
      Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>;
```

**EN:** This alias defines `StrideSFA` as `conditional_t<majorSFA == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>`, shortening later template or member declarations.

**CN:** 这里把 `StrideSFA` 定义为 `conditional_t<majorSFA == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>` 的别名，以简化后续模板或成员声明。

### Lines 61-63

```cpp
  using StrideSFB = conditional_t<majorSFB == UMMA::Major::MN, 
      Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, 
      Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>;
```

**EN:** This alias defines `StrideSFB` as `conditional_t<majorSFB == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>`, shortening later template or member declarations.

**CN:** 这里把 `StrideSFB` 定义为 `conditional_t<majorSFB == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>` 的别名，以简化后续模板或成员声明。

### Lines 65-65

```cpp
  using LayoutSFA = Layout<ShapeSFA, StrideSFA>;
```

**EN:** This alias defines `LayoutSFA` as `Layout<ShapeSFA, StrideSFA>`, shortening later template or member declarations.

**CN:** 这里把 `LayoutSFA` 定义为 `Layout<ShapeSFA, StrideSFA>` 的别名，以简化后续模板或成员声明。

### Lines 66-66

```cpp
  using LayoutSFB = Layout<ShapeSFB, StrideSFB>;
```

**EN:** This alias defines `LayoutSFB` as `Layout<ShapeSFB, StrideSFB>`, shortening later template or member declarations.

**CN:** 这里把 `LayoutSFB` 定义为 `Layout<ShapeSFB, StrideSFB>` 的别名，以简化后续模板或成员声明。

### Lines 68-72

```cpp
  CUTE_HOST_DEVICE
  static constexpr auto
  deduce_layoutSFA() {
    return LayoutSFA{};
  }
```

**EN:** The function `deduce_layoutSFA` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `deduce_layoutSFA` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 74-95

```cpp
  template<typename CtaShape_MNK>
  CUTE_HOST_DEVICE
  static constexpr auto
  smem_atom_layoutSFA(CtaShape_MNK cta_shape_mnk) {
    static_assert(cute::is_static_v<CtaShape_MNK>, "Expect static CTA shape");
    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto [M, N, K] = cta_shape_mnk;
      if constexpr (majorSFA == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, Int<cute::ceil_div(size<0>(CtaShape_MNK{}), SFVecSizeM)>{}));
      }
      else {
        return make_stride(make_stride(_0{}, Int<cute::ceil_div(size<2>(CtaShape_MNK{}), SFVecSizeK)>{}), make_stride(_0{}, _1{}));
      }
    }();

    auto [M, N, K] = cta_shape_mnk;
    return make_layout(
      make_shape(make_shape(Int<SFVecSizeM>{}, Int<cute::ceil_div(size<0>(CtaShape_MNK{}), SFVecSizeM)>{}),
                 make_shape(Int<SFVecSizeK>{}, Int<cute::ceil_div(size<2>(CtaShape_MNK{}), SFVecSizeK)>{})),
      strides
    );
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 98-102

```cpp
  CUTE_HOST_DEVICE
  static constexpr auto
  deduce_layoutSFB() {
    return LayoutSFB{};
  }
```

**EN:** The function `deduce_layoutSFB` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `deduce_layoutSFB` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 104-124

```cpp
  template<typename CtaShape_MNK>
  CUTE_HOST_DEVICE
  static constexpr auto
  smem_atom_layoutSFB(CtaShape_MNK cta_shape_mnk) {
    static_assert(cute::is_static_v<CtaShape_MNK>, "Expect static CTA shape");
    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      if constexpr (majorSFA == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, Int<cute::ceil_div(size<1>(CtaShape_MNK{}), SFVecSizeN)>{}));
      }
      else {
        return make_stride(make_stride(_0{}, Int<cute::ceil_div(size<2>(CtaShape_MNK{}), SFVecSizeK)>{}), make_stride(_0{}, _1{}));
      }
    }();

    auto [M, N, K] = cta_shape_mnk;
    return make_layout(
      make_shape(make_shape(Int<SFVecSizeN>{}, Int<cute::ceil_div(size<1>(CtaShape_MNK{}), SFVecSizeN)>{}),
                 make_shape(Int<SFVecSizeK>{}, Int<cute::ceil_div(size<2>(CtaShape_MNK{}), SFVecSizeK)>{})),
      strides
    );
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 126-151

```cpp
  // The following function is provided for user fill dynamic problem size to the layout_SFA.
  template <class ProblemShape>
  CUTE_HOST_DEVICE
  static constexpr auto 
  tile_atom_to_shape_SFA(ProblemShape problem_shape) {
    auto problem_shape_MNKL = append<4>(problem_shape, 1);

    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto [M, N, K, L] = problem_shape_MNKL;
      if constexpr (majorSFA == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, cute::ceil_div(M, SFVecSizeM)));
      }
      else {
        return make_stride(make_stride(_0{}, cute::ceil_div(K, SFVecSizeK)), make_stride(_0{}, _1{}));
      }
    }();

    auto [M, N, K, L] = problem_shape_MNKL;
    auto mk_layout = make_layout(
      make_shape(make_shape(Int<SFVecSizeM>{}, cute::ceil_div(M, SFVecSizeM)),
                 make_shape(Int<SFVecSizeK>{}, cute::ceil_div(K, SFVecSizeK))),
      strides
    );

    return make_layout(append(shape(mk_layout), L), append(stride(mk_layout), size(filter_zeros(mk_layout))));
  }
```

**EN:** The preceding comment documents this block. The function `problem_shape_MNKL` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`problem_shape_MNKL` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 153-179

```cpp
  // The following function is provided for user fill dynamic problem size to the layout_SFB.
  template <class ProblemShape>
  CUTE_HOST_DEVICE
  static constexpr auto 
  tile_atom_to_shape_SFB(ProblemShape problem_shape) {
    auto problem_shape_MNKL = append<4>(problem_shape, 1);

    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto [M, N, K, L] = problem_shape_MNKL;

      if constexpr (majorSFB == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, cute::ceil_div(N, SFVecSizeN)));
      }
      else {
        return make_stride(make_stride(_0{}, cute::ceil_div(K, SFVecSizeK)), make_stride(_0{}, _1{}));
      }
    }();

    auto [M, N, K, L] = problem_shape_MNKL;
    auto nk_layout = make_layout(
      make_shape(make_shape(Int<SFVecSizeN>{}, cute::ceil_div(N, SFVecSizeN)),
                 make_shape(Int<SFVecSizeK>{}, cute::ceil_div(K, SFVecSizeK))),
      strides
    );

    return make_layout(append(shape(nk_layout), L), append(stride(nk_layout), size(filter_zeros(nk_layout))));
  }
```

**EN:** The preceding comment documents this block. The function `problem_shape_MNKL` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`problem_shape_MNKL` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 183-184

```cpp
template<UMMA::Major majorSFA = UMMA::Major::MN, UMMA::Major majorSFB = UMMA::Major::MN>
struct RuntimeBlockwiseScaleConfig {
```

**EN:** This block begins the definition of `RuntimeBlockwiseScaleConfig`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `RuntimeBlockwiseScaleConfig` 这个 `struct`，其成员会在后续代码中展开。

### Lines 186-186

```cpp
  using ShapeSFA = Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>;
```

**EN:** This alias defines `ShapeSFA` as `Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>`, shortening later template or member declarations.

**CN:** 这里把 `ShapeSFA` 定义为 `Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>` 的别名，以简化后续模板或成员声明。

### Lines 187-187

```cpp
  using ShapeSFB = Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>;
```

**EN:** This alias defines `ShapeSFB` as `Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>`, shortening later template or member declarations.

**CN:** 这里把 `ShapeSFB` 定义为 `Shape<Shape<int32_t, int32_t>, Shape<int32_t, int32_t>, int32_t>` 的别名，以简化后续模板或成员声明。

### Lines 189-191

```cpp
  using StrideSFA = conditional_t<majorSFA == UMMA::Major::MN, 
      Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, 
      Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>;
```

**EN:** This alias defines `StrideSFA` as `conditional_t<majorSFA == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>`, shortening later template or member declarations.

**CN:** 这里把 `StrideSFA` 定义为 `conditional_t<majorSFA == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>` 的别名，以简化后续模板或成员声明。

### Lines 193-195

```cpp
  using StrideSFB = conditional_t<majorSFB == UMMA::Major::MN, 
      Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, 
      Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>;
```

**EN:** This alias defines `StrideSFB` as `conditional_t<majorSFB == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>`, shortening later template or member declarations.

**CN:** 这里把 `StrideSFB` 定义为 `conditional_t<majorSFB == UMMA::Major::MN, Stride<Stride<_0,_1>,Stride<_0,int32_t>, int32_t>, Stride<Stride<_0,int32_t>,Stride<_0,_1>, int32_t>>` 的别名，以简化后续模板或成员声明。

### Lines 197-197

```cpp
  using LayoutSFA = Layout<ShapeSFA, StrideSFA>;
```

**EN:** This alias defines `LayoutSFA` as `Layout<ShapeSFA, StrideSFA>`, shortening later template or member declarations.

**CN:** 这里把 `LayoutSFA` 定义为 `Layout<ShapeSFA, StrideSFA>` 的别名，以简化后续模板或成员声明。

### Lines 198-198

```cpp
  using LayoutSFB = Layout<ShapeSFB, StrideSFB>;
```

**EN:** This alias defines `LayoutSFB` as `Layout<ShapeSFB, StrideSFB>`, shortening later template or member declarations.

**CN:** 这里把 `LayoutSFB` 定义为 `Layout<ShapeSFB, StrideSFB>` 的别名，以简化后续模板或成员声明。

### Lines 200-204

```cpp
  CUTE_HOST_DEVICE
  static constexpr auto
  deduce_layoutSFA() {
    return LayoutSFA{};
  }
```

**EN:** The function `deduce_layoutSFA` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `deduce_layoutSFA` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 206-210

```cpp
  CUTE_HOST_DEVICE
  static constexpr auto
  deduce_layoutSFB() {
    return LayoutSFB{};
  }
```

**EN:** The function `deduce_layoutSFB` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `deduce_layoutSFB` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 212-239

```cpp
  // The following function is provided for user fill dynamic problem size to the layout_SFA.
  template <class ProblemShape, class SFVecShape>
  CUTE_HOST_DEVICE
  static constexpr auto 
  tile_atom_to_shape_SFA(ProblemShape problem_shape, SFVecShape sf_vec_shape) {
    auto problem_shape_MNKL = append<4>(problem_shape, 1);

    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto [M, N, K, L] = problem_shape_MNKL;
      auto [sfm, sfn, sfk] = sf_vec_shape;
      if constexpr (majorSFA == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, cute::ceil_div(M, sfm)));
      }
      else {
        return make_stride(make_stride(_0{}, cute::ceil_div(K, sfk)), make_stride(_0{}, _1{}));
      }
    }();

    auto [M, N, K, L] = problem_shape_MNKL;
    auto [sfm, sfn, sfk] = sf_vec_shape;
    auto mk_layout = make_layout(
      make_shape(make_shape(sfm, cute::ceil_div(M, sfm)),
                 make_shape(sfk, cute::ceil_div(K, sfk))),
      strides
    );

    return make_layout(append(shape(mk_layout), L), append(stride(mk_layout), size(filter_zeros(mk_layout))));
  }
```

**EN:** The preceding comment documents this block. The function `problem_shape_MNKL` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`problem_shape_MNKL` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 241-269

```cpp
  // The following function is provided for user fill dynamic problem size to the layout_SFB.
  template <class ProblemShape, class SFVecShape>
  CUTE_HOST_DEVICE
  static constexpr auto 
  tile_atom_to_shape_SFB(ProblemShape problem_shape, SFVecShape sf_vec_shape) {
    auto problem_shape_MNKL = append<4>(problem_shape, 1);

    auto strides = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto [M, N, K, L] = problem_shape_MNKL;
      auto [sfm, sfn, sfk] = sf_vec_shape;

      if constexpr (majorSFB == UMMA::Major::MN) {
        return make_stride(make_stride(_0{}, _1{}), make_stride(_0{}, cute::ceil_div(N, sfn)));
      }
      else {
        return make_stride(make_stride(_0{}, cute::ceil_div(K, sfk)), make_stride(_0{}, _1{}));
      }
    }();

    auto [M, N, K, L] = problem_shape_MNKL;
    auto [sfm, sfn, sfk] = sf_vec_shape;
    auto nk_layout = make_layout(
      make_shape(make_shape(sfn, cute::ceil_div(N, sfn)),
                 make_shape(sfk, cute::ceil_div(K, sfk))),
      strides
    );

    return make_layout(append(shape(nk_layout), L), append(stride(nk_layout), size(filter_zeros(nk_layout))));
  }
```

**EN:** The preceding comment documents this block. The function `problem_shape_MNKL` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`problem_shape_MNKL` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 273-280

```cpp
// Sm90 only supports MN major for SFA and SFB for now
template<int SFVecSizeM, int SFVecSizeN, int SFVecSizeK, cute::GMMA::Major majorSFA = cute::GMMA::Major::MN, cute::GMMA::Major majorSFB = cute::GMMA::Major::MN>
using Sm90BlockwiseScaleConfig = Sm1xxBlockwiseScaleConfig<
    SFVecSizeM, 
    SFVecSizeN, 
    SFVecSizeK, 
    majorSFA == cute::GMMA::Major::MN ? UMMA::Major::MN : UMMA::Major::K, 
    majorSFB == cute::GMMA::Major::MN ? UMMA::Major::MN : UMMA::Major::K>;
```

**EN:** The preceding comment documents this block. This alias defines `Sm90BlockwiseScaleConfig` as `Sm1xxBlockwiseScaleConfig< SFVecSizeM, SFVecSizeN, SFVecSizeK, majorSFA == cute::GMMA::Major::MN ? UMMA::Major::MN : UMMA::Major::K, majorSFB == cute::GMMA::Major::MN ? UMMA::Major::MN : UMMA::Major::K>`, shortening later template or member declarations.

**CN:** 前面的注释说明了这个代码块。这里把 `Sm90BlockwiseScaleConfig` 定义为 `Sm1xxBlockwiseScaleConfig< SFVecSizeM, SFVecSizeN, SFVecSizeK, majorSFA == cute::GMMA::Major::MN ? UMMA::Major::MN : UMMA::Major::K, majorSFB == cute::GMMA::Major::MN ? UMMA::Major::MN : UMMA::Major::K>` 的别名，以简化后续模板或成员声明。

### Lines 282-283

```cpp
template<int SFVecSizeM, int SFVecSizeN, int SFVecSizeK, UMMA::Major majorSFA = UMMA::Major::MN, UMMA::Major majorSFB = UMMA::Major::MN>
using Sm100BlockwiseScaleConfig = Sm1xxBlockwiseScaleConfig<SFVecSizeM, SFVecSizeN, SFVecSizeK, majorSFA, majorSFB>;
```

**EN:** This alias defines `Sm100BlockwiseScaleConfig` as `Sm1xxBlockwiseScaleConfig<SFVecSizeM, SFVecSizeN, SFVecSizeK, majorSFA, majorSFB>`, shortening later template or member declarations.

**CN:** 这里把 `Sm100BlockwiseScaleConfig` 定义为 `Sm1xxBlockwiseScaleConfig<SFVecSizeM, SFVecSizeN, SFVecSizeK, majorSFA, majorSFB>` 的别名，以简化后续模板或成员声明。

### Lines 285-286

```cpp
template<int SFVecSizeM, int SFVecSizeN, int SFVecSizeK, UMMA::Major majorSFA = UMMA::Major::MN, UMMA::Major majorSFB = UMMA::Major::MN>
using Sm120BlockwiseScaleConfig = Sm1xxBlockwiseScaleConfig<SFVecSizeM, SFVecSizeN, SFVecSizeK, majorSFA, majorSFB>;
```

**EN:** This alias defines `Sm120BlockwiseScaleConfig` as `Sm1xxBlockwiseScaleConfig<SFVecSizeM, SFVecSizeN, SFVecSizeK, majorSFA, majorSFB>`, shortening later template or member declarations.

**CN:** 这里把 `Sm120BlockwiseScaleConfig` 定义为 `Sm1xxBlockwiseScaleConfig<SFVecSizeM, SFVecSizeN, SFVecSizeK, majorSFA, majorSFB>` 的别名，以简化后续模板或成员声明。

### Lines 288-291

```cpp
template<class MmaTileShape_MNK>
constexpr auto sm90_trivial_blockwise_scale_config(MmaTileShape_MNK) {
  return Sm90BlockwiseScaleConfig<size<0>(MmaTileShape_MNK{}), size<1>(MmaTileShape_MNK{}), size<2>(MmaTileShape_MNK{})>{};
}
```

**EN:** The function `sm90_trivial_blockwise_scale_config` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `sm90_trivial_blockwise_scale_config` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 293-296

```cpp
template<class MmaTileShape_MNK>
constexpr auto sm100_trivial_blockwise_scale_config(MmaTileShape_MNK) {
  return Sm100BlockwiseScaleConfig<size<0>(MmaTileShape_MNK{}), size<1>(MmaTileShape_MNK{}), size<2>(MmaTileShape_MNK{})>{};
}
```

**EN:** The function `sm100_trivial_blockwise_scale_config` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `sm100_trivial_blockwise_scale_config` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 298-301

```cpp
template<class MmaTileShape_MNK>
constexpr auto sm120_trivial_blockwise_scale_config(MmaTileShape_MNK) {
  return Sm120BlockwiseScaleConfig<size<0>(MmaTileShape_MNK{}), size<1>(MmaTileShape_MNK{}), size<2>(MmaTileShape_MNK{})>{};
}
```

**EN:** The function `sm120_trivial_blockwise_scale_config` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `sm120_trivial_blockwise_scale_config` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Type traits and compile-time checks constrain valid instantiations.
  **CN:** 类型萃取与编译期检查用于约束合法的模板实例化。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/layout/matrix.h`, `cute/int_tuple.hpp`, `cute/atom/mma_traits_sm100.hpp`, `cute/arch/mma_sm90.hpp`.
  **CN:** 直接包含：`cutlass/layout/matrix.h`, `cute/int_tuple.hpp`, `cute/atom/mma_traits_sm100.hpp`, `cute/arch/mma_sm90.hpp`。

- **EN:** Primary namespaces: `cutlass::detail`, `cute;`.
  **CN:** 主要命名空间：`cutlass::detail`, `cute;`。

- **EN:** Important macros or compile flags: `CUTLASS_LAMBDA_FUNC_INLINE`.
  **CN:** 重要宏或编译开关：`CUTLASS_LAMBDA_FUNC_INLINE`。
