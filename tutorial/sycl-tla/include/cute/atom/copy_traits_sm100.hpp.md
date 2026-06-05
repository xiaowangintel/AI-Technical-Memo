# copy_traits_sm100.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/atom/copy_traits_sm100.hpp`
- **EN:** Defines copy-trait specializations that map logical copy layouts onto NVIDIA SM100 instructions.
- **CN:** 定义 copy trait 特化，把逻辑拷贝布局映射到 NVIDIA SM100 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
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

### Lines 34-35
```cpp
#include <cute/arch/copy_sm100.hpp>
#include <cute/arch/tmem_allocator_sm100.hpp>
```
- **EN:** Imports `cute/arch/copy_sm100.hpp` (related definitions from `cute/arch/copy_sm100.hpp`); `cute/arch/tmem_allocator_sm100.hpp` (related definitions from `cute/arch/tmem_allocator_sm100.hpp`).
- **CN:** 引入 `cute/arch/copy_sm100.hpp`（来自 `cute/arch/copy_sm100.hpp` 的相关定义）；`cute/arch/tmem_allocator_sm100.hpp`（来自 `cute/arch/tmem_allocator_sm100.hpp` 的相关定义）。

### Lines 37-40
```cpp
#include <cute/atom/copy_traits.hpp>
#include <cute/atom/copy_atom.hpp>
#include <cute/atom/partitioner.hpp>
#include <cute/numeric/numeric_types.hpp>
```
- **EN:** Imports `cute/atom/copy_traits.hpp` (generic copy-trait interfaces and utilities); `cute/atom/copy_atom.hpp` (copy atoms that combine traits with tiled tensor views); `cute/atom/partitioner.hpp` (related definitions from `cute/atom/partitioner.hpp`); `cute/numeric/numeric_types.hpp` (numeric helper types such as fixed-width bit wrappers).
- **CN:** 引入 `cute/atom/copy_traits.hpp`（通用 copy trait 接口与工具）；`cute/atom/copy_atom.hpp`（把 traits 与分块张量视图结合起来的 copy atom）；`cute/atom/partitioner.hpp`（来自 `cute/atom/partitioner.hpp` 的相关定义）；`cute/numeric/numeric_types.hpp`（数值辅助类型，例如固定位宽包装类型）。

### Lines 42-42
```cpp
#include <cute/layout.hpp>
```
- **EN:** Imports `cute/layout.hpp` (related definitions from `cute/layout.hpp`).
- **CN:** 引入 `cute/layout.hpp`（来自 `cute/layout.hpp` 的相关定义）。

### Lines 44-50
```cpp
namespace cute
{
template <>
struct Copy_Traits<SM100_LOAD_256bit_CACHE_NOALLOCATION>
{
  // Logical thread id to thread idx (one-thread)
  using ThrID = Layout<_1>;
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 52-55
```cpp
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_1,_256>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_1,_256>>;
```
- **EN:** Introduces the alias `SrcLayout` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SrcLayout`，便于复用周边的寄存器、布局或策略类型。

### Lines 57-59
```cpp
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Introduces the alias `RefLayout` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `RefLayout`，便于复用周边的寄存器、布局或策略类型。

### Lines 61-74
```cpp
template <>
struct Copy_Traits<SM100_STORE_256bit_CACHE_NOALLOCATION>
{
  // Logical thread id to thread idx (one-thread)
  using ThrID = Layout<_1>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_1,_256>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_1,_256>>;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 76-91
```cpp
template <>
struct Copy_Traits<SM100_U8x8_LDSM_T>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape <  _2,   _2,  _4,_2>,_128>,
                           Stride<Stride<_128,_1024,_256,_0>,  _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <Shape <  _4,_8>,Shape <_8,  _2, _2,   _2>>,
                           Stride<Stride<_256,_8>,Stride<_1,_128,_64,_1024>>>;

  // Reference map from (thr,val) to bit
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 93-108
```cpp
template <>
struct Copy_Traits<SM100_U8x16_LDSM_T>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape <  _2,   _2,  _4,   _2>,_128>,
                           Stride<Stride<_128,_1024,_256,_2048>,  _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <Shape <  _4,_8>,Shape <_8,  _2, _2,   _4>>,
                           Stride<Stride<_256,_8>,Stride<_1,_128,_64,_1024>>>;

  // Reference map from (thr,val) to bit
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 110-125
```cpp
template <>
struct Copy_Traits<SM100_SU4_DU8x16_x1_LDSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape <  _8,_4>,_128>,
                           Stride<Stride<_128,_0>,  _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <_32,_32>,
                           Stride<_32, _1>>;

  // Reference map from (thr,val) to bit
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 127-142
```cpp
template <>
struct Copy_Traits<SM100_SU6_DU8x16_x1_LDSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape <  _8,_4>,_128>,
                           Stride<Stride<_128,_0>,  _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <_32,_32>,
                           Stride<_32, _1>>;

  // Reference map from (thr,val) to bit
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 144-159
```cpp
template <>
struct Copy_Traits<SM100_SU4_DU8x16_x2_LDSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape < _16,_2>,_128>,
                           Stride<Stride<_128,_0>,  _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <_32,Shape <_32,   _2>>,
                           Stride<_32,Stride< _1,_1024>>>;

  // Reference map from (thr,val) to bit
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 161-176
```cpp
template <>
struct Copy_Traits<SM100_SU6_DU8x16_x2_LDSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape < _16,_2>,_128>,
                           Stride<Stride<_128,_0>,  _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <_32,Shape <_32,   _2>>,
                           Stride<_32,Stride< _1,_1024>>>;

