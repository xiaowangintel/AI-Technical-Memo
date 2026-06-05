# copy_traits_sm100_tma.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/atom/copy_traits_sm100_tma.hpp`
- **EN:** Defines TMA-oriented copy-trait specializations for NVIDIA SM100.
- **CN:** 为 NVIDIA SM100 定义面向 TMA 的 copy trait 特化。

## Line-by-Line Analysis / 逐行分析

### Lines 1-34
```cpp
/***************************************************************************************************
 * Copyright (c) 2021 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 36-38
```cpp
#if !defined(__CUDACC_RTC__) && !defined(CUTLASS_ENABLE_SYCL)
#include <cuda.h>
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ENABLE_SYCL) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ENABLE_SYCL）只启用当前目标有效的构建路径。

### Lines 40-43
```cpp
#include <cute/tensor.hpp>
#include <cute/atom/copy_traits_sm90_tma.hpp>
#include <cute/arch/copy_sm100_tma.hpp>
#include <cute/atom/copy_traits.hpp>
```
- **EN:** Imports `cute/tensor.hpp` (related definitions from `cute/tensor.hpp`); `cute/atom/copy_traits_sm90_tma.hpp` (related definitions from `cute/atom/copy_traits_sm90_tma.hpp`); `cute/arch/copy_sm100_tma.hpp` (related definitions from `cute/arch/copy_sm100_tma.hpp`); `cute/atom/copy_traits.hpp` (generic copy-trait interfaces and utilities).
- **CN:** 引入 `cute/tensor.hpp`（来自 `cute/tensor.hpp` 的相关定义）；`cute/atom/copy_traits_sm90_tma.hpp`（来自 `cute/atom/copy_traits_sm90_tma.hpp` 的相关定义）；`cute/arch/copy_sm100_tma.hpp`（来自 `cute/arch/copy_sm100_tma.hpp` 的相关定义）；`cute/atom/copy_traits.hpp`（通用 copy trait 接口与工具）。

### Lines 45-46
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 48-52
```cpp
//////////////////////////////////////////////////////////////////////////////
////////////////////////////// TMA_LOAD ////////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////

struct SM100_TMA_2SM_LOAD_OP : SM100_TMA_2SM_LOAD {};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_OP`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_OP`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 54-117
```cpp
// The non-executable SM100_TMA_2SM_LOAD with tma_desc and no tma_mbar
// Use .with(tma_mbar) to construct an executable version
template <class NumBitsPerTMA, class AuxParams_>
struct Copy_Traits<SM100_TMA_2SM_LOAD, NumBitsPerTMA, AuxParams_>
{
  using ThrID     = Layout<_2>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_2,NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_2,NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM100_TMA_2SM_LOAD arguments
  TmaDescriptor tma_desc_;
  using AuxParams = AuxParams_;
  AuxParams aux_params_;

  // Return TmaDescriptor/TensorMap
  CUTE_HOST_DEVICE constexpr
  TmaDescriptor const*
  get_tma_descriptor() const {
    return &tma_desc_;
  }

  // Construct an executable SM100_TMA_2SM_LOAD with tma_mbar
  CUTE_HOST_DEVICE constexpr
  Copy_Traits<SM100_TMA_2SM_LOAD_OP, NumBitsPerTMA>
  with(
    uint64_t& tma_mbar,
    [[maybe_unused]] uint16_t const& multicast_mask = 0,
    TMA::CacheHintSm100 const& cache_hint = TMA::CacheHintSm100::EVICT_NORMAL) const {
    // We accept multicast_mask here to keep the API for both atoms consistent
    return {{}, {&tma_desc_, &tma_mbar, static_cast<uint64_t>(cache_hint)}};
  }

