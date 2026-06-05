# copy_traits_sm100_im2col.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/atom/copy_traits_sm100_im2col.hpp`
- **EN:** Defines im2col-related copy-trait specializations for NVIDIA SM100.
- **CN:** 为 NVIDIA SM100 定义与 im2col 相关的 copy trait 特化。

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

### Lines 33-34
```cpp
/*! \file
  \brief im2col make_tma_copy
```
- **EN:** Implements a supporting fragment of the surrounding algorithm, trait, or architecture wrapper.
- **CN:** 实现周边算法、trait 或体系结构封装所需的辅助片段。

### Lines 36-41
```cpp
*/

#include "cute/arch/copy_sm90.hpp"
#include "cute/arch/copy_sm90_desc.hpp"
#include "cute/atom/copy_traits_sm90_im2col.hpp"
#include "cute/tensor.hpp"
```
- **EN:** Imports `cute/arch/copy_sm90.hpp` (related definitions from `cute/arch/copy_sm90.hpp`); `cute/arch/copy_sm90_desc.hpp` (related definitions from `cute/arch/copy_sm90_desc.hpp`); `cute/atom/copy_traits_sm90_im2col.hpp` (related definitions from `cute/atom/copy_traits_sm90_im2col.hpp`); `cute/tensor.hpp` (related definitions from `cute/tensor.hpp`).
- **CN:** 引入 `cute/arch/copy_sm90.hpp`（来自 `cute/arch/copy_sm90.hpp` 的相关定义）；`cute/arch/copy_sm90_desc.hpp`（来自 `cute/arch/copy_sm90_desc.hpp` 的相关定义）；`cute/atom/copy_traits_sm90_im2col.hpp`（来自 `cute/atom/copy_traits_sm90_im2col.hpp` 的相关定义）；`cute/tensor.hpp`（来自 `cute/tensor.hpp` 的相关定义）。

