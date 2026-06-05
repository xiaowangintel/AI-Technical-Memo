# copy.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/algorithm/copy.hpp`
- **EN:** Implements generic copy algorithms over CuTe tensors and layouts.
- **CN:** 实现面向 CuTe 张量与布局的通用拷贝算法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
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
#pragma once
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 33-35
```cpp
#include <cute/config.hpp>            // CUTE_HOST_DEVICE
#include <cute/tensor_impl.hpp>       // cute::Tensor
#include <cute/atom/copy_atom.hpp>    // cute::Copy_Atom
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/tensor_impl.hpp` (tensor storage, indexing, and layout implementation details); `cute/atom/copy_atom.hpp` (copy atoms that combine traits with tiled tensor views).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/tensor_impl.hpp`（张量存储、索引与布局实现细节）；`cute/atom/copy_atom.hpp`（把 traits 与分块张量视图结合起来的 copy atom）。

### Lines 37-38
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 40-62
```cpp
//
// copy_if -- Predicated Copy
//

template <class PrdTensor,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_if(PrdTensor                    const& pred,
        Tensor<SrcEngine, SrcLayout> const& src,
        Tensor<DstEngine, DstLayout>      & dst)
{
  using SrcType = typename SrcEngine::value_type;
  using DstType = typename DstEngine::value_type;

  CUTE_UNROLL
  for (int i = 0; i < size(dst); ++i) {
    if (pred(i)) {
      dst(i) = static_cast<DstType>(static_cast<SrcType>(src(i)));
    }
  }
}
```
- **EN:** Defines `copy_if`, iterating over tensor elements or coordinates to perform the intended operation.
- **CN:** 定义 `copy_if`，通过遍历张量元素或坐标来执行目标操作。

### Lines 64-100
```cpp
//
// copy_if -- Predicated CopyAtom
//

// Predicate Tensor is an Actual Tensor
template <class... CopyArgs,
          class PrdEngine, class PrdLayout,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_if(Copy_Atom<CopyArgs...>       const& copy_atom,
        Tensor<PrdEngine, PrdLayout> const& prd,       // ([V],Rest...)
        Tensor<SrcEngine, SrcLayout> const& src,       // ( V, Rest...)
        Tensor<DstEngine, DstLayout>      & dst)       // ( V, Rest...)
{
  if constexpr (PrdLayout::rank == SrcLayout::rank - 1) {
    // Back-compat ONLY -- Delete?
    copy_if(copy_atom, make_tensor(prd.data(), prepend(prd.layout(), Layout<_1,_0>{})), src, dst);
  } else {
    static_assert(SrcLayout::rank == DstLayout::rank, "CopyAtom rank-mismatch.");
    static_assert(SrcLayout::rank == PrdLayout::rank, "CopyAtom rank-mismatch.");

    if constexpr (SrcLayout::rank == 1) {   // Dispatch the copy
      copy_atom.call(prd, src, dst);
    } else {                                // Loop over all but the first mode
      constexpr int R = SrcLayout::rank;
      Tensor prd_v = group_modes<1,R>(prd);
      Tensor src_v = group_modes<1,R>(src);
      Tensor dst_v = group_modes<1,R>(dst);
      CUTE_UNROLL
      for (int i = 0; i < size<1>(dst_v); ++i) {
        copy_atom.call(prd_v(_,i), src_v(_,i), dst_v(_,i));
      }
    }
  }
}
```
- **EN:** Defines `copy_if`, iterating over tensor elements or coordinates to perform the intended operation.
- **CN:** 定义 `copy_if`，通过遍历张量元素或坐标来执行目标操作。

### Lines 102-116
```cpp
template <class... CopyArgs,
          class PredTensor,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