  // Construct an executable SM100_TMA_2SM_LOAD with tma_mbar (temp. overloaded for grouped gemm/ptr array gemm)
  CUTE_HOST_DEVICE constexpr
  Copy_Traits<SM100_TMA_2SM_LOAD_OP, NumBitsPerTMA>
  with(
    TmaDescriptor const* new_tma_desc,
    uint64_t& tma_mbar,
    [[maybe_unused]] uint16_t const& multicast_mask = 0,
    TMA::CacheHintSm100 const& cache_hint = TMA::CacheHintSm100::EVICT_NORMAL) const {
    // We accept multicast_mask here to keep the API for both atoms consistent
    return {{}, {new_tma_desc, &tma_mbar, static_cast<uint64_t>(cache_hint)}};
  }

  template <class GShape>
  CUTE_HOST_DEVICE constexpr
  auto
  get_tma_tensor(GShape const& g_shape) const {
    static_assert(is_congruent<decltype(g_shape), decltype(aux_params_.g_stride_)>::value);
    return make_coord_tensor(make_layout(g_shape, aux_params_.g_stride_));
  }

  // Don't try to execute a copy with SM100_TMA_2SM_LOAD before calling .with()
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

### Lines 119-145
```cpp
// The executable SM100_TMA_2SM_LOAD with tma_desc and tma_mbar
template <class NumBitsPerTMA>
struct Copy_Traits<SM100_TMA_2SM_LOAD_OP, NumBitsPerTMA>
     : TMA_LOAD_Unpack<SM100_TMA_2SM_LOAD_OP, NumBitsPerTMA>
{
  using ThrID     = Layout<_2>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_2,NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_2,NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM100_TMA_2SM_LOAD arguments
  tuple<
  TmaDescriptor const*,
  uint64_t*, // smem mbarrier
  uint64_t   // cache hint
  > const opargs_;

  // Return TmaDescriptor/TensorMap
  CUTE_HOST_DEVICE constexpr
  TmaDescriptor const*
  get_tma_descriptor() const {
    return get<0>(opargs_);
  }
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 147-151
```cpp
//////////////////////////////////////////////////////////////////////////////
///////////////////////////// TMA_LOAD_MULTICAST /////////////////////////////
//////////////////////////////////////////////////////////////////////////////

struct SM100_TMA_2SM_LOAD_MULTICAST_OP : SM100_TMA_2SM_LOAD_MULTICAST {};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_MULTICAST_OP`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_MULTICAST_OP`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 153-212
```cpp
template <class NumBitsPerTMA, class AuxParams_>
struct Copy_Traits<SM100_TMA_2SM_LOAD_MULTICAST, NumBitsPerTMA, AuxParams_>
{
  using ThrID     = Layout<_2>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_2,NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_2,NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM100_TMA_2SM_LOAD_MULTICAST_OP arguments
  TmaDescriptor tma_desc_;
  using AuxParams = AuxParams_;
  AuxParams aux_params_;

  // Return TmaDescriptor/TensorMap
  CUTE_HOST_DEVICE constexpr
  TmaDescriptor const*
  get_tma_descriptor() const {
    return &tma_desc_;
  }

  // Construct an executable SM100_TMA_2SM_LOAD_MULTICAST_OP with tma_mbar
  CUTE_HOST_DEVICE constexpr
  Copy_Traits<SM100_TMA_2SM_LOAD_MULTICAST_OP, NumBitsPerTMA>
  with(
    uint64_t& tma_load_mbar,
    uint16_t const& multicast_mask,
    TMA::CacheHintSm100 const& cache_hint = TMA::CacheHintSm100::EVICT_NORMAL) const {
    return {{}, {&tma_desc_, &tma_load_mbar, multicast_mask, static_cast<uint64_t>(cache_hint)}};
  }

  // Construct an executable SM100_TMA_2SM_LOAD_MULTICAST_OP with tma_mbar (temp. overloaded for grouped gemm/ptr array gemm)
  CUTE_HOST_DEVICE constexpr
  Copy_Traits<SM100_TMA_2SM_LOAD_MULTICAST_OP, NumBitsPerTMA>
  with(
    TmaDescriptor const* new_tma_desc,
    uint64_t& tma_load_mbar,
    uint16_t const& multicast_mask,
    TMA::CacheHintSm100 const& cache_hint = TMA::CacheHintSm100::EVICT_NORMAL) const {
    return {{}, {new_tma_desc, &tma_load_mbar, multicast_mask, static_cast<uint64_t>(cache_hint)}};
  }

  template <class GShape>
  CUTE_HOST_DEVICE constexpr
  auto
  get_tma_tensor(GShape const& g_shape) const {
    static_assert(is_congruent<decltype(g_shape), decltype(aux_params_.g_stride_)>::value);
    return make_coord_tensor(make_layout(g_shape, aux_params_.g_stride_));
  }

  // Don't try to execute a copy with SM100_TMA_2SM_LOAD_MULTICAST_OP before calling .with()
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

### Lines 214-240
```cpp
template <class NumBitsPerTMA>
struct Copy_Traits<SM100_TMA_2SM_LOAD_MULTICAST_OP, NumBitsPerTMA>
     : TMA_LOAD_Unpack<SM100_TMA_2SM_LOAD_MULTICAST_OP, NumBitsPerTMA>
{
  using ThrID     = Layout<_2>;
  // Map from (src-thr,src-val) to bit
  using SrcLayout = Layout<Shape<_2,NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = Layout<Shape<_2,NumBitsPerTMA>, Stride<NumBitsPerTMA,_1>>;
  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;

  // SM100_TMA_2SM_LOAD_MULTICAST_OP arguments
  tuple<
  TmaDescriptor const*,
  uint64_t*, // smem mbarrier
  uint16_t,  // multicast mask
  uint64_t   // cache hint
  > const opargs_;

  // Return TmaDescriptor/TensorMap
  CUTE_HOST_DEVICE constexpr
  TmaDescriptor const*
  get_tma_descriptor() const {
    return get<0>(opargs_);
  }
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 242-311
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
template <class TmaInternalType = void,
          class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class Cluster_Tiler,
          class... Args>
CUTE_HOST
auto
make_tma_copy_A_sm100(CopyOp                  const& copy_op,
                      Tensor<GEngine,GLayout> const& gtensor,        // (M, K, ...)
                      SLayout                 const& slayout,        // (MMA, MMA_M, MMA_K, ...)
                      Cluster_Tiler           const& cluster_tiler,  // (TILER_M, TILER_N, TILER_K, ...)
                      TiledMMA<Args...>       const& mma)
{
  // Keep only MK modes from MNK
  auto cluster_tiler_mk = remove<1>(cluster_tiler);
  // cluster tile coord -> gtensor coord
  auto g_tile = make_identity_layout(shape(gtensor)).compose(cluster_tiler_mk);     // (TILE_M, TILE_K, ...)
  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_A(g_tile))(_, repeat<rank(g_tile)>(_));    // (MMA, MMA_M, MMA_K, ...)

  auto cta_t_vmnk_strides = [](){
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_MULTICAST> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_MULTICAST>) {
      return Stride<_0,_0,_1,_0>{};                    // VMNK: Use only the N-CTAs in the Multicast
    } else
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD>  ||
                  is_same_v<CopyOp, SM90_TMA_STORE> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD>) {
      return Stride<_0,_0,_0,_0>{};                    // VMNK: Use no CTAs in Non-Multicast
    } else {
      static_assert(dependent_false<CopyOp>, "Unsupported TMA");
    }
  }();

  auto cta_t_shape = shape(mma.get_thr_layout_vmnk());
  // cta rank -> logical cta idx
  auto cta_t_map  = coalesce(make_layout(cta_t_shape, compact_col_major(cta_t_shape, cta_t_vmnk_strides)));

  // Prefer TmaInternalType if specified. Fallback to GEngine::value_type
  using TmaType = conditional_t<is_same<void, TmaInternalType>::value, typename GEngine::value_type, TmaInternalType>;
  return detail::make_tma_copy_tiled<TmaType>(copy_op, gtensor, slayout, cta_t_map, cta_v_tile);
}
```
- **EN:** Defines or forwards `make_tma_copy_A_sm100` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tma_copy_A_sm100`，作为该头文件可执行接口的一部分。

