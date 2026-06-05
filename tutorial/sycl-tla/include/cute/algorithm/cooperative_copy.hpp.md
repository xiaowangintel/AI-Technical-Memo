# cooperative_copy.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/algorithm/cooperative_copy.hpp`
- **EN:** Implements cooperative copy algorithms that distribute movement across threads or lanes.
- **CN:** 实现在线程或 lane 之间分摊数据搬运的协作式拷贝算法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
/***************************************************************************************************
* Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 33-40
```cpp
#include <cute/config.hpp>
#include <cute/layout.hpp>
#include <cute/layout_composed.hpp> // cute::logical_divide
#include <cute/swizzle.hpp>         // cute::Swizzle
#include <cute/swizzle_layout.hpp>  // cute::get_nonswizzle_portion
#include <cute/tensor_impl.hpp>     // cute::Tensor
#include <cute/algorithm/copy.hpp>
#include <cute/atom/copy_atom.hpp>
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/layout.hpp` (related definitions from `cute/layout.hpp`); `cute/layout_composed.hpp` (related definitions from `cute/layout_composed.hpp`); `cute/swizzle.hpp` (related definitions from `cute/swizzle.hpp`); `cute/swizzle_layout.hpp` (related definitions from `cute/swizzle_layout.hpp`); `cute/tensor_impl.hpp` (tensor storage, indexing, and layout implementation details); `cute/algorithm/copy.hpp` (related definitions from `cute/algorithm/copy.hpp`); `cute/atom/copy_atom.hpp` (copy atoms that combine traits with tiled tensor views).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/layout.hpp`（来自 `cute/layout.hpp` 的相关定义）；`cute/layout_composed.hpp`（来自 `cute/layout_composed.hpp` 的相关定义）；`cute/swizzle.hpp`（来自 `cute/swizzle.hpp` 的相关定义）；`cute/swizzle_layout.hpp`（来自 `cute/swizzle_layout.hpp` 的相关定义）；`cute/tensor_impl.hpp`（张量存储、索引与布局实现细节）；`cute/algorithm/copy.hpp`（来自 `cute/algorithm/copy.hpp` 的相关定义）；`cute/atom/copy_atom.hpp`（把 traits 与分块张量视图结合起来的 copy atom）。

### Lines 42-43
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 45-62
```cpp
template <uint32_t NumThreads,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE void
naive_cooperative_copy(uint32_t                     const& tid,
                       Tensor<SrcEngine, SrcLayout> const& src,
                       Tensor<DstEngine, DstLayout>      & dst)
{
  auto N = size(dst);
  auto R = N % Int<NumThreads>{};
  if (R > 0 && tid < R) {                                             // Likely static condition && Residue in-bounds
    dst[tid] = src[tid];
  }
  CUTE_UNROLL
  for (uint32_t i = uint32_t(R); i < uint32_t(N); i += NumThreads) {  // All in-bounds
    dst[tid + i] = src[tid + i];
  }
}
```
- **EN:** Defines `naive_cooperative_copy`, iterating over tensor elements or coordinates to perform the intended operation.
- **CN:** 定义 `naive_cooperative_copy`，通过遍历张量元素或坐标来执行目标操作。

### Lines 64-74
```cpp
// Accept mutable temporaries
template <uint32_t NumThreads,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout>
CUTE_HOST_DEVICE void
naive_cooperative_copy(uint32_t                     const& tid,
                       Tensor<SrcEngine, SrcLayout> const& src,
                       Tensor<DstEngine, DstLayout>     && dst)
{
  return naive_cooperative_copy(tid, src, dst);
}
```
- **EN:** Defines or forwards `naive_cooperative_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `naive_cooperative_copy`，作为该头文件可执行接口的一部分。

### Lines 76-104
```cpp
// A heuristic to determine a "good" permutation of two tensors for later vectorization and thr-assignment
template <class AEngine, class ALayout,
          class BEngine, class BLayout>
CUTE_HOST_DEVICE constexpr
auto
heuristic_permutation(Tensor<AEngine, ALayout> const& a,
                      Tensor<BEngine, BLayout> const& b)
{
  constexpr bool swizzleA = get_swizzle_t<AEngine>::num_bits != 0 or
                            get_swizzle_t<ALayout>::num_bits != 0;
  constexpr bool swizzleB = get_swizzle_t<BEngine>::num_bits != 0 or
                            get_swizzle_t<BLayout>::num_bits != 0;
  auto a_inv = right_inverse(get_nonswizzle_portion(a.layout()));
  auto b_inv = right_inverse(get_nonswizzle_portion(b.layout()));

  constexpr uint8_t scoreA = (uint8_t(swizzleA)                  << 2) |
                             (uint8_t(is_smem<AEngine>::value)   << 1) |
                             (uint8_t(size(a_inv) > size(b_inv)) << 0);

  constexpr uint8_t scoreB = (uint8_t(swizzleB)                  << 2) |
                             (uint8_t(is_smem<BEngine>::value)   << 1) |
                             (uint8_t(size(b_inv) > size(a_inv)) << 0);

  if constexpr (scoreA >= scoreB) {
    return a_inv;
  } else {
    return b_inv;
  }
}
```
- **EN:** Defines or forwards `heuristic_permutation` as part of this header's executable interface.
- **CN:** 定义或转发 `heuristic_permutation`，作为该头文件可执行接口的一部分。

### Lines 106-287
```cpp
// cooperative_copy<NumThreads, MaxVecBits>(thr_idx, src, dst)
// Use NumThreads to copy Tensor src to Tensor dst with element-wise vectorization up to MaxVecBits.
// @pre 0 <= @a tid < NumThreads
// @pre Tensors @a src and @a dst are aligned up to MaxVecBits.
//      That is, pointers and dynamic strides are assumed to be aligned up to MaxVecBits.
//
template <uint32_t NumThreads, uint32_t MaxVecBits,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout,
          class CopyPolicy = DefaultCopy>
CUTE_HOST_DEVICE
void
cooperative_copy(uint32_t                     const& tid,
                 Tensor<SrcEngine, SrcLayout> const& src,
                 Tensor<DstEngine, DstLayout>      & dst,
                 CopyPolicy                   const& cpy = {})
{
  // Assumes the shapes are static, can generalize/fallback
  CUTE_STATIC_ASSERT_V(is_static<decltype(shape(src))>{} && is_static<decltype(shape(dst))>{});
  CUTE_STATIC_ASSERT_V(size(src) == size(dst));
  // Assumes the types are the same, can generalize/fallback
  static_assert(cute::is_same<typename SrcEngine::value_type, typename DstEngine::value_type>::value);
  static_assert(MaxVecBits == sizeof_bits_v<typename SrcEngine::value_type> ||
                MaxVecBits == 8 || MaxVecBits == 16 || MaxVecBits == 32 || MaxVecBits == 64 || MaxVecBits == 128,
                "Expected MaxVecBits to be value size or 8 or 16 or 32 or 64 or 128 for alignment and performance.");
  // Check that the tensors are likely shared across threads: either gmem or smem
  static_assert((is_gmem<SrcEngine>::value || is_smem<SrcEngine>::value),
                "cooperative_copy expects shared gmem or smem source tensor.");
  static_assert((is_gmem<DstEngine>::value || is_smem<DstEngine>::value),
                "cooperative_copy expects shared gmem or smem destination tensor.");
  // Precondition on tid in DEBUG
  assert(tid < NumThreads);
  // Precondition on pointer alignment in DEBUG
  assert(is_byte_aligned<ceil_div(MaxVecBits,8u)>(raw_pointer_cast(src.data())));
  assert(is_byte_aligned<ceil_div(MaxVecBits,8u)>(raw_pointer_cast(dst.data())));

#if 0
      if (thread0()) {
        print("   "); print("cooperative_copy\n");
        print("   "); print("NumThreads: "); print(NumThreads); print("\n");
        print("   "); print("MaxVecBits: "); print(MaxVecBits); print("\n");
        print("   "); print("src: "); print(src); print("\n");
        print("   "); print("dst: "); print(dst); print("\n");
      }
#ifdef __CUDA_ARCH__
      __syncthreads();
#endif
#endif

  // The common layout of the two tensors that can be vectorized over elements and threads
  // vidx -> coord
  auto common_layout = heuristic_permutation(src, dst);

  // Apply
  // (V, rest)
  Tensor src_a = coalesce(logical_divide(src, common_layout), Shape<_1,_1>{});
  Tensor dst_a = coalesce(logical_divide(dst, common_layout), Shape<_1,_1>{});

  //
  // Determine vectorization of elems and thrs based on src/dst size and number of threads
  // NOTE: This heuristic promotes parallelization over vectorization
  //

  // The number of elements and number of bits
  constexpr int  elem_bits = sizeof_bits_v<typename SrcEngine::value_type>;
  constexpr int total_elem = size(SrcLayout{});

  // The number of elements that can be vectorized in values
  constexpr int common_elem = decltype(max_common_vector(src_a, dst_a))::value;

#if 0
      if (thread0()) {
        print("   "); print("common_layout: "); print(common_layout); print("\n");
        print("   "); print("src_a: "); print(src_a); print("\n");
        print("   "); print("dst_a: "); print(dst_a); print("\n");
      }
#ifdef __CUDA_ARCH__
      __syncthreads();
#endif
#endif

  //
  if constexpr (total_elem % NumThreads != 0) {
    // Not attempting to find a partitioning pattern, fallback to dynamically indexed slowpath

    if constexpr (common_elem > 1 && MaxVecBits > elem_bits) {
      // If the vectorization is non-trivial and divides the maximum vectorizations, then vectorize
      constexpr auto max_align_src = elem_bits * decltype(max_alignment(src_a.layout()))::value;
      constexpr auto max_align_dst = elem_bits * decltype(max_alignment(dst_a.layout()))::value;
      constexpr auto vec_bits = gcd(max_align_src, max_align_dst, MaxVecBits);
      using VecType = uint_bit_t<vec_bits>;

      static_assert(vec_bits % elem_bits == 0, "Expected divisibility");
      static_assert((vec_bits >= 8), "No support for subbyte copying");

      Tensor src_v = recast<VecType const>(src_a);
      Tensor dst_v = recast<VecType      >(dst_a);

#if 0
      if (thread0()) {
        print("   "); print("cooperative_copy -- naive\n");
        print("   "); print("src_v: "); print(src_v); print("\n");
        print("   "); print("dst_v: "); print(dst_v); print("\n");
      }
#ifdef __CUDA_ARCH__
      __syncthreads();
#endif
#endif

      naive_cooperative_copy<NumThreads>(tid, src_v, dst_v);
    } else {
      naive_cooperative_copy<NumThreads>(tid, src_a, dst_a);
    }
  } else {
    // If the tensors can be equally partitioned by the threads,
    // compute vectorization widths in elements and threads.

    // If there are too many threads to allow a full vectorized copy, trunc the vectorization
    constexpr int total_bits = total_elem * elem_bits;
    constexpr int max_bits_per_thr = total_bits / NumThreads;
    // At least elem_bits, at most common_bits
    constexpr int common_bits = common_elem * elem_bits;
    constexpr int vec_bits = cute::max(elem_bits, cute::gcd(common_bits, int(MaxVecBits), max_bits_per_thr));

    // Should account for vec_bits < 8 and/or vec_elem <= 1
    // And also account for subbyte types, which could cause race conditions
    // Want to ENFORCE sufficient vectorization in those cases
    static_assert(vec_bits % elem_bits == 0, "Expected divisibility");
    static_assert(vec_bits >= 8, "No support for subbyte copying");

    using VecType = uint_bit_t<vec_bits>;
    constexpr int vec_elem = vec_bits / elem_bits;

    constexpr int vec_thrs = cute::min(int(NumThreads), total_elem / vec_elem);

    //
    // Determine the partitioning patterns for the vec_elems and vec_thrs
    //

    // Distribute the rest of the V*T to some consistent portion outside of the common_layout, if needed
    auto common_domain_src = domain_distribute(shape(src_a), Int<vec_elem*vec_thrs>{});
    auto common_domain_dst = domain_distribute(shape(dst_a), Int<vec_elem*vec_thrs>{});

    // Make sure for now, could fall back here instead
    CUTE_STATIC_ASSERT_V(size(common_domain_src) == Int<vec_elem*vec_thrs>{});
    CUTE_STATIC_ASSERT_V(compatible(common_domain_src, common_domain_dst) ||
                         compatible(common_domain_dst, common_domain_src));
    // Use the "more specific" domain for the extra elements of V*T
    auto common_domain = conditional_return(compatible(common_domain_src, common_domain_dst),
                                            common_domain_dst, common_domain_src);

    // Construct the tiler
    auto tiler_vt = common_domain.with_shape(Int<vec_elem>{}, Int<vec_thrs>{});

    // Apply and slice
    Tensor src_v = logical_divide(src_a, tiler_vt)(make_coord(_,tid),_);
    Tensor dst_v = logical_divide(dst_a, tiler_vt)(make_coord(_,tid),_);

#if 0
      if (thread0()) {
        print("   "); print("cooperative_copy -- vec\n");
        print("   "); print("Used vector: ");  print(vec_elem); print("\n");
        print("   "); print("Used threads: ");  print(vec_thrs); print("\n");
        print("   "); print("tiler_vt: "); print(tiler_vt); print("\n");
        print("   "); print("src_v: "); print(src_v); print("\n");
        print("   "); print("dst_v: "); print(dst_v); print("\n");
        print("   "); print("recast<VecType const>(src_v): "); print(recast<VecType const>(src_v)); print("\n");
        print("   "); print("recast<VecType      >(dst_v): "); print(recast<VecType      >(dst_v)); print("\n");
      }
#ifdef __CUDA_ARCH__
      __syncthreads();
#endif
#endif

    // If we're using all threads (static) or the tid is in-range (dynamic)
    if (vec_thrs == NumThreads or tid < vec_thrs) {
      auto src_c = recast<VecType const>(src_v);
      auto dst_c = recast<VecType>(dst_v);
      return copy(cpy, src_c, dst_c);
    }
  }
}
```
- **EN:** Defines or forwards `cooperative_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `cooperative_copy`，作为该头文件可执行接口的一部分。

### Lines 290-304
```cpp
// Default max-vectorization size to value_type size
template <uint32_t NumThreads,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout,
          class CopyPolicy = DefaultCopy>
CUTE_HOST_DEVICE
void
cooperative_copy(uint32_t                     const& tid,
                 Tensor<SrcEngine, SrcLayout> const& src,
                 Tensor<DstEngine, DstLayout>      & dst,
                 CopyPolicy                   const& cpy = {})
{
  constexpr uint32_t MaxVecBits = sizeof_bits_v<typename SrcEngine::value_type>;
  return cooperative_copy<NumThreads, MaxVecBits>(tid, src, dst, cpy);
}
```
- **EN:** Defines or forwards `cooperative_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `cooperative_copy`，作为该头文件可执行接口的一部分。

### Lines 306-322
```cpp
//
// Accept mutable temporaries
//

template <uint32_t NumThreads,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout,
          class CopyPolicy = DefaultCopy>
CUTE_HOST_DEVICE
void
cooperative_copy(uint32_t                     const& tid,
                 Tensor<SrcEngine, SrcLayout> const& src,
                 Tensor<DstEngine, DstLayout>     && dst,
                 CopyPolicy                   const& cpy = {})
{
  return cooperative_copy<NumThreads>(tid, src, dst, cpy);
}
```
- **EN:** Defines or forwards `cooperative_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `cooperative_copy`，作为该头文件可执行接口的一部分。

### Lines 324-336
```cpp
template <uint32_t NumThreads, uint32_t MaxVecBits,
          class SrcEngine, class SrcLayout,
          class DstEngine, class DstLayout,
          class CopyPolicy = DefaultCopy>
CUTE_HOST_DEVICE
void
cooperative_copy(uint32_t                     const& tid,
                 Tensor<SrcEngine, SrcLayout> const& src,
                 Tensor<DstEngine, DstLayout>     && dst,
                 CopyPolicy                   const& cpy = {})
{
  return cooperative_copy<NumThreads, MaxVecBits>(tid, src, dst, cpy);
}
```
- **EN:** Defines or forwards `cooperative_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `cooperative_copy`，作为该头文件可执行接口的一部分。

### Lines 338-338
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Tensor abstractions organize element access, shapes, and coordinate transforms.
  **CN:** 张量抽象负责组织元素访问、形状以及坐标变换。
- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/layout.hpp` supplies related definitions from `cute/layout.hpp`.
  **CN:** `cute/layout.hpp` 提供了来自 `cute/layout.hpp` 的相关定义。
- **EN:** `cute/layout_composed.hpp` supplies related definitions from `cute/layout_composed.hpp`.
  **CN:** `cute/layout_composed.hpp` 提供了来自 `cute/layout_composed.hpp` 的相关定义。
- **EN:** `cute/swizzle.hpp` supplies related definitions from `cute/swizzle.hpp`.
  **CN:** `cute/swizzle.hpp` 提供了来自 `cute/swizzle.hpp` 的相关定义。
- **EN:** `cute/swizzle_layout.hpp` supplies related definitions from `cute/swizzle_layout.hpp`.
  **CN:** `cute/swizzle_layout.hpp` 提供了来自 `cute/swizzle_layout.hpp` 的相关定义。
- **EN:** `cute/tensor_impl.hpp` supplies tensor storage, indexing, and layout implementation details.
  **CN:** `cute/tensor_impl.hpp` 提供了张量存储、索引与布局实现细节。
- **EN:** `cute/algorithm/copy.hpp` supplies related definitions from `cute/algorithm/copy.hpp`.
  **CN:** `cute/algorithm/copy.hpp` 提供了来自 `cute/algorithm/copy.hpp` 的相关定义。
- **EN:** `cute/atom/copy_atom.hpp` supplies copy atoms that combine traits with tiled tensor views.
  **CN:** `cute/atom/copy_atom.hpp` 提供了把 traits 与分块张量视图结合起来的 copy atom。
- **EN:** CUDA architecture macros gate device-only fast paths and inline instructions.
  **CN:** CUDA 架构宏用于控制仅设备端可用的快速路径与内联指令。