[[deprecated("Use a bool-tensor or transform-tensor as predication.")]]
CUTE_HOST_DEVICE
void
copy_if(Copy_Atom<CopyArgs...>       const& copy_atom,
        PredTensor                   const& pred,      // (Rest...)
        Tensor<SrcEngine, SrcLayout> const& src,       // (V,Rest...)
        Tensor<DstEngine, DstLayout>      & dst)       // (V,Rest...)
{
  Tensor tpred = cute::lazy::transform(make_tensor(counting_iterator<int>{}, replace<0>(shape(dst), _1{})), pred);
  return copy_if(copy_atom, tpred, src, dst);
}
```
- **EN:** Defines or forwards `copy_if` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_if`，作为该头文件可执行接口的一部分。

### Lines 118-163
```cpp
//
// copy_if -- AutoCopyAsync
//

template <class PrdTensor,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_if(AutoCopyAsync                const& cpy,
        PrdTensor                    const& pred,
        Tensor<SrcEngine, SrcLayout> const& src,
        Tensor<DstEngine, DstLayout>      & dst)
{
  using SrcElemWithConst = remove_reference_t<typename SrcEngine::reference>;
  using SrcType = typename SrcEngine::value_type;
  using DstType = typename DstEngine::value_type;

  auto copy_op = []() {
#if defined(CUTE_ARCH_CP_ASYNC_SM80_ENABLED)
    if constexpr (is_gmem<SrcEngine>::value && is_smem<DstEngine>::value &&
                  sizeof(SrcType) == sizeof(DstType)) {
      if constexpr (is_const_v<SrcElemWithConst> && sizeof(SrcType) == 16) {
          return SM80_CP_ASYNC_CACHEGLOBAL<SrcType,DstType>{};
      } else if constexpr (sizeof(SrcType) == 4 || sizeof(SrcType) == 8 || sizeof(SrcType) == 16) {
          return SM80_CP_ASYNC_CACHEALWAYS<SrcType,DstType>{};
      } else {
          return UniversalCopy<SrcType,DstType>{};
      }
    } else {
        return UniversalCopy<SrcType,DstType>{};
    }

    CUTE_GCC_UNREACHABLE;
#else
    return UniversalCopy<SrcType,DstType>{};
#endif
  }();

  CUTE_UNROLL
  for (int i = 0; i < size(dst); ++i) {
    if (pred(i)) {
      copy_op.copy(src(i), dst(i));
    }
  }
}
```
- **EN:** Defines `copy_if`, iterating over tensor elements or coordinates to perform the intended operation.
- **CN:** 定义 `copy_if`，通过遍历张量元素或坐标来执行目标操作。

### Lines 165-178
```cpp
//
// copy -- AutoCopyAsync
//

template <class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(AutoCopyAsync                const& cpy,
     Tensor<SrcEngine, SrcLayout> const& src,       // (V,Rest...)
     Tensor<DstEngine, DstLayout>      & dst)       // (V,Rest...)
{
  copy_if(cpy, constant_fn<true_type>{}, src, dst);
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 180-235
```cpp
//
// copy -- CopyAtom
//