### Lines 313-355
```cpp
template <class TmaInternalType = void,
          class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class Cluster_Tiler,
          class... Args>
CUTE_HOST
auto
make_tma_copy_B_sm100(CopyOp                  const& copy_op,
                      Tensor<GEngine,GLayout> const& gtensor,        // (N, K, ...)
                      SLayout                 const& slayout,        // (MMA, MMA_N, MMA_K, ...)
                      Cluster_Tiler           const& cluster_tiler,  // (TILE_M, TILE_N, TILE_K, ...)
                      TiledMMA<Args...>       const& mma)
{
  // Keep only NK modes from MNK
  auto cluster_tiler_nk = remove<0>(cluster_tiler);
  // cluster tile coord -> gtensor coord
  auto g_tile = make_identity_layout(shape(gtensor)).compose(cluster_tiler_nk);     // (TILE_N, TILE_K, ...)
  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_B(g_tile))(_, repeat<rank(g_tile)>(_));    // (MMA, MMA_N, MMA_K, ...)

  auto cta_t_vmnk_strides = [](){
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_MULTICAST> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_MULTICAST>) {
      return Stride<_0,_1,_0,_0>{};                    // VMNK: Use only the M-CTAs in the Multicast
    } else
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD>  ||
                  is_same_v<CopyOp, SM90_TMA_STORE> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD>) {
      return Stride<_0,_0,_0,_0>{};                    // VMNK: Use no CTAs in Non-Multicast
    } else {
      static_assert(dependent_false<CopyOp>, "Unsupported TMA");
    }
  }();

  auto cta_t_shape = shape(mma.get_thr_layout_vmnk());
  // cta rank -> logical cta idx
  auto cta_t_map  = coalesce(make_layout(cta_t_shape, compact_col_major(cta_t_shape, cta_t_vmnk_strides)));

  // Prefer TmaInternalType if specified. Fallback to GEngine::value_type
  using TmaType = conditional_t<is_same<void, TmaInternalType>::value, typename GEngine::value_type, TmaInternalType>;
  return detail::make_tma_copy_tiled<TmaType>(copy_op, gtensor, slayout, cta_t_map, cta_v_tile);
}
```
- **EN:** Defines or forwards `make_tma_copy_B_sm100` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tma_copy_B_sm100`，作为该头文件可执行接口的一部分。

### Lines 357-389
```cpp
template <class TmaInternalType = void,
          class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class Cluster_Tiler,
          class... Args>