### Lines 43-43
```cpp
namespace cute {
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 45-45
```cpp
struct SM100_TMA_2SM_LOAD_IM2COL_OP : SM100_TMA_2SM_LOAD_IM2COL {};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_OP`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_OP`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 47-110
```cpp
/// @brief Non-executable specialization of Copy_Traits for SM100
///   im2col TMA load, with TMA descriptor but no barrier.
///
/// Use `.with(memory_barrier)` to construct an executable version.
template <class NumBitsPerTMA, class TMATensor>
struct Copy_Traits<SM100_TMA_2SM_LOAD_IM2COL, NumBitsPerTMA, TMATensor>
{
  using ThrID = Layout<_2>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_2, NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_2, NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
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
  /// Copy_Traits specializations with SM100_TMA_2SM_LOAD_IM2COL are not
  /// directly executable.  Instead, call this "with" member function
  /// to get an executable specialization.  "Executable" means that
  /// @c copy_unpack works.
  ///
  /// @param tma_mbar Memory barrier for synchronization
  ///
  /// @param multicast_mask Multicast mask (unused; only exists
  ///   for consistency with the actual multicast Copy_Traits
  ///   specialization)
  ///
  /// @return Executable specialization of @c Copy_Traits
  CUTE_HOST_DEVICE constexpr
  Copy_Traits<SM100_TMA_2SM_LOAD_IM2COL_OP, NumBitsPerTMA>
  with(uint64_t& tma_mbar, [[maybe_unused]] uint16_t const& multicast_mask = 0) const
  {
    return {{}, {&tma_desc_, &tma_mbar}};
  }

  // Copy_Traits specializations with SM100_TMA_2SM_LOAD_IM2COL
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

### Lines 112-130
```cpp
///   TMA load, with TMA descriptor and barrier.
template <class NumBitsPerTMA>
struct Copy_Traits<SM100_TMA_2SM_LOAD_IM2COL_OP, NumBitsPerTMA>
     : TMA_LOAD_IM2COL_Unpack<SM100_TMA_2SM_LOAD_IM2COL_OP>
{
  using ThrID = Layout<_2>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_2, NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_2, NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM100_TMA_2SM_LOAD_IM2COL arguments
  tuple<
  Im2ColTmaDescriptor const*,
  uint64_t* // smem mbarrier
  > const opargs_;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 132-136
```cpp
//////////////////////////////////////////////////////////////////////////////
///////////////////////////// TMA_LOAD_MULTICAST /////////////////////////////
//////////////////////////////////////////////////////////////////////////////

struct SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_OP : SM100_TMA_2SM_LOAD_IM2COL_MULTICAST {};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_OP`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_OP`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 138-200
```cpp
/// @brief Non-executable specialization of Copy_Traits for SM100
///   im2col TMA load, with TMA descriptor but no barrier or multicast
///   mask.
///
/// Use `.with(memory_barrier)` to construct an executable version.
template <class NumBitsPerTMA, class TMATensor>
struct Copy_Traits<SM100_TMA_2SM_LOAD_IM2COL_MULTICAST, NumBitsPerTMA, TMATensor>
{
  using ThrID = Layout<_2>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_2, NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_2, NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
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
  /// Copy_Traits specializations with SM100_TMA_2SM_LOAD_IM2COL_MULTICAST
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
  Copy_Traits<SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_OP, NumBitsPerTMA>
  with(uint64_t& tma_mbar, uint16_t const& multicast_mask) const
  {
    return {{}, {&tma_desc_, &tma_mbar, multicast_mask}};
  }

  // Copy_Traits specializations with SM100_TMA_LOAD_IM2COL_MULTICAST
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

### Lines 202-222
```cpp
/// @brief Executable specialization of Copy_Traits for SM100 multicast
///   im2col TMA load, with TMA descriptor, barrier, and multicast mask.
template <class NumBitsPerTMA>
struct Copy_Traits<SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_OP, NumBitsPerTMA>
     : TMA_LOAD_IM2COL_Unpack<SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_OP>
{
  using ThrID = Layout<_2>;
  // Map from (src-thr,src-val) to bit.
  using SrcLayout = Layout<Shape<_2, NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_2, NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM100_TMA_2SM_LOAD_IM2COL_MULTICAST arguments
  tuple<
  Im2ColTmaDescriptor const*,
  uint64_t*, // smem mbarrier
  uint16_t   // multicast mask
  > const opargs_;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 224-307
```cpp
////////////////////////////////////
// Make TMA
///////////////////////////////////

#if !defined(__CUDACC_RTC__)
/** Make a CuTe CTA-collective TiledCopy for a TMA operation.
 *
 * @param CopyOp The target copy operation: SM100_TMA_2SM_LOAD
 * @param gtensor The GMEM Tensor to be involved in the TMA.
 * @param slayout The SMEM Layout to be involved in the TMA.
 * @param cluster_tile The Cluster-local tile that each Cluster will be tiling GMEM with.
 *                     This is often the cluster_tile_shape that is used to tile the GMEM:
 *                       local_tile(gtensor, cluster_tile_shape, cluster_coord)
 *                         -> Cluster-local tile of GMEM
 * @param mma The TiledMMA that defines the Cluster-Tile to Block-Tile partitioning.
 *
 * This code attempts to maximize the TMA box size. It does this by tracing
 * the SMEM "vector" -- the inverse of the smem layout -- to find the largest
 * contiguous array of smem that can be written to/from global memory given
 * the constraints that the TMA instruction imposes.
 *
 * This is accomplished by assigning "basis" strides to the GMEM to track which
 * modes of SMEM map to which modes of GMEM, then reordering the modes of GMEM according
 * to the SMEM vector, and then using those GMEM/SMEM modes to fill in the desc.
 *
 * Examples:
 */
template <class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class Cluster_Tile,
          class... Args,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST
auto
make_im2col_tma_copy_A_sm100(CopyOp                    const& copy_op,
                             Tensor<GEngine,GLayout>   const& gtensor,        // (M,K,...)
                             SLayout                   const& slayout,        // (MMA, MMA_M, MMA_K)
                             Cluster_Tile              const& cluster_tile,   // (TILE_M,TILE_N,TILE_K)
                             TiledMMA<Args...>         const& mma,
                             LowerCornerStride         const& lower_corner_whd,
                             UpperCornerStride         const& upper_corner_whd,
                             LowerPaddingStride        const& lower_padding_whd,
                             UpperPaddingStride        const& upper_padding_whd,
                             TraversalStride           const& stride_whd,
                             LowerSRTStride            const& lower_srt,
                             DilationStride            const& stride_srt,
                             TMA::DescriptorAuxParams  const& aux_params = {})
{
  constexpr int R = GLayout::rank;
  // Keep only MK modes from MNK
  auto cluster_tile_shape = append<R>(make_shape(get<0>(cluster_tile), get<2>(cluster_tile)), Int<1>{});
  auto cluster_layout = make_identity_layout(cluster_tile_shape);
  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_A(cluster_layout))(_, repeat<R>(_));

  auto cta_t_vmnk_strides = [](){
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_IM2COL_MULTICAST> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_IM2COL_MULTICAST>) {
      return Stride<_0,_0,_1,_0>{};                    // VMNK: Use only the N-CTAs in the Multicast
    } else
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_IM2COL> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_IM2COL>) {
      return Stride<_0,_0,_0,_0>{};                    // VMNK: Use no CTAs in Non-Multicast
    } else {
      static_assert(dependent_false<CopyOp>, "Unsupported TMA");
    }
  }();

  auto cta_t_shape = shape(mma.get_thr_layout_vmnk());
  // cta rank -> logical cta idx
  auto cta_t_map  = make_layout(cta_t_shape, compact_col_major(cta_t_shape, cta_t_vmnk_strides));

  return detail::make_tma_copy_im2col(copy_op, gtensor, slayout,
                                      cta_t_map, cta_v_tile,
                                      lower_corner_whd, upper_corner_whd, lower_padding_whd, upper_padding_whd, stride_whd,
                                      lower_srt, stride_srt, aux_params);
}
```
- **EN:** Performs compile-time validation so unsupported combinations fail before code generation.
- **CN:** 执行编译期校验，使不受支持的组合在生成代码前就失败。

### Lines 309-365
```cpp
template <class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class Cluster_Tile,
          class... Args,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST
