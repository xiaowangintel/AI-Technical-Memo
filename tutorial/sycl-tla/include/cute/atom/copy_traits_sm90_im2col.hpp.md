# copy_traits_sm90_im2col.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/atom/copy_traits_sm90_im2col.hpp`
- **EN:** Defines im2col-related copy-trait specializations for NVIDIA SM90.
- **CN:** 为 NVIDIA SM90 定义与 im2col 相关的 copy trait 特化。

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
/*! \file
  \brief im2col make_tma_copy
*/
```
- **EN:** Implements a supporting fragment of the surrounding algorithm, trait, or architecture wrapper.
- **CN:** 实现周边算法、trait 或体系结构封装所需的辅助片段。

### Lines 37-39
```cpp
#include "cute/arch/copy_sm90.hpp"
#include "cute/arch/copy_sm90_desc.hpp"
#include "cute/tensor.hpp"
```
- **EN:** Imports `cute/arch/copy_sm90.hpp` (related definitions from `cute/arch/copy_sm90.hpp`); `cute/arch/copy_sm90_desc.hpp` (related definitions from `cute/arch/copy_sm90_desc.hpp`); `cute/tensor.hpp` (related definitions from `cute/tensor.hpp`).
- **CN:** 引入 `cute/arch/copy_sm90.hpp`（来自 `cute/arch/copy_sm90.hpp` 的相关定义）；`cute/arch/copy_sm90_desc.hpp`（来自 `cute/arch/copy_sm90_desc.hpp` 的相关定义）；`cute/tensor.hpp`（来自 `cute/tensor.hpp` 的相关定义）。

### Lines 41-43
```cpp
#include "cute/algorithm/prefetch.hpp"
#include "cutlass/fast_math.h"
#include "cutlass/cuda_host_adapter.hpp"
```
- **EN:** Imports `cute/algorithm/prefetch.hpp` (higher-level prefetch helpers); `cutlass/fast_math.h` (related definitions from `cutlass/fast_math.h`); `cutlass/cuda_host_adapter.hpp` (CUTLASS host-side CUDA adapter utilities).
- **CN:** 引入 `cute/algorithm/prefetch.hpp`（更高层的预取辅助工具）；`cutlass/fast_math.h`（来自 `cutlass/fast_math.h` 的相关定义）；`cutlass/cuda_host_adapter.hpp`（CUTLASS 主机侧 CUDA 适配工具）。

### Lines 45-46
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 48-90
```cpp
// Utility for unpacking TMA_LOAD_IM2COL arguments into a CopyOp
template <class CopyOp>
struct TMA_LOAD_IM2COL_Unpack
{
  /// Copy from src to dst.
  ///
  /// @param traits Copy traits created with a TMA descriptor that
  ///   correctly matches the input tensor and other convolution
  ///   parameters.
  ///
  /// @param src Tile of the im2col-transformed coordinate tensor
  ///   (result of get_tma_tensor), representing the global-memory
  ///   tensor from which to load.
  ///
  /// @param dst Shared memory tile, into which to load.
  template <class... Args,
            class TS, class SLayout,
            class TD, class DLayout>
  CUTE_HOST_DEVICE friend constexpr void
  copy_unpack(Copy_Traits<CopyOp, Args...> const& traits,
              Tensor<TS,SLayout>           const& src, // tile of the transformed global activation (A) tensor
              Tensor<TD,DLayout>                & dst) // shared memory tile
  {
    auto src_coord_offset = src(Int<0>{});
    auto src_coord_cwhdn_offset_srt = flatten(src_coord_offset);
    // Interpret the TMA IM2COL coordinate as  (c, ([w,h,d]), n, ([s,r,t]))
    CUTE_STATIC_ASSERT_V(rank(src_coord_offset) == _4{});
    CUTE_STATIC_ASSERT_V(rank<1>(src_coord_offset) == rank<3>(src_coord_offset));

    if constexpr (detail::is_prefetch<CopyOp>) {
      return detail::explode_tuple(detail::CallCOPY<CopyOp>{},
                                   traits.opargs_, tuple_seq<decltype(traits.opargs_)>{},
                                   src_coord_cwhdn_offset_srt, tuple_seq<decltype(src_coord_cwhdn_offset_srt)>{});
    } else {
      static_assert(is_smem<TD>::value, "SM90_TMA_LOAD_IM2COL requires the destination be shared memory.");
      void* dst_ptr = cute::raw_pointer_cast(dst.data());
      return detail::explode_tuple(detail::CallCOPY<CopyOp>{},
                                   traits.opargs_, tuple_seq<decltype(traits.opargs_)>{},
                                   make_tuple(dst_ptr), seq<0>{},
                                   src_coord_cwhdn_offset_srt, tuple_seq<decltype(src_coord_cwhdn_offset_srt)>{});
    }
  }
};
```
- **EN:** Defines `TMA_LOAD_IM2COL_Unpack`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `TMA_LOAD_IM2COL_Unpack`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 92-106
```cpp
// Copy_Traits for SM90 im2col TMA load comes in two layers.
//
// 1. Copy_Traits<SM90_TMA_LOAD_IM2COL>
// 2. Copy_Traits<SM90_TMA_LOAD_IM2COL_OP>
//
// Copy_Traits<SM90_TMA_LOAD_IM2COL>
// is the "outer" layer.  It has a TMA descriptor,
// but no barrier ("tma_mbar"), so it's "nonexecutable."
// One calls its "with" member function with a barrier,
// to get an executable "inner"-layer
// Copy_Traits<SM90_TMA_LOAD_IM2COL_OP> object.
// That object's "copy_unpack" member function
// actually invokes im2col TMA load.