CUTE_HOST
auto
make_tma_copy_C_sm100(CopyOp                  const& copy_op,
                      Tensor<GEngine,GLayout> const& gtensor,        // (M, N, ...)
                      SLayout                 const& slayout,        // (MMA, MMA_M, MMA_N, ...)
                      Cluster_Tiler           const& cluster_tiler,  // (TILE_M, TILE_N, TILE_K, ...)
                      TiledMMA<Args...>       const& mma)
{
  // Keep only MN modes from MNK
  auto cluster_tiler_mn = remove<2>(cluster_tiler);
  // cluster tile coord -> gtensor coord
  auto g_tile = make_identity_layout(shape(gtensor)).compose(cluster_tiler_mn);     // (TILE_M, TILE_N, ...)
  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_C(g_tile))(_, repeat<rank(g_tile)>(_));    // (MMA, MMA_M, MMA_N, ...)

  static_assert(is_same_v<CopyOp, SM90_TMA_LOAD>  ||
                is_same_v<CopyOp, SM90_TMA_STORE> ||
                is_same_v<CopyOp, SM100_TMA_2SM_LOAD>,
                "Unsupported TMA Op, expected a non-multicast TMA");

  // No multicast, so only 1 CTA involved
  auto cta_t_map = Layout<_1,_0>{};

  // Prefer TmaInternalType if specified. Fallback to GEngine::value_type
  using TmaType = conditional_t<is_same<void, TmaInternalType>::value, typename GEngine::value_type, TmaInternalType>;
  return detail::make_tma_copy_tiled<TmaType>(copy_op, gtensor, slayout, cta_t_map, cta_v_tile);
}
```
- **EN:** Defines or forwards `make_tma_copy_C_sm100` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tma_copy_C_sm100`，作为该头文件可执行接口的一部分。