  // Reference map from (thr,val) to bit
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 178-192
```cpp
template <>
struct Copy_Traits<SM100_SU4_DU8x16_x4_LDSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape < _32,_128>,
                           Stride<_128,  _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <_32,Shape <_32,   _4>>,
                           Stride<_32,Stride< _1,_1024>>>;
  // Reference map from (thr,val) to bit
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 194-208
```cpp
template <>
struct Copy_Traits<SM100_SU6_DU8x16_x4_LDSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape < _32,_128>,
                           Stride<_128,  _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <_32,Shape <_32,   _4>>,
                           Stride<_32,Stride< _1,_1024>>>;
  // Reference map from (thr,val) to bit
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 210-225
```cpp
template <>
struct Copy_Traits<SM100_U8x4_STSM_T>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape <  _4,_8>,Shape <_8,  _2, _2>>,
                           Stride<Stride<_256,_8>,Stride<_1,_128,_64>>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <Shape <  _8,_4>,_128>,
                           Stride<Stride<_128,_0>,  _1>>;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 227-242
```cpp
template <>
struct Copy_Traits<SM100_U8x8_STSM_T>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape <  _4,_8>,Shape <_8,  _2, _2,   _2>>,
                           Stride<Stride<_256,_8>,Stride<_1,_128,_64,_1024>>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <Shape < _16,_2>,_128>,
                           Stride<Stride<_128,_0>,  _1>>;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 244-259
```cpp
template <>
struct Copy_Traits<SM100_U8x16_STSM_T>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <Shape <  _4,_8>,Shape <_8,  _2, _2,   _4>>,
                           Stride<Stride<_256,_8>,Stride<_1,_128,_64,_1024>>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape < _32,_128>,
                           Stride<_128,  _1>>;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 261-268
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
//
// TMEM Traits and Utilities
//
////////////////////////////////////////////////////////////////////////////////////////////////////

template <class... Args>
struct Copy_Atom;
```
- **EN:** Defines `Copy_Atom`, a tag or wrapper that describes how data movement should be performed on this backend.
- **CN:** 定义 `Copy_Atom`，它是一个标签或封装，用于描述该后端上的数据搬运方式。

### Lines 270-303
```cpp
/** Generate a TiledCopy from a CopyAtom and a TMEM tensor
 * Example:
 *   Tensor gmem_tensor = ...                                            // (M,N,...)
 *   Tensor tmem_tensor = ...                                            // (M,N,...)
 *   auto tiled_tmem_load = make_tmem_copy(TMEM_LOAD_Operation, tmem_tensor);
 *   auto thr_tmem_load = tiled_tmem_load.get_slice(thread_idx);
 *
 *   Tensor tDtC = thr_tmem_load.partition_S(tmem_tensor);                    // (TMEM_LOAD,TMEM_LOAD_M,TMEM_LOAD_N,...)
 *   Tensor tDgC = thr_tmem_load.partition_D(gmem_tensor);                    // (TMEM_LOAD,TMEM_LOAD_M,TMEM_LOAD_N,...)
 *   Tensor tDrC = make_tensor<ElementAccumulator>(shape(tDgD));         // (TMEM_LOAD,TMEM_LOAD_M,TMEM_LOAD_N,...)
 *
 *   copy(tiled_tmem_load, tDtC, tDrC);       // tmem -> rmem
 *   copy(tDrC, tDgC);                   // rmem -> gmem
 */
template <class CopyOp, class CopyT,
          class TEngine, class TLayout>
CUTE_HOST_DEVICE constexpr
auto
make_tmem_copy(Copy_Atom<CopyOp,CopyT> const& atom,
               Tensor<TEngine,TLayout> const& tmem)
{
  static_assert(is_tmem<TEngine>::value, "Expected TMEM tensor.");
  using T      = typename TEngine::value_type;
  using Traits = typename Copy_Atom<CopyOp, CopyT>::Traits;
  static_assert(sizeof_bits_v<CopyT> == sizeof_bits_v<T>,
                "Expected a CopyAtom with the same type-width as the Tensor.");

  // atom thr idx -> tmem addr    4warps where each warp points to the same position within it's own subpartition
  auto atom_t_layout = Layout<Shape<_32,_4>, Stride<_0, decltype(Int<32>{} * TMEM::DP<T>{})>>{};
  // atom val idx -> tmem addr    Cast the CopyOp's value ids to the proper data width
  auto atom_v_layout = coalesce(upcast<sizeof_bits<T>::value>(typename Traits::ValID{}));

  return make_cotiled_copy(atom, make_layout(atom_t_layout, atom_v_layout), tmem.layout());
}
```
- **EN:** Defines or forwards `make_tmem_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tmem_copy`，作为该头文件可执行接口的一部分。

### Lines 305-313
```cpp
template <class CopyOp,
          class TEngine, class TLayout>
CUTE_HOST_DEVICE constexpr
auto
make_tmem_copy(CopyOp const&,
               Tensor<TEngine,TLayout> const& tmem)
{
  return make_tmem_copy(Copy_Atom<CopyOp, typename TEngine::value_type>{}, tmem);
}
```
- **EN:** Defines or forwards `make_tmem_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tmem_copy`，作为该头文件可执行接口的一部分。

### Lines 315-366
```cpp
/** Generate a TV_Tiler from a TMEM tensor
 * Example:
 *   Tensor gmem_tensor = ...                                            // (M,N,...)
 *   Tensor tmem_tensor = ...                                            // (M,N,...)
 *   auto tmem_tiler = make_tmem_warp_partitioner(tmem_tensor);
 *   auto warp_tiler  = tmem_tiler.get_slice(warp_idx);
 *
 *   Tensor tWtC = warp_tiler.partition(tmem_tensor);                    // (WARP_M,WARP_N,...)
 *   Tensor tWgC = warp_tiler.partition(gmem_tensor);                    // (WARP_M,WARP_N,...)
 */
template <class TEngine, class TLayout>
CUTE_HOST_DEVICE constexpr
auto
make_tmem_warp_partitioner(Tensor<TEngine,TLayout> const& tmem)
{
  static_assert(is_tmem<TEngine>::value, "Expected TMEM tensor.");
  using T = typename TEngine::value_type;

  // warp idx -> tmem addr    This is the T in the Layout_TV
  auto atom_t_layout = Layout<_4, decltype(Int<32>{} * TMEM::DP<T>{})>{};

  // tmem coord -> tmem addr
  auto tmem_layout = tmem.layout();
  // tmem addr -> tmem coord    Append 1:0 so off-the-ends get the stride-0
  auto inv_tmem_layout = make_layout(left_inverse(tmem_layout), Layout<_1,_0>{});

  // wid -> tmem_coord
  auto layout_t_tmem = composition(inv_tmem_layout, atom_t_layout);
  //
  // Tiler -- Find the active elements in the TMEM tensor and generate a tiler to extract them
  //

  // Convert to the awkward by-mode tiler to preserve the modes of the tiled TMEM
  auto flat_tmem_shape = product_each(shape(tmem_layout));
  auto flat_tmem_zeros = repeat<rank(flat_tmem_shape)>(Int<0>{});

  auto tiler = transform(make_seq<rank(flat_tmem_shape)>{}, [&](auto i) {
    return filter(composition(make_layout(flat_tmem_shape, replace<i>(flat_tmem_zeros, Int<1>{})), layout_t_tmem));
  });

  //
  // Layout_TV -- Find the (tid,vid) -> tile coord transformation
  //

  // Apply the tiler to a reference and transform the codomain
  // tile_coord -> tmem_coord
  auto tile2tmem = composition(make_layout(flat_tmem_shape), tiler);

  // wid -> tile_coord
  auto layout_tv = composition(left_inverse(tile2tmem), layout_t_tmem);
  return make_tiler_impl(layout_tv, tiler);
}
```
- **EN:** Defines or forwards `make_tmem_warp_partitioner` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tmem_warp_partitioner`，作为该头文件可执行接口的一部分。

### Lines 368-368
```cpp
namespace SM100::TMEM::LOAD {
```
- **EN:** Enters or leaves namespace scope `SM100::TMEM::LOAD` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM100::TMEM::LOAD`，以便把相关符号组织在一起。

### Lines 370-408
```cpp
//
// Specialized copy_unpack implementation for SM100::TMEM::LOAD instructions
//

template <class CopyOp,
          class TS, class SLayout,
          class TD, class DLayout>
CUTE_HOST_DEVICE constexpr
void
copy_unpack(Copy_Traits<CopyOp> const& traits,
            Tensor<TS,SLayout>  const& src,
            Tensor<TD,DLayout>       & dst)
{
  static_assert(is_tmem<TS>::value, "Expected TMEM src.");
  static_assert(is_rmem<TD>::value, "Expected RMEM dst.");

  using SrcType = typename TS::value_type;
  CUTE_STATIC_ASSERT_V((coalesce(layout(src)) == coalesce(upcast<sizeof_bits<SrcType>::value>(typename Copy_Traits<CopyOp>::ValID{}))),
    "Expected src to have the specific TMEM layout required by CopyOp.");

  uint32_t tmem_addr = raw_pointer_cast(src.data());

  using RegTypeDst = typename remove_extent<typename CopyOp::DRegisters>::type;
  Tensor rD = recast<RegTypeDst>(dst);

  constexpr int RegNumDst = extent<typename CopyOp::DRegisters>::value;
  CUTE_STATIC_ASSERT_V(size(rD) == Int<RegNumDst>{},
    "In CopyAtom, dst layout doesn't vectorize into registers. This dst layout is incompatible with this CopyOp.");

  // thread idx <=> DP lane assert.
  // ASSERT TMEM_LOAD thread attemping to access DP lane within sub-partition.
#if defined(__CUDA_ARCH__) && !defined(NDEBUG)
  assert(((uint32_t(threadIdx.x) / 32) % 4) == (((tmem_addr >> 16) / 32) % 4));
#endif

  detail::explode(CopyOp::copy,
                  &tmem_addr, seq<0>{},
                  rD, make_seq<RegNumDst>{});
}
```
- **EN:** Defines `copy_unpack`, which reshapes tensor operands into the register view expected by the selected copy operation and dispatches it.
- **CN:** 定义 `copy_unpack`：把张量操作数重塑为所选拷贝操作期望的寄存器视图，然后完成派发。

### Lines 410-410
```cpp
} // end namespace SM100::TMEM::LOAD
```
- **EN:** Enters or leaves namespace scope `SM100::TMEM::LOAD` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM100::TMEM::LOAD`，以便把相关符号组织在一起。

### Lines 412-412
```cpp
namespace SM100::TMEM::STORE {
```
- **EN:** Enters or leaves namespace scope `SM100::TMEM::STORE` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM100::TMEM::STORE`，以便把相关符号组织在一起。

### Lines 414-452
```cpp
//
// Specialized copy_unpack implementation for SM100::TMEM::STORE instructions
//

template <class CopyOp,
          class TS, class SLayout,
          class TD, class DLayout>
CUTE_HOST_DEVICE constexpr
void
copy_unpack(Copy_Traits<CopyOp> const& traits,
            Tensor<TS,SLayout>  const& src,
            Tensor<TD,DLayout>       & dst)
{
  static_assert(is_rmem<TS>::value, "Expected RMEM src.");
  static_assert(is_tmem<TD>::value, "Expected TMEM dst.");

  using RegTypeSrc = typename remove_extent<typename CopyOp::SRegisters>::type;
  Tensor rS = recast<RegTypeSrc>(src);

  constexpr int RegNumSrc = extent<typename CopyOp::SRegisters>::value;
  CUTE_STATIC_ASSERT_V(size(rS) == Int<RegNumSrc>{},
    "In CopyAtom, src layout doesn't vectorize into registers. This src layout is incompatible with this tiled copy.");

  using DstType = typename TD::value_type;
  CUTE_STATIC_ASSERT_V((coalesce(layout(dst)) == coalesce(upcast<sizeof_bits<DstType>::value>(typename Copy_Traits<CopyOp>::ValID{}))),
    "Expected dst to have the specific TMEM layout required by CopyOp.");

  uint32_t tmem_addr = raw_pointer_cast(dst.data());

  // thread idx <=> DP lane assert.
  // ASSERT TMEM_LOAD thread attemping to access DP lane within sub-partition.
#if defined(__CUDA_ARCH__) && !defined(NDEBUG)
  assert(((uint32_t(threadIdx.x) / 32) % 4) == (((tmem_addr >> 16) / 32) % 4));
#endif

  detail::explode(CopyOp::copy,
                  rS, make_seq<RegNumSrc>{},
                  &tmem_addr, seq<0>{});
}
```
- **EN:** Defines `copy_unpack`, which reshapes tensor operands into the register view expected by the selected copy operation and dispatches it.
- **CN:** 定义 `copy_unpack`：把张量操作数重塑为所选拷贝操作期望的寄存器视图，然后完成派发。

### Lines 454-454
```cpp
} // end namespace SM100::TMEM::STORE
```
- **EN:** Enters or leaves namespace scope `SM100::TMEM::STORE` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM100::TMEM::STORE`，以便把相关符号组织在一起。

### Lines 456-462
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
//
// TMEM_LOAD Copy Traits
//
////////////////////////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Section comment introducing the next logical part of the file: TMEM_LOAD Copy Traits
- **CN:** 分节注释，用来引出文件中的下一段逻辑：TMEM_LOAD Copy Traits

### Lines 464-464
```cpp
using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 466-482
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b1x>
{
  // Logical thread id to thread idx (warp)
  using ThrID = Layout<_32>;
  // Logical bit id to bit idx (address)
  using ValID = Layout<Shape <_256,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_64,   _2>>,
                           Stride<Stride<_64,_256>,Stride< _1,_2048>>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 484-486
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 488-499
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b1x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_16>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_64,   _2>>,
                           Stride<Stride<_64,_256>,Stride< _1,_2048>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 501-503
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 505-516
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b2x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_512,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_64,   _2,  _2>>,
                           Stride<Stride<_64,_512>,Stride< _1,_4096,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 518-520
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 522-533
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b2x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_32>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_64,   _2,  _2>>,
                           Stride<Stride<_64,_512>,Stride< _1,_4096,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 535-537
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 539-550
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b4x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_1024,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_64,   _2,  _4>>,
                           Stride<Stride<_64,_1024>,Stride< _1,_8192,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 552-554
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 556-567
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b4x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_64>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_64,   _2,  _4>>,
                           Stride<Stride<_64,_1024>,Stride< _1,_8192,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 569-571
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 573-584
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b8x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_2048,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_64,    _2,  _8>>,
                           Stride<Stride<_64,_2048>,Stride< _1,_16384,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 586-588
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 590-601
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b8x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_128>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_64,    _2,  _8>>,
                           Stride<Stride<_64,_2048>,Stride< _1,_16384,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 603-605
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 607-618
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b16x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_4096,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_64,    _2, _16>>,
                           Stride<Stride<_64,_4096>,Stride< _1,_32768,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 620-622
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 624-635
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b16x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_256>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_64,    _2, _16>>,
                           Stride<Stride<_64,_4096>,Stride< _1,_32768,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 637-639
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 641-652
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b32x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_8192,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_64,    _2, _32>>,
                           Stride<Stride<_64,_8192>,Stride< _1,_65536,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 654-656
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp256b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 658-669
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp256b32x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_512>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_64,    _2, _32>>,
                           Stride<Stride<_64,_8192>,Stride< _1,_65536,_256>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 671-673
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 675-686
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b1x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_128,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_2048>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_32,   _2>>,
                           Stride<Stride<_32,_128>,Stride< _1,_1024>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 688-690
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 692-703
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b1x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16, _8>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_2048>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_32,   _2>>,
                           Stride<Stride<_32,_128>,Stride< _1,_1024>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 705-707
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 709-720
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b2x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_256,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_32,   _2,  _2>>,
                           Stride<Stride<_32,_256>,Stride< _1,_2048,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 722-724
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 726-737
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b2x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_16>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_32,   _2,  _2>>,
                           Stride<Stride<_32,_256>,Stride< _1,_2048,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 739-741
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 743-754
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b4x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_512,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_32,   _2,  _4>>,
                           Stride<Stride<_32,_512>,Stride< _1,_4096,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 756-758
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 760-771
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b4x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_32>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _4,  _8>,Shape <_32,   _2,  _4>>,
                           Stride<Stride<_32,_512>,Stride< _1,_4096,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 773-775
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 777-788
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b8x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_1024,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_32,   _2,  _8>>,
                           Stride<Stride<_32,_1024>,Stride< _1,_8192,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 790-792
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 794-805
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b8x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_64>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_32,   _2,  _8>>,
                           Stride<Stride<_32,_1024>,Stride< _1,_8192,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 807-809
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 811-822
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b16x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_2048,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_32,    _2, _16>>,
                           Stride<Stride<_32,_2048>,Stride< _1,_16384,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 824-826
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 828-839
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b16x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_128>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_32,    _2, _16>>,
                           Stride<Stride<_32,_2048>,Stride< _1,_16384,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 841-843
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 845-856
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b32x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_4096,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_32,    _2, _32>>,
                           Stride<Stride<_32,_4096>,Stride< _1,_32768,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 858-860
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 862-873
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b32x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_256>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_32,    _2, _32>>,
                           Stride<Stride<_32,_4096>,Stride< _1,_32768,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 875-877
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b64x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 879-890
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b64x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_8192,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_32,    _2, _64>>,
                           Stride<Stride<_32,_8192>,Stride< _1,_65536,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 892-894
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp128b64x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 896-907
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp128b64x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_512>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <Shape < _4,   _8>,Shape <_32,    _2, _64>>,
                           Stride<Stride<_32,_8192>,Stride< _1,_65536,_128>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 909-911
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 913-924
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b1x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_64,       _16>,
                       Stride< _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_1024>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <  _2, _2, _8>,_32>,
                           Stride<Stride<_512,_32,_64>, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 926-928
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 930-941
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b1x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16, _4>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_1024>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <  _2, _2, _8>,_32>,
                           Stride<Stride<_512,_32,_64>, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 943-945
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 947-958
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b2x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_128,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_2048>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <   _2, _2,  _8>,Shape <_32, _2>>,
                           Stride<Stride<_1024,_32,_128>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 960-962
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 964-975
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b2x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16, _8>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_2048>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <   _2, _2,  _8>,Shape <_32, _2>>,
                           Stride<Stride<_1024,_32,_128>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 977-979
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 981-992
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b4x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_256,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <   _2, _2,  _8>,Shape <_32, _4>>,
                           Stride<Stride<_2048,_32,_256>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 994-996
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 998-1009
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b4x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_16>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <   _2, _2,  _8>,Shape <_32, _4>>,
                           Stride<Stride<_2048,_32,_256>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1011-1013
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1015-1026
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b8x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_512,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <   _2, _2,  _8>,Shape <_32, _8>>,
                           Stride<Stride<_4096,_32,_512>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1028-1030
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1032-1043
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b8x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_32>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <   _2, _2,  _8>,Shape <_32, _8>>,
                           Stride<Stride<_4096,_32,_512>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1045-1047
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1049-1060
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b16x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_1024,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <   _2, _2,   _8>,Shape <_32,_16>>,
                           Stride<Stride<_8192,_32,_1024>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1062-1064
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1066-1077
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b16x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_64>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <   _2, _2,   _8>,Shape <_32,_16>>,
                           Stride<Stride<_8192,_32,_1024>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1079-1081
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1083-1094
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b32x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_2048,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <    _2, _2,   _8>,Shape <_32,_32>>,
                           Stride<Stride<_16384,_32,_2048>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1096-1098
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1100-1111
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b32x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_128>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <    _2, _2,   _8>,Shape <_32,_32>>,
                           Stride<Stride<_16384,_32,_2048>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1113-1115
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b64x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1117-1128
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b64x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_4096,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <    _2, _2,   _8>,Shape <_32,_64>>,
                           Stride<Stride<_32768,_32,_4096>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1130-1132
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b64x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1134-1145
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b64x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_256>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <    _2, _2,   _8>,Shape <_32,_64>>,
                           Stride<Stride<_32768,_32,_4096>,Stride< _1,_64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1147-1149
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b128x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1151-1162
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b128x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_8192,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <Shape <    _2, _2,   _8>,Shape <_32,_128>>,
                           Stride<Stride<_65536,_32,_8192>,Stride< _1, _64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1164-1166
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp64b128x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1168-1179
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp64b128x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_512>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <Shape <    _2, _2,   _8>,Shape <_32,_128>>,
                           Stride<Stride<_65536,_32,_8192>,Stride< _1, _64>>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1181-1183
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1185-1196
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b1x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_64,       _16>,
                       Stride< _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_1024>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <_16, _2>,_32>,
                           Stride<Stride<_64,_32>, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1198-1200
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1202-1213
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b1x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16, _4>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_1024>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape <_16, _2>,_32>,
                           Stride<Stride<_64,_32>, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1215-1217
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1219-1230
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b2x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_128,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_2048>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _16, _2>,_64>,
                           Stride<Stride<_128,_64>, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1232-1234
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1236-1247
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b2x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16, _8>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_2048>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _16, _2>,_64>,
                           Stride<Stride<_128,_64>, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1249-1251
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1253-1264
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b4x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_256,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _16,  _2>,_128>,
                           Stride<Stride<_256,_128>,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1266-1268
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1270-1281
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b4x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_16>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _16,  _2>,_128>,
                           Stride<Stride<_256,_128>,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1283-1285
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1287-1298
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b8x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_512,       _16>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _16,  _2>,_256>,
                           Stride<Stride<_512,_256>,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1300-1302
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1304-1315
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b8x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_32>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <Shape < _16,  _2>,_256>,
                           Stride<Stride<_512,_256>,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1317-1319
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1321-1332
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b16x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_1024,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <  _16,  _2>,_512>,
                           Stride<Stride<_1024,_512>,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1334-1336
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1338-1349
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b16x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_64>,       _16>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <  _16,  _2>,_512>,
                           Stride<Stride<_1024,_512>,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1351-1353
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1355-1366
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b32x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_2048,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <  _16,   _2>,_1024>,
                           Stride<Stride<_2048,_1024>,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1368-1370
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1372-1383
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b32x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_128>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <  _16,   _2>,_1024>,
                           Stride<Stride<_2048,_1024>,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1385-1387
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b64x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1389-1400
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b64x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_4096,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <  _16,   _2>,_2048>,
                           Stride<Stride<_4096,_2048>,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1402-1404
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b64x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1406-1417
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b64x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_256>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <Shape <  _16,   _2>,_2048>,
                           Stride<Stride<_4096,_2048>,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1419-1421
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b128x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1423-1434
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b128x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_8192,       _16>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <Shape <  _16,   _2>,_4096>,
                           Stride<Stride<_8192,_4096>,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1436-1438
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_16dp32b128x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1440-1451
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_16dp32b128x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_512>,       _16>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <Shape <  _16,   _2>,_4096>,
                           Stride<Stride<_8192,_4096>,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1453-1455
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1457-1468
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b1x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_32,       _32>,
                       Stride< _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_1024>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <_32,_32>,
                           Stride<_32, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1470-1472
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1474-1485
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b1x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16, _2>,       _32>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_1024>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <_32,_32>,
                           Stride<_32, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1487-1489
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1491-1502
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b2x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_64,       _32>,
                       Stride< _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_2048>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <_32,_64>,
                           Stride<_64, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1504-1506
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1508-1519
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b2x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16, _4>,       _32>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_2048>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape <_32,_64>,
                           Stride<_64, _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1521-1523
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1525-1536
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b4x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_128,       _32>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape < _32,_128>,
                           Stride<_128,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1538-1540
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1542-1553
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b4x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16, _8>,       _32>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_4096>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape < _32,_128>,
                           Stride<_128,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1555-1557
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1559-1570
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b8x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_256,       _32>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape < _32,_256>,
                           Stride<_256,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1572-1574
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1576-1587
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b8x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_16>,       _32>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_8192>,
                           Stride< _0,   _1>>;
  using DstLayout = Layout<Shape < _32,_256>,
                           Stride<_256,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1589-1591
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1593-1604
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b16x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_512,       _32>,
                       Stride<  _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape < _32,_512>,
                           Stride<_512,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1606-1608
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1610-1621
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b16x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_32>,       _32>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_16384>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape < _32,_512>,
                           Stride<_512,  _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1623-1625
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1627-1638
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b32x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_1024,       _32>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <  _32,_1024>,
                           Stride<_1024,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1640-1642
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1644-1655
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b32x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_64>,       _32>,
                       Stride<Stride< _1,_32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_32768>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <  _32,_1024>,
                           Stride<_1024,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1657-1659
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b64x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1661-1672
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b64x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_2048,       _32>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <  _32,_2048>,
                           Stride<_2048,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1674-1676
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b64x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1678-1689
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b64x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_128>,       _32>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_65536>,
                           Stride< _0,    _1>>;
  using DstLayout = Layout<Shape <  _32,_2048>,
                           Stride<_2048,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1691-1693
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b128x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1695-1706
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b128x>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <_4096,       _32>,
                       Stride<   _1,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <  _32,_4096>,
                           Stride<_4096,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1708-1710
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::LOAD::SM100_TMEM_LOAD_32dp32b128x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1712-1723
```cpp
template <>
struct Copy_Traits<SM100_TMEM_LOAD_32dp32b128x_16b>
{
  using ThrID = Layout<_32>;
  using ValID = Layout<Shape <Shape <_16,_256>,       _32>,
                       Stride<Stride< _1, _32>,TMEM::DP_b>>;
  using SrcLayout = Layout<Shape <_32,_131072>,
                           Stride< _0,     _1>>;
  using DstLayout = Layout<Shape <  _32,_4096>,
                           Stride<_4096,   _1>>;
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1725-1731
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////////////////////////
//
// TMEM_STORE Copy Traits
//
////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Section comment introducing the next logical part of the file: TMEM_STORE Copy Traits
- **CN:** 分节注释，用来引出文件中的下一段逻辑：TMEM_STORE Copy Traits

### Lines 1733-1735
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1737-1745
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b1x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1747-1749
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1751-1759
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b1x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b1x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1761-1763
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1765-1773
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b2x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1775-1777
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1779-1787
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b2x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b2x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1789-1791
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1793-1801
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b4x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1803-1805
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1807-1815
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b4x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b4x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1817-1819
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1821-1829
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b8x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1831-1833
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1835-1843
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b8x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b8x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1845-1847
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1849-1857
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b16x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1859-1861
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1863-1871
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b16x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b16x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1873-1875
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1877-1885
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b32x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1887-1889
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp256b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1891-1899
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp256b32x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp256b32x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1901-1903
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1905-1913
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b1x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1915-1917
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1919-1927
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b1x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b1x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1929-1931
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1933-1941
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b2x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1943-1945
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1947-1955
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b2x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b2x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1957-1959
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1961-1969
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b4x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1971-1973
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1975-1983
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b4x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b4x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1985-1987
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 1989-1997
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b8x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 1999-2001
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2003-2011
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b8x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b8x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2013-2015
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2017-2025
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b16x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2027-2029
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2031-2039
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b16x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b16x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2041-2043
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2045-2053
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b32x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2055-2057
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2059-2067
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b32x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b32x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2069-2071
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b64x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2073-2081
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b64x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2083-2085
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp128b64x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2087-2095
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp128b64x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp128b64x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2097-2099
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2101-2109
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b1x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2111-2113
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2115-2123
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b1x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b1x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2125-2127
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2129-2137
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b2x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2139-2141
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2143-2151
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b2x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b2x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2153-2155
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2157-2165
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b4x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2167-2169
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2171-2179
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b4x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b4x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2181-2183
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2185-2193
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b8x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2195-2197
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2199-2207
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b8x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b8x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2209-2211
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2213-2221
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b16x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2223-2225
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2227-2235
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b16x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b16x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2237-2239
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2241-2249
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b32x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2251-2253
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2255-2263
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b32x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b32x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2265-2267
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b64x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2269-2277
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b64x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2279-2281
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b64x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2283-2291
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b64x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b64x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2293-2295
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b128x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2297-2305
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b128x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2307-2309
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp64b128x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2311-2319
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp64b128x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp64b128x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2321-2323
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2325-2333
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b1x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2335-2337
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2339-2347
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b1x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b1x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2349-2351
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2353-2361
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b2x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2363-2365
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2367-2375
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b2x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b2x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2377-2379
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2381-2389
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b4x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2391-2393
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2395-2403
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b4x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b4x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2405-2407
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2409-2417
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b8x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2419-2421
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2423-2431
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b8x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b8x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2433-2435
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2437-2445
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b16x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2447-2449
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2451-2459
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b16x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b16x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2461-2463
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2465-2473
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b32x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2475-2477
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2479-2487
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b32x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b32x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2489-2491
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b64x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2493-2501
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b64x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2503-2505
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b64x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2507-2515
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b64x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b64x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2517-2519
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b128x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2521-2529
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b128x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2531-2533
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_16dp32b128x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2535-2543
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_16dp32b128x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_16dp32b128x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2545-2547
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b1x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2549-2557
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b1x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2559-2561
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b1x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2563-2571
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b1x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b1x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2573-2575
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b2x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2577-2585
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b2x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2587-2589
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b2x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2591-2599
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b2x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b2x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2601-2603
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b4x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2605-2613
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b4x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2615-2617
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b4x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2619-2627
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b4x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b4x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2629-2631
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b8x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2633-2641
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b8x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2643-2645
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b8x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2647-2655
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b8x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b8x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2657-2659
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b16x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2661-2669
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b16x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2671-2673
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b16x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2675-2683
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b16x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b16x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2685-2687
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b32x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2689-2697
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b32x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2699-2701
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b32x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2703-2711
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b32x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b32x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2713-2715
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b64x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2717-2725
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b64x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2727-2729
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b64x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2731-2739
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b64x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b64x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2741-2743
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b128x;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2745-2753
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b128x>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2755-2757
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::STORE::SM100_TMEM_STORE_32dp32b128x_16b;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 2759-2767
```cpp
template <>
struct Copy_Traits<SM100_TMEM_STORE_32dp32b128x_16b>
{
  using ThrID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x_16b>::ThrID;
  using ValID = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x_16b>::ValID;
  using SrcLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x_16b>::DstLayout;
  using DstLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x_16b>::SrcLayout;
  using RefLayout = typename Copy_Traits<SM100_TMEM_LOAD_32dp32b128x_16b>::RefLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 2769-2771
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Section comment introducing the next logical part of the file: 
- **CN:** 分节注释，用来引出文件中的下一段逻辑：

### Lines 2773-2773
```cpp
namespace TMEM {
```
- **EN:** Enters or leaves namespace scope `TMEM` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `TMEM`，以便把相关符号组织在一起。

### Lines 2775-3270
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// Given a 1x tmem copy op, returns the widest repeated variant that divides the specified bits in the N-mode
template <class CopyOp, int bits_n>
CUTE_HOST_DEVICE constexpr
auto
op_repeater()
{
  if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b1x>) {
    if constexpr (bits_n % (256 * 32) == 0) {
      return SM100_TMEM_LOAD_16dp256b32x{};
    }
    else if constexpr (bits_n % (256 * 16) == 0) {
      return SM100_TMEM_LOAD_16dp256b16x{};
    }
    else if constexpr (bits_n % (256 *  8) == 0) {
      return SM100_TMEM_LOAD_16dp256b8x{};
    }
    else if constexpr (bits_n % (256 *  4) == 0) {
      return SM100_TMEM_LOAD_16dp256b4x{};
    }
    else if constexpr (bits_n % (256 *  2) == 0) {
      return SM100_TMEM_LOAD_16dp256b2x{};
    }
    else if constexpr (bits_n % (256 *  1) == 0) {
      return SM100_TMEM_LOAD_16dp256b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b1x_16b>) {
    if constexpr (bits_n % (256 * 32) == 0) {
      return SM100_TMEM_LOAD_16dp256b32x_16b{};
    }
    else if constexpr (bits_n % (256 * 16) == 0) {
      return SM100_TMEM_LOAD_16dp256b16x_16b{};
    }
    else if constexpr (bits_n % (256 *  8) == 0) {
      return SM100_TMEM_LOAD_16dp256b8x_16b{};
    }
    else if constexpr (bits_n % (256 *  4) == 0) {
      return SM100_TMEM_LOAD_16dp256b4x_16b{};
    }
    else if constexpr (bits_n % (256 *  2) == 0) {
      return SM100_TMEM_LOAD_16dp256b2x_16b{};
    }
    else if constexpr (bits_n % (256 *  1) == 0) {
      return SM100_TMEM_LOAD_16dp256b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b1x>) {
    if constexpr (bits_n % (128 * 64) == 0) {
      return SM100_TMEM_LOAD_16dp128b64x{};
    }
    else if constexpr (bits_n % (128 * 32) == 0) {
      return SM100_TMEM_LOAD_16dp128b32x{};
    }
    else if constexpr (bits_n % (128 * 16) == 0) {
      return SM100_TMEM_LOAD_16dp128b16x{};
    }
    else if constexpr (bits_n % (128 *  8) == 0) {
      return SM100_TMEM_LOAD_16dp128b8x{};
    }
    else if constexpr (bits_n % (128 *  4) == 0) {
      return SM100_TMEM_LOAD_16dp128b4x{};
    }
    else if constexpr (bits_n % (128 *  2) == 0) {
      return SM100_TMEM_LOAD_16dp128b2x{};
    }
    else if constexpr (bits_n % (128 *  1) == 0) {
      return SM100_TMEM_LOAD_16dp128b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b1x_16b>) {
    if constexpr (bits_n % (128 * 64) == 0) {
      return SM100_TMEM_LOAD_16dp128b64x_16b{};
    }
    else if constexpr (bits_n % (128 * 32) == 0) {
      return SM100_TMEM_LOAD_16dp128b32x_16b{};
    }
    else if constexpr (bits_n % (128 * 16) == 0) {
      return SM100_TMEM_LOAD_16dp128b16x_16b{};
    }
    else if constexpr (bits_n % (128 *  8) == 0) {
      return SM100_TMEM_LOAD_16dp128b8x_16b{};
    }
    else if constexpr (bits_n % (128 *  4) == 0) {
      return SM100_TMEM_LOAD_16dp128b4x_16b{};
    }
    else if constexpr (bits_n % (128 *  2) == 0) {
      return SM100_TMEM_LOAD_16dp128b2x_16b{};
    }
    else if constexpr (bits_n % (128 *  1) == 0) {
      return SM100_TMEM_LOAD_16dp128b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b1x>) {
    if constexpr (bits_n % (64 * 128) == 0) {
      return SM100_TMEM_LOAD_16dp64b128x{};
    }
    else if constexpr (bits_n % (64 * 64) == 0) {
      return SM100_TMEM_LOAD_16dp64b64x{};
    }
    else if constexpr (bits_n % (64 * 32) == 0) {
      return SM100_TMEM_LOAD_16dp64b32x{};
    }
    else if constexpr (bits_n % (64 * 16) == 0) {
      return SM100_TMEM_LOAD_16dp64b16x{};
    }
    else if constexpr (bits_n % (64 *  8) == 0) {
      return SM100_TMEM_LOAD_16dp64b8x{};
    }
    else if constexpr (bits_n % (64 *  4) == 0) {
      return SM100_TMEM_LOAD_16dp64b4x{};
    }
    else if constexpr (bits_n % (64 *  2) == 0) {
      return SM100_TMEM_LOAD_16dp64b2x{};
    }
    else if constexpr (bits_n % (64 *  1) == 0) {
      return SM100_TMEM_LOAD_16dp64b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b1x_16b>) {
    if constexpr (bits_n % (64 * 128) == 0) {
      return SM100_TMEM_LOAD_16dp64b128x_16b{};
    }
    else if constexpr (bits_n % (64 * 64) == 0) {
      return SM100_TMEM_LOAD_16dp64b64x_16b{};
    }
    else if constexpr (bits_n % (64 * 32) == 0) {
      return SM100_TMEM_LOAD_16dp64b32x_16b{};
    }
    else if constexpr (bits_n % (64 * 16) == 0) {
      return SM100_TMEM_LOAD_16dp64b16x_16b{};
    }
    else if constexpr (bits_n % (64 *  8) == 0) {
      return SM100_TMEM_LOAD_16dp64b8x_16b{};
    }
    else if constexpr (bits_n % (64 *  4) == 0) {
      return SM100_TMEM_LOAD_16dp64b4x_16b{};
    }
    else if constexpr (bits_n % (64 *  2) == 0) {
      return SM100_TMEM_LOAD_16dp64b2x_16b{};
    }
    else if constexpr (bits_n % (64 *  1) == 0) {
      return SM100_TMEM_LOAD_16dp64b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b1x>) {
    if constexpr (bits_n % (64 * 128) == 0) {
      return SM100_TMEM_LOAD_16dp32b128x{};
    }
    else if constexpr (bits_n % (64 * 64) == 0) {
      return SM100_TMEM_LOAD_16dp32b64x{};
    }
    else if constexpr (bits_n % (64 * 32) == 0) {
      return SM100_TMEM_LOAD_16dp32b32x{};
    }
    else if constexpr (bits_n % (64 * 16) == 0) {
      return SM100_TMEM_LOAD_16dp32b16x{};
    }
    else if constexpr (bits_n % (64 *  8) == 0) {
      return SM100_TMEM_LOAD_16dp32b8x{};
    }
    else if constexpr (bits_n % (64 *  4) == 0) {
      return SM100_TMEM_LOAD_16dp32b4x{};
    }
    else if constexpr (bits_n % (64 *  2) == 0) {
      return SM100_TMEM_LOAD_16dp32b2x{};
    }
    else if constexpr (bits_n % (64 *  1) == 0) {
      return SM100_TMEM_LOAD_16dp32b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b1x_16b>) {
    if constexpr (bits_n % (64 * 128) == 0) {
      return SM100_TMEM_LOAD_16dp32b128x_16b{};
    }
    else if constexpr (bits_n % (64 * 64) == 0) {
      return SM100_TMEM_LOAD_16dp32b64x_16b{};
    }
    else if constexpr (bits_n % (64 * 32) == 0) {
      return SM100_TMEM_LOAD_16dp32b32x_16b{};
    }
    else if constexpr (bits_n % (64 * 16) == 0) {
      return SM100_TMEM_LOAD_16dp32b16x_16b{};
    }
    else if constexpr (bits_n % (64 *  8) == 0) {
      return SM100_TMEM_LOAD_16dp32b8x_16b{};
    }
    else if constexpr (bits_n % (64 *  4) == 0) {
      return SM100_TMEM_LOAD_16dp32b4x_16b{};
    }
    else if constexpr (bits_n % (64 *  2) == 0) {
      return SM100_TMEM_LOAD_16dp32b2x_16b{};
    }
    else if constexpr (bits_n % (64 *  1) == 0) {
      return SM100_TMEM_LOAD_16dp32b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b1x>) {
    if constexpr (bits_n % (32 * 128) == 0) {
      return SM100_TMEM_LOAD_32dp32b128x{};
    }
    else if constexpr (bits_n % (32 * 64) == 0) {
      return SM100_TMEM_LOAD_32dp32b64x{};
    }
    else if constexpr (bits_n % (32 * 32) == 0) {
      return SM100_TMEM_LOAD_32dp32b32x{};
    }
    else if constexpr (bits_n % (32 * 16) == 0) {
      return SM100_TMEM_LOAD_32dp32b16x{};
    }
    else if constexpr (bits_n % (32 *  8) == 0) {
      return SM100_TMEM_LOAD_32dp32b8x{};
    }
    else if constexpr (bits_n % (32 *  4) == 0) {
      return SM100_TMEM_LOAD_32dp32b4x{};
    }
    else if constexpr (bits_n % (32 *  2) == 0) {
      return SM100_TMEM_LOAD_32dp32b2x{};
    }
    else if constexpr (bits_n % (32 *  1) == 0) {
      return SM100_TMEM_LOAD_32dp32b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b1x_16b>) {
    if constexpr (bits_n % (32 * 128) == 0) {
      return SM100_TMEM_LOAD_32dp32b128x_16b{};
    }
    else if constexpr (bits_n % (32 * 64) == 0) {
      return SM100_TMEM_LOAD_32dp32b64x_16b{};
    }
    else if constexpr (bits_n % (32 * 32) == 0) {
      return SM100_TMEM_LOAD_32dp32b32x_16b{};
    }
    else if constexpr (bits_n % (32 * 16) == 0) {
      return SM100_TMEM_LOAD_32dp32b16x_16b{};
    }
    else if constexpr (bits_n % (32 *  8) == 0) {
      return SM100_TMEM_LOAD_32dp32b8x_16b{};
    }
    else if constexpr (bits_n % (32 *  4) == 0) {
      return SM100_TMEM_LOAD_32dp32b4x_16b{};
    }
    else if constexpr (bits_n % (32 *  2) == 0) {
      return SM100_TMEM_LOAD_32dp32b2x_16b{};
    }
    else if constexpr (bits_n % (32 *  1) == 0) {
      return SM100_TMEM_LOAD_32dp32b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_16dp256b1x>) {
    if constexpr (bits_n % (256 * 32) == 0) {
      return SM100_TMEM_STORE_16dp256b32x{};
    }
    else if constexpr (bits_n % (256 * 16) == 0) {
      return SM100_TMEM_STORE_16dp256b16x{};
    }
    else if constexpr (bits_n % (256 *  8) == 0) {
      return SM100_TMEM_STORE_16dp256b8x{};
    }
    else if constexpr (bits_n % (256 *  4) == 0) {
      return SM100_TMEM_STORE_16dp256b4x{};
    }
    else if constexpr (bits_n % (256 *  2) == 0) {
      return SM100_TMEM_STORE_16dp256b2x{};
    }
    else if constexpr (bits_n % (256 *  1) == 0) {
      return SM100_TMEM_STORE_16dp256b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_16dp256b1x_16b>) {
    if constexpr (bits_n % (256 * 32) == 0) {
      return SM100_TMEM_STORE_16dp256b32x_16b{};
    }
    else if constexpr (bits_n % (256 * 16) == 0) {
      return SM100_TMEM_STORE_16dp256b16x_16b{};
    }
    else if constexpr (bits_n % (256 *  8) == 0) {
      return SM100_TMEM_STORE_16dp256b8x_16b{};
    }
    else if constexpr (bits_n % (256 *  4) == 0) {
      return SM100_TMEM_STORE_16dp256b4x_16b{};
    }
    else if constexpr (bits_n % (256 *  2) == 0) {
      return SM100_TMEM_STORE_16dp256b2x_16b{};
    }
    else if constexpr (bits_n % (256 *  1) == 0) {
      return SM100_TMEM_STORE_16dp256b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_16dp128b1x>) {
    if constexpr (bits_n % (128 * 64) == 0) {
      return SM100_TMEM_STORE_16dp128b64x{};
    }
    else if constexpr (bits_n % (128 * 32) == 0) {
      return SM100_TMEM_STORE_16dp128b32x{};
    }
    else if constexpr (bits_n % (128 * 16) == 0) {
      return SM100_TMEM_STORE_16dp128b16x{};
    }
    else if constexpr (bits_n % (128 *  8) == 0) {
      return SM100_TMEM_STORE_16dp128b8x{};
    }
    else if constexpr (bits_n % (128 *  4) == 0) {
      return SM100_TMEM_STORE_16dp128b4x{};
    }
    else if constexpr (bits_n % (128 *  2) == 0) {
      return SM100_TMEM_STORE_16dp128b2x{};
    }
    else if constexpr (bits_n % (128 *  1) == 0) {
      return SM100_TMEM_STORE_16dp128b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_16dp128b1x_16b>) {
    if constexpr (bits_n % (128 * 64) == 0) {
      return SM100_TMEM_STORE_16dp128b64x_16b{};
    }
    else if constexpr (bits_n % (128 * 32) == 0) {
      return SM100_TMEM_STORE_16dp128b32x_16b{};
    }
    else if constexpr (bits_n % (128 * 16) == 0) {
      return SM100_TMEM_STORE_16dp128b16x_16b{};
    }
    else if constexpr (bits_n % (128 *  8) == 0) {
      return SM100_TMEM_STORE_16dp128b8x_16b{};
    }
    else if constexpr (bits_n % (128 *  4) == 0) {
      return SM100_TMEM_STORE_16dp128b4x_16b{};
    }
    else if constexpr (bits_n % (128 *  2) == 0) {
      return SM100_TMEM_STORE_16dp128b2x_16b{};
    }
    else if constexpr (bits_n % (128 *  1) == 0) {
      return SM100_TMEM_STORE_16dp128b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_16dp64b1x>) {
    if constexpr (bits_n % (64 * 128) == 0) {
      return SM100_TMEM_STORE_16dp64b128x{};
    }
    else if constexpr (bits_n % (64 * 64) == 0) {
      return SM100_TMEM_STORE_16dp64b64x{};
    }
    else if constexpr (bits_n % (64 * 32) == 0) {
      return SM100_TMEM_STORE_16dp64b32x{};
    }
    else if constexpr (bits_n % (64 * 16) == 0) {
      return SM100_TMEM_STORE_16dp64b16x{};
    }
    else if constexpr (bits_n % (64 *  8) == 0) {
      return SM100_TMEM_STORE_16dp64b8x{};
    }
    else if constexpr (bits_n % (64 *  4) == 0) {
      return SM100_TMEM_STORE_16dp64b4x{};
    }
    else if constexpr (bits_n % (64 *  2) == 0) {
      return SM100_TMEM_STORE_16dp64b2x{};
    }
    else if constexpr (bits_n % (64 *  1) == 0) {
      return SM100_TMEM_STORE_16dp64b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_16dp64b1x_16b>) {
    if constexpr (bits_n % (64 * 128) == 0) {
      return SM100_TMEM_STORE_16dp64b128x_16b{};
    }
    else if constexpr (bits_n % (64 * 64) == 0) {
      return SM100_TMEM_STORE_16dp64b64x_16b{};
    }
    else if constexpr (bits_n % (64 * 32) == 0) {
      return SM100_TMEM_STORE_16dp64b32x_16b{};
    }
    else if constexpr (bits_n % (64 * 16) == 0) {
      return SM100_TMEM_STORE_16dp64b16x_16b{};
    }
    else if constexpr (bits_n % (64 *  8) == 0) {
      return SM100_TMEM_STORE_16dp64b8x_16b{};
    }
    else if constexpr (bits_n % (64 *  4) == 0) {
      return SM100_TMEM_STORE_16dp64b4x_16b{};
    }
    else if constexpr (bits_n % (64 *  2) == 0) {
      return SM100_TMEM_STORE_16dp64b2x_16b{};
    }
    else if constexpr (bits_n % (64 *  1) == 0) {
      return SM100_TMEM_STORE_16dp64b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_16dp32b1x>) {
    if constexpr (bits_n % (64 * 128) == 0) {
      return SM100_TMEM_STORE_16dp32b128x{};
    }
    else if constexpr (bits_n % (64 * 64) == 0) {
      return SM100_TMEM_STORE_16dp32b64x{};
    }
    else if constexpr (bits_n % (64 * 32) == 0) {
      return SM100_TMEM_STORE_16dp32b32x{};
    }
    else if constexpr (bits_n % (64 * 16) == 0) {
      return SM100_TMEM_STORE_16dp32b16x{};
    }
    else if constexpr (bits_n % (64 *  8) == 0) {
      return SM100_TMEM_STORE_16dp32b8x{};
    }
    else if constexpr (bits_n % (64 *  4) == 0) {
      return SM100_TMEM_STORE_16dp32b4x{};
    }
    else if constexpr (bits_n % (64 *  2) == 0) {
      return SM100_TMEM_STORE_16dp32b2x{};
    }
    else if constexpr (bits_n % (64 *  1) == 0) {
      return SM100_TMEM_STORE_16dp32b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_16dp32b1x_16b>) {
    if constexpr (bits_n % (64 * 128) == 0) {
      return SM100_TMEM_STORE_16dp32b128x_16b{};
    }
    else if constexpr (bits_n % (64 * 64) == 0) {
      return SM100_TMEM_STORE_16dp32b64x_16b{};
    }
    else if constexpr (bits_n % (64 * 32) == 0) {
      return SM100_TMEM_STORE_16dp32b32x_16b{};
    }
    else if constexpr (bits_n % (64 * 16) == 0) {
      return SM100_TMEM_STORE_16dp32b16x_16b{};
    }
    else if constexpr (bits_n % (64 *  8) == 0) {
      return SM100_TMEM_STORE_16dp32b8x_16b{};
    }
    else if constexpr (bits_n % (64 *  4) == 0) {
      return SM100_TMEM_STORE_16dp32b4x_16b{};
    }
    else if constexpr (bits_n % (64 *  2) == 0) {
      return SM100_TMEM_STORE_16dp32b2x_16b{};
    }
    else if constexpr (bits_n % (64 *  1) == 0) {
      return SM100_TMEM_STORE_16dp32b1x_16b{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_32dp32b1x>) {
    if constexpr (bits_n % (32 * 128) == 0) {
      return SM100_TMEM_STORE_32dp32b128x{};
    }
    else if constexpr (bits_n % (32 * 64) == 0) {
      return SM100_TMEM_STORE_32dp32b64x{};
    }
    else if constexpr (bits_n % (32 * 32) == 0) {
      return SM100_TMEM_STORE_32dp32b32x{};
    }
    else if constexpr (bits_n % (32 * 16) == 0) {
      return SM100_TMEM_STORE_32dp32b16x{};
    }
    else if constexpr (bits_n % (32 *  8) == 0) {
      return SM100_TMEM_STORE_32dp32b8x{};
    }
    else if constexpr (bits_n % (32 *  4) == 0) {
      return SM100_TMEM_STORE_32dp32b4x{};
    }
    else if constexpr (bits_n % (32 *  2) == 0) {
      return SM100_TMEM_STORE_32dp32b2x{};
    }
    else if constexpr (bits_n % (32 *  1) == 0) {
      return SM100_TMEM_STORE_32dp32b1x{};
    }
  }
  else if constexpr (cute::is_same_v<CopyOp, SM100_TMEM_STORE_32dp32b1x_16b>) {
    if constexpr (bits_n % (32 * 128) == 0) {
      return SM100_TMEM_STORE_32dp32b128x_16b{};
    }
    else if constexpr (bits_n % (32 * 64) == 0) {
      return SM100_TMEM_STORE_32dp32b64x_16b{};
    }
    else if constexpr (bits_n % (32 * 32) == 0) {
      return SM100_TMEM_STORE_32dp32b32x_16b{};
    }
    else if constexpr (bits_n % (32 * 16) == 0) {
      return SM100_TMEM_STORE_32dp32b16x_16b{};
    }
    else if constexpr (bits_n % (32 *  8) == 0) {
      return SM100_TMEM_STORE_32dp32b8x_16b{};
    }
    else if constexpr (bits_n % (32 *  4) == 0) {
      return SM100_TMEM_STORE_32dp32b4x_16b{};
    }
    else if constexpr (bits_n % (32 *  2) == 0) {
      return SM100_TMEM_STORE_32dp32b2x_16b{};
    }
    else if constexpr (bits_n % (32 *  1) == 0) {
      return SM100_TMEM_STORE_32dp32b1x_16b{};
    }
  }
  else {
    static_assert(dependent_false<CopyOp>, "Must pass 1x tmem copy operator");
  }
}
```
- **EN:** Defines or forwards `op_repeater` as part of this header's executable interface.
- **CN:** 定义或转发 `op_repeater`，作为该头文件可执行接口的一部分。

### Lines 3272-3503
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// Select TMEM store corresponding to the provided TMEM load
template <class CopyOp>
CUTE_HOST_DEVICE constexpr auto
tmem_load_to_store(CopyOp) {
  if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b1x>) {
    return SM100_TMEM_STORE_16dp256b1x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b1x_16b>) {
    return SM100_TMEM_STORE_16dp256b1x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b2x>) {
    return SM100_TMEM_STORE_16dp256b2x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b2x_16b>) {
    return SM100_TMEM_STORE_16dp256b2x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b4x>) {
    return SM100_TMEM_STORE_16dp256b4x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b4x_16b>) {
    return SM100_TMEM_STORE_16dp256b4x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b8x>) {
    return SM100_TMEM_STORE_16dp256b8x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b8x_16b>) {
    return SM100_TMEM_STORE_16dp256b8x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b16x>) {
    return SM100_TMEM_STORE_16dp256b16x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b16x_16b>) {
    return SM100_TMEM_STORE_16dp256b16x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b32x>) {
    return SM100_TMEM_STORE_16dp256b32x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp256b32x_16b>) {
    return SM100_TMEM_STORE_16dp256b32x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b1x>) {
    return SM100_TMEM_STORE_16dp128b1x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b1x_16b>) {
    return SM100_TMEM_STORE_16dp128b1x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b2x>) {
    return SM100_TMEM_STORE_16dp128b2x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b2x_16b>) {
    return SM100_TMEM_STORE_16dp128b2x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b4x>) {
    return SM100_TMEM_STORE_16dp128b4x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b4x_16b>) {
    return SM100_TMEM_STORE_16dp128b4x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b8x>) {
    return SM100_TMEM_STORE_16dp128b8x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b8x_16b>) {
    return SM100_TMEM_STORE_16dp128b8x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b16x>) {
    return SM100_TMEM_STORE_16dp128b16x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b16x_16b>) {
    return SM100_TMEM_STORE_16dp128b16x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b32x>) {
    return SM100_TMEM_STORE_16dp128b32x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b32x_16b>) {
    return SM100_TMEM_STORE_16dp128b32x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b64x>) {
    return SM100_TMEM_STORE_16dp128b64x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp128b64x_16b>) {
    return SM100_TMEM_STORE_16dp128b64x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b1x>) {
    return SM100_TMEM_STORE_16dp64b1x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b1x_16b>) {
    return SM100_TMEM_STORE_16dp64b1x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b2x>) {
    return SM100_TMEM_STORE_16dp64b2x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b2x_16b>) {
    return SM100_TMEM_STORE_16dp64b2x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b4x>) {
    return SM100_TMEM_STORE_16dp64b4x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b4x_16b>) {
    return SM100_TMEM_STORE_16dp64b4x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b8x>) {
    return SM100_TMEM_STORE_16dp64b8x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b8x_16b>) {
    return SM100_TMEM_STORE_16dp64b8x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b16x>) {
    return SM100_TMEM_STORE_16dp64b16x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b16x_16b>) {
    return SM100_TMEM_STORE_16dp64b16x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b32x>) {
    return SM100_TMEM_STORE_16dp64b32x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b32x_16b>) {
    return SM100_TMEM_STORE_16dp64b32x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b64x>) {
    return SM100_TMEM_STORE_16dp64b64x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b64x_16b>) {
    return SM100_TMEM_STORE_16dp64b64x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b128x>) {
    return SM100_TMEM_STORE_16dp64b128x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp64b128x_16b>) {
    return SM100_TMEM_STORE_16dp64b128x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b1x>) {
    return SM100_TMEM_STORE_16dp32b1x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b1x_16b>) {
    return SM100_TMEM_STORE_16dp32b1x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b2x>) {
    return SM100_TMEM_STORE_16dp32b2x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b2x_16b>) {
    return SM100_TMEM_STORE_16dp32b2x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b4x>) {
    return SM100_TMEM_STORE_16dp32b4x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b4x_16b>) {
    return SM100_TMEM_STORE_16dp32b4x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b8x>) {
    return SM100_TMEM_STORE_16dp32b8x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b8x_16b>) {
    return SM100_TMEM_STORE_16dp32b8x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b16x>) {
    return SM100_TMEM_STORE_16dp32b16x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b16x_16b>) {
    return SM100_TMEM_STORE_16dp32b16x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b32x>) {
    return SM100_TMEM_STORE_16dp32b32x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b32x_16b>) {
    return SM100_TMEM_STORE_16dp32b32x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b64x>) {
    return SM100_TMEM_STORE_16dp32b64x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b64x_16b>) {
    return SM100_TMEM_STORE_16dp32b64x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b128x>) {
    return SM100_TMEM_STORE_16dp32b128x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_16dp32b128x_16b>) {
    return SM100_TMEM_STORE_16dp32b128x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b1x>) {
    return SM100_TMEM_STORE_32dp32b1x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b1x_16b>) {
    return SM100_TMEM_STORE_32dp32b1x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b2x>) {
    return SM100_TMEM_STORE_32dp32b2x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b2x_16b>) {
    return SM100_TMEM_STORE_32dp32b2x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b4x>) {
    return SM100_TMEM_STORE_32dp32b4x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b4x_16b>) {
    return SM100_TMEM_STORE_32dp32b4x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b8x>) {
    return SM100_TMEM_STORE_32dp32b8x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b8x_16b>) {
    return SM100_TMEM_STORE_32dp32b8x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b16x>) {
    return SM100_TMEM_STORE_32dp32b16x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b16x_16b>) {
    return SM100_TMEM_STORE_32dp32b16x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b32x>) {
    return SM100_TMEM_STORE_32dp32b32x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b32x_16b>) {
    return SM100_TMEM_STORE_32dp32b32x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b64x>) {
    return SM100_TMEM_STORE_32dp32b64x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b64x_16b>) {
    return SM100_TMEM_STORE_32dp32b64x_16b{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b128x>) {
    return SM100_TMEM_STORE_32dp32b128x{};
  }
  else if constexpr (is_same_v<CopyOp, SM100_TMEM_LOAD_32dp32b128x_16b>) {
    return SM100_TMEM_STORE_32dp32b128x_16b{};
  }
  else {
    static_assert(dependent_false<CopyOp>, "No TMEM_STORE matching for provided TMEM_LOAD");
  }
}
```
- **EN:** Defines or forwards `tmem_load_to_store` as part of this header's executable interface.
- **CN:** 定义或转发 `tmem_load_to_store`，作为该头文件可执行接口的一部分。

### Lines 3505-3505
```cpp
} // namespace TMEM
```
- **EN:** Enters or leaves namespace scope `TMEM` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `TMEM`，以便把相关符号组织在一起。

### Lines 3507-3513
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////////////////////////
//
// UTCCP Copy Traits
//
////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Section comment introducing the next logical part of the file: UTCCP Copy Traits
- **CN:** 分节注释，用来引出文件中的下一段逻辑：UTCCP Copy Traits

### Lines 3515-3515
```cpp
namespace SM100::TMEM::UTCCP {
```
- **EN:** Enters or leaves namespace scope `SM100::TMEM::UTCCP` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM100::TMEM::UTCCP`，以便把相关符号组织在一起。

### Lines 3517-3533
```cpp
//
// Specialized copy_unpack implementation for SM100::TMEM::UTCCP instructions
//

template <class CopyOp,
          class TS, class SLayout,
          class TD, class DLayout>
CUTE_HOST_DEVICE constexpr
void
copy_unpack(Copy_Traits<CopyOp> const&,
            Tensor<TS,SLayout>  const& src,
            Tensor<TD,DLayout>       & dst)
{
  static_assert(is_rmem<TS>::value, "Expected smem_desc src for SM100_UTCCP");
  static_assert(is_tmem<TD>::value, "Expected tmem dst for SM100_UTCCP");
  CopyOp::copy(src[0], raw_pointer_cast(dst.data()));
}
```
- **EN:** Defines `copy_unpack`, which reshapes tensor operands into the register view expected by the selected copy operation and dispatches it.
- **CN:** 定义 `copy_unpack`：把张量操作数重塑为所选拷贝操作期望的寄存器视图，然后完成派发。

### Lines 3535-3535
```cpp
} // end namespace SM100::TMEM::UTCCP
```
- **EN:** Enters or leaves namespace scope `SM100::TMEM::UTCCP` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM100::TMEM::UTCCP`，以便把相关符号组织在一起。

### Lines 3537-3546
```cpp
// In the following UTCCP traits, the ValID is representing:
// logical_bit_idx -> tmem_addr_offset.
// And the logical_bit_idx is numbered in the order of:
// [core_matrix_strided, core_matrix_leading, broadcast, repeat].
// The first two modes provide convenience for smem_desc construtction.
// The last two modes provide boradcast transformation for 4x32DP and 2x64DP.
// With above, the strides of first two modes are neccessary to be TMEM::DP_b and 1.
// And the stride of the third mode in the SrcLayout must be zero.

////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Section comment introducing the next logical part of the file: In the following UTCCP traits, the ValID is representing: logical_bit_idx -> tmem_addr_offset. And the logical...
- **CN:** 分节注释，用来引出文件中的下一段逻辑：In the following UTCCP traits, the ValID is representing: logical_bit_idx -> tmem_addr_offset. And the logical...

### Lines 3548-3548
```cpp
using SM100::TMEM::UTCCP::SM100_UTCCP_128dp256bit_1cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3550-3561
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_128dp256bit_1cta>
{
  using ThrID = Layout<_1>;
  using ValID = Layout<Shape <_128,      _256>,
                       Stride<TMEM::DP_b, _1>>;
  using SrcLayout = Layout<Shape<_1, _32768>,
                           Stride<_0, _1>>;
  using DstLayout = Layout<Shape<_1, _32768>,
                           Stride<_0,_1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3563-3565
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_128dp256bit_2cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3567-3577
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_128dp256bit_2cta>
{
  using ThrID = Layout<_2>;
  using ValID = typename Copy_Traits<SM100_UTCCP_128dp256bit_1cta>::ValID;
  using SrcLayout = Layout<Shape <_2, _32768>,
                           Stride<_0, _1>>;
  using DstLayout = Layout<Shape <_2, _32768>,
                           Stride<_0, _1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3579-3581
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_128dp128bit_1cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3583-3594
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_128dp128bit_1cta>
{
  using ThrID = Layout<_1>;
  using ValID = Layout<Shape <_128,      _128>,
                       Stride<TMEM::DP_b, _1>>;
  using SrcLayout = Layout<Shape<_1, _16384>,
                           Stride<_0, _1>>;
  using DstLayout = Layout<Shape<_1, _16384>,
                           Stride<_0,_1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3596-3598
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_128dp128bit_2cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3600-3610
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_128dp128bit_2cta>
{
  using ThrID = Layout<_2>;
  using ValID = typename Copy_Traits<SM100_UTCCP_128dp128bit_1cta>::ValID;
  using SrcLayout = Layout<Shape <_2, _16384>,
                           Stride<_0, _1>>;
  using DstLayout = Layout<Shape <_2, _16384>,
                           Stride<_0, _1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3612-3614
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_4dp256bit_1cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3616-3638
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_4dp256bit_1cta>
{
  /*
  4DP is really hard to model if we consider this instruction as a "copy" instruction.
  But, if we take it as "TMEM refresh" instruction, then everything goes out naturally.
  4DP utccp is designed to refresh the last 4 lanes of each tmem subpartition.
  So, in the kernel implementation, we usually only don't need to iterate on MMA_M dimension,
  but only need to iterate on MMA_K dimension.
  And in each refresh, logically we are refreshing MMA's 128 rows M + 256bit K.
  So the "atom_v" should be (refresh_m, refresh_k) instead of (copy_m, copy_k).
  And the Src/DstLayout below is: copy_bits -> logical_refresh_bits.
  */

  using ThrID = Layout<_1>;
  using ValID = Layout<Shape <_128,    _256>,
                       Stride<TMEM::DP_b,_1>>;
  using SrcLayout = Layout<Shape <_1,Shape <_4, _256>>,
                           Stride<_0,Stride<_32,_128>>>;
  using DstLayout = Layout<Shape <_1,Shape <_4, _256>>,
                           Stride<_0,Stride<_32,_128>>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3640-3642
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_4dp256bit_2cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3644-3654
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_4dp256bit_2cta>
{
  using ThrID = Layout<_2>;
  using ValID = typename Copy_Traits<SM100_UTCCP_4dp256bit_1cta>::ValID;
  using SrcLayout = Layout<Shape <_2,Shape <_4, _256>>,
                           Stride<_0,Stride<_32,_128>>>;
  using DstLayout = Layout<Shape <_2,Shape <_4, _256>>,
                           Stride<_0,Stride<_32,_128>>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3656-3658
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_4x32dp128bit_1cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3660-3676
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_4x32dp128bit_1cta>
{
  using _DP = TMEM::DP_b;
  using _DPx32 = Int<_DP{}*32>;

  using ThrID = Layout<_1>;
  // logical bit_idx -> tmem_addr
  // [core_matrix_strided, core_matrix_leading, broadcast]
  using ValID = Layout<Shape <_32,_128,_4>,
                       Stride<_DP,_1,  _DPx32>>;
  using SrcLayout = Layout<Shape <_1,Shape <_32,_128,_4>>,
                           Stride<_0,Stride<_1, _32, _0>>>;
  using DstLayout = Layout<Shape <_1,_16384>,
                           Stride<_0,_1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3678-3680
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_4x32dp128bit_2cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3682-3692
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_4x32dp128bit_2cta>
{
  using ThrID = Layout<_2>;
  using ValID = typename Copy_Traits<SM100_UTCCP_4x32dp128bit_1cta>::ValID;
  using SrcLayout = Layout<Shape <_2,Shape <_32,_128,_4>>,
                           Stride<_0,Stride<_1, _32, _0>>>;
  using DstLayout = Layout<Shape<_2, _16384>,
                           Stride<_0,_1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3694-3696
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_2x64dp128bitlw0213_1cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3698-3714
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_2x64dp128bitlw0213_1cta>
{
  using _DP = TMEM::DP_b;
  using _DPx64 = Int<_DP{}*64>;

  using ThrID = Layout<_1>;
  // logical bit_idx -> tmem_addr
  // [core_matrix_strided, core_matrix_leading, broadcast]
  using ValID = Layout<Shape <_64,_128,_2>,
                       Stride<_DP,_1,  _DPx64>>;
  using SrcLayout = Layout<Shape <_1,Shape <_64,_128,_2>>,
                           Stride<_0,Stride<_1, _64, _0>>>;
  using DstLayout = Layout<Shape<_1, _16384>,
                           Stride<_0, _1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3716-3718
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_2x64dp128bitlw0213_2cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3720-3731
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_2x64dp128bitlw0213_2cta>
{
  using ThrID = Layout<_2>;
  using ValID = typename Copy_Traits<SM100_UTCCP_2x64dp128bitlw0213_1cta>::ValID;

  using SrcLayout = Layout<Shape <_2,Shape <_64,_128,_2>>,
                           Stride<_0,Stride<_1, _64, _0>>>;
  using DstLayout = Layout<Shape<_2, _16384>,
                           Stride<_0, _1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3733-3735
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_2x64dp128bitlw0123_1cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3737-3755
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_2x64dp128bitlw0123_1cta>
{
  using _DP = TMEM::DP_b;
  using _DPx32 = Int<_DP{}*32>;
  using _DPx64 = Int<_DP{}*64>;

  using ThrID = Layout<_1>;
  // logical bit_idx -> tmem_addr
  // [core_matrix_strided, core_matrix_leading, repeat, broadcast]
  using ValID = Layout<Shape <_32,_128,_2,    _2>,
                       Stride<_DP,_1  ,_DPx64,_DPx32>>;

  using SrcLayout = Layout<Shape <_1,Shape <_32,_128,_2,_2>>,
                           Stride<_0,Stride<_1, _32,_4096,_0>>>;
  using DstLayout = Layout<Shape<_1, _16384>,
                           Stride<_0, _1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3757-3759
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

using SM100::TMEM::UTCCP::SM100_UTCCP_2x64dp128bitlw0123_2cta;
```
- **EN:** Introduces the alias `SM100` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `SM100`，便于复用周边的寄存器、布局或策略类型。

### Lines 3761-3771
```cpp
template <>
struct Copy_Traits<SM100_UTCCP_2x64dp128bitlw0123_2cta>
{
  using ThrID = Layout<_2>;
  using ValID = typename Copy_Traits<SM100_UTCCP_2x64dp128bitlw0123_1cta>::ValID;
  using SrcLayout = Layout<Shape <_2,Shape <_32,_128,_2,_2>>,
                           Stride<_0,Stride<_1, _32, _4096,_0>>>;
  using DstLayout = Layout<Shape <_2,_16384>,
                           Stride<_0,_1>>;
  using RefLayout = DstLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 3773-3793
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

template <class CopyOp,
          class TEngine, class TLayout>
CUTE_HOST_DEVICE constexpr
auto
make_utccp_copy(CopyOp const&,
                Tensor<TEngine,TLayout> const& tmem)
{
  static_assert(is_tmem<TEngine>::value, "Expected TMEM tensor.");
  using T      = typename TEngine::value_type;
  using Traits = Copy_Traits<CopyOp>;
  using Atom   = Copy_Atom<Traits, T>;

  // atom thr idx -> tmem addr    This is the T in the Layout_TV
  auto atom_t_layout = make_layout(size(typename Traits::ThrID{}), Int<0>{});
  // atom val idx -> tmem addr    Cast the CopyOp's value ids to the proper data width
  auto atom_v_layout = coalesce(upcast<sizeof_bits<T>::value>(typename Traits::ValID{}));

  return make_cotiled_copy(Atom{}, make_layout(atom_t_layout, atom_v_layout), tmem.layout());
}
```
- **EN:** Defines or forwards `make_utccp_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_utccp_copy`，作为该头文件可执行接口的一部分。

### Lines 3795-3797
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace cute
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
- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。

## Dependencies / 依赖关系

- **EN:** `cute/arch/copy_sm100.hpp` supplies related definitions from `cute/arch/copy_sm100.hpp`.
  **CN:** `cute/arch/copy_sm100.hpp` 提供了来自 `cute/arch/copy_sm100.hpp` 的相关定义。
- **EN:** `cute/arch/tmem_allocator_sm100.hpp` supplies related definitions from `cute/arch/tmem_allocator_sm100.hpp`.
  **CN:** `cute/arch/tmem_allocator_sm100.hpp` 提供了来自 `cute/arch/tmem_allocator_sm100.hpp` 的相关定义。
- **EN:** `cute/atom/copy_traits.hpp` supplies generic copy-trait interfaces and utilities.
  **CN:** `cute/atom/copy_traits.hpp` 提供了通用 copy trait 接口与工具。
- **EN:** `cute/atom/copy_atom.hpp` supplies copy atoms that combine traits with tiled tensor views.
  **CN:** `cute/atom/copy_atom.hpp` 提供了把 traits 与分块张量视图结合起来的 copy atom。
- **EN:** `cute/atom/partitioner.hpp` supplies related definitions from `cute/atom/partitioner.hpp`.
  **CN:** `cute/atom/partitioner.hpp` 提供了来自 `cute/atom/partitioner.hpp` 的相关定义。
- **EN:** `cute/numeric/numeric_types.hpp` supplies numeric helper types such as fixed-width bit wrappers.
  **CN:** `cute/numeric/numeric_types.hpp` 提供了数值辅助类型，例如固定位宽包装类型。
- **EN:** `cute/layout.hpp` supplies related definitions from `cute/layout.hpp`.
  **CN:** `cute/layout.hpp` 提供了来自 `cute/layout.hpp` 的相关定义。
- **EN:** CUDA architecture macros gate device-only fast paths and inline instructions.
  **CN:** CUDA 架构宏用于控制仅设备端可用的快速路径与内联指令。
