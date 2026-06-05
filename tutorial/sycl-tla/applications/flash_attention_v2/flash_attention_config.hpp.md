# flash_attention_config.hpp — Code Analysis / 代码分析

## Source / 来源

- **Requested Path / 请求路径:** `applications/flash_attention_v2/flash_attention_config.hpp`
- **Analyzed Source / 实际分析源码:** `benchmarks/flash_attention/fmha_configuration.hpp`
- **Purpose / 用途:** Compile-time FlashAttention configuration aliases for datatype, tile shape, scheduling, and mode selection.
- **Note / 说明:** The requested config header does not exist in this checkout; the benchmark configuration header now carries the same role.

## Line-by-Line Analysis / 逐行分析

### Lines 1-31

```cpp
/***************************************************************************************************
 * Copyright (c) 2026 Intel Corporation. All rights reserved.
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
- **EN:** Provides the license header and ownership notice for this source file.
- **CN:** 给出该源文件的许可证头和版权归属说明。

### Lines 33-33

```cpp
#include <type_traits>
```
- **EN:** Imports dependencies such as `type_traits` so this file can reuse CUTLASS/CUTE, benchmark, or FlashAttention helpers.
- **CN:** 引入依赖头文件，例如 `type_traits`，使本文件能够复用 CUTLASS/CUTE、benchmark 或 FlashAttention 辅助组件。

### Lines 35-38

```cpp
#include "cutlass/gemm/dispatch_policy.hpp"
#include "flash_attention_v2/collective/fmha_fusion.hpp"
#include "flash_attention_v2/kernel/xe_fmha_fwd_kernel.hpp"
#include "flash_attention_v2/kernel/xe_tile_scheduler.hpp"
```
- **EN:** Imports dependencies such as `cutlass/gemm/dispatch_policy.hpp`, `flash_attention_v2/collective/fmha_fusion.hpp`, `flash_attention_v2/kernel/xe_fmha_fwd_kernel.hpp`, `flash_attention_v2/kernel/xe_tile_scheduler.hpp` so this file can reuse CUTLASS/CUTE, benchmark, or FlashAttention helpers.
- **CN:** 引入依赖头文件，例如 `cutlass/gemm/dispatch_policy.hpp`, `flash_attention_v2/collective/fmha_fusion.hpp`, `flash_attention_v2/kernel/xe_fmha_fwd_kernel.hpp`, `flash_attention_v2/kernel/xe_tile_scheduler.hpp`，使本文件能够复用 CUTLASS/CUTE、benchmark 或 FlashAttention 辅助组件。

### Lines 40-41

```cpp
namespace cutlass {
namespace flash_attention {
```
- **EN:** Opens or closes namespaces so the declarations live in the intended CUTLASS, CUTE, or benchmark scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS、CUTE 或 benchmark 作用域中。

### Lines 43-62

```cpp
template <typename ElementQ, typename ElementK, typename ElementV, typename ElementO,
          typename LayoutQ_, typename LayoutK_, typename LayoutV_, typename LayoutO_,
          typename TileShapeQK, typename TileShapePV, typename TileShapeOutput, 
          typename SubgroupLayoutQK, typename SubgroupLayoutPV_,
          bool Causal_, bool VarLen_, bool CachedKV_, bool PagedKV_, bool Persistent_,
          int PipelineStages,
          typename GmemTiledCopyQ = void, 
          typename GmemTiledCopyK = void, 
          typename GmemTiledCopyV = void, 
          typename GmemTiledCopyO = void,
          typename MMAOperation_ = void,
          typename StrideQ = Stride<int, _1, int, int>, 
          typename StrideK = Stride<int, _1, int, int>,
          typename StrideV = Stride<_1, int, int, int>, 
          typename StrideO = Stride<int, _1, int, int>>          
struct FMHAConfig {
  using LayoutQ = LayoutQ_;
  using LayoutK = LayoutK_;
  using LayoutV = LayoutV_;
  using LayoutO = LayoutO_;
```
- **EN:** Defines templated type `FMHAConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `FMHAConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 64-69

```cpp
  static constexpr bool Causal = Causal_;
  static constexpr bool VarLen = VarLen_;
  static constexpr bool CachedKV = CachedKV_;
  static constexpr bool PagedKV = PagedKV_;
  static constexpr bool Persistent = Persistent_;
  static_assert(!(Persistent & Causal), "persistent SDPA kernel not support Causal yet");
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 71-71

```cpp
  static constexpr int SGTileQ = get<0>(shape_div(TileShapeQK{}, shape(SubgroupLayoutQK{})))();
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 73-74

```cpp
  using DefaultMMA = XE_DPAS_TT<cute::gcd(SGTileQ, 8), float, ElementQ>;
  using MMAOperationPV = XE_DPAS_TT<cute::gcd(SGTileQ, 8), float, ElementV>;
```
- **EN:** Defines aliases such as `DefaultMMA`, `MMAOperationPV` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `DefaultMMA`, `MMAOperationPV`，用于简化冗长的模板表达式或命名空间限定。

### Lines 76-78

```cpp
  using MMAOperation = cute::conditional_t<is_void_v<MMAOperation_>,
                                           DefaultMMA,
                                           MMAOperation_>;
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 80-82

```cpp
  using SubgroupLayoutPV = cute::conditional_t<is_void_v<SubgroupLayoutPV_>,
                                               decltype(cutlass::fmha::collective::get_sg_layout_pv(SubgroupLayoutQK{})),
                                               SubgroupLayoutPV_>;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 84-88

```cpp
  using TiledMMAQK = typename TiledMMAHelper<MMA_Atom<MMAOperation>, Layout<TileShapeQK>, SubgroupLayoutQK>::TiledMMA;
  using TiledMMAPV = typename TiledMMAHelper<MMA_Atom<MMAOperationPV>, Layout<TileShapePV>, SubgroupLayoutPV>::TiledMMA;
  static_assert(get<0>(TileShapeOutput{}) == get<0>(TileShapePV{}),
      "Output tile and P*V tile have different sizes in Q dimension");    
  static constexpr int VTiles = get<1>(TileShapeOutput{}) / get<1>(TileShapePV{});
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 90-94

```cpp
  template <typename ElementType, typename Stride>
  static constexpr auto make_dummy_tensor(ElementType val, Stride stride) {
    return make_tensor(make_gmem_ptr(&val),
                        make_layout(repeat<rank_v<decltype(stride)>>(1), stride));
  };
```
- **EN:** Builds a lightweight tensor object used only to propagate layout/type information through templates.
- **CN:** 构造轻量级张量对象，仅用于在模板中传播布局和类型信息。

### Lines 96-103

```cpp
  using TensorQ = decltype(make_dummy_tensor(ElementQ{}, StrideQ{}));
  using TensorK = decltype(make_dummy_tensor(ElementK{}, StrideK{}));
  using TensorV = decltype(make_dummy_tensor(ElementV{}, StrideV{}));
  using TensorO = decltype(make_dummy_tensor(ElementO{}, StrideO{}));
  using TensorK_cache = TensorK;
  using TensorV_cache = TensorV;
  using GmemTiledCopyK_cache = GmemTiledCopyK;
  using GmemTiledCopyV_cache = GmemTiledCopyV;
```
- **EN:** Defines aliases such as `TensorQ`, `TensorK`, `TensorV`, `TensorO`, `TensorK_cache`, `TensorV_cache`, ... (+2) to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `TensorQ`, `TensorK`, `TensorV`, `TensorO`, `TensorK_cache`, `TensorV_cache`, ... (+2)，用于简化冗长的模板表达式或命名空间限定。

### Lines 105-105

```cpp
  using ProblemShapeType = cutlass::fmha::kernel::FMHAProblemShape<VarLen>;
```
- **EN:** Defines aliases such as `ProblemShapeType` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `ProblemShapeType`，用于简化冗长的模板表达式或命名空间限定。

### Lines 108-117

```cpp
  // Mainloop
  using MainloopDispatchPolicy = cutlass::fmha::XeDefault<PipelineStages>;
  using CollectiveMainloop = cutlass::fmha::collective::FMHAFwdMainloop<
      MainloopDispatchPolicy, Causal, CachedKV, PagedKV,
      TiledMMAQK, TiledMMAPV, VTiles,
      TensorQ, TensorK, TensorV,
      TensorK_cache, TensorV_cache,
      GmemTiledCopyQ, GmemTiledCopyK, GmemTiledCopyV,
      GmemTiledCopyK_cache, GmemTiledCopyV_cache
  >;
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 119-124

```cpp
  using CollectiveEpilogue = cutlass::fmha::collective::FMHAFwdEpilogue<
    CollectiveMainloop,
    TileShapeOutput,
    TensorO,
    GmemTiledCopyO
  >;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 126-136

```cpp
  using Scheduler = cute::conditional_t<Persistent,
      cutlass::fmha::kernel::XeFHMAIndividualPersistentTileScheduler,
      cutlass::fmha::kernel::XeFHMAIndividualTileScheduler
  >;
  using FMHAKernel = cute::conditional_t<Persistent,
      cutlass::fmha::kernel::XeFMHAFwdDynamicSplitKernel<
        ProblemShapeType, CollectiveMainloop, CollectiveEpilogue, Scheduler>,
      cutlass::fmha::kernel::XeFMHAFwdKernel<
        ProblemShapeType, CollectiveMainloop, CollectiveEpilogue, Scheduler>
  >;
};
```
- **EN:** Describes scheduling logic that maps FlashAttention tiles onto work-groups or subgroups.
- **CN:** 描述将 FlashAttention tile 映射到工作组或子组的调度逻辑。

### Lines 138-138

```cpp
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 140-140

```cpp
enum class FMHAMode { Decode, Prefill };
```
- **EN:** Declares enumeration `FMHAMode` to encode compile-time mode or policy choices.
- **CN:** 声明枚举 `FMHAMode`，用于编码编译期模式或策略选择。

### Lines 142-143

```cpp
template <FMHAMode Mode, int HeadDim, bool Persistent>
struct ShapeConfig;
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 145-152

```cpp
// Prefill configs
template <bool Persistent>
struct ShapeConfig<FMHAMode::Prefill, 16, Persistent> {
  using ShapeQK = Shape<_16, _16, _32>;
  using ShapePV = Shape<_16, _32, _16>;
  using ShapeOutput = Shape<_16, _16>;
  using SubgroupLayout = Layout<Shape<_1, _1, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 154-160

```cpp
template <bool Persistent>
struct ShapeConfig<FMHAMode::Prefill, 64, Persistent> {
  using ShapeQK = Shape<_128, _64, _32>;
  using ShapePV = Shape<_128, _32, _64>;
  using ShapeOutput = Shape<_128, _64>;
  using SubgroupLayout = Layout<Shape<_8, _1, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 162-168

```cpp
template <bool Persistent>
struct ShapeConfig<FMHAMode::Prefill, 96, Persistent> {
  using ShapeQK = Shape<_128, _64, _32>;
  using ShapePV = Shape<_128, _32, _64>;
  using ShapeOutput = Shape<_128, _96>;
  using SubgroupLayout = Layout<Shape<_8, _1, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 170-176

```cpp
template <bool Persistent>
struct ShapeConfig<FMHAMode::Prefill, 128, Persistent> {
  using ShapeQK = Shape<_128, _64, _32>;
  using ShapePV = Shape<_128, _32, _64>;
  using ShapeOutput = Shape<_128, _128>;
  using SubgroupLayout = Layout<Shape<_16, _1, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 178-184

```cpp
template <bool Persistent>
struct ShapeConfig<FMHAMode::Prefill, 192, Persistent> {
  using ShapeQK = Shape<_256, _64, _32>;
  using ShapePV = Shape<_256, _32, _64>;
  using ShapeOutput = Shape<_256, _192>;
  using SubgroupLayout = Layout<Shape<_16, _1, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 186-193

```cpp
// Decode configs
template <bool Persistent>
struct ShapeConfig<FMHAMode::Decode, 16, Persistent> {
  using ShapeQK = Shape<_1, _16, _16>;
  using ShapePV = Shape<_1, _16, _16>;
  using ShapeOutput = Shape<_1, _16>;
  using SubgroupLayout = Layout<Shape<_1, _2, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 195-203

```cpp
template <bool Persistent>
struct ShapeConfig<FMHAMode::Decode, 64, Persistent> {
  using num_sg = cute::conditional_t<Persistent, _16, _8>;
  using kv_tile_size = cute::conditional_t<Persistent, _256, _512>;
  using ShapeQK = Shape<_1, kv_tile_size, _64>;
  using ShapePV = Shape<_1, _32, kv_tile_size>;
  using ShapeOutput = Shape<_1, _64>;
  using SubgroupLayout = Layout<Shape<_1, num_sg, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 205-213

```cpp
template <bool Persistent>
struct ShapeConfig<FMHAMode::Decode, 96, Persistent> {
  using num_sg = cute::conditional_t<Persistent, _16, _8>;
  using kv_tile_size = cute::conditional_t<Persistent, _256, _512>;
  using ShapeQK = Shape<_1, kv_tile_size, _64>;
  using ShapePV = Shape<_1, _32, kv_tile_size>;
  using ShapeOutput = Shape<_1, _96>;
  using SubgroupLayout = Layout<Shape<_1, num_sg, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 215-223

```cpp
template <bool Persistent>
struct ShapeConfig<FMHAMode::Decode, 128, Persistent> {
  using num_sg = cute::conditional_t<Persistent, _16, _8>;
  using kv_tile_size = cute::conditional_t<Persistent, _256, _512>;
  using ShapeQK = Shape<_1, kv_tile_size, _64>;
  using ShapePV = Shape<_1, _32, kv_tile_size>;
  using ShapeOutput = Shape<_1, _128>;
  using SubgroupLayout = Layout<Shape<_1, num_sg, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 225-233

```cpp
template <bool Persistent>
struct ShapeConfig<FMHAMode::Decode, 192, Persistent> {
  using num_sg = cute::conditional_t<Persistent, _16, _8>;
  using kv_tile_size = cute::conditional_t<Persistent, _256, _512>;
  using ShapeQK = Shape<_1, kv_tile_size, _64>;
  using ShapePV = Shape<_1, _32, kv_tile_size>;
  using ShapeOutput = Shape<_1, _192>;
  using SubgroupLayout = Layout<Shape<_1, num_sg, _1>>;
};
```
- **EN:** Defines templated type `ShapeConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `ShapeConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 235-241

```cpp
// Mode-dependent pipeline stages
template <FMHAMode Mode>
struct PipelineStagesConfig;
template <>
struct PipelineStagesConfig<FMHAMode::Decode> { static constexpr int value = 1; };
template <>
struct PipelineStagesConfig<FMHAMode::Prefill> { static constexpr int value = 2; };
```
- **EN:** Defines templated type `PipelineStagesConfig` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `PipelineStagesConfig`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 243-255

```cpp
// FMHAConfigGen using ShapeConfig lookup table
template<FMHAMode Mode,
         class ElementQ, class ElementK, class ElementV, class ElementO,
         class LayoutQ, class LayoutK, class LayoutV, class LayoutO,
         bool Causal, bool VarLen, bool CachedKV, bool PagedKV, bool Persistent, int HeadDim>
struct FMHAConfigGen{
  using TileShapeConfig = ShapeConfig<Mode, HeadDim, Persistent>;
  using type = cutlass::flash_attention::FMHAConfig<
    ElementQ, ElementK, ElementV, ElementO, LayoutQ, LayoutK, LayoutV, LayoutO,
    typename TileShapeConfig::ShapeQK, typename TileShapeConfig::ShapePV, typename TileShapeConfig::ShapeOutput,
    typename TileShapeConfig::SubgroupLayout, void,
    Causal, VarLen, CachedKV, PagedKV, Persistent, PipelineStagesConfig<Mode>::value>;
};
```
- **EN:** Defines templated type `FMHAConfigGen` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `FMHAConfigGen`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 257-268

```cpp
// FMHAConfigGen with explicit tile and subgroup specification
template<FMHAMode Mode,
         class ElementQ, class ElementK, class ElementV, class ElementO,
         class LayoutQ, class LayoutK, class LayoutV, class LayoutO,
         bool Causal, bool VarLen, bool CachedKV, bool PagedKV, bool Persistent,
         int WgTileQ, int WgTileK, int WgTileV,
         int SgTileQ, int SgTileK,
         int HeadDimQK, int HeadDimV>
struct FMHAConfigGenWithTileShape{
  using ShapeQK = Shape<Int<WgTileQ>, Int<WgTileK>, Int<HeadDimQK>>;
  using ShapePV = Shape<Int<WgTileQ>, Int<WgTileV>, Int<WgTileK>>;  // Third dimension = WgTileK (K sequence tile, shared with ShapeQK[1])
  using ShapeOutput = Shape<Int<WgTileQ>, Int<HeadDimV>>;
```
- **EN:** Defines templated type `FMHAConfigGenWithTileShape` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `FMHAConfigGenWithTileShape`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 270-272

```cpp
  // Derive subgroup counts from tile ratios for QK matmul
  static_assert(WgTileQ % SgTileQ == 0, "WgTileQ must be divisible by SgTileQ");
  static_assert(WgTileK % SgTileK == 0, "WgTileK must be divisible by SgTileK");
```
- **EN:** Adds a compile-time constraint so unsupported combinations fail early during template instantiation.
- **CN:** 添加编译期约束，使不受支持的组合在模板实例化阶段及早失败。

### Lines 274-280

```cpp
  // SubgroupLayoutQK: (num_sg_q, num_sg_k, 1)
  // Head dimension is never split across subgroups (always _1)
  using SubgroupLayoutQK = Layout<Shape<
    Int<WgTileQ / SgTileQ>,
    Int<WgTileK / SgTileK>,
    _1
  >>;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 282-288

```cpp
  // SubgroupLayoutPV: (num_sg_p = num_sg_q, 1, num_sg_k)
  // number of subgroups in PV GEMM equals that in QK GEMM
  using SubgroupLayoutPV = Layout<Shape<
    Int<WgTileQ / SgTileQ>,
    _1,
    Int<WgTileK / SgTileK>
  >>;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 290-295

```cpp
  using type = cutlass::flash_attention::FMHAConfig<
    ElementQ, ElementK, ElementV, ElementO, LayoutQ, LayoutK, LayoutV, LayoutO,
    ShapeQK, ShapePV, ShapeOutput,
    SubgroupLayoutQK, SubgroupLayoutPV,
    Causal, VarLen, CachedKV, PagedKV, Persistent, PipelineStagesConfig<Mode>::value>;
};
```
- **EN:** Defines compile-time configuration objects that choose tile sizes, layouts, datatypes, and policies.
- **CN:** 定义编译期配置对象，用于选择 tile 大小、布局、数据类型和策略。

### Lines 297-298

```cpp
} // namespace flash_attention
} // namespace cutlass
```
- **EN:** Opens or closes namespaces so the declarations live in the intended CUTLASS, CUTE, or benchmark scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS、CUTE 或 benchmark 作用域中。

## Key Concepts / 关键概念

- **EN:** Template configuration: many choices are fixed at compile time through type aliases and boolean policy flags.
- **CN:** 模板配置：许多选择通过类型别名和布尔策略标志在编译期固定。
- **EN:** MMA tiling: matrix-multiply-accumulate building blocks are specialized for Xe subgroup execution.
- **CN:** MMA 分块：矩阵乘加构件针对 Xe 子组执行进行专门化。
- **EN:** Benchmark harness: the file mostly registers or launches predefined kernel configurations.
- **CN:** 基准框架：该文件主要负责注册或启动预定义的内核配置。

## Dependencies / 依赖关系

- **EN:** Direct dependencies referenced here include `type_traits`, `cutlass/gemm/dispatch_policy.hpp`, `flash_attention_v2/collective/fmha_fusion.hpp`, `flash_attention_v2/kernel/xe_fmha_fwd_kernel.hpp`, `flash_attention_v2/kernel/xe_tile_scheduler.hpp`.
- **CN:** 这里引用的直接依赖包括 `type_traits`, `cutlass/gemm/dispatch_policy.hpp`, `flash_attention_v2/collective/fmha_fusion.hpp`, `flash_attention_v2/kernel/xe_fmha_fwd_kernel.hpp`, `flash_attention_v2/kernel/xe_tile_scheduler.hpp`。
- **EN:** Benchmark-side code depends on runner/configuration helpers rather than implementing the low-level math directly.
- **CN:** 基准侧代码依赖 runner/配置辅助模块，而不是直接实现底层数学过程。