### Lines 391-446
```cpp
////////////////////////////////////
// Experimental Make TMA Atom
///////////////////////////////////

template <class TmaInternalType = void,
          class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class MMA_Tiler,
          class... Args,
          class ClusterShapeVMNK>
CUTE_HOST
auto
make_tma_atom_A_sm100(CopyOp                  const& copy_op,
                      Tensor<GEngine,GLayout> const& gtensor,        // (M, K, ...)
                      SLayout                 const& slayout,        // (MMA, MMA_M, MMA_K, ...)
                      MMA_Tiler               const& mma_tiler,      // (TILE_M, TILE_N, TILE_K, ...)
                      TiledMMA<Args...>       const& mma,
                      ClusterShapeVMNK        const& cluster_shape)  // (CTA_V, CTA_M, CTA_N, CTA_K)
{
  // Keep only MK modes from MNK
  auto mma_tiler_mk = remove<1>(mma_tiler);

  // cluster tile coord -> gtensor coord
  auto g_tile = make_identity_layout(shape(gtensor)).compose(mma_tiler_mk);         // (TILE_M, TILE_K, ...)

  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_A(g_tile))(_, repeat<rank(g_tile)>(_));    // (MMA, MMA_M, MMA_K, ...)

#if 0
  print("(tma_a) slayout:      "); print(slayout);      print("\n");
  print("(tma_a) mma_tiler_nk: "); print(mma_tiler_nk); print("\n");
  print("(tma_a) g_tile:       "); print(g_tile);       print("\n");
  print("(tma_a) mma_tiler:    "); print(mma_tiler);    print("\n");
  print("(tma_a) cta_v_tile:   "); print(cta_v_tile);   print("\n");
#endif

  // The size of the multicasting
  auto num_multicast = [&](){
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_MULTICAST> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_MULTICAST>) {
      return size<2>(cluster_shape);                   // VMNK: Use only the N-CTAs in the Multicast
    } else
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD>  ||
                  is_same_v<CopyOp, SM90_TMA_STORE> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD>) {
      return Int<1>{};                                 // VMNK: Use no CTAs in Non-Multicast
    } else {
      static_assert(dependent_false<CopyOp>, "Unsupported TMA");
    }
  }();

  // Prefer TmaInternalType if specified. Fallback to GEngine::value_type
  using TmaType = conditional_t<is_same<void, TmaInternalType>::value, typename GEngine::value_type, TmaInternalType>;
  return detail::make_tma_copy_atom<TmaType>(copy_op, gtensor, slayout, num_multicast, cta_v_tile);
}
```
- **EN:** Defines or forwards `make_tma_atom_A_sm100` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tma_atom_A_sm100`，作为该头文件可执行接口的一部分。

### Lines 448-497
```cpp
template <class TmaInternalType = void,
          class CopyOp,
          class GEngine, class GLayout,
          class SLayout,
          class MMA_Tiler,
          class... Args,
          class ClusterShapeVMNK>