template <class... CopyArgs,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(Copy_Atom<CopyArgs...>       const& copy_atom,
     Tensor<SrcEngine, SrcLayout> const& src,       // (V,Rest...)
     Tensor<DstEngine, DstLayout>      & dst)       // (V,Rest...)
{
  static_assert(SrcLayout::rank == DstLayout::rank, "CopyAtom rank-mismatch.");

  if constexpr (SrcLayout::rank == 1) {   // Dispatch the copy
    copy_atom.call(src, dst);
  } else {                                // Loop over all but the first mode
    constexpr int R = SrcLayout::rank;
    Tensor src_v = group_modes<1,R>(src);
    Tensor dst_v = group_modes<1,R>(dst);

    if constexpr (is_static<decltype(shape(src_v))>::value && is_static<decltype(shape(dst_v))>::value) {
      CUTE_STATIC_ASSERT_V(size<1>(src_v) == size<1>(dst_v));

      // AutoFilter on the Rest-mode
      auto dst_null = nullspace(layout<1>(dst_v));

      Tensor dst_n = zipped_divide(dst_v, make_tile(shape<0>(dst_v), dst_null));  // ((V, NLL), (_1, Rest))
      Tensor src_n = zipped_divide(src_v, make_tile(shape<0>(src_v), dst_null));  // ((V, NLL), (_1, Rest))

      CUTE_STATIC_ASSERT_V(size<1>(src_n) == size<1>(dst_n));
      CUTE_STATIC_ASSERT_V((cosize<0,1>(dst_n.layout()) == Int<1>{}), "Nullspace definition error");
      CUTE_STATIC_ASSERT_V((cosize<0,1>(src_n.layout()) == Int<1>{}), "Error: Ambiguous scatter detected in copy");
      CUTE_STATIC_ASSERT_V((size<1,0>(dst_n) == Int<1>{}));
      CUTE_STATIC_ASSERT_V((size<1,0>(src_n) == Int<1>{}));

      Tensor dst_c = dst_n(make_coord(_,Int<0>{}),make_coord(Int<0>{},_));        // (V, Rest)
      Tensor src_c = src_n(make_coord(_,Int<0>{}),make_coord(Int<0>{},_));        // (V, Rest)

      CUTE_STATIC_ASSERT_V( size<1>(src_c) ==  size<1>(dst_c));
      CUTE_STATIC_ASSERT_V(shape<0>(dst_c) == shape<0>(dst));
      CUTE_STATIC_ASSERT_V(shape<0>(src_c) == shape<0>(src));

      CUTE_UNROLL
      for (int i = 0; i < size<1>(dst_c); ++i) {
        copy_atom.call(src_c(_,i), dst_c(_,i));
      }
    } else {
      CUTE_UNROLL
      for (int i = 0; i < size<1>(dst_v); ++i) {
        copy_atom.call(src_v(_,i), dst_v(_,i));
      }
    }
  }
}
```
- **EN:** Defines `copy`, iterating over tensor elements or coordinates to perform the intended operation.
- **CN:** 定义 `copy`，通过遍历张量元素或坐标来执行目标操作。

### Lines 237-274
```cpp
////////////////////////////////////////////////////////
// Special Auto-Vectorizing, Auto-Filtering Overloads //
////////////////////////////////////////////////////////