auto
make_im2col_tma_copy_B_sm100(CopyOp                    const& copy_op,
                             Tensor<GEngine,GLayout>   const& gtensor,        // (N,K,...)
                             SLayout                   const& slayout,        // (MMA, MMA_N, MMA_K)
                             Cluster_Tile              const& cluster_tile,   // (TILE_M,TILE_N,TILE_K)
                             TiledMMA<Args...>         const& mma,
                             LowerCornerStride         const& lower_corner_whd,
                             UpperCornerStride         const& upper_corner_whd,
                             LowerPaddingStride        const& lower_padding_whd,
                             UpperPaddingStride        const& upper_padding_whd,
                             TraversalStride           const& stride_whd,
                             LowerSRTStride            const& lower_srt,
                             DilationStride            const& stride_srt,
                             TMA::DescriptorAuxParams  const& aux_params = {})
{
  constexpr int R = GLayout::rank;
  // Keep only NK modes from MNK
  auto cluster_tile_shape = append<R>(make_shape(get<1>(cluster_tile), get<2>(cluster_tile)), Int<1>{});
  auto cluster_layout = make_identity_layout(cluster_tile_shape);
  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_B(cluster_layout))(_, repeat<R>(_));

  auto cta_t_vmnk_strides = [](){
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_IM2COL_MULTICAST> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_IM2COL_MULTICAST>) {
      return Stride<_0,_1,_0,_0>{};                    // VMNK: Use only the M-CTAs in the Multicast
    } else
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_IM2COL> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_IM2COL>) {
      return Stride<_0,_0,_0,_0>{};                    // VMNK: Use no CTAs in Non-Multicast
    } else {
      static_assert(dependent_false<CopyOp>, "Unsupported TMA");
    }
  }();

  auto cta_t_shape = shape(mma.get_thr_layout_vmnk());
  // cta rank -> logical cta idx
  auto cta_t_map  = make_layout(cta_t_shape, compact_col_major(cta_t_shape, cta_t_vmnk_strides));

  return detail::make_tma_copy_im2col(copy_op, gtensor, slayout,
                                      cta_t_map, cta_v_tile,
                                      lower_corner_whd, upper_corner_whd, lower_padding_whd, upper_padding_whd, stride_whd,
                                      lower_srt, stride_srt, aux_params);
}
```
- **EN:** Defines or forwards `make_im2col_tma_copy_B_sm100` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_copy_B_sm100`，作为该头文件可执行接口的一部分。