CUTE_HOST
auto
make_tma_atom_B_sm100(CopyOp                  const& copy_op,
                      Tensor<GEngine,GLayout> const& gtensor,        // (N, K, ...)
                      SLayout                 const& slayout,        // (MMA, MMA_N, MMA_K, ...)
                      MMA_Tiler               const& mma_tiler,      // (TILE_M, TILE_N, TILE_K, ...)
                      TiledMMA<Args...>       const& mma,
                      ClusterShapeVMNK        const& cluster_shape)  // (CTA_V, CTA_M, CTA_N, CTA_K)
{
  // Keep only NK modes from MNK
  auto mma_tiler_nk = remove<0>(mma_tiler);
  // cluster tile coord -> gtensor coord
  auto g_tile = make_identity_layout(shape(gtensor)).compose(mma_tiler_nk);         // (TILE_N, TILE_K, ...)
  // cta val idx -> gmem mode
  auto cta_v_tile = layout<1>(mma.thrfrg_B(g_tile))(_, repeat<rank(g_tile)>(_));    // (MMA, MMA_N, MMA_K, ...)

#if 0
  print("(tma_b) slayout:      "); print(slayout);      print("\n");
  print("(tma_b) mma_tiler_nk: "); print(mma_tiler_nk); print("\n");
  print("(tma_b) g_tile:       "); print(g_tile);       print("\n");
  print("(tma_b) mma_tiler:    "); print(mma_tiler);    print("\n");
  print("(tma_b) cta_v_tile:   "); print(cta_v_tile);   print("\n");
#endif

  // The size of the multicasting
  auto num_multicast = [&](){
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD_MULTICAST> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD_MULTICAST>) {
      return size<1>(cluster_shape);                   // VMNK: Use only the M-CTAs in the Multicast
    } else
    if constexpr (is_same_v<CopyOp, SM90_TMA_LOAD>  ||
                  is_same_v<CopyOp, SM90_TMA_STORE> ||
                  is_same_v<CopyOp, SM100_TMA_2SM_LOAD>) {
      return Int<1>{};                                 // VMNK: Use no CTAs in Non-Multicast
    } else {
      static_assert(dependent_false<CopyOp>, "Unsupported TMA");
    }
  }();

  // Prefer TmaInternalType if specified. Fallback to GEngine::value_type
  using TmaType = conditional_t<is_same<void, TmaInternalType>::value, typename GEngine::value_type, TmaInternalType>;
  return detail::make_tma_copy_atom<TmaType>(copy_op, gtensor, slayout, num_multicast, cta_v_tile);
}
```
- **EN:** Defines or forwards `make_tma_atom_B_sm100` as part of this header's executable interface.
- **CN:** 定义或转发 `make_tma_atom_B_sm100`，作为该头文件可执行接口的一部分。

### Lines 499-499
```cpp
#endif // !defined(__CUDACC_RTC__)
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 501-501
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

- **EN:** `cuda.h` supplies CUDA driver/runtime declarations used by low-level backends.
  **CN:** `cuda.h` 提供了底层后端使用的 CUDA 驱动/运行时声明。
- **EN:** `cute/tensor.hpp` supplies related definitions from `cute/tensor.hpp`.
  **CN:** `cute/tensor.hpp` 提供了来自 `cute/tensor.hpp` 的相关定义。
- **EN:** `cute/atom/copy_traits_sm90_tma.hpp` supplies related definitions from `cute/atom/copy_traits_sm90_tma.hpp`.
  **CN:** `cute/atom/copy_traits_sm90_tma.hpp` 提供了来自 `cute/atom/copy_traits_sm90_tma.hpp` 的相关定义。
- **EN:** `cute/arch/copy_sm100_tma.hpp` supplies related definitions from `cute/arch/copy_sm100_tma.hpp`.
  **CN:** `cute/arch/copy_sm100_tma.hpp` 提供了来自 `cute/arch/copy_sm100_tma.hpp` 的相关定义。
- **EN:** `cute/atom/copy_traits.hpp` supplies generic copy-trait interfaces and utilities.
  **CN:** `cute/atom/copy_traits.hpp` 提供了通用 copy trait 接口与工具。
- **EN:** SYCL/SPIR-V feature macros select alternate code paths for Intel/Xe-style backends.
  **CN:** SYCL/SPIR-V 特性宏会为 Intel/Xe 风格后端选择替代代码路径。