// Specialization for AutoVectorizingCopyAssumedAlignment<MaxVecBits>
template <int MaxVecBits,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(AutoVectorizingCopyWithAssumedAlignment<MaxVecBits> const&,
     Tensor<SrcEngine, SrcLayout>                        const& src,
     Tensor<DstEngine, DstLayout>                             & dst)
{
  constexpr int common_elem = CUTE_STATIC_V(max_common_vector(src, dst));
  static_assert(is_integral<decltype(Int<common_elem>{} * sizeof_bits_v<typename DstEngine::value_type>)>::value, "Error: Attempting a subbit write!");

  if constexpr (common_elem > 1)
  {
    constexpr int align_bits = CUTE_STATIC_V(gcd(max_alignment(src), max_alignment(dst), Int<MaxVecBits>{}));
    constexpr int vec_bits   = gcd(common_elem * sizeof_bits_v<typename DstEngine::value_type>, align_bits);

    if constexpr ((vec_bits % 8) == 0 && sizeof_bits_v<typename DstEngine::value_type> < Int<vec_bits>{})
    {
      // If more than one element vectorizes to a multiple of 8bits that is larger than the value_type, then recast and copy
      using VecType = uint_bit_t<vec_bits>;

      // Recast
      Tensor src_v = recast<VecType>(src);
      Tensor dst_v = recast<VecType>(dst);
      return copy_if(constant_fn<true_type>{}, src_v, dst_v);
    } else {
      return copy_if(constant_fn<true_type>{}, src, dst);
    }
  } else {
    return copy_if(constant_fn<true_type>{}, src, dst);
  }
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 276-280
```cpp
template <class Base>
struct AutoFilter {
  Base const& base;
  CUTE_HOST_DEVICE AutoFilter(Base const& b) : base(b) {}
};
```
- **EN:** Defines `AutoFilter` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `AutoFilter`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 282-305
```cpp
// Specialization for AutoFilter
template <class CopyOp,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(AutoFilter<CopyOp>           const& copy_op,
     Tensor<SrcEngine, SrcLayout> const& src,
     Tensor<DstEngine, DstLayout>      & dst)
{
  if constexpr (is_constant<true, decltype(size(src) == size(dst))>::value) {
    auto dst_null = nullspace(dst.layout());

    Tensor dst_n = zipped_divide(dst, dst_null);
    Tensor src_n = zipped_divide(src, dst_null);

    CUTE_STATIC_ASSERT_V(cosize<0>(dst_n.layout()) == Int<1>{}, "Nullspace definition error");
    CUTE_STATIC_ASSERT_V(cosize<0>(src_n.layout()) == Int<1>{}, "Error: Ambiguous race-condition detected.");

    copy(copy_op.base, src_n(Int<0>{},_), dst_n(Int<0>{},_));
  } else {
    copy(copy_op.base, src, dst);
  }
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 307-326
```cpp
// Auto-vectorizing copy for static layouts
template <class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(Tensor<SrcEngine, SrcLayout> const& src,
     Tensor<DstEngine, DstLayout>      & dst)
{
  if constexpr (is_static<SrcLayout>::value && is_static<DstLayout>::value) {
    // Assume Tensors with static layouts (e.g. registers) have pointers that are 128b aligned
    return copy(AutoFilter(AutoVectorizingCopyWithAssumedAlignment<128>{}), src, dst);
  } else
  if constexpr (is_static<decltype(shape(src))>::value && is_static<decltype(shape(dst))>::value) {
    // Tensors with static shapes can be filtered, but do not assume that dynamic layouts are aligned.
    return copy(AutoFilter(AutoVectorizingCopyWithAssumedAlignment<8>{}), src, dst);
  } else {
    // Do not assume that dynamic layouts are aligned.
    return copy(AutoVectorizingCopyWithAssumedAlignment<8>{}, src, dst);
  }
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 328-342
```cpp
// Auto-vectorizing copy with assumed alignment up to 128bit.
template <class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_aligned(Tensor<SrcEngine, SrcLayout> const& src,
             Tensor<DstEngine, DstLayout>      & dst)
{
  if constexpr (is_static<decltype(shape(src))>::value && is_static<decltype(shape(dst))>::value) {
    // Tensors with static shapes can be filtered
    return copy(AutoFilter(AutoVectorizingCopyWithAssumedAlignment<128>{}), src, dst);
  } else {
    return copy(AutoVectorizingCopyWithAssumedAlignment<128>{}, src, dst);
  }
}
```
- **EN:** Defines or forwards `copy_aligned` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_aligned`，作为该头文件可执行接口的一部分。

### Lines 344-355
```cpp
// Specializaton for Atom AutoVectorizingCopyAssumedAlignment
template <int MaxVecBits, class... Args,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(Copy_Atom<AutoVectorizingCopyWithAssumedAlignment<MaxVecBits>, Args...> const&,
     Tensor<SrcEngine, SrcLayout>                                            const& src,
     Tensor<DstEngine, DstLayout>                                                 & dst)
{
  return copy(AutoVectorizingCopyWithAssumedAlignment<MaxVecBits>{}, src, dst);
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 357-367
```cpp
template <int MaxVecBits, class... Args,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(Copy_Atom<Copy_Traits<AutoVectorizingCopyWithAssumedAlignment<MaxVecBits>>, Args...> const&,
     Tensor<SrcEngine, SrcLayout>                                                         const& src,
     Tensor<DstEngine, DstLayout>                                                              & dst)
{
  return copy(AutoVectorizingCopyWithAssumedAlignment<MaxVecBits>{}, src, dst);
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 369-388
```cpp
#if defined(CUTE_COPY_ATOM_TMA_SM90_ENABLED)
template <class... CT_Args,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(Copy_Traits<SM90_BULK_COPY_AUTO, CT_Args...> const& atom,  // Copy_Traits may or may not have the memory barrier in it already
     Tensor<SrcEngine, SrcLayout>                 const& src,
     Tensor<DstEngine, DstLayout>                      & dst)
{
  using SrcType = typename SrcEngine::value_type;
  using DstType = typename DstEngine::value_type;
  static_assert(cute::is_same<SrcType, DstType>::value);
  static_assert((is_gmem<SrcEngine>::value && is_smem<DstEngine>::value) ||
                (is_smem<SrcEngine>::value && is_gmem<DstEngine>::value),
                "Bulk Copy only supports gmem -> smem or smem -> gmem movement.");
  // G2S or S2G dispatch
  using BULK_COPY_OP = conditional_t<is_gmem<SrcEngine>::value,
                                     SM90_BULK_COPY_G2S,
                                     SM90_BULK_COPY_S2G>;
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 390-394
```cpp
  // Find the common subtensor of src and dst
  auto tiler = max_common_layout(src, dst);
  constexpr int vec_elem = decltype(size(tiler))::value;
  constexpr int vec_bits = vec_elem * sizeof_bits_v<SrcType>;
  static_assert(vec_bits >= 128, "Expected at least 128-bits for BLKCP");
```
- **EN:** Defines or forwards `max_common_layout` as part of this header's executable interface.
- **CN:** 定义或转发 `max_common_layout`，作为该头文件可执行接口的一部分。

### Lines 396-400
```cpp
  // Construct a new concrete Atom of the vector size
  using BulkAtom = Copy_Atom<Copy_Traits<BULK_COPY_OP, Int<vec_bits>, CT_Args...>, SrcType>;
  auto bulk_atom = apply(atom.opargs_, [](auto const&... args) { return BulkAtom{args...}; });
  return copy(bulk_atom, logical_divide(src, tiler), logical_divide(dst, tiler));
}
```
- **EN:** Introduces the alias `BulkAtom` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `BulkAtom`，便于复用周边的寄存器、布局或策略类型。

### Lines 402-414
```cpp
// Backwards-compat. Throw out any extra Copy_Atom args.
template <class... CT_Args, class... CA_Args,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(Copy_Atom<Copy_Traits<SM90_BULK_COPY_AUTO, CT_Args...>, CA_Args...> const& atom,
     Tensor<SrcEngine, SrcLayout>                                        const& src,
     Tensor<DstEngine, DstLayout>                                             & dst)
{
  return copy(static_cast<Copy_Traits<SM90_BULK_COPY_AUTO, CT_Args...> const&>(atom), src, dst);
}
#endif // #if defined(CUTE_COPY_ATOM_TMA_SM90_ENABLED)
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 416-432
```cpp
//
// Decay TiledCopy to CopyAtom
//

template <class CopyAtom, class TV, class Tiler,
          class PrdTensor,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_if(TiledCopy<CopyAtom, TV, Tiler> const& tiled_copy,
        PrdTensor                      const& pred,
        Tensor<SrcEngine, SrcLayout>   const& src,
        Tensor<DstEngine, DstLayout>        & dst)
{
  return copy_if(static_cast<CopyAtom const&>(tiled_copy), pred, src, dst);
}
```
- **EN:** Defines or forwards `copy_if` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_if`，作为该头文件可执行接口的一部分。

### Lines 434-444
```cpp
template <class CopyAtom, class TV, class Tiler,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(TiledCopy<CopyAtom, TV, Tiler> const& tiled_copy,
     Tensor<SrcEngine, SrcLayout>   const& src,
     Tensor<DstEngine, DstLayout>        & dst)
{
  return copy(static_cast<CopyAtom const&>(tiled_copy), src, dst);
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 446-455
```cpp
template <class TiledCopy, class ThrIdx,
          class PrdTensor,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_if(ThrCopy<TiledCopy, ThrIdx>   const& thr_copy,
        PrdTensor                    const& pred,
        Tensor<SrcEngine, SrcLayout> const& src,
        Tensor<DstEngine, DstLayout>      & dst) = delete;
```
- **EN:** Defines or forwards `copy_if` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_if`，作为该头文件可执行接口的一部分。

### Lines 457-464
```cpp
template <class TiledCopy, class ThrIdx,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(ThrCopy<TiledCopy, ThrIdx>   const& thr_copy,
     Tensor<SrcEngine, SrcLayout> const& src,
     Tensor<DstEngine, DstLayout>      & dst) = delete;
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 466-482
```cpp
//
// Catch uncaught policies
//

template <class CopyPolicy,
          class PredTensor,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_if(CopyPolicy                   const& cpy,
        PredTensor                   const& prd,
        Tensor<SrcEngine, SrcLayout> const& src,
        Tensor<DstEngine, DstLayout>      & dst)
{
  static_assert(dependent_false<CopyPolicy>, "Unrecognized CopyPolicy.");
}
```
- **EN:** Defines or forwards `copy_if` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_if`，作为该头文件可执行接口的一部分。

### Lines 484-494
```cpp
template <class CopyPolicy,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(CopyPolicy                   const& cpy,
     Tensor<SrcEngine, SrcLayout> const& src,
     Tensor<DstEngine, DstLayout>      & dst)
{
  static_assert(dependent_false<CopyPolicy>, "Unrecognized CopyPolicy.");
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 496-510
```cpp
//
// Accept mutable temporaries
//

template <class PrdTensor,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_if(PrdTensor                    const& pred,
        Tensor<SrcEngine, SrcLayout> const& src,
        Tensor<DstEngine, DstLayout>     && dst)
{
  return copy_if(pred, src, dst);
}
```
- **EN:** Defines or forwards `copy_if` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_if`，作为该头文件可执行接口的一部分。

### Lines 512-524
```cpp
template <class CopyPolicy,
          class PrdTensor,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_if(CopyPolicy                   const& copy_policy,
        PrdTensor                    const& pred,
        Tensor<SrcEngine, SrcLayout> const& src,
        Tensor<DstEngine, DstLayout>     && dst)
{
  return copy_if(copy_policy, pred, src, dst);
}
```
- **EN:** Defines or forwards `copy_if` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_if`，作为该头文件可执行接口的一部分。

### Lines 526-534
```cpp
template <class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(Tensor<SrcEngine, SrcLayout> const& src,
     Tensor<DstEngine, DstLayout>     && dst)
{
  return copy(src, dst);
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 536-546
```cpp
template <class CopyPolicy,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy(CopyPolicy                   const& copy_policy,
     Tensor<SrcEngine, SrcLayout> const& src,
     Tensor<DstEngine, DstLayout>     && dst)
{
  return copy(copy_policy, src, dst);
}
```
- **EN:** Defines or forwards `copy` as part of this header's executable interface.
- **CN:** 定义或转发 `copy`，作为该头文件可执行接口的一部分。

### Lines 548-556
```cpp
template <class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE
void
copy_aligned(Tensor<SrcEngine, SrcLayout> const& src,
             Tensor<DstEngine, DstLayout>     && dst)
{
  return copy_aligned(src, dst);
}
```
- **EN:** Defines or forwards `copy_aligned` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_aligned`，作为该头文件可执行接口的一部分。

### Lines 558-558
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Tensor abstractions organize element access, shapes, and coordinate transforms.
  **CN:** 张量抽象负责组织元素访问、形状以及坐标变换。
- **EN:** Compile-time layouts/strides describe how logical coordinates map onto storage.
  **CN:** 编译期布局/步长描述了逻辑坐标如何映射到实际存储。
- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** TMA-related code packages descriptors, barriers, swizzles, and coordinates for bulk memory movement.
  **CN:** TMA 相关代码会封装描述符、屏障、swizzle 与坐标，以支持批量内存搬运。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/tensor_impl.hpp` supplies tensor storage, indexing, and layout implementation details.
  **CN:** `cute/tensor_impl.hpp` 提供了张量存储、索引与布局实现细节。
- **EN:** `cute/atom/copy_atom.hpp` supplies copy atoms that combine traits with tiled tensor views.
  **CN:** `cute/atom/copy_atom.hpp` 提供了把 traits 与分块张量视图结合起来的 copy atom。