### Lines 367-427
```cpp
/////////////////////////////////////
// Experimental Make Im2col TMA Atom
/////////////////////////////////////

template <class TmaInternalType = void,
          class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class MMA_Tiler,
          class... Args,
          class ClusterShapeVMNK,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST
auto
make_im2col_tma_atom_A_sm100(CopyOp                    const& copy_op,
                             Tensor<GEngine,GLayout>   const& gtensor,           // (M, K, ...)
                             SLayout                   const& slayout,           // (MMA, MMA_M, MMA_K, ...)
                             MMA_Tiler                 const& mma_tiler,         // (TILE_M, TILE_N, TILE_K, ...)
                             TiledMMA<Args...>         const& mma,
                             ClusterShapeVMNK          const& cluster_shape,     // (CTA_V, CTA_M, CTA_N, CTA_K)
                             LowerCornerStride         const& lower_corner_whd,
                             UpperCornerStride         const& upper_corner_whd,
                             LowerPaddingStride        const& lower_padding_whd,
                             UpperPaddingStride        const& upper_padding_whd,
                             TraversalStride           const& stride_whd,
                             LowerSRTStride            const& lower_srt,
                             DilationStride            const& stride_srt,
                             TMA::DescriptorAuxParams  const& aux_params = {})
{
  constexpr int R = GLayout::rank;
  // Keep only MK modes from MNK
  auto cluster_tile_shape = append<R>(make_shape(get<0>(mma_tiler), get<2>(mma_tiler)), Int<1>{});
  auto cluster_layout = make_identity_layout(cluster_tile_shape);
  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_A(cluster_layout))(_, repeat<R>(_));

  // The size of the multicasting
  auto num_multicast = [&](){
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_IM2COL_MULTICAST> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_IM2COL_MULTICAST>) {
      return size<2>(cluster_shape);                   // VMNK: Use only the N-CTAs in the Multicast
    } else
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_IM2COL>  ||
                  is_same_v<CopyOp, SM90_TMA_STORE_IM2COL> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_IM2COL>) {
      return Int<1>{};                                 // VMNK: Use no CTAs in Non-Multicast
    } else {
      static_assert(dependent_false<CopyOp>, "Unsupported TMA");
    }
  }();

  return detail::make_tma_atom_im2col(copy_op, gtensor, slayout, num_multicast, cta_v_tile,
                                      lower_corner_whd, upper_corner_whd, lower_padding_whd, upper_padding_whd,
                                      stride_whd, lower_srt, stride_srt, aux_params);
}
```
- **EN:** Defines or forwards `make_im2col_tma_atom_A_sm100` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_atom_A_sm100`，作为该头文件可执行接口的一部分。

### Lines 429-486
```cpp
template <class TmaInternalType = void,
          class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class MMA_Tiler,
          class... Args,
          class ClusterShapeVMNK,
          class LowerCornerStride,
          class UpperCornerStride,
          class LowerPaddingStride,
          class UpperPaddingStride,
          class TraversalStride,
          class LowerSRTStride,
          class DilationStride>