struct SM90_TMA_LOAD_IM2COL_OP : SM90_TMA_LOAD_IM2COL {};
```
- **EN:** Defines `SM90_TMA_LOAD_IM2COL_OP`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM90_TMA_LOAD_IM2COL_OP`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 108-170
```cpp
/// @brief Non-executable specialization of Copy_Traits for SM90
///   im2col TMA load, with TMA descriptor but no barrier.
///
/// Use `.with(memory_barrier)` to construct an executable version.
template <class NumBitsPerTMA, class TMATensor>
struct Copy_Traits<SM90_TMA_LOAD_IM2COL, NumBitsPerTMA, TMATensor>
{
  using ThrID = Layout<_1>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  Im2ColTmaDescriptor tma_desc_;
  TMATensor tma_tensor_;

  CUTE_HOST_DEVICE constexpr
  Im2ColTmaDescriptor const*
  get_tma_descriptor() const
  {
    return &tma_desc_;
  }

  template <class GShape>
  CUTE_HOST_DEVICE constexpr
  TMATensor const
  get_tma_tensor(GShape const&) const
  {
    return tma_tensor_;
  }

  /// @brief Get an executable specialization.
  ///
  /// Copy_Traits specializations with SM90_TMA_LOAD_IM2COL are not
  /// directly executable.  Instead, call this "with" member function
  /// to get an executable specialization.  "Executable" means that
  /// @c copy_unpack works.
  ///
  /// @param tma_mbar Memory barrier for synchronization
  ///
  /// @param multicast_mask Multicast mask (unused; only exists
  ///   for interface compatibility with the actual multicast Copy_Traits)
  ///
  /// @return Executable specialization of @c Copy_Traits
  CUTE_HOST_DEVICE constexpr
  Copy_Traits<SM90_TMA_LOAD_IM2COL_OP, NumBitsPerTMA>
  with(uint64_t& tma_mbar, [[maybe_unused]] uint16_t const& multicast_mask = 0) const
  {
    return {{}, {&tma_desc_, &tma_mbar}};
  }

  // Copy_Traits specializations with SM90_TMA_LOAD_IM2COL
  // are not directly executable.  Instead, call .with
  // to get an executable specialization.
  template <class TS, class SLayout,
            class TD, class DLayout>
  CUTE_HOST_DEVICE friend constexpr void
  copy_unpack(Copy_Traits        const& traits,
              Tensor<TS,SLayout> const& src,
              Tensor<TD,DLayout>      & dst) = delete;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 172-191
```cpp
/// @brief Executable specialization of Copy_Traits for SM90 im2col
///   TMA load, with TMA descriptor and barrier.
template <class NumBitsPerTMA>
struct Copy_Traits<SM90_TMA_LOAD_IM2COL_OP, NumBitsPerTMA>
     : TMA_LOAD_IM2COL_Unpack<SM90_TMA_LOAD_IM2COL_OP>
{
  using ThrID = Layout<_1>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM90_TMA_LOAD_IM2COL arguments
  tuple<
  Im2ColTmaDescriptor const*,
  uint64_t* // smem mbarrier
  > const opargs_;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 193-211
```cpp
template <class NumBitsPerTMA, class... Args>
struct Copy_Traits<SM90_TMA_LOAD_IM2COL::PREFETCH, NumBitsPerTMA, Args...>
     : TMA_LOAD_IM2COL_Unpack<SM90_TMA_LOAD_IM2COL::PREFETCH>
{
  using ThrID = Layout<_1>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM90_TMA_LOAD_IM2COL::PREFETCH arguments
  tuple<Im2ColTmaDescriptor const*> const opargs_;

  CUTE_HOST_DEVICE
  Copy_Traits(Copy_Traits<SM90_TMA_LOAD_IM2COL, NumBitsPerTMA, Args...> const& traits)
    : opargs_({&traits.tma_desc_}) {}
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 213-217
```cpp
//////////////////////////////////////////////////////////////////////////////
///////////////////////////// TMA_LOAD_MULTICAST /////////////////////////////
//////////////////////////////////////////////////////////////////////////////

struct SM90_TMA_LOAD_IM2COL_MULTICAST_OP : SM90_TMA_LOAD_IM2COL_MULTICAST {};
```
- **EN:** Defines `SM90_TMA_LOAD_IM2COL_MULTICAST_OP`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM90_TMA_LOAD_IM2COL_MULTICAST_OP`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 219-279
```cpp
/// @brief Non-executable specialization of Copy_Traits for SM90
///   im2col TMA load, with TMA descriptor but no barrier or multicast
///   mask.
///
/// Use `.with(memory_barrier)` to construct an executable version.
template <class NumBitsPerTMA, class TMATensor>
struct Copy_Traits<SM90_TMA_LOAD_IM2COL_MULTICAST, NumBitsPerTMA, TMATensor>
{
  using ThrID = Layout<_1>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  Im2ColTmaDescriptor tma_desc_;
  TMATensor tma_tensor_;

  CUTE_HOST_DEVICE constexpr
  Im2ColTmaDescriptor const*
  get_tma_descriptor() const {
    return &tma_desc_;
  }

  template <class GShape>
  CUTE_HOST_DEVICE constexpr
  TMATensor const
  get_tma_tensor(GShape const&) const
  {
    return tma_tensor_;
  }

  /// @brief Get an executable specialization.
  ///
  /// Copy_Traits specializations with SM90_TMA_LOAD_IM2COL_MULTICAST
  /// are not directly executable.  Instead, call this "with" member
  /// function to get an executable specialization.  "Executable"
  /// means that @c copy_unpack works.
  ///
  /// @param tma_mbar Memory barrier for synchronization
  ///
  /// @param multicast_mask Multicast mask (defaults to a single CTA)
  ///
  /// @return Executable specialization of @c Copy_Traits
  CUTE_HOST_DEVICE constexpr
  Copy_Traits<SM90_TMA_LOAD_IM2COL_MULTICAST_OP, NumBitsPerTMA>
  with(uint64_t& tma_mbar, uint16_t const& multicast_mask) const {
    return {{}, {&tma_desc_, &tma_mbar, multicast_mask}};
  }

  // Copy_Traits specializations with SM90_TMA_LOAD_IM2COL_MULTICAST
  // are not directly executable.  Instead, call .with to get an
  // executable specialization.
  template <class TS, class SLayout,
            class TD, class DLayout>
  CUTE_HOST_DEVICE friend constexpr void
  copy_unpack(Copy_Traits        const& traits,
              Tensor<TS,SLayout> const& src,
              Tensor<TD,DLayout>      & dst) = delete;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 281-301
```cpp
/// @brief Executable specialization of Copy_Traits for SM90 multicast
///   im2col TMA load, with TMA descriptor, barrier, and multicast mask.
template <class NumBitsPerTMA>
struct Copy_Traits<SM90_TMA_LOAD_IM2COL_MULTICAST_OP, NumBitsPerTMA>
     : TMA_LOAD_IM2COL_Unpack<SM90_TMA_LOAD_IM2COL_MULTICAST_OP>
{
  using ThrID = Layout<_1>;
  // Map from (src-thr,src-val) to bit.
  using SrcLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_1, NumBitsPerTMA>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM90_TMA_LOAD_IM2COL_MULTICAST arguments
  tuple<
  Im2ColTmaDescriptor const*,
  uint64_t*, // smem mbarrier
  uint16_t   // multicast mask
  > const opargs_;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 303-360
```cpp
//////////////////////////////////////////////////////////////////////////////
///////////////////////////// TMA_STORE IM2COL////////////////////////////////
//////////////////////////////////////////////////////////////////////////////

// The executable SM90_TMA_STORE_IM2COL with tma_desc
template <class NumBitsPerTMA, class TMATensor>
struct Copy_Traits<SM90_TMA_STORE_IM2COL, NumBitsPerTMA, TMATensor>
{
  using ThrID   = Layout<_1>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_1,NumBitsPerTMA>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_1,NumBitsPerTMA>>;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM90_TMA_STORE_IM2COL arguments
  Im2ColTmaDescriptor tma_desc_;
  TMATensor tma_tensor_;

  // Return TmaDescriptor/TensorMap
  CUTE_HOST_DEVICE constexpr
  Im2ColTmaDescriptor const*
  get_tma_descriptor() const {
    return &tma_desc_;
  }

  template <class GShape>
  CUTE_HOST_DEVICE constexpr
  TMATensor const
  get_tma_tensor(GShape const&) const
  {
    return tma_tensor_;
  }

  // This is the copy_unpack dispatch for this Copy_Traits
  // Src needs to be a smem tensor
  // Dst needs to be a gmem tensor with TmaCoordIterator .data()
  template <class TS, class SLayout,
            class TD, class DLayout>
  CUTE_HOST_DEVICE friend constexpr void
  copy_unpack(Copy_Traits        const& traits,
              Tensor<TS,SLayout> const& src,
              Tensor<TD,DLayout>      & dst)
  {
    static_assert(is_smem<TS>::value, "Expected smem src for SM90_TMA_STORE_IM2COL");

    void const* const desc_ptr = &(traits.tma_desc_);
    void const* const src_ptr  = cute::raw_pointer_cast(src.data());
    auto dst_coord = flatten(take<0,3>(dst(Int<0>{})));

    return detail::explode_tuple(detail::CallCOPY<SM90_TMA_STORE_IM2COL>{},
                                 make_tuple(desc_ptr, src_ptr), seq<0,1>{},
                                 dst_coord, tuple_seq<decltype(dst_coord)>{});
  }
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 362-362
```cpp
namespace detail {
```
- **EN:** Enters or leaves namespace scope `detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `detail`，以便把相关符号组织在一起。

### Lines 364-550
```cpp
/// @brief Creates a TMA descriptor for im2col TMA load.
///
/// @param tensor_cwhdn Global activation tensor (A matrix of Fprop).
///   This is the original (not im2col-transformed) tensor in global
///   memory.
///
/// @param slayout Rank 2 (M,K) shared memory layout of the activation
///   tensor.  Here, K is "GEMM K," not the filter tensor's mode of
///   the same name.
//////
/// @param traversal_stride Traversal strides convolution parameter
//////
/// Each of padding_shape, traversal_stride, and dilation_shape is a
/// tuple whose size is the number of spatial modes (e.g., 3 for a 5-D
/// convolution).
///
/// @return TMA descriptor for im2col TMA load
template <class EngineA, class LayoutA,
          class SmemSwizzle, class TMALayout,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST
auto
make_im2col_tma_copy_desc(
    Tensor<EngineA, LayoutA>    const& tensor_cwhdn,       // (C,W,H,D,N)
    uint32_t                           range_c,            // TILE_C
    uint32_t                           range_whdn,         // TILE_WHDN
    SmemSwizzle                 const& smem_swizzle,       // Swizzle
    TMALayout                   const& tma_layout_vt,      // TMA layout
    LowerCornerStride           const& lower_corner_whd,   // WHD offset of the "base pointer"
    UpperCornerStride           const& upper_corner_whd,   // WHD upper corner
    LowerPaddingStride          const& lower_padding_whd,  // WHD lower padding
    UpperPaddingStride          const& upper_padding_whd,  // WHD upper padding
    TraversalStride             const& stride_whd,         // WHD traversal stride
    LowerSRTStride              const& lower_srt,          // SRT offset of the "base pointer"
    DilationStride              const& stride_srt,          // SRT stride - dilation
    TMA::DescriptorAuxParams    const& aux_params = {})
{
  static_assert(is_gmem<EngineA>::value, "Tensor must point to GPU global memory.");
  using value_type = typename EngineA::value_type;

  constexpr uint32_t num_total_modes   = LayoutA::rank;
  constexpr int      num_spatial_modes = num_total_modes - 2;

  // Gmem starting address
  void* gmem_address = (void*) raw_pointer_cast(tensor_cwhdn.data());

  // Gmem extents are just the tensor shape
  cute::array<uint64_t, 5> gmem_prob_shape = {1,1,1,1,1};
  for_each(make_seq<num_total_modes>{}, [&](auto i) {
    gmem_prob_shape[i] = static_cast<uint64_t>(shape<i>(tensor_cwhdn));
  });

  // Gmem strides are byte strides of the activation tensor in CWHDN order
  cute::array<uint64_t, 5> gmem_prob_stride = {0,0,0,0,0};
  for_each(make_seq<num_total_modes>{}, [&](auto i) {
    gmem_prob_stride[i] = sizeof(value_type) * stride<i>(tensor_cwhdn);
  });

  // Traversal strides are a function of the dilation shape
  // corresponding to spatial (WHD) modes.
  cute::array<uint32_t, 5> tma_traversal_strides = {1,1,1,1,1};
  for_each(make_seq<num_spatial_modes>{}, [&](auto i) {
    tma_traversal_strides[i+1] = static_cast<uint32_t>(get<i>(stride_whd));
  });

  cute::array<int32_t, num_spatial_modes> tma_lower_corner{};
  for_each(make_seq<num_spatial_modes>{}, [&](auto i) {
    tma_lower_corner[i] = static_cast<int32_t>(get<i>(lower_corner_whd));
  });

  cute::array<int32_t, num_spatial_modes> tma_upper_corner{};
  for_each(make_seq<num_spatial_modes>{}, [&](auto i) {
    tma_upper_corner[i] = static_cast<int32_t>(get<i>(upper_corner_whd));
  });

  Im2ColTmaDescriptor tma_desc;

#if (__CUDACC_VER_MAJOR__ >= 12)

  CUtensorMapDataType     tma_format      = TMA::to_CUtensorMapDataType<value_type>();
  CUtensorMapInterleave   tma_interleave  = CU_TENSOR_MAP_INTERLEAVE_NONE;
  CUtensorMapL2promotion  tma_l2Promotion = to_CUtensorMapL2promotion(aux_params.l2promo_);
  CUtensorMapFloatOOBfill tma_oob_fill    = to_CUtensorMapFloatOOBfill(aux_params.oobfill_);
  TMA::SmemSwizzleBits    swizzle_bits    = detail::get_tma_swizzle_bits(smem_swizzle);
  TMA::SmemSwizzleBase    swizzle_base    = detail::get_tma_swizzle_base(smem_swizzle);
  CUtensorMapSwizzle      tma_swizzle     = TMA::to_CUtensorMapSwizzle(swizzle_bits, swizzle_base);

  CUresult encode_result = CUTLASS_CUDA_DRIVER_WRAPPER_CALL(cuTensorMapEncodeIm2col)(
      &tma_desc,
      tma_format,
      num_total_modes,
      gmem_address,
      gmem_prob_shape.data(),
      gmem_prob_stride.data() + 1, // gmem_prob_stride[0] implicitly sizeof(value_type)
      tma_lower_corner.data(),
      tma_upper_corner.data(),
      range_c,
      range_whdn,
      tma_traversal_strides.data(),
      tma_interleave,
      tma_swizzle,
      tma_l2Promotion,
      tma_oob_fill);

  // The extra asserts help indicate the error's cause.
  assert(encode_result != CUDA_ERROR_DEINITIALIZED);
  assert(encode_result != CUDA_ERROR_NOT_INITIALIZED);
  assert(encode_result != CUDA_ERROR_INVALID_CONTEXT);
  assert(encode_result != CUDA_ERROR_INVALID_VALUE);
  assert(encode_result == CUDA_SUCCESS);

#endif // (__CUDACC_VER_MAJOR__ >= 12)
  //
  // Calculate gemm shapes and linearized shapes based on tma layout tiling.
  //

  // Compute [w, h, d, n]
  // q/p/z = (w/h/d + (upper_corner_whd - lower_corner_whd - 1)) / stride_whd + 1
  auto gemm_mn_ = cute::transform(cute::make_seq<num_spatial_modes>{}, [&](auto i) {
    return (shape<i+1>(tensor_cwhdn) + get<i>(upper_corner_whd) - get<i>(lower_corner_whd) - Int<1>{}) / get<i>(stride_whd) + Int<1>{};
  });
  auto gemm_mn = append(gemm_mn_, shape<num_spatial_modes+1>(tensor_cwhdn));

  // Compute [c, s, r, t]
  // fprop/wgrad, s/r/t = 1 + (upper_padding_whd - upper_corner_whd) / stride_srt
  // wgrad,       s/r/t = 1 + (lower_padding_whd - lower_corner_whd) / stride_srt
  auto gemm_k_ = cute::transform(cute::make_seq<num_spatial_modes>{}, [&](auto i) {
    auto padding_size = conditional_return(get<i>(stride_srt) > Int<0>{},
                                           get<i>(upper_padding_whd) - get<i>(upper_corner_whd),
                                           get<i>(lower_corner_whd)  - get<i>(lower_padding_whd));
    return Int<1>{} + padding_size / get<i>(stride_srt);
  });
  auto gemm_k = prepend(gemm_k_, shape<0>(tensor_cwhdn));

  // For fprop/dgrad kernel, gemm_shapes is ((q, p, z, n), (c, s, r, t))
  // For wgrad kernel, gemm_shapes is ((c, s, r, t), (q, p, z, n))
  auto gemm_shapes_common = make_shape(
      transform_leaf(gemm_mn, [](auto s) {
        return conditional_return(cute::is_static<decltype(s)>{}, s, cutlass::FastDivmod(s));
      }),
      gemm_k);
  auto gemm_shapes = make_shape(
      basis_get(stride<0,1>(tma_layout_vt), gemm_shapes_common),
      basis_get(stride<0,0>(tma_layout_vt), gemm_shapes_common));

  // For fprop/dgrad kernel, linearized shapes is (whdn, (c, s, r, t))
  // For wgrad kernel linearized shapes is ((c, s, r, t), whdn)
  auto linear_shapes_common = make_shape(size(gemm_mn), gemm_k);
  auto linear_shapes = make_shape(
      basis_get(stride<0,1>(tma_layout_vt), linear_shapes_common),
      basis_get(stride<0,0>(tma_layout_vt), linear_shapes_common));

  //
  // Calculate gmem basis stride based on tma layout tiling.
  //

  auto tma_basis_scale = make_shape(Int<1>{}, stride_whd, Int<1>{}, stride_srt);
  auto tma_basis = elem_scale(tma_basis_scale, make_basis_like(tma_basis_scale));

  auto gbasis_strides_common = make_stride(
      append(get<1>(tma_basis), get<2>(tma_basis)),
      prepend(get<3>(tma_basis), get<0>(tma_basis)));    // ((w,h,d,n),(c,s,r,t))
  auto gbasis_strides = make_stride(
      basis_get(stride<0,1>(tma_layout_vt), gbasis_strides_common),
      basis_get(stride<0,0>(tma_layout_vt), gbasis_strides_common));

  //
  // Create tma tensor
  //

  auto lower_corner = make_arithmetic_tuple(Int<0>{}, lower_corner_whd, Int<0>{}, lower_srt);

  auto tensor_multimode = make_tensor(ArithmeticTupleIterator(lower_corner), gemm_shapes, gbasis_strides);
  auto tensor_linear = make_identity_tensor(linear_shapes);
  auto tma_tensor = make_tensor(tensor_multimode.data(), composition(
      tensor_multimode.layout(),
      tensor_linear(Int<0>{}),
      tensor_linear.layout()));

  return cute::make_tuple(tma_desc, tma_tensor);
}
```
- **EN:** Defines or forwards `make_im2col_tma_copy_desc` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_copy_desc`，作为该头文件可执行接口的一部分。

### Lines 552-679
```cpp
template <class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class VShape, class VStride,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST_RTC
auto
make_tma_atom_im2col(CopyOp,
                     Tensor<GEngine,GLayout>      const& gtensor,           // Full GMEM Tensor: ((w, h, d, n), c)
                     SLayout                      const& slayout,           // CTA Tile of SMEM, potentially swizzled
                     int32_t                      const& num_multicast,     // The number of CTAs involved in multicasting
                     Layout<VShape,VStride>       const& cta_v_map,         // V: CTA val idx -> gmem mode
                     LowerCornerStride            const& lower_corner_whd,
                     UpperCornerStride            const& upper_corner_whd,
                     LowerPaddingStride           const& lower_padding_whd,
                     UpperPaddingStride           const& upper_padding_whd,
                     TraversalStride              const& stride_whd,        // traversal stride
                     LowerSRTStride               const& lower_srt,
                     DilationStride               const& stride_srt,        // dilation
                     TMA::DescriptorAuxParams     const& aux_params = {})
{
  //
  // TMA parameter checking
  //

  CUTE_STATIC_ASSERT_V(product_each(shape(slayout)) == product_each(shape(cta_v_map)),
    "TMA requires CTA_Tile and SLayout top-level shape equivalence.");

  //
  // TMA slayout manipulation
  //

  // Invert the smem to get the largest contiguous vector in the smem layout
  auto inv_smem_layout = right_inverse(get_nonswizzle_portion(slayout));
  // trunc_smem_idx -> trunc_smem_coord

  // Map from smem idx to a gmem mode
  auto sidx_to_gmode = coalesce(composition(cta_v_map, inv_smem_layout));

#if 0
  print("g_layout         : "); print(gtensor.layout()); print("\n");
  print("s_layout         : "); print(slayout); print("\n");
  print("cta_t_map        : "); print(cta_t_map); print("\n");
  print("cta_v_map        : "); print(cta_v_map); print("\n");
  print("inv_smem         : "); print(inv_smem_layout); print("\n");
  print("sidx_to_gmode    : "); print(sidx_to_gmode); print("\n");
#endif

  //
  // TMA gtensor manipulation
  //

  // Generate a TupleBasis for the gtensor
  auto glayout_basis = make_identity_layout(product_each(shape(gtensor)));

  // Tile the modes of gtensor with the truncated cta_v_map o inv_smem_layout_trunc
  auto tma_layout_full = flatten(composition(glayout_basis, sidx_to_gmode));

  // Truncate any incompatibilities -- no starting in the middle of gmodes
  auto smem_rank = find_if(stride(tma_layout_full), [](auto e) {
    [[maybe_unused]] auto v = basis_value(e);
    return not is_constant<1,decltype(v)>{};
  });
  static_assert(smem_rank >= 2, "IM2COL expects at least 2 modes of the smem to vectorize with gmem.");
  // IM2COL uses a maximum of 2 modes
  constexpr int smem_tma_rank = cute::min(int(smem_rank), 2);

  // Keep only the static-1 basis modes into gmem
  auto tma_layout_trunc = take<0,smem_tma_rank>(tma_layout_full);

  // Split according to the portion each multicast CTA will be responsible for
  auto tma_layout_vt = logical_divide(tma_layout_trunc, safe_div(size(tma_layout_trunc), num_multicast));

#if 0
  print("glayout_basis   : "); print(glayout_basis); print("\n");
  print("tma_layout_full : "); print(tma_layout_full); print("\n");

  print("tma_layout_trunc: "); print(tma_layout_trunc); print("\n");
  print("tma_layout_vt   : "); print(tma_layout_vt); print("\n");
#endif

  auto range_c    = size<0,0>(tma_layout_vt);
  auto range_whdn = size<0,1>(tma_layout_vt);
  Tensor gtensor_cwhdn = make_tensor(gtensor.data(),
                                     flatten(make_layout(make_layout(basis_get(stride<0,0>(tma_layout_vt), gtensor.shape()),
                                                                     basis_get(stride<0,0>(tma_layout_vt), gtensor.stride())),
                                                         make_layout(basis_get(stride<0,1>(tma_layout_vt), gtensor.shape()),
                                                                     basis_get(stride<0,1>(tma_layout_vt), gtensor.stride())))));
  auto [tma_desc, tma_tensor] = make_im2col_tma_copy_desc(
      gtensor_cwhdn,
      range_c,
      range_whdn,
      get_swizzle_portion(slayout),
      tma_layout_vt,
      lower_corner_whd,
      upper_corner_whd,
      lower_padding_whd,
      upper_padding_whd,
      stride_whd,
      lower_srt,
      stride_srt,
      aux_params);

  //
  // Construct the Copy_Traits
  //

  using T = typename GEngine::value_type;
  constexpr int num_bits_per_tma = decltype(size(tma_layout_trunc))::value * sizeof(T) * 8;

  using Traits = Copy_Traits<CopyOp, cute::C<num_bits_per_tma>, decltype(tma_tensor)>;
  using Atom = Copy_Atom<Traits, typename GEngine::value_type>;

#if 0
  print("num_bits      :  "); print(num_bits_per_tma); print("\n");
#endif

  Traits tma_traits{tma_desc, tma_tensor};

  // Return the Copy_Atom
  return Atom{tma_traits};
}
```
- **EN:** Defines or forwards `make_tma_atom_im2col` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tma_atom_im2col`，作为该头文件可执行接口的一部分。

### Lines 681-767
```cpp
/// Make a TiledCopy for im2col TMA load.
///
/// @param copy_op The copy implementation: either
///   SM90_TMA_LOAD_IM2COL or SM90_TMA_LOAD_IM2COL_MULTICAST.
///
/// @param tensor_cwhdn The global tensor to use for im2col TMA loads.
///   For Fprop convolutions, this is the activation tensor.  This is
///   the "original tensor that points to global memory, not the
///   coordinate (im2col-transformed) tensor.
///
/// @param slayout Layout of shared memory tile.
///
/// @param stride_whd The traversal strides convolution
///   parameter.
///
/// @return TiledCopy specialization for im2col TMA loads.
template <class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class TShape, class TStride,
          class VShape, class VStride,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST_RTC
auto
make_tma_copy_im2col(CopyOp                       const& copy_op,
                     Tensor<GEngine,GLayout>      const& gtensor,
                     SLayout                      const& slayout,
                     Layout<TShape,TStride>       const& cta_t_map,          // CTA tid -> logical TMA tid
                     Layout<VShape,VStride>       const& cta_v_map,          // CTA vid -> gmem coord
                     LowerCornerStride            const& lower_corner_whd,
                     UpperCornerStride            const& upper_corner_whd,
                     LowerPaddingStride           const& lower_padding_whd,
                     UpperPaddingStride           const& upper_padding_whd,
                     TraversalStride              const& stride_whd,         // traversal stride
                     LowerSRTStride               const& lower_srt,
                     DilationStride               const& stride_srt,         // dilation
                     TMA::DescriptorAuxParams     const& aux_params = {})
{
  //
  // TMA parameter checking
  //

  CUTE_STATIC_ASSERT_V(size(slayout) % cosize(cta_t_map) == Int<0>{},
    "Number of active CTAs in TMA must divide domain size of slayout.");

  Copy_Atom atom = make_tma_atom_im2col(copy_op, gtensor, slayout, cosize(cta_t_map), cta_v_map,
                                        lower_corner_whd, upper_corner_whd, lower_padding_whd,
                                        upper_padding_whd, stride_whd, lower_srt, stride_srt, aux_params);

  //
  // Construct the TiledCopy
  //

  auto cta_tiler = product_each(shape(cta_v_map));

  auto num_elems_per_tma = size<1>(typename decltype(atom)::RefLayout{}) / static_value<sizeof_bits<typename GEngine::value_type>>();

  // smem idx -> smem coord
  auto inv_smem_layout = right_inverse(get_nonswizzle_portion(slayout));
  // CTA V -> smem_coord
  auto layout_v = composition(inv_smem_layout, num_elems_per_tma);
  // Scale that up to cover all of the smem_coords
  auto layout_V = tile_to_shape(make_layout(layout_v), size(cta_v_map));
  // CTA T -> smem idx
  auto layout_t = make_layout(cosize(cta_t_map), safe_div(num_elems_per_tma, cosize(cta_t_map)));
  // CTA TID -> smem coord
  auto layout_T = composition(inv_smem_layout, composition(layout_t, cta_t_map));
  // Combine with the T mapping
  [[maybe_unused]] auto layout_TV = make_layout(layout_T, layout_V);

#if 0
  print("cta_tiler : "); print(cta_tiler); print("\n");
  print("layout_v : "); print(layout_v); print("\n");
  print("layout_V : "); print(layout_V); print("\n");
  print("layout_t : "); print(layout_t); print("\n");
  print("layout_T : "); print(layout_T); print("\n");
  print("layout_TV : "); print(layout_TV); print("\n");
#endif

  return TiledCopy<decltype(atom), decltype(layout_TV), decltype(cta_tiler)>{atom};
}
```
- **EN:** Defines or forwards `make_tma_copy_im2col` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tma_copy_im2col`，作为该头文件可执行接口的一部分。

### Lines 769-793
```cpp
/// Make a TiledCopy for im2col TMA with no offsets.
/// E.g. im2col TMA load for C and im2col TMA store for D.
template <class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class TShape, class TStride,
          class VShape, class VStride>
CUTE_HOST_RTC
auto
make_tma_copy_im2col(CopyOp                  const& copy_op,
                     Tensor<GEngine,GLayout> const& gtensor,
                     SLayout                 const& slayout,
                     Layout<TShape,TStride>  const& cta_t_map,          // CTA tid -> logical TMA tid
                     Layout<VShape,VStride>  const& cta_v_map)          // CTA vid -> gmem coord
{
  constexpr int num_spatial_modes = rank<0>(GLayout{}) - 1;
  return make_tma_copy_im2col(copy_op, gtensor, slayout, cta_t_map, cta_v_map,
                              append<num_spatial_modes>(Stride<_0>{}, Int<0>{}),  // lower_corner_whd
                              append<num_spatial_modes>(Stride<_0>{}, Int<0>{}),  // upper_corner_whd
                              append<num_spatial_modes>(Stride<_0>{}, Int<0>{}),  // lower_padding_whd
                              append<num_spatial_modes>(Stride<_0>{}, Int<0>{}),  // upper_padding_whd
                              append<num_spatial_modes>(Stride<_1>{}, Int<1>{}),  // stride_whd
                              append<num_spatial_modes>(Stride<_0>{}, Int<0>{}),  // lower_srt
                              append<num_spatial_modes>(Stride<_1>{}, Int<1>{})); // stride_srt
}
```
- **EN:** Defines or forwards `make_tma_copy_im2col` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tma_copy_im2col`，作为该头文件可执行接口的一部分。

### Lines 795-795
```cpp
} // namespace detail
```
- **EN:** Enters or leaves namespace scope `detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `detail`，以便把相关符号组织在一起。

### Lines 799-832
```cpp
template <class CopyOp,
          class Engine0, class Layout0,
          class SLayout,
          class CTATiler,
          class MulticastSize,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST_RTC
auto
make_im2col_tma_copy(CopyOp                   const& copy_op,
                     Tensor<Engine0, Layout0> const& tensor_cwhdn,
                     SLayout                  const& slayout,
                     CTATiler                 const& cta_tiler,
                     MulticastSize            const& multicast_size,
                     LowerCornerStride        const& lower_corner_whd,
                     UpperCornerStride        const& upper_corner_whd,
                     LowerPaddingStride       const& lower_padding_whd,
                     UpperPaddingStride       const& upper_padding_whd,
                     TraversalStride          const& stride_whd,
                     LowerSRTStride           const& lower_srt,
                     DilationStride           const& stride_srt)
{
  auto cta_v_tile = make_identity_layout(product_each(shape(tensor_cwhdn))).compose(cta_tiler);
  auto cta_t_tile = make_layout(multicast_size);

  return detail::make_tma_copy_im2col(copy_op, tensor_cwhdn,
                                      slayout, cta_t_tile, cta_v_tile,
                                      lower_corner_whd, upper_corner_whd, lower_padding_whd, upper_padding_whd, stride_whd, lower_srt, stride_srt);
}
```
- **EN:** Defines or forwards `make_im2col_tma_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_copy`，作为该头文件可执行接口的一部分。

### Lines 834-862
```cpp
// Explicit default for multicast_size
template <class CopyOp,
          class Engine0, class Layout0,
          class SLayout,
          class CTATiler,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST_RTC
auto
make_im2col_tma_copy(CopyOp                   const& copy_op,
                     Tensor<Engine0, Layout0> const& tensor_cwhdn,
                     SLayout                  const& slayout,
                     CTATiler                 const& cta_tiler,
                     LowerCornerStride        const& lower_corner_whd,
                     UpperCornerStride        const& upper_corner_whd,
                     LowerPaddingStride       const& lower_padding_whd,
                     UpperPaddingStride       const& upper_padding_whd,
                     TraversalStride          const& stride_whd,
                     LowerSRTStride           const& lower_srt,
                     DilationStride           const& stride_srt)
{
  return make_im2col_tma_copy(copy_op, tensor_cwhdn, slayout, cta_tiler, Int<1>{},
                              lower_corner_whd, upper_corner_whd, lower_padding_whd, upper_padding_whd, stride_whd, lower_srt, stride_srt);
}
```
- **EN:** Defines or forwards `make_im2col_tma_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_copy`，作为该头文件可执行接口的一部分。

### Lines 864-890
```cpp
// Explicit default for cta_tiler and multicast_size
template <class CopyOp,
          class Engine0, class Layout0,
          class SLayout,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST_RTC
auto
make_im2col_tma_copy(CopyOp                   const& copy_op,
                     Tensor<Engine0, Layout0> const& tensor_cwhdn,
                     SLayout                  const& slayout,
                     LowerCornerStride        const& lower_corner_whd,
                     UpperCornerStride        const& upper_corner_whd,
                     LowerPaddingStride       const& lower_padding_whd,
                     UpperPaddingStride       const& upper_padding_whd,
                     TraversalStride          const& stride_whd,
                     LowerSRTStride           const& lower_srt,
                     DilationStride           const& stride_srt)
{
  return make_im2col_tma_copy(copy_op, tensor_cwhdn, slayout, product_each(shape(slayout)), Int<1>{},
                              lower_corner_whd, upper_corner_whd, lower_padding_whd, upper_padding_whd, stride_whd, lower_srt, stride_srt);
}
```
- **EN:** Defines or forwards `make_im2col_tma_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_copy`，作为该头文件可执行接口的一部分。

### Lines 892-910
```cpp
// No offsets copy.
template <class CopyOp,
          class Engine0, class Layout0,
          class SLayout,
          class CTATiler,
          class MulticastSize>
CUTE_HOST_RTC
auto
make_im2col_tma_copy(CopyOp                   const& copy_op,
                     Tensor<Engine0, Layout0> const& tensor_cwhdn,
                     SLayout                  const& slayout,
                     CTATiler                 const& cta_tiler,
                     MulticastSize            const& multicast_size)
{
  auto cta_v_tile = make_identity_layout(product_each(shape(tensor_cwhdn))).compose(cta_tiler);
  auto cta_t_tile = make_layout(multicast_size);

  return detail::make_tma_copy_im2col(copy_op, tensor_cwhdn, slayout, cta_t_tile, cta_v_tile);
}
```
- **EN:** Defines or forwards `make_im2col_tma_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_copy`，作为该头文件可执行接口的一部分。

### Lines 912-925
```cpp
// Explicit default for multicast_size
template <class CopyOp,
          class Engine0, class Layout0,
          class SLayout,
          class CTATiler>
CUTE_HOST_RTC
auto
make_im2col_tma_copy(CopyOp                   const& copy_op,
                     Tensor<Engine0, Layout0> const& tensor_cwhdn,
                     SLayout                  const& slayout,
                     CTATiler                 const& cta_tiler)
{
  return make_im2col_tma_copy(copy_op, tensor_cwhdn, slayout, cta_tiler, Int<1>{});
}
```
- **EN:** Defines or forwards `make_im2col_tma_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_copy`，作为该头文件可执行接口的一部分。

### Lines 927-938
```cpp
// Explicit default for cta_tiler and multicast_size
template <class CopyOp,
          class Engine0, class Layout0,
          class SLayout>
CUTE_HOST_RTC
auto
make_im2col_tma_copy(CopyOp                   const& copy_op,
                     Tensor<Engine0, Layout0> const& tensor_cwhdn,
                     SLayout                  const& slayout)
{
  return make_im2col_tma_copy(copy_op, tensor_cwhdn, slayout, product_each(shape(slayout)), Int<1>{});
}
```
- **EN:** Defines or forwards `make_im2col_tma_copy` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_copy`，作为该头文件可执行接口的一部分。

### Lines 940-940
```cpp
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
- **EN:** TMA-related code packages descriptors, barriers, swizzles, and coordinates for bulk memory movement.
  **CN:** TMA 相关代码会封装描述符、屏障、swizzle 与坐标，以支持批量内存搬运。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。
- **EN:** Prefetch paths try to reduce latency by staging data or metadata early.
  **CN:** 预取路径尝试通过提前准备数据或元数据来降低延迟。

## Dependencies / 依赖关系

- **EN:** `cute/arch/copy_sm90.hpp` supplies related definitions from `cute/arch/copy_sm90.hpp`.
  **CN:** `cute/arch/copy_sm90.hpp` 提供了来自 `cute/arch/copy_sm90.hpp` 的相关定义。
- **EN:** `cute/arch/copy_sm90_desc.hpp` supplies related definitions from `cute/arch/copy_sm90_desc.hpp`.
  **CN:** `cute/arch/copy_sm90_desc.hpp` 提供了来自 `cute/arch/copy_sm90_desc.hpp` 的相关定义。
- **EN:** `cute/tensor.hpp` supplies related definitions from `cute/tensor.hpp`.
  **CN:** `cute/tensor.hpp` 提供了来自 `cute/tensor.hpp` 的相关定义。
- **EN:** `cute/algorithm/prefetch.hpp` supplies higher-level prefetch helpers.
  **CN:** `cute/algorithm/prefetch.hpp` 提供了更高层的预取辅助工具。
- **EN:** `cutlass/fast_math.h` supplies related definitions from `cutlass/fast_math.h`.
  **CN:** `cutlass/fast_math.h` 提供了来自 `cutlass/fast_math.h` 的相关定义。
- **EN:** `cutlass/cuda_host_adapter.hpp` supplies CUTLASS host-side CUDA adapter utilities.
  **CN:** `cutlass/cuda_host_adapter.hpp` 提供了CUTLASS 主机侧 CUDA 适配工具。
- **EN:** CUTLASS headers provide adapter types, synchronization hooks, or shared low-level GPU infrastructure.
  **CN:** CUTLASS 头文件提供适配类型、同步钩子或共享的底层 GPU 基础设施。
