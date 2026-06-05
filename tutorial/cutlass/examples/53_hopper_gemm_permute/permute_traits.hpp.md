# permute_traits.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/53_hopper_gemm_permute/permute_traits.hpp`  
**Purpose / 用途**: This header defines trait specializations that translate older CUTLASS permutation layout tags into CuTe shape/stride descriptions. It is the bridge that lets the Hopper permutation example reason about permuted tensors using CuTe-native layout objects. / 该头文件通过一组 trait 特化，把旧版 CUTLASS 的 permutation 布局标签翻译成 CuTe 的 shape/stride 描述。它相当于 Hopper permutation 示例中的“桥接层”，让示例能够用 CuTe 原生布局对象来理解经过排列的张量。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-74 — Base trait shell, placeholder symbol, and reshape helper

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
    \brief Additional permutation information for the example.
*/

#include "cutlass/layout/permute.h"
#include "cutlass/gemm/gemm.h"

namespace example
{

using namespace cute;

// This struct is specialized below for different CUTLASS 2.x permutation ops
// to describe the operation in terms of target CuTe shape and stride order.
template<class Permute>
struct PermuteTraits {};

// Use X as a placeholder for shape division result
using X = Underscore;

// Reshape a rank-2 shape into a multidimensional shape.
// Input:
//   shape = (A, B, ...)
//   target_shape = ((A1, ..., X, ..., Am), (B1, ..., X, ..., Bn), ...)
// Output:
//   ((A1, ..., A/prod(A1..Am), ..., Am), (B1, ..., B/prod(B1..Bn), ..., Bn), ...)
template<class Shape, class TargetShape>
constexpr auto
reshape(Shape const& shape, TargetShape const& target_shape)
{
  if constexpr (is_tuple<Shape>::value) {
    return cute::transform(shape, target_shape, [](auto && s, auto && t){ return reshape(s, t); });
  }
  else {
    auto idx = find_if(target_shape, [](auto x){ return is_underscore<decltype(x)>{}; });
    constexpr int I = decltype(idx)::value;
    static_assert(I < tuple_size_v<TargetShape>, "Each mode of TargetShape must contain a placeholder X");
    auto divisors = remove<I>(target_shape);
    assert(shape % product(divisors) == 0);
    return replace<I>(target_shape, shape / product(divisors));
  }
}
```

**EN**: The file starts by declaring an empty `PermuteTraits` template that will later be specialized per permutation type. The placeholder alias `X = Underscore` is then used by `reshape()`, which takes a flat rank-2 shape and expands it into a nested multidimensional shape profile while solving for one inferred dimension in each mode. This is the foundational utility that lets permutation traits describe how a logical matrix should be interpreted as a higher-rank tensor.

**CN**: 文件首先声明一个空的 `PermuteTraits` 模板，后面会针对不同排列类型进行特化。接着定义占位符 `X = Underscore`，并由 `reshape()` 使用：它能把扁平的 rank-2 形状扩展成嵌套的多维 shape profile，同时自动推导每个 mode 中唯一未知的那一维。这个工具是整份头文件的基础，因为 permutation trait 正是依靠它把“逻辑矩阵”重新解释成更高维张量。

### Lines 75-100 — Building permuted CuTe layouts

```cpp
// Given a tensor layout, compute a permutation layout consisting of:
// - sub-modes corresponding to the implied multidimensional shape of the source tensor
// - strides accounting for the permutation operation being performed
template<class Permute, bool Transpose, class Shape, class Stride>
constexpr auto
make_permute_layout(Layout<Shape,Stride> const& layout) {
  static_assert(cute::rank(Shape{}) == 3, "Only rank-3 layouts are supported");
  if constexpr (Transpose) {
    // Deal with tensor B by transposing appropriately before and after computing the permute layout.
    // Its CuTe-canonical mode order is [N,K,L], while permute operations expect [row,col,batch].
    return select<1,0,2>(make_permute_layout<Permute, false>(select<1,0,2>(layout)));
  }
  else {
    if constexpr (cutlass::layout::is_trivial_permute<Permute>) {
      // Special case for NoPermute. Use a depth-2 layout for consistency with other permutations.
      using ShapeProfile = tuple<tuple<X>, tuple<X>, tuple<X>>;
      return unflatten(layout, ShapeProfile{});
    }
    else {
      // Here's where the permutation layout is actually built
      using ShapeProfile = typename PermuteTraits<Permute>::ShapeProfile;
      using StrideOrder  = typename PermuteTraits<Permute>::StrideOrder;
      return make_ordered_layout(reshape(layout.shape(), ShapeProfile{}), StrideOrder{});
    }
  }
}
```

**EN**: `make_permute_layout()` converts a CUTLASS layout into a CuTe layout that explicitly encodes the permutation structure. It handles B-tensor transposition specially because CuTe’s canonical mode order for B differs from the row/column/batch order expected by the permutation operators. For trivial permutations it returns a consistent depth-2 layout, and for real permutations it combines `ShapeProfile` and `StrideOrder` from the corresponding trait specialization to produce an ordered layout.

**CN**: `make_permute_layout()` 会把 CUTLASS 布局转换成显式编码排列结构的 CuTe 布局。由于 CuTe 中 B 张量的规范 mode 顺序与 permutation 运算期望的“行/列/批次”顺序不同，所以它对 B 的转置做了专门处理。若排列是平凡的（例如 NoPermute），它会返回一个统一风格的 depth-2 布局；若是真正的排列，则会结合对应 trait 特化中的 `ShapeProfile` 和 `StrideOrder` 生成有序布局。

### Lines 101-154 — Permutation inversion and reconstruction of original layouts

```cpp

namespace detail
{

template<int I>
struct is_constant_pred {
  template <class T>
  constexpr auto operator()(T) {
    return is_constant<I, T>{};
  }
};

template<class Permutation, int... I>
constexpr auto
inverse_impl(Permutation const & perm, seq<I...>) {
  return cute::make_tuple(Int<find_if(Permutation{}, is_constant_pred<I>{})>{}...);
}

} // namespace detail

// Compute an inverse of a permutation represented as a tuple of cute::Int<>
template<class Permutation>
constexpr auto
inverse(Permutation const & perm) {
  auto flat_perm = flatten(perm);
  return unflatten(detail::inverse_impl(flat_perm, tuple_seq<decltype(flat_perm)>{}), perm);
}

template<class T>
using inverse_t = decltype(inverse(T{}));

// Given a rank-2 layout of tensor that is assumed to have been permuted,
// compute the original rank-2 layout of the tensor prior to the permutation.
// This is needed to form the correct input to the standalone permutation kernel.
template<class Permute, bool Transpose, class Shape, class Stride>
constexpr auto
make_original_layout(Layout<Shape,Stride> const& layout) {
  static_assert(cute::rank(Shape{}) == 3, "Only rank-3 layouts are supported");
  if constexpr (Transpose) {
    // Deal with tensor B by transposing appropriately before and after computing the permute layout.
    // Its CuTe-canonical mode order is [N,K,L], while permute operations expect [row,col,batch].
    return select<1,0,2>(make_original_layout<Permute, false>(select<1,0,2>(layout)));
  }
  else {
    using ShapeProfile = typename PermuteTraits<Permute>::ShapeProfile;
    auto re_shape   = flatten(reshape(layout.shape(), ShapeProfile{}));
    using IndexOrder   = typename PermuteTraits<Permute>::IndexOrder;
    auto orig_shape = transform_leaf(IndexOrder{}, [&](auto i){ return get<i>(re_shape); });
    using OrigOrder    = conditional_t<cutlass::gemm::detail::is_major<0,Stride>(), seq<0,1,2>, seq<1,0,2>>;
    // print("Permuted shape: "); print(reshape(layout.shape(), ShapeProfile{})); print("\n");
    // print("Original shape: "); print(orig_shape); print("\n");
    return make_ordered_layout(product_each(orig_shape), OrigOrder{});
  }
}
```

**EN**: The `detail::inverse_impl()` utility and the public `inverse()` wrapper compute inverse permutations expressed as tuples of `cute::Int<>`. `make_original_layout()` then uses those trait-level descriptions to reconstruct the pre-permutation rank-2 layout of a tensor. This is important because the standalone permutation kernel needs to know not only how the output is permuted, but also how the original unpermuted storage should be interpreted.

**CN**: `detail::inverse_impl()` 与外层 `inverse()` 负责对由 `cute::Int<>` 元组表示的排列求逆。随后 `make_original_layout()` 利用这些 trait 层面的描述，重建张量在排列之前的 rank-2 原始布局。这一点非常重要，因为独立的 permutation kernel 不仅要知道输出如何被排列，还要知道原始未排列存储应当如何解释。

### Lines 155-274 — Trait specializations for 4D/5D and batched permutation operators

```cpp

/////////////// Tensor4DPermute0213 ////////////////////

template<int D1, int D2>
struct PermuteTraits<cutlass::layout::Tensor4DPermute0213ColumnMajor<D1, D2>>
{
  static constexpr bool kBatched = false;
  using ShapeProfile = Shape<Shape<X,Int<D1>>, Shape<Int<D2>,X>, Shape<X>>;
  using IndexOrder   = Step<Step<_0,_2>, Step<_1,_3>, Step<_4>>;
  using StrideOrder = inverse_t<IndexOrder>; // Step<Step<_0,_2>, Step<_1,_3>, Step<_4>>;
};

template<int D1, int D2>
struct PermuteTraits<cutlass::layout::Tensor4DPermute0213ColumnMajorInverse<D1, D2>>
{
  static constexpr bool kBatched = false;
  using ShapeProfile = Shape<Shape<X,Int<D2>>, Shape<Int<D1>,X>, Shape<X>>;
  using IndexOrder   = Step<Step<_0,_2>, Step<_1,_3>, Step<_4>>;
  using StrideOrder  = inverse_t<IndexOrder>; // Step<Step<_0,_2>, Step<_1,_3>, Step<_4>>;
};

template<int D1, int D2>
struct PermuteTraits<cutlass::layout::Tensor4DPermute0213RowMajor<D1, D2>>
{
  static constexpr bool kBatched = false;
  using ShapeProfile = Shape<Shape<Int<D1>,X>, Shape<X,Int<D2>>, Shape<X>>;
  using IndexOrder   = Step<Step<_1,_3>, Step<_0,_2>, Step<_4>>;
  using StrideOrder  = Step<Step<_1,_3>, Step<_0,_2>, Step<_4>>;
};

template<int D1, int D2>
struct PermuteTraits<cutlass::layout::Tensor4DPermute0213RowMajorInverse<D1, D2>>
{
  static constexpr bool kBatched = false;
  using ShapeProfile = Shape<Shape<Int<D2>,X>, Shape<X,Int<D1>>, Shape<X>>;
  using IndexOrder   = Step<Step<_1,_3>, Step<_0,_2>, Step<_4>>;
  using StrideOrder  = Step<Step<_1,_3>, Step<_0,_2>, Step<_4>>;
};

/////////////// Tensor4DPermuteBMM0321 ////////////////////

template<int D>
struct PermuteTraits<cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D>>
{
  static constexpr bool kBatched = true;
  using ShapeProfile = Shape<Shape<X>, Shape<X>, Shape<Int<D>,X>>;
  using IndexOrder   = Step<Step<_0,_2>, Step<_1>, Step<_3>>;
  using StrideOrder  = Step<Step<_0>, Step<_2>, Step<_1,_3>>;
};

template<int D>
struct PermuteTraits<cutlass::layout::Tensor4DPermuteBMM0321ColumnMajorInverse<D>>
{
  static constexpr bool kBatched = true;
  using ShapeProfile = Shape<Shape<X,Int<D>>, Shape<X>, Shape<X>>;
  using IndexOrder   = Step<Step<_0>, Step<_2>, Step<_1,_3>>;
  using StrideOrder  = Step<Step<_0,_2>, Step<_1>, Step<_3>>;
};

/////////////// Tensor4DPermuteBMM0213 ////////////////////

template<int D>
struct PermuteTraits<cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D>>
{
  static constexpr bool kBatched = true;
  using ShapeProfile = Shape<Shape<X>, Shape<X>, Shape<Int<D>,X>>;
  using IndexOrder   = Step<Step<_0>, Step<_1,_2>, Step<_3>>;
  using StrideOrder  = Step<Step<_2>, Step<_0>, Step<_1,_3>>;
};

template<int D>
struct PermuteTraits<cutlass::layout::Tensor4DPermuteBMM0213RowMajorInverse<D>>
{
  static constexpr bool kBatched = true;
  using ShapeProfile = Shape<Shape<X>, Shape<X,Int<D>>, Shape<X>>;
  using IndexOrder   = Step<Step<_0>, Step<_1>, Step<_2,_3>>;
  using StrideOrder  = Step<Step<_1>, Step<_0,_2>, Step<_3>>;
};

/////////////// Tensor5DPermute02413 ////////////////////

template<int D1, int D2, int D3>
struct PermuteTraits<cutlass::layout::Tensor5DPermute02413ColumnMajor<D1, D2, D3>>
{
  static constexpr bool kBatched = false;
  using ShapeProfile = Shape<Shape<X,Int<D1>>, Shape<Int<D2>,Int<D3>,X>, Shape<X>>;
  using IndexOrder   = Step<Step<_0,_2>, Step<_4,_1,_3>, Step<_5>>;
  using StrideOrder  = inverse_t<IndexOrder>; // Step<Step<_0,_3>, Step<_1,_4,_2>, Step<_5>>;
};

template<int D1, int D2, int D3>
struct PermuteTraits<cutlass::layout::Tensor5DPermute02413ColumnMajorInverse<D1, D2, D3>>
{
  static constexpr bool kBatched = false;
  using ShapeProfile = Shape<Shape<X,Int<D2>>, Shape<X,Int<D1>,Int<D3>>, Shape<X>>;
  using IndexOrder   = Step<Step<_0,_3>, Step<_1,_4,_2>, Step<_5>>;
  using StrideOrder  = inverse_t<IndexOrder>; // Step<Step<_0,_2>, Step<_4,_1,_3>, Step<_5>>;
};

/////////////// Tensor5DPermute20314 ////////////////////

template<int D1, int D2, int D3>
struct PermuteTraits<cutlass::layout::Tensor5DPermute20314RowMajor<D1, D2, D3>>
{
  static constexpr bool kBatched = false;
  using ShapeProfile = Shape<Shape<Int<D1>,X>, Shape<X,Int<D3>,Int<D2>>, Shape<X>>;
  using IndexOrder   = Step<Step<_2,_0>, Step<_3,_1,_4>, Step<_5>>;
  using StrideOrder  = Step<Step<_1,_3>, Step<_0,_2,_4>, Step<_5>>;
};

template<int D1, int D2, int D3>
struct PermuteTraits<cutlass::layout::Tensor5DPermute20314RowMajorInverse<D1, D2, D3>>
{
  static constexpr bool kBatched = false;
  using ShapeProfile = Shape<Shape<X,Int<D2>>, Shape<X,Int<D1>,Int<D3>>, Shape<X>>;
  using IndexOrder   = Step<Step<_3,_0>, Step<_2,_4,_1>, Step<_5>>;
  using StrideOrder  = Step<Step<_4,_2>, Step<_0,_3,_1>, Step<_5>>;
};

} // namespace example
```

**EN**: The remainder of the file is effectively a catalog of permutation metadata. Each specialization encodes whether the permutation is batched, how the logical matrix should be split into a nested `ShapeProfile`, how original indices map to permuted indices (`IndexOrder`), and how the resulting CuTe layout should order strides (`StrideOrder`). The examples cover both forward and inverse forms, row-major and column-major variants, batched BMM-style permutations, and higher-rank 5D permutations. Together these specializations make permutation traits declarative: the example can ask the trait for shape/stride rules instead of hard-coding per-layout logic in the kernel path.

**CN**: 文件后半部分本质上是一张“排列元数据表”。每个特化都编码了该排列是否为 batched、逻辑矩阵应如何拆成嵌套的 `ShapeProfile`、原始索引到排列后索引的映射方式（`IndexOrder`），以及 CuTe 布局应采用怎样的步长顺序（`StrideOrder`）。这些特化同时覆盖了正向与逆向版本、行主序与列主序版本、batched BMM 风格排列以及更高阶的 5D 排列。借助这些定义，示例就能以声明式方式查询 shape/stride 规则，而不必在 kernel 路径里为每种布局硬编码特殊逻辑。

---

## Key Concepts / 关键概念

- **ShapeProfile**
  - **EN**: A `ShapeProfile` tells CuTe how to split a flat logical extent into nested submodes that match the intended tensor permutation.
  - **CN**: `ShapeProfile` 告诉 CuTe 如何把扁平逻辑维度拆成嵌套子模式，以匹配目标张量排列。
- **IndexOrder vs StrideOrder**
  - **EN**: `IndexOrder` describes how logical coordinates are permuted, while `StrideOrder` describes how the resulting CuTe layout should order its strides.
  - **CN**: `IndexOrder` 描述逻辑坐标如何被重新排列，而 `StrideOrder` 描述结果 CuTe 布局的步长应按什么顺序组织。
- **Inverse layout recovery**
  - **EN**: The helper functions can reconstruct the original layout before permutation, which is necessary when a separate permutation kernel must read the source tensor correctly.
  - **CN**: 这些辅助函数能够恢复排列之前的原始布局，这在独立 permutation kernel 需要正确读取源张量时是必需的。
- **Batched permutation traits**
  - **EN**: The `kBatched` flag distinguishes permutations that reinterpret one mode as a batch-oriented structure rather than a simple spatial split.
  - **CN**: `kBatched` 标志用于区分“把某个 mode 解释成 batch 结构”的排列与普通空间维拆分。

## Dependencies / 依赖项

- `cutlass/layout/permute.h`
  - **EN**: Defines the legacy CUTLASS permutation layout tags that are being translated into trait metadata here.
  - **CN**: 定义了这里要被翻译成 trait 元数据的旧版 CUTLASS permutation 布局标签。
- CuTe tuple/layout utilities
  - **EN**: Tuple transforms, flatten/unflatten, ordered layouts, and compile-time constants are all provided by CuTe.
  - **CN**: 元组变换、flatten/unflatten、有序布局以及编译期常量等能力都由 CuTe 提供。
- `cutlass/gemm/gemm.h`
  - **EN**: Provides GEMM-related helpers such as major-order inspection used when reconstructing original layouts.
  - **CN**: 提供了在重建原始布局时会用到的 GEMM 相关辅助能力，例如主序判断。