CUTE_HOST
auto
make_im2col_tma_atom_B_sm100(CopyOp                    const& copy_op,
                             Tensor<GEngine,GLayout>   const& gtensor,           // (N, K, ...)
                             SLayout                   const& slayout,           // (MMA, MMA_N, MMA_K, ...)
                             MMA_Tiler                 const& mma_tiler,         // (TILE_M, TILE_N, TILE_K, ...)
                             TiledMMA<Args...>         const& mma,
                             ClusterShapeVMNK          const& cluster_shape,     // (CTA_V, CTA_M, CTA_N, CTA_K)
                             LowerCornerStride         const& lower_corner_whd,
                             UpperCornerStride         const& upper_corner_whd,
                             LowerPaddingStride        const& lower_padding_whd,
                             UpperPaddingStride        const& upper_padding_whd,
                             TraversalStride           const& stride_whd,
                             LowerSRTStride            const& lower_srt,
                             DilationStride            const& stride_srt,
                             TMA::DescriptorAuxParams  const& aux_params = {})
{
  constexpr int R = GLayout::rank;
  // Keep only NK modes from MNK
  auto cluster_tile_shape = append<R>(make_shape(get<1>(mma_tiler), get<2>(mma_tiler)), Int<1>{});
  auto cluster_layout = make_identity_layout(cluster_tile_shape);
  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_B(cluster_layout))(_, repeat<R>(_));

  // The size of the multicasting
  auto num_multicast = [&](){
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_IM2COL_MULTICAST> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_IM2COL_MULTICAST>) {
      return size<1>(cluster_shape);                   // VMNK: Use only the M-CTAs in the Multicast
    } else
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_IM2COL>  ||
                  is_same_v<CopyOp, SM90_TMA_STORE_IM2COL> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_IM2COL>) {
      return Int<1>{};                                 // VMNK: Use no CTAs in Non-Multicast
    } else {
      static_assert(dependent_false<CopyOp>, "Unsupported TMA");
    }
  }();

  return detail::make_tma_atom_im2col(copy_op, gtensor, slayout, num_multicast, cta_v_tile,
                                           lower_corner_whd, upper_corner_whd, lower_padding_whd, upper_padding_whd,
                                           stride_whd, lower_srt, stride_srt, aux_params);
}
#endif // !defined(__CUDACC_RTC__)
```
- **EN:** Defines or forwards `make_im2col_tma_atom_B_sm100` as part of this header's executable interface.
- **CN:** 定义或转发 `make_im2col_tma_atom_B_sm100`，作为该头文件可执行接口的一部分。

### Lines 488-488
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
- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。

## Dependencies / 依赖关系

- **EN:** `cute/arch/copy_sm90.hpp` supplies related definitions from `cute/arch/copy_sm90.hpp`.
  **CN:** `cute/arch/copy_sm90.hpp` 提供了来自 `cute/arch/copy_sm90.hpp` 的相关定义。
- **EN:** `cute/arch/copy_sm90_desc.hpp` supplies related definitions from `cute/arch/copy_sm90_desc.hpp`.
  **CN:** `cute/arch/copy_sm90_desc.hpp` 提供了来自 `cute/arch/copy_sm90_desc.hpp` 的相关定义。
- **EN:** `cute/atom/copy_traits_sm90_im2col.hpp` supplies related definitions from `cute/atom/copy_traits_sm90_im2col.hpp`.
  **CN:** `cute/atom/copy_traits_sm90_im2col.hpp` 提供了来自 `cute/atom/copy_traits_sm90_im2col.hpp` 的相关定义。
- **EN:** `cute/tensor.hpp` supplies related definitions from `cute/tensor.hpp`.
  **CN:** `cute/tensor.hpp` 提供了来自 `cute/tensor.hpp` 的相关定义。
