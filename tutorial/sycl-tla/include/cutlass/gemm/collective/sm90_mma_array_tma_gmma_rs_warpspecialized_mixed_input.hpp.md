# sm90_mma_array_tma_gmma_rs_warpspecialized_mixed_input.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/sm90_mma_array_tma_gmma_rs_warpspecialized_mixed_input.hpp`
- **Purpose (EN):** Implements an architecture-specific `CollectiveMma` specialization for SM90, covering mixed input types, TMA transfers, GMMA instructions, warp-specialized scheduling, array-based MMA tiling, and 1 more.
- **用途 (CN):** 为 SM90 实现架构特化的 `CollectiveMma` 特化版本，重点覆盖 混合输入类型、TMA 传输、GMMA 指令、warp-specialized 调度、数组式 MMA 分块 等 6 项。
- **Lines / 行数:** 1380

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
**EN:** This opening block combines the license banner with `#pragma once`, documenting legal terms and making the header safe to include multiple times.
**CN:** 这个开头块同时包含许可证说明和 `#pragma once`：前者给出法律信息，后者保证头文件可被重复包含而不重复定义。

### Lines 33-39

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/numeric_types.h"
#include "cutlass/pipeline/pipeline.hpp"
#include "cutlass/trace.h"
#include "cutlass/cuda_host_adapter.hpp"
#include "cutlass/detail/collective/mixed_input_utils.hpp"
```
**EN:** This include block imports cutlass.h, dispatch_policy.hpp, numeric_types.h, pipeline.hpp, and 3 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cutlass.h、dispatch_policy.hpp、numeric_types.h、pipeline.hpp 等 7 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 41-46

```cpp
#include "cute/arch/cluster_sm90.hpp"
#include "cute/arch/copy_sm90.hpp"
#include "cute/algorithm/functional.hpp"
#include "cute/atom/mma_atom.hpp"
#include "cute/algorithm/gemm.hpp"
#include "cute/numeric/arithmetic_tuple.hpp"
```
**EN:** This include block imports cluster_sm90.hpp, copy_sm90.hpp, functional.hpp, mma_atom.hpp, and 2 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cluster_sm90.hpp、copy_sm90.hpp、functional.hpp、mma_atom.hpp 等 6 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 48-48

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 50-51

```cpp
namespace cutlass::gemm::collective {
using namespace cute;
```
**EN:** This short block both opens the collective namespace and pulls CuTe symbols into scope, setting up a concise vocabulary for the rest of the file.
**CN:** 这个短代码块既打开了 collective 命名空间，也把 CuTe 符号引入作用域，为后续代码建立更简洁的表达方式。

### Lines 53-53

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 55-98

```cpp
// WarpSpecialized Mainloop
template <
  int Stages,
  class ClusterShape,
  class KernelSchedule_,
  class TileShape_,
  class ElementAOptionalTuple,
  class StrideA_,
  class ElementBOptionalTuple,
  class StrideB_,
  class TiledMma_,
  class GmemTiledCopyA_,
  class SmemLayoutAtomA_,
  class SmemCopyAtomA_,
  class TransformA_,
  class GmemTiledCopyB_,
  class SmemLayoutAtomB_,
  class SmemCopyAtomB_,
  class TransformB_>
struct CollectiveMma<
    MainloopSm90ArrayTmaGmmaWarpSpecializedMixedInput<Stages, ClusterShape, KernelSchedule_>,
    TileShape_,
    ElementAOptionalTuple,
    StrideA_,
    ElementBOptionalTuple,
    StrideB_,
    TiledMma_,
    GmemTiledCopyA_,
    SmemLayoutAtomA_,
    SmemCopyAtomA_,
    TransformA_,
    GmemTiledCopyB_,
    SmemLayoutAtomB_,
    SmemCopyAtomB_,
    TransformB_>
{
public:
  //
  // Type Aliases
  //
  using ConversionMode = cutlass::detail::ConversionMode;
  using DispatchPolicy = MainloopSm90ArrayTmaGmmaWarpSpecializedMixedInput<Stages, ClusterShape, KernelSchedule_>;
  using TileShape = TileShape_;
  using KernelSchedule = KernelSchedule_;
```
**EN:** This template block declares or specializes `CollectiveMma`, the mainloop object that coordinates tile movement and matrix-multiply work.
**CN:** 这个模板块声明或特化了 `CollectiveMma`，它负责协调 tile 搬运与矩阵乘主循环。

### Lines 100-110

```cpp
private:
  template<class T> friend struct detail::MixedInputUtils;
  using CollectiveType = CollectiveMma<DispatchPolicy, TileShape_,
                                       ElementAOptionalTuple, StrideA_,
                                       ElementBOptionalTuple, StrideB_,
                                       TiledMma_,
                                       GmemTiledCopyA_, SmemLayoutAtomA_, SmemCopyAtomA_,
                                       TransformA_,
                                       GmemTiledCopyB_, SmemLayoutAtomB_, SmemCopyAtomB_,
                                       TransformB_>;
  using Utils = detail::MixedInputUtils<CollectiveType>;
```
**EN:** This block introduces `T` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `T`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 112-118

```cpp
  //
  // Type Aliases
  //
  using ScaleA = detail::deduce_mixed_width_dtype_t<1, ElementAOptionalTuple>;
  using ScaleB = detail::deduce_mixed_width_dtype_t<1, ElementBOptionalTuple>;
  using ZeroA = detail::deduce_mixed_width_dtype_t<2, ElementAOptionalTuple>;
  using ZeroB = detail::deduce_mixed_width_dtype_t<2, ElementBOptionalTuple>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 120-122

```cpp
public:
  static_assert(cute::is_tuple<ElementAOptionalTuple>::value ^ cute::is_tuple<ElementBOptionalTuple>::value,
    "Either A OR B must be a tuple. It must take the from {ElementOperand, [ElementScale], [ElementZero]}. Inputs in [] are optional.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 124-131

```cpp
  using ElementA = detail::deduce_mixed_width_dtype_t<0, ElementAOptionalTuple>;
  using ElementB = detail::deduce_mixed_width_dtype_t<0, ElementBOptionalTuple>;
  static constexpr bool IsATransformed = cute::is_tuple<ElementAOptionalTuple>::value;
  using ElementScale = cute::conditional_t<IsATransformed, ScaleA, ScaleB>;
  using ElementZero = cute::conditional_t<IsATransformed, ZeroA, ZeroB>;
  // For cases where we can't have a void type, we can use this to allow the code to compile when the scale / zero is void.
  using NonVoidElementScale = cute::conditional_t<cute::is_void_v<ElementScale>, float, ElementScale>;
  using NonVoidElementZero = cute::conditional_t<cute::is_void_v<ElementZero>, float, ElementZero>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 133-136

```cpp
  using StrideA = StrideA_;
  using InternalStrideA = cute::remove_pointer_t<StrideA>;
  using StrideB = StrideB_;
  using InternalStrideB = cute::remove_pointer_t<StrideB>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 138-139

```cpp
  using StrideScale = cute::Stride<cute::Int<1>, int64_t, int64_t>;
  using NonVoidStrideScale = cute::conditional_t<cute::is_void_v<StrideScale>, cute::Stride<_1, int64_t, int64_t>, StrideScale>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 141-143

```cpp
  static_assert(( IsATransformed && (cutlass::gemm::detail::is_k_major<StrideA>() || is_layout<StrideA>::value || is_layout<InternalStrideA>::value)) ||
                (!IsATransformed && (cutlass::gemm::detail::is_k_major<StrideB>() || is_layout<StrideB>::value || is_layout<InternalStrideB>::value)),
                "The transformed type must be K-major.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 145-149

```cpp
  static_assert(( IsATransformed && (sizeof(ElementB) == 2)) ||
                (!IsATransformed && (sizeof(ElementA) == 2)) ||
                ((cutlass::gemm::detail::is_k_major<StrideA>() || is_layout<StrideA>::value || is_layout<InternalStrideA>::value) &&
                 (cutlass::gemm::detail::is_k_major<StrideB>() || is_layout<StrideB>::value || is_layout<InternalStrideB>::value)),
                "The unscaled element must be 2 bytes OR both inputs must be K-major");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 151-152

```cpp
  static_assert(cutlass::gemm::detail::is_mn_major<NonVoidStrideScale>(),
    "Scale must be MN major [Col Major if A is scaled, Row Major if B is scaled].");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 154-164

```cpp
  using CtaShape_MNK = decltype(shape_div(TileShape{}, ClusterShape{}));
  using TiledMma = TiledMma_;
  using ElementAccumulator = typename TiledMma::ValTypeC;
  using GmemTiledCopyA = GmemTiledCopyA_;
  using GmemTiledCopyB = GmemTiledCopyB_;
  using GmemTiledCopyScale = cute::SM90_TMA_LOAD;
  using SmemLayoutAtomA = SmemLayoutAtomA_;
  using SmemLayoutAtomB = SmemLayoutAtomB_;
  using SmemCopyAtomA = SmemCopyAtomA_;
  using SmemCopyAtomB = SmemCopyAtomB_;
  using SmemCopyAtomScale = Copy_Atom<cute::AutoVectorizingCopy, NonVoidElementScale>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 166-185

```cpp
  // We must ensure the type to be scaled goes to RF
  static constexpr bool SwapAB = !IsATransformed;
  using SwappedStrideA = cute::conditional_t<!SwapAB, StrideA, StrideB>;
  using SwappedStrideB = cute::conditional_t<!SwapAB, StrideB, StrideA>;
  using InternalSwappedStrideA = cute::conditional_t<!SwapAB, InternalStrideA, InternalStrideB>;
  using InternalSwappedStrideB = cute::conditional_t<!SwapAB, InternalStrideB, InternalStrideA>;
  using SwappedSmemLayoutAtomA = cute::conditional_t<!SwapAB, SmemLayoutAtomA, SmemLayoutAtomB>;
  using SwappedSmemLayoutAtomB = cute::conditional_t<!SwapAB, SmemLayoutAtomB, SmemLayoutAtomA>;
  using SwappedSmemCopyAtomA   = cute::conditional_t<!SwapAB, SmemCopyAtomA, SmemCopyAtomB>;
  using SwappedSmemCopyAtomB   = cute::conditional_t<!SwapAB, SmemCopyAtomB, SmemCopyAtomA>;
  // TMA converts f32 input to tf32 when copying from GMEM to SMEM
  // For all other types, cast to size equivalent uint type to avoid any rounding by TMA.
  static constexpr bool ConvertF32toTF32A = cute::is_same_v<float, ElementA>;
  static constexpr bool ConvertF32toTF32B = cute::is_same_v<float, ElementB>;
  using ConvertedElementA = cute::conditional_t<ConvertF32toTF32A, tfloat32_t, uint_bit_t<sizeof_bits_v<ElementA>>>;
  using ConvertedElementB = cute::conditional_t<ConvertF32toTF32B, tfloat32_t, uint_bit_t<sizeof_bits_v<ElementB>>>;
  using RealSwappedElementA = cute::conditional_t<!SwapAB, ElementA, ElementB>;
  using RealSwappedElementB = cute::conditional_t<!SwapAB, ElementB, ElementA>;
  using SwappedElementA = cute::conditional_t<!SwapAB, ConvertedElementA, ConvertedElementB>;
  using SwappedElementB = cute::conditional_t<!SwapAB, ConvertedElementB, ConvertedElementA>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 187-191

```cpp
  using TransformA = TransformA_;
  using TransformB = TransformB_;
  using SwappedTransformA  = cute::conditional_t<!SwapAB, TransformA, TransformB>;
  using SwappedTransformB  = cute::conditional_t<!SwapAB, TransformB, TransformA>;
  using ArchTag = typename DispatchPolicy::ArchTag;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 193-195

```cpp
  static constexpr int IsSubbyteA = cute::sizeof_bits_v<SwappedElementA> < 8;
  using TmaElementA = cute::conditional_t<IsSubbyteA, uint8_t, SwappedElementA>;
  using TmaElementScale = uint_bit_t<sizeof_bits_v<NonVoidElementScale> >; // in case we have array. translating to uint to satisfy tma descriptor's specialization
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 197-199

```cpp
  using MainloopPipeline = cutlass::PipelineTmaAsync<DispatchPolicy::Stages>;
  using PipelineState = cutlass::PipelineState<DispatchPolicy::Stages>;
  using PipelineParams = typename MainloopPipeline::Params;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 201-201

```cpp
  static constexpr int NumProducerThreadEvents = 1;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 203-204

```cpp
  using SmemLayoutAtomScale = Layout<Shape<decltype(cute::shape<0>(SwappedSmemLayoutAtomA{})), cute::Int<1>>>;
  using ScaleTileShape = decltype(make_shape(shape<0>(TileShape{}), shape<1>(SmemLayoutAtomScale{})));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 206-208

```cpp
  static_assert(cute::rank(SwappedSmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SwappedSmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SwappedSmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 210-212

```cpp
  static_assert(cute::rank(SwappedSmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<1>(TileShape{}) % size<0>(SwappedSmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SwappedSmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 214-216

```cpp
  static_assert(rank(SmemLayoutAtomScale{}) == 2, "SmemLayoutAtomScale must be rank 2");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomScale{})) == 0, "SmemLayoutAtomScale must equal the tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomScale{})) == 0, "SmemLayoutAtomScale must evenly divide tile k shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 218-220

```cpp
  /// Tile along modes in a way that maximizes the TMA box size.
  using SmemLayoutA = decltype(detail::get_smem_layout<DispatchPolicy::Stages>(SwappedSmemLayoutAtomA{}, select<0,2>(TileShape{}), InternalSwappedStrideA{}));
  using SmemLayoutB = decltype(detail::get_smem_layout<DispatchPolicy::Stages>(SwappedSmemLayoutAtomB{}, select<1,2>(TileShape{}), InternalSwappedStrideB{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 222-226

```cpp
  // It is assumed that the scales and zero-points share the same smem layout
  using SmemLayoutScale = decltype(tile_to_shape(
    SmemLayoutAtomScale{},
    make_shape(shape<0>(ScaleTileShape{}), shape<1>(ScaleTileShape{}), Int<Stages>{}),
    cute::conditional_t< ::cutlass::gemm::detail::is_major<0,NonVoidStrideScale>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 228-235

```cpp
  static_assert(DispatchPolicy::Stages >= 2, "Specialization requires Stages set to value 2 or more.");
  static_assert(not cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value &&
                    cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeB>::value,
                "MMA atom must source A from rmem and B operand from smem_desc for this mainloop.");
  static_assert(cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>,
      "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
  static_assert(cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD_MULTICAST>,
      "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 237-239

```cpp
  // To relax them, we need to handle loading more than 1 row of scales for every main loop iteration.
  // We must also handle updating the pipeline transaction bytes on the fly.
  static_assert(size<1>(SmemLayoutAtomScale{}) == 1, "size<1>(SmemLayoutAtomScale) must be 1.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 241-253

```cpp
private:
  static constexpr ConversionMode
  get_conversion_mode() {
    if constexpr (cute::is_void_v<ElementScale>) {
      return ConversionMode::DirectConvert;
    }
    else if constexpr (cute::is_void_v<ElementZero>) {
      return ConversionMode::ConvertAndScale;
    }
    else {
      return ConversionMode::ConvertAndScaleWithZero;
    }
  }
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 255-263

```cpp
public:
  static constexpr ConversionMode KernelConversionMode = get_conversion_mode();
  static constexpr bool ModeHasScales = KernelConversionMode == ConversionMode::ConvertAndScale ||
                                        KernelConversionMode == ConversionMode::ConvertAndScaleWithZero;
  static constexpr bool UseScaleLookupTable = KernelConversionMode == ConversionMode::ConvertAndScale &&
                                              cutlass::detail::is_Array_v<ElementScale>;
  static constexpr size_t SmemAlignmentA = cutlass::detail::alignment_for_swizzle(SmemLayoutA{});
  static constexpr size_t SmemAlignmentB = cutlass::detail::alignment_for_swizzle(SmemLayoutB{});
  static constexpr size_t SmemAlignmentScale = cute::max(SmemAlignmentA, SmemAlignmentB);
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 265-265

```cpp
  static_assert(SmemAlignmentA >= 128 and SmemAlignmentB >= 128, "Require at least 128B alignment");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 267-275

```cpp
  struct SharedStorage {
    static constexpr int scale_elements = Utils::elements_per_smem_scale();
    static constexpr int zero_elements = Utils::elements_per_smem_zero();
    struct TensorStorage {
      CUTE_ALIGNAS(SmemAlignmentA) cute::ArrayEngine<RealSwappedElementA, cute::cosize_v<SmemLayoutA>> smem_A;
      CUTE_ALIGNAS(SmemAlignmentB) cute::ArrayEngine<typename TiledMma::ValTypeB, cute::cosize_v<SmemLayoutB>> smem_B;
      cute::ArrayEngine<NonVoidElementScale, scale_elements> smem_scale;
      cute::ArrayEngine<NonVoidElementZero, zero_elements> smem_zero;
    } tensors;
```
**EN:** This block introduces `SharedStorage` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `SharedStorage`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 277-282

```cpp
    struct TensorMapStorage {
      cute::TmaDescriptor smem_tensormap_A;
      cute::TmaDescriptor smem_tensormap_B;
      cute::TmaDescriptor smem_tensormap_scale;
      cute::TmaDescriptor smem_tensormap_zero;
    };
```
**EN:** This block introduces `TensorMapStorage` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TensorMapStorage`，并围绕该符号组织相关声明。

### Lines 284-289

```cpp
    using PipelineStorage = typename MainloopPipeline::SharedStorage;
    PipelineStorage pipeline;
  };
  using TensorStorage = typename SharedStorage::TensorStorage;
  using TensorMapStorage = typename SharedStorage::TensorMapStorage;
  using PipelineStorage = typename SharedStorage::PipelineStorage;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 291-291

```cpp
  static constexpr bool IsGroupedGemmKernel = !cute::is_same_v<InternalStrideA, StrideA>;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 293-303

```cpp
  // Host side kernel arguments
  struct Arguments {
    ElementA const** ptr_A;
    StrideA dA;
    ElementB const** ptr_B;
    StrideB dB;
    ElementScale const** ptr_S = nullptr;
    NonVoidStrideScale const* dS{};
    int chunk_size = 0;
    ElementZero const** ptr_Z = nullptr;
  };
```
**EN:** This block introduces `Arguments` and groups related declarations around that symbol. `Arguments` is the host-side bundle: callers fill it with pointers, strides, and other launch-time values before parameter lowering.
**CN:** 这一段引入了 `Arguments`，并围绕该符号组织相关声明。 `Arguments` 是主机侧参数包：调用者在参数下沉前把指针、步长及其他启动期数据填入其中。

### Lines 305-309

```cpp
  // Device side kernel params
  struct Params {
    // Assumption: StrideA is congruent with Problem_MK
    using LayoutA = decltype(detail::get_gmem_layout(repeat_like(InternalSwappedStrideA{}, int32_t(0)), InternalSwappedStrideA{}));
    using LayoutB = decltype(detail::get_gmem_layout(repeat_like(InternalSwappedStrideB{}, int32_t(0)), InternalSwappedStrideB{}));
```
**EN:** This block introduces `Params` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `Params`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 311-323

```cpp
    using TMA_A = decltype(make_tma_copy<TmaElementA>(
        GmemTiledCopyA{},
        make_tensor(detail::get_logical_ptr(static_cast<SwappedElementA const*>(nullptr)), LayoutA{}),
        SmemLayoutA{}(_,_,cute::Int<0>{}),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        size<1>(ClusterShape{})));  // mcast along N mode for this M load, if any
    // Assumption: StrideB is congruent with Problem_NK
    using TMA_B = decltype(make_tma_copy(
        GmemTiledCopyB{},
        make_tensor(detail::get_logical_ptr(static_cast<SwappedElementB const*>(nullptr)), LayoutB{}),
        SmemLayoutB{}(_,_,cute::Int<0>{}),
        make_shape(shape<1>(TileShape{}), shape<2>(TileShape{})),
        size<0>(ClusterShape{}))); // mcast along M mode for this N load, if any
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 325-330

```cpp
    using TMA_Scale = decltype(make_tma_copy<TmaElementScale>(
        GmemTiledCopyScale{},
        make_tensor(detail::get_logical_ptr(static_cast<NonVoidElementScale const*>(nullptr)), repeat_like(NonVoidStrideScale{}, int32_t(0)), NonVoidStrideScale{}),
        SmemLayoutScale{}(_,_,cute::Int<0>{}),
        ScaleTileShape{},
        _1{}));  // mcast along N mode for this M load, if any. Scale is ALWAYS loaded with A for RF kernel
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 332-337

```cpp
   using TMA_Zero = decltype(make_tma_copy(
        GmemTiledCopyScale{},
        make_tensor(detail::get_logical_ptr(static_cast<NonVoidElementZero const*>(nullptr)), repeat_like(NonVoidStrideScale{}, int32_t(0)), NonVoidStrideScale{}),
        SmemLayoutScale{}(_,_,cute::Int<0>{}),
        ScaleTileShape{},
        _1{}));  // mcast along N mode for this M load, if any. Scale is ALWAYS loaded with A for RF kernel
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 339-357

```cpp
    TMA_A tma_load_a;
    TMA_B tma_load_b;
    uint32_t tma_transaction_bytes = TmaTransactionBytes;
    TMA_Scale tma_load_scale;
    TMA_Zero tma_load_zero;
    void* tensormaps;
    SwappedElementA const** ptr_A;
    SwappedStrideA ptr_dA;
    SwappedElementB const** ptr_B;
    SwappedStrideB ptr_dB;
    NonVoidElementScale const** ptr_S;
    NonVoidStrideScale const* dS;
    NonVoidElementZero const** ptr_Z;
    int64_t scale_k;
    int chunk_size;
    int reload_factor = (chunk_size + size<2>(TileShape{}) - 1) / size<2>(TileShape{});
    InternalSwappedStrideA dA;
    InternalSwappedStrideB dB;
  };
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 359-361

```cpp
  //
  // Methods
  //
```
**EN:** This comment starts a method section where the collective exposes its runtime behavior.
**CN:** 这条注释表示进入方法区，collective 的运行时行为将在这里定义。

### Lines 363-368

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(
      ProblemShape problem_shapes,
      Arguments const& args,
      void* workspace) {
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 370-375

```cpp
    // These tensor shapes (only applicable for grouped gemm) and pointers are only used to create tensormap/tma desc.
    // These will be replaced with correct values before the initial tma load.
    auto init_shape = repeat_like(typename ProblemShape::UnderlyingProblemShape{}, int32_t(1));
    auto init_M = get<0>(init_shape);
    auto init_N = get<1>(init_shape);
    auto init_K = get<2>(init_shape);
```
**EN:** The surrounding comments explain the local purpose of this block: These tensor shapes (only applicable for grouped gemm) and pointers are only used to create tensormap/tma desc. These will be replaced with correct values before the initial tma load..
**CN:** 周围注释解释了这一段的局部作用：These tensor shapes (only applicable for grouped gemm) and pointers are only used to create tensormap/tma desc. These will be replaced with correct values before the initial tma load.。

### Lines 377-388

```cpp
    if constexpr (SwapAB) {
      init_M = get<1>(init_shape);
      init_N = get<0>(init_shape);
    }
    // Batches/Groups are managed by using appropriate pointers to input matrices
    const uint32_t mock_L = 1;
    SwappedElementA const* ptr_A_first_batch;
    SwappedElementB const* ptr_B_first_batch;
    SwappedStrideA ptr_dA;
    SwappedStrideB ptr_dB;
    InternalSwappedStrideA dA;
    InternalSwappedStrideB dB;
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 390-397

```cpp
    if constexpr (not SwapAB) {
      ptr_A_first_batch = reinterpret_cast<SwappedElementA const*>(reinterpret_cast<uint64_t>(args.ptr_A) & 0xFFFFFFFFFFFFFFF0);  // Address must be 16B-aligned
      ptr_B_first_batch = reinterpret_cast<SwappedElementB const*>(reinterpret_cast<uint64_t>(args.ptr_B) & 0xFFFFFFFFFFFFFFF0);  // Address must be 16B-aligned
    }
    else {
      ptr_A_first_batch = reinterpret_cast<SwappedElementA const*>(reinterpret_cast<uint64_t>(args.ptr_B) & 0xFFFFFFFFFFFFFFF0);  // Address must be 16B-aligned
      ptr_B_first_batch = reinterpret_cast<SwappedElementB const*>(reinterpret_cast<uint64_t>(args.ptr_A) & 0xFFFFFFFFFFFFFFF0);  // Address must be 16B-aligned
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 399-432

```cpp
    if constexpr (IsGroupedGemmKernel) {
      // Strides for Grouped Gemm will be replaced prior to the first access regardless.
      if constexpr (not SwapAB) {
        ptr_dA = args.dA;
        ptr_dB = args.dB;
      }
      else {
        ptr_dA = args.dB;
        ptr_dB = args.dA;
      }
      dA = InternalSwappedStrideA{};
      if constexpr (is_layout<InternalSwappedStrideA>::value) {
        dA = make_layout(
          transform_leaf(dA.shape(), [](auto x){
            if constexpr (not is_static_v<decltype(x)>) {
              return static_cast<decltype(x)>(1);
            } else {
              return x;
            }
          }),
          dA.stride());
      }
      dB = InternalSwappedStrideB{};
    }
    else {
      // Tensor shapes for Ptr-Array are initialized correctly only here.
      auto problem_shape_MNK = problem_shapes.get_host_problem_shape(0);
      init_M = get<0>(problem_shape_MNK);
      init_N = get<1>(problem_shape_MNK);
      init_K = get<2>(problem_shape_MNK);
      if constexpr (SwapAB) {
        init_M = get<1>(problem_shape_MNK);
        init_N = get<0>(problem_shape_MNK);
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 434-446

```cpp
      if constexpr (not SwapAB) {
        dA = args.dA;
        dB = args.dB;
      }
      else {
        dA = args.dB;
        dB = args.dA;
      }
      ptr_dA = SwappedStrideA{};
      ptr_dB = SwappedStrideB{};
    }
    Tensor tensor_a = make_tensor(ptr_A_first_batch, detail::get_gmem_layout(make_shape(init_M,init_K,mock_L), dA));
    Tensor tensor_b = make_tensor(ptr_B_first_batch, detail::get_gmem_layout(make_shape(init_N,init_K,mock_L), dB));
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 448-461

```cpp
    typename Params::TMA_A tma_load_a = make_tma_copy<TmaElementA>(
        GmemTiledCopyA{},
        tensor_a,
        SmemLayoutA{}(_,_,cute::Int<0>{}),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        size<1>(ClusterShape{})); // mcast along N mode for this M load, if any
    typename Params::TMA_B tma_load_b = make_tma_copy(
        GmemTiledCopyB{},
        tensor_b,
        SmemLayoutB{}(_,_,cute::Int<0>{}),
        make_shape(shape<1>(TileShape{}), shape<2>(TileShape{})),
        size<0>(ClusterShape{})); // mcast along M mode for this N load, if any
    typename Params::TMA_Scale tma_load_scale{};
    typename Params::TMA_Zero tma_load_zero{};
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 463-485

```cpp
    void* tensormaps = workspace;
    auto args_setup = [&](auto ptr_A, auto ptr_B, int64_t scale_k = 0, int chunk_size = 0, int reload_factor = 1) -> Params {
      return {
          tma_load_a,
          tma_load_b,
          TmaTransactionBytes,
          tma_load_scale,
          tma_load_zero,
          tensormaps,
          reinterpret_cast<SwappedElementA const**>(ptr_A),
          ptr_dA,
          reinterpret_cast<SwappedElementB const**>(ptr_B),
          ptr_dB,
          reinterpret_cast<NonVoidElementScale const**>(args.ptr_S),
          args.dS,
          reinterpret_cast<NonVoidElementZero const**>(args.ptr_Z),
          scale_k,
          chunk_size,
          reload_factor,
          dA,
          dB
      };
    };
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 487-501

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return SwapAB ? args_setup(args.ptr_B, args.ptr_A)
                    : args_setup(args.ptr_A, args.ptr_B);
    }
    else if constexpr (ModeHasScales) {
      auto scale_k = ceil_div(init_K, args.chunk_size);
      ElementScale const* ptr_S = reinterpret_cast<ElementScale const*>(args.ptr_S);
      StrideScale dS{};
      Tensor tensor_scale = make_tensor(detail::get_logical_ptr(ptr_S), make_layout(make_shape(init_M,scale_k,mock_L), dS));
      tma_load_scale = make_tma_copy<TmaElementScale>(
          GmemTiledCopyScale{},
          tensor_scale,
          SmemLayoutScale{}(_,_,cute::Int<0>{}),
          ScaleTileShape{},
          _1{}); // mcast along N mode for this M load, if any
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 503-517

```cpp
      if constexpr(KernelConversionMode == ConversionMode::ConvertAndScale) {
        return SwapAB ? args_setup(args.ptr_B, args.ptr_A, scale_k, args.chunk_size, (args.chunk_size + size<2>(TileShape{}) - 1) / size<2>(TileShape{}))
                      : args_setup(args.ptr_A, args.ptr_B, scale_k, args.chunk_size, (args.chunk_size + size<2>(TileShape{}) - 1) / size<2>(TileShape{}));
      }
      else if constexpr(KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        ElementZero const* ptr_Z = reinterpret_cast<ElementZero const*>(args.ptr_Z);
        Tensor tensor_zero = make_tensor(detail::get_logical_ptr(ptr_Z), make_layout(make_shape(init_M,scale_k,mock_L), dS));
        tma_load_zero = make_tma_copy(
            GmemTiledCopyScale{},
            tensor_zero,
            SmemLayoutScale{}(_,_,cute::Int<0>{}),
            ScaleTileShape{},
            _1{}); // mcast along N mode for this M load, if any
        return SwapAB ? args_setup(args.ptr_B, args.ptr_A, scale_k, args.chunk_size, (args.chunk_size + size<2>(TileShape{}) - 1) / size<2>(TileShape{}))
                      : args_setup(args.ptr_A, args.ptr_B, scale_k, args.chunk_size, (args.chunk_size + size<2>(TileShape{}) - 1) / size<2>(TileShape{}));
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 519-527

```cpp
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in to_underlying_arguments.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in to_underlying_arguments.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. This method lowers high-level problem-shape and argument objects into the concrete descriptors or tensors consumed by the collective.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这个方法把高层 problem shape 和参数对象下沉为 collective 真正消费的具体描述符或张量。

### Lines 529-532

```cpp
  template <class ProblemShape>
  static size_t
  get_workspace_size(ProblemShape const& problem_shape, Arguments const& args, int sm_count) {
    constexpr size_t SizeOfCuTensorMap = sizeof(cute::TmaDescriptor);
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。

### Lines 534-537

```cpp
    // Calculating workspace size
    auto calculate_workspace_size = [SizeOfCuTensorMap, sm_count](uint32_t num_input_tensors) {
        return num_input_tensors * SizeOfCuTensorMap * sm_count;
    };
```
**EN:** The surrounding comments explain the local purpose of this block: Calculating workspace size.
**CN:** 周围注释解释了这一段的局部作用：Calculating workspace size。

### Lines 539-554

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      // Allocate gmem space for input tensormaps per each SM, A tensormap copies followed by B tensormap copies
      return calculate_workspace_size(2);
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      // Allocate gmem space for input tensormaps per each SM, A tensormap copies followed by B tensormap copies, followed by scale tensormap copies
      return calculate_workspace_size(3);
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      // Allocate gmem space for input tensormaps per each SM, A tensormap copies followed by B tensormap copies, followed by scale and zeros tensormap copies
      return calculate_workspace_size(4);
    }
    else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in get_workspace_size.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 556-560

```cpp
  template <class ProblemShape>
  static cutlass::Status
  initialize_workspace(ProblemShape const& problem_shape, Arguments const& args, void* workspace, cudaStream_t stream, CudaHostAdapter* cuda_adapter = nullptr) {
    return cutlass::Status::kSuccess;
  }
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 563-570

```cpp
  template<class ProblemShape>
  CUTLASS_HOST_DEVICE static bool
  can_implement(
      ProblemShape problem_shapes,
      Arguments const& args) {
    constexpr int tma_alignment_bits = 128;
    constexpr int min_tma_aligned_elements_A = tma_alignment_bits / cutlass::sizeof_bits<ElementA>::value;
    constexpr int min_tma_aligned_elements_B = tma_alignment_bits / cutlass::sizeof_bits<ElementB>::value;
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. This method performs runtime feasibility checks, usually validating alignment, layout assumptions, or shape constraints before launching the kernel.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这个方法执行运行时可实现性检查，通常会在启动内核前验证对齐、布局假设或形状约束。

### Lines 572-618

```cpp
    bool implementable = true;
    if (problem_shapes.is_host_problem_shape_available()) {
      // Check alignment for all problem sizes
      for (int i = 0; i < problem_shapes.groups(); i++) {
        auto problem_shape_MNKL = append<4>(problem_shapes.get_host_problem_shape(i), 1);
        auto [M,N,K,L] = problem_shape_MNKL;
        auto get_stride = [](auto stride) {
          if constexpr (cute::is_pointer_v<cute::decay_t<decltype(stride)>>) {
            return *stride;
          }
          else {
            return stride;
          }
        };
        auto dA = get_stride(args.dA);
        auto dB = get_stride(args.dB);
        implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_A>(detail::get_gmem_layout(cute::make_shape(M,K,L), dA));
        implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_B>(detail::get_gmem_layout(cute::make_shape(N,K,L), dB));
        if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
          implementable = implementable && (args.ptr_S == nullptr);
          implementable = implementable && (args.ptr_Z == nullptr);
        }
        else if constexpr (ModeHasScales) {
          const int scale_mn = SwapAB ? N : M;
          const int scale_k = ceil_div(K, args.chunk_size);
          constexpr int min_tma_aligned_elements_scale = tma_alignment_bits / cutlass::sizeof_bits<ElementScale>::value;
          implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_scale>(cute::make_shape(scale_mn,scale_k,L), StrideScale{});
          implementable = implementable && (args.chunk_size == K || ((args.chunk_size % size<2>(TileShape{})) == 0));
          implementable = implementable && args.chunk_size != 0;
          implementable = implementable && (args.ptr_S != nullptr);
          if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
            implementable = implementable && (args.ptr_Z == nullptr);
          }
          else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
            constexpr int min_tma_aligned_elements_zero = tma_alignment_bits / cutlass::sizeof_bits<ElementZero>::value;
            implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_zero>(cute::make_shape(scale_mn,scale_k,L), StrideScale{});
            implementable = implementable && (args.ptr_Z != nullptr);
          }
          else {
            static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in can_implement.");
          }
        }
        else {
          static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in can_implement.");
        }
      }
    }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. This method performs runtime feasibility checks, usually validating alignment, layout assumptions, or shape constraints before launching the kernel.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这个方法执行运行时可实现性检查，通常会在启动内核前验证对齐、布局假设或形状约束。

### Lines 620-624

```cpp
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA.\n");
    }
    return implementable;
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 626-631

```cpp
  static constexpr int K_PIPE_MAX = DispatchPolicy::Stages;
  static constexpr int K_PIPE_MMAS = 1;
  static constexpr uint32_t TmaTransactionBytesMK = Utils::compute_tma_transaction_bytes_mk();
  static constexpr uint32_t TmaTransactionBytesNK = Utils::compute_tma_transaction_bytes_nk();
  static constexpr uint32_t TmaTransactionBytesExtra = Utils::compute_tma_transaction_bytes_extra();
  static constexpr uint32_t TmaTransactionBytes = TmaTransactionBytesMK + TmaTransactionBytesNK + TmaTransactionBytesExtra;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 633-645

```cpp
  // Set up the data needed by this collective for load and mma.
  // Returns a tuple of tensors. The collective and the kernel layer have the contract that the
  // returned tuple must contain at least two elements, with the first two elements being:
  // gA_mkl - The tma tensor, A after a local tile so it has shape  (BLK_M,BLK_K,m,k,l)
  // gB_nkl - The tma tensor, B after a local tile so it has shape  (BLK_N,BLK_K,n,k,l)
  // The rest of the tensors can be specified as needed by this collective.
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE auto
  load_init(ProblemShape_MNKL const& problem_shape_MNKL, Params const& mainloop_params) const {
    using X = Underscore;
    // Separate out problem shape for convenience
    auto [M,N,K,L] = problem_shape_MNKL;
    const int32_t mock_L = 1;
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 647-650

```cpp
    // TMA requires special handling of strides to deal with coord codomain mapping
    // Represent the full tensors -- get these from TMA
    Tensor mA_mkl = mainloop_params.tma_load_a.get_tma_tensor(shape(detail::get_gmem_layout(make_shape(M,K,mock_L), mainloop_params.dA))); // (m,k,l)
    Tensor mB_nkl = mainloop_params.tma_load_b.get_tma_tensor(shape(detail::get_gmem_layout(make_shape(N,K,mock_L), mainloop_params.dB))); // (n,k,l)
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 652-654

```cpp
    // Make tiled views, defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});  // (BLK_M,BLK_K,m,k,l)
    Tensor gB_nkl = local_tile(mB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});  // (BLK_N,BLK_K,n,k,l)
```
**EN:** The surrounding comments explain the local purpose of this block: Make tiled views, defer the slice.
**CN:** 周围注释解释了这一段的局部作用：Make tiled views, defer the slice。

### Lines 656-679

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return cute::make_tuple(gA_mkl, gB_nkl);
    }
    else if constexpr (ModeHasScales) {
      const int scale_mn = SwapAB ? N : M;
      auto scale_k = mainloop_params.scale_k;
      Tensor mS_mkl = mainloop_params.tma_load_scale.get_tma_tensor(make_shape(scale_mn,scale_k,L));
      Tensor gS_mkl = local_tile(mS_mkl, ScaleTileShape{}, make_coord(_,_));       // (BLK_M,BLK_Scale_K,m,scale_k,l)
      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return cute::make_tuple(gA_mkl, gB_nkl, gS_mkl);
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        Tensor mZ_mkl = mainloop_params.tma_load_zero.get_tma_tensor(make_shape(scale_mn,scale_k,L));
        Tensor gZ_mkl = local_tile(mZ_mkl, ScaleTileShape{}, make_coord(_,_));      // (BLK_M,BLK_Scale_K,m,scale_k,l)
        return cute::make_tuple(gA_mkl, gB_nkl, gS_mkl, gZ_mkl);
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in load_init.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in load_init.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 681-699

```cpp
  // Perform a collective-scoped matrix multiply-accumulate
  // Producer Perspective
  template <
    class... Ts,
    class... TMs,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load(
      Params const& mainloop_params,
      MainloopPipeline pipeline,
      PipelineState smem_pipe_write,
      cute::tuple<Ts...> const& load_inputs,
      cute::tuple<TMs...> const& input_tensormaps,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
```
**EN:** This block introduces `KTileIterator` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `KTileIterator`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 701-715

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      static_assert(sizeof... (Ts) == 2, "Direct convert needs two inputs");
      static_assert(sizeof... (TMs) == 2, "Direct convert needs two tensormaps");
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      static_assert(sizeof... (Ts) == 3, "Scaled convert needs three inputs");
      static_assert(sizeof... (TMs) == 3, "Scaled convert needs three tensormaps");
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      static_assert(sizeof... (Ts) == 4, "Scaled and zero convert needs four inputs");
      static_assert(sizeof... (TMs) == 4, "Scaled and zero convert needs four tensormaps");
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in TMA load.");
    }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 717-720

```cpp
    Tensor sA_ = make_tensor(make_smem_ptr(shared_tensors.smem_A.begin()), SmemLayoutA{});          // (BLK_M,BLK_K,PIPE)
    Tensor sB_ = make_tensor(make_smem_ptr(shared_tensors.smem_B.begin()), SmemLayoutB{});          // (BLK_N,BLK_K,PIPE)
    Tensor sA  = as_position_independent_swizzle_tensor(sA_);                                       // (BLK_M,BLK_K,PIPE)
    Tensor sB  = as_position_independent_swizzle_tensor(sB_);                                       // (BLK_N,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 722-724

```cpp
    //
    // Prepare the TMA loads for A and B
    //
```
**EN:** This comment-only block labels or explains the following section:  Prepare the TMA loads for A and B .
**CN:** 这个纯注释块用于标记或解释后续区域： Prepare the TMA loads for A and B 。

### Lines 726-727

```cpp
    constexpr uint32_t cluster_shape_x = get<0>(typename DispatchPolicy::ClusterShape());
    uint2 cluster_local_block_id = {block_rank_in_cluster % cluster_shape_x, block_rank_in_cluster / cluster_shape_x};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 729-730

```cpp
    Tensor gA_mkl = get<0>(load_inputs);
    Tensor gB_nkl = get<1>(load_inputs);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 732-733

```cpp
    auto block_tma_a = mainloop_params.tma_load_a.get_slice(cluster_local_block_id.y);
    auto block_tma_b = mainloop_params.tma_load_b.get_slice(cluster_local_block_id.x);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 735-738

```cpp
    // Partition the inputs based on the current block coordinates.
    auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
    Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                     // (BLK_M,BLK_K,k)
    Tensor gB = gB_nkl(_,_,n_coord,_,l_coord);                                                     // (BLK_N,BLK_K,k)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition the inputs based on the current block coordinates..
**CN:** 周围注释解释了这一段的局部作用：Partition the inputs based on the current block coordinates.。

### Lines 740-742

```cpp
    // Applies the mapping from block_tma_a
    Tensor tAgA = block_tma_a.partition_S(gA);                                                 // (TMA,TMA_M,TMA_K,k)
    Tensor tAsA = block_tma_a.partition_D(sA);                                              // (TMA,TMA_M,TMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Applies the mapping from block_tma_a.
**CN:** 周围注释解释了这一段的局部作用：Applies the mapping from block_tma_a。

### Lines 744-745

```cpp
    Tensor tBgB = block_tma_b.partition_S(gB);                                                 // (TMA,TMA_N,TMA_K,k)
    Tensor tBsB = block_tma_b.partition_D(sB);                                              // (TMA,TMA_N,TMA_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 747-749

```cpp
    uint16_t mcast_mask_a = 0;
    uint16_t mcast_mask_b = 0;
    uint16_t mcast_mask_s = 0;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 751-758

```cpp
    // Issue TmaLoads
    // Maps the tile -> block, value
    if constexpr (cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>) {
      auto block_layout = Layout<typename DispatchPolicy::ClusterShape>{}; // (m,n) -> block_id
      for (int n = 0; n < size<1>(block_layout); ++n) {
        mcast_mask_a |= (uint16_t(1) << block_layout(cluster_local_block_id.x,n,Int<0>{}));
      }
    }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 760-765

```cpp
    if constexpr (cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD_MULTICAST>) {
      auto block_layout = Layout<typename DispatchPolicy::ClusterShape>{}; // (m,n) -> block_id
      for (int m = 0; m < size<0>(block_layout); ++m) {
        mcast_mask_b |= (uint16_t(1) << block_layout(m,cluster_local_block_id.y,Int<0>{}));
      }
    }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 767-767

```cpp
    auto extra_input_partitions = Utils::partition_extra_tma_inputs(mainloop_params, load_inputs, shared_tensors, cluster_local_block_id, m_coord, l_coord);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 769-774

```cpp
    // Mainloop
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count)
    {
      // LOCK smem_pipe_write for _writing_
      pipeline.producer_acquire(smem_pipe_write);
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 776-778

```cpp
      //
      // Copy gmem to smem for *k_tile_iter
      //
```
**EN:** This comment-only block labels or explains the following section:  Copy gmem to smem for *k_tile_iter .
**CN:** 这个纯注释块用于标记或解释后续区域： Copy gmem to smem for *k_tile_iter 。

### Lines 780-781

```cpp
      using BarrierType = typename MainloopPipeline::ProducerBarrierType;
      BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 783-793

```cpp
      int write_stage = smem_pipe_write.index();
      if (cute::elect_one_sync()) {
        copy(mainloop_params.tma_load_a.with(get<0>(input_tensormaps), *tma_barrier, mcast_mask_a), tAgA(_,_,_,*k_tile_iter), tAsA(_,_,_,write_stage));
        copy(mainloop_params.tma_load_b.with(get<1>(input_tensormaps), *tma_barrier, mcast_mask_b), tBgB(_,_,_,*k_tile_iter), tBsB(_,_,_,write_stage));
      }
      if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
        // Nothing extra to do.
      }
      else if constexpr (ModeHasScales) {
        auto tSgS = get<0>(extra_input_partitions);
        auto tSsS = get<1>(extra_input_partitions);
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 795-802

```cpp
        // Temporary factor which will determine which k tile to reload from gmem. Needed so we don't modify tma transaction bytes
        // on the fly.
        // We must do a ceiling divide here to correctly handle with chunk_size == K. In that case, we don't require that K
        // is a multiple of the threadblock tile K
        const int scale_load_k = *k_tile_iter / mainloop_params.reload_factor; // This will always be 0 when chunk_size == K.
        if (cute::elect_one_sync()) {
          copy(mainloop_params.tma_load_scale.with(get<2>(input_tensormaps), *tma_barrier, mcast_mask_s), tSgS(_,_,_,scale_load_k), tSsS(_,_,_,write_stage));
        }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 804-821

```cpp
        if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
          // Nothing extra to do
        }
        else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
          auto tZgZ = get<2>(extra_input_partitions);
          auto tZsZ = get<3>(extra_input_partitions);
          if (cute::elect_one_sync()) {
            copy(mainloop_params.tma_load_zero.with(get<3>(input_tensormaps), *tma_barrier, mcast_mask_s), tZgZ(_,_,_,scale_load_k), tZsZ(_,_,_,write_stage));
          }
        }
        else {
          static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled for TMA copy op.");
        }
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled for TMA copy op.");
      }
      ++k_tile_iter;
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 823-826

```cpp
      // Advance smem_pipe_write
      ++smem_pipe_write;
    }
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Advance smem_pipe_write.
**CN:** 周围注释解释了这一段的局部作用：Advance smem_pipe_write。

### Lines 828-831

```cpp
  // Perform a Producer Epilogue to prevent early exit of blocks in a Cluster
  CUTLASS_DEVICE void
  load_tail(MainloopPipeline pipeline, PipelineState smem_pipe_write) {
    int lane_predicate = cute::elect_one_sync();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 833-842

```cpp
    // Issue the epilogue waits
    if (lane_predicate) {
      // This helps avoid early exit of blocks in Cluster.
      // Waits for all stages to either be released (all
      // Consumer UNLOCKs), or if the stage was never used
      // then it would just be acquired since the phase was
      // still inverted from make_producer_start_state.
      pipeline.producer_tail(smem_pipe_write);
    }
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 844-856

```cpp
  /// Perform a collective-scoped matrix multiply-accumulate
  /// Consumer Perspective
  template <
    class FrgTensorC
  >
  CUTLASS_DEVICE void
  mma(MainloopPipeline pipeline,
      PipelineState smem_pipe_read,
      FrgTensorC& accum,
      int k_tile_count,
      int thread_idx,
      TensorStorage& shared_tensors,
      Params const& mainloop_params) {
```
**EN:** This block introduces `FrgTensorC` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `FrgTensorC`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 858-866

```cpp
    static_assert(is_rmem<FrgTensorC>::value, "C tensor must be rmem resident.");
    static_assert(cute::rank(SmemLayoutA{}) == 3, "Smem layout must be rank 3.");
    static_assert(cute::rank(SmemLayoutB{}) == 3, "Smem layout must be rank 3.");
    static_assert(cute::rank(SwappedSmemLayoutAtomA{}) == 2, "SwappedSmemLayoutAtomA must be rank 2.");
    static_assert(cute::rank(SwappedSmemLayoutAtomB{}) == 2, "SwappedSmemLayoutAtomB must be rank 2.");
    static_assert(!cute::is_void_v<SwappedSmemCopyAtomA>,
      "SM90 GMMA mainloops must specify a non-void copy atom for smem sourced instructions.");
    static_assert(cute::is_void_v<SwappedSmemCopyAtomB>,
      "SM90 GMMA mainloops cannot have a non-void copy atom for smem sourced instructions.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 868-870

```cpp
    // Obtain warp index
    int warp_idx = canonical_warp_idx_sync();
    [[maybe_unused]] int warp_group_thread_idx = thread_idx % 128;
```
**EN:** This comment highlights a warp-specialized design choice for the following implementation.
**CN:** 这条注释强调后续实现采用了 warp-specialized 的设计。

### Lines 873-874

```cpp
    Tensor sA_ = make_tensor(make_smem_ptr(shared_tensors.smem_A.begin()), SmemLayoutA{});        // (BLK_M,BLK_K,PIPE)
    Tensor sA = as_position_independent_swizzle_tensor(sA_);                                      // (BLK_M,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 876-876

```cpp
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.begin()), SmemLayoutB{});         // (BLK_N,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 879-881

```cpp
    //
    // Define C accumulators and A/B partitioning
    //
```
**EN:** This comment-only block labels or explains the following section:  Define C accumulators and A/B partitioning .
**CN:** 这个纯注释块用于标记或解释后续区域： Define C accumulators and A/B partitioning 。

### Lines 883-883

```cpp
    // Layout of warp group to thread mapping
```
**EN:** This comment highlights a warp-specialized design choice for the following implementation.
**CN:** 这条注释强调后续实现采用了 warp-specialized 的设计。

### Lines 885-887

```cpp
    static_assert(stride<0>(typename TiledMma::BLayout{}) == 0 and
                  size<0>(typename TiledMma::BLayout{}) == NumThreadsPerWarpGroup,
                  "Stride of the first mode must be 0 and the size of the mode must be NumThreadsPerWarpGroup");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 889-891

```cpp
    constexpr int MmaWarpGroups = size(TiledMma{}) / NumThreadsPerWarpGroup;
    Layout warp_group_thread_layout = make_layout(Int<MmaWarpGroups>{},
                                                  Int<NumThreadsPerWarpGroup>{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 893-893

```cpp
    int warp_group_idx = shfl_sync(0xFFFFFFFF, thread_idx / NumThreadsPerWarpGroup, 0);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 895-898

```cpp
    TiledMma tiled_mma;
    auto mma_thread_slice = tiled_mma.get_thread_slice(thread_idx);
    Tensor tCsA = mma_thread_slice.partition_A(sA);
    auto mma_warpgroup_slice = tiled_mma.get_slice(warp_group_thread_layout(warp_group_idx));
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 900-913

```cpp
    // Allocate fragments and descriptors
    Tensor tCrA_mma = mma_thread_slice.partition_fragment_A(sA(_,_,Int<0>{}));                // (MMA,MMA_M,MMA_K,PIPE)
    Tensor tCrA_load = [&]{
      if constexpr (not is_layout<InternalSwappedStrideA>::value) {
        // Make register tensor with MMA layout
        return make_fragment_like<RealSwappedElementA>(tCrA_mma);
      }
      else {
        // Make register tensor matching smem layout, converter will take care of de-swizzling
        return make_tensor_like<RealSwappedElementA>(tCsA(_,_,_,Int<0>{}));
      }
    }();
    Tensor tCsB = mma_warpgroup_slice.partition_B(sB);                                        // (MMA,MMA_N,MMA_K,PIPE)
    Tensor tCrB = mma_warpgroup_slice.make_fragment_B(tCsB);                                  // (MMA,MMA_N,MMA_K,PIPE)
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 915-919

```cpp
    //
    // Copy Atom A retiling
    //
    auto smem_tiled_copy_A = make_tiled_copy_A(SwappedSmemCopyAtomA{}, tiled_mma);
    auto smem_thr_copy_A   = smem_tiled_copy_A.get_thread_slice(warp_group_thread_idx);
```
**EN:** The surrounding comments explain the local purpose of this block:  Copy Atom A retiling .
**CN:** 周围注释解释了这一段的局部作用： Copy Atom A retiling 。

### Lines 921-921

```cpp
    Tensor tCrA_copy_view  = smem_thr_copy_A.retile_D(tCrA_load);                                  // (CPY,CPY_M,CPY_K)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 923-925

```cpp
    // Partition of thread -> shared and thread -> RF
    auto partitioned_extra_info = Utils::partition_extra_mma_info(mma_thread_slice, shared_tensors);
    auto copy_partitions_extra_info = Utils::retile_extra_mma_info(tiled_mma, partitioned_extra_info, warp_group_thread_idx);
```
**EN:** The surrounding comments explain the local purpose of this block: Partition of thread -> shared and thread -> RF.
**CN:** 周围注释解释了这一段的局部作用：Partition of thread -> shared and thread -> RF。

### Lines 927-934

```cpp
    CUTE_STATIC_ASSERT_V(size<1>(tCsA) == size<1>(tCrA_copy_view));                                            // CPY_M
    CUTE_STATIC_ASSERT_V(size<2>(tCsA) == size<2>(tCrA_copy_view));                                            // CPY_K
    CUTE_STATIC_ASSERT_V(size<1>(tCrA_mma) == size<1>(accum));                                                 // MMA_M
    CUTE_STATIC_ASSERT_V(size<1>(tCsB) == size<2>(accum));                                                         // N
    CUTE_STATIC_ASSERT_V(size<2>(tCsA) == size<2>(tCsB));                                                          // K
    CUTE_STATIC_ASSERT_V(size<3>(tCsA) == size<3>(tCsB));                                                       // PIPE
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<2>(sA));                                         // PIPE
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<2>(sB));                                         // PIPE
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 936-938

```cpp
    //
    // PIPELINED MAIN LOOP
    //
```
**EN:** This comment-only block labels or explains the following section:  PIPELINED MAIN LOOP .
**CN:** 这个纯注释块用于标记或解释后续区域： PIPELINED MAIN LOOP 。

### Lines 940-941

```cpp
    // We release buffers to producer warps(dma load) with some mmas in flight
    PipelineState smem_pipe_release = smem_pipe_read;
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 943-943

```cpp
    tiled_mma.accumulate_ = GMMA::ScaleOut::Zero;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 945-945

```cpp
    warpgroup_fence_operand(accum);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 947-949

```cpp
    constexpr int K_BLOCK_MAX = size<2>(tCrA_load);
    constexpr int K_WAIT_MAX = cute::min(K_BLOCK_MAX - 1, 7);
    static_assert(K_BLOCK_MAX >= 4, "Consider increasing TileShapeK");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 951-955

```cpp
    ConsumerToken barrier_token = {BarrierStatus::WaitAgain};
    // first k tile
    {
      barrier_token = pipeline.consumer_try_wait(smem_pipe_read);
      pipeline.consumer_wait(smem_pipe_read, barrier_token);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 957-957

```cpp
      int read_stage = smem_pipe_read.index();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 959-960

```cpp
      ++smem_pipe_read;
      barrier_token = pipeline.consumer_try_wait(smem_pipe_read);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 962-962

```cpp
      // copy smem->rmem for A operand
```
**EN:** This comment-only block labels or explains the following section: copy smem->rmem for A operand.
**CN:** 这个纯注释块用于标记或解释后续区域：copy smem->rmem for A operand。

### Lines 964-969

```cpp
      Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
        partitioned_extra_info, copy_partitions_extra_info, 0, read_stage);
      if (K_BLOCK_MAX > 1) {
        Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
          partitioned_extra_info, copy_partitions_extra_info, 1, read_stage);
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 971-971

```cpp
      Utils::dequantize_A_kblock(tCrA_load, tCrA_mma, partitioned_extra_info, 0);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 973-980

```cpp
      // Unroll the K mode manually to set scale D to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < K_BLOCK_MAX; ++k_block) {
        warpgroup_arrive();
        // (V,M) x (V,N) => (V,M,N)
        cute::gemm(tiled_mma, tCrA_mma(_,_,k_block), tCrB(_,_,k_block,read_stage), accum);
        tiled_mma.accumulate_ = GMMA::ScaleOut::One;
        warpgroup_commit_batch();
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 982-989

```cpp
        if (k_block < K_BLOCK_MAX - 2) {
          Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
            partitioned_extra_info, copy_partitions_extra_info, k_block + 2, read_stage);
        }
        if (k_block < K_BLOCK_MAX - 1) {
          Utils::dequantize_A_kblock(tCrA_load, tCrA_mma, partitioned_extra_info, k_block + 1);
        }
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 991-994

```cpp
      --k_tile_count;
      if (k_tile_count > 0) {
        // Wait for K_BLOCK_MAX - 1 to be in flight to ensure that it is safe to overwrite the A registers for the first mma.
        pipeline.consumer_wait(smem_pipe_read, barrier_token);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 996-997

```cpp
        Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
          partitioned_extra_info, copy_partitions_extra_info, 0, smem_pipe_read.index());
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 999-1000

```cpp
        Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
          partitioned_extra_info, copy_partitions_extra_info, 1, smem_pipe_read.index());
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1002-1005

```cpp
        warpgroup_wait<K_WAIT_MAX>();
        Utils::dequantize_A_kblock(tCrA_load, tCrA_mma, partitioned_extra_info, 0);
      }
    }
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1007-1009

```cpp
    if (k_tile_count == 0) {
      return;
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1011-1014

```cpp
    warpgroup_fence_operand(accum);
    // Mainloop GMMAs
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 1; --k_tile_count) {
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1016-1018

```cpp
      //
      // Compute on k_tile
      //
```
**EN:** This comment-only block labels or explains the following section:  Compute on k_tile .
**CN:** 这个纯注释块用于标记或解释后续区域： Compute on k_tile 。

### Lines 1020-1021

```cpp
      int read_stage = smem_pipe_read.index();
      ++smem_pipe_read;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1023-1026

```cpp
      warpgroup_fence_operand(accum);
      // Unroll the K mode manually to set scale D to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < K_BLOCK_MAX; ++k_block) {
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1028-1032

```cpp
        warpgroup_arrive();
        // (V,M) x (V,N) => (V,M,N)
        cute::gemm(tiled_mma, tCrA_mma(_,_,k_block), tCrB(_,_,k_block,read_stage), accum);
        tiled_mma.accumulate_ = GMMA::ScaleOut::One;
        warpgroup_commit_batch();
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1034-1038

```cpp
        warpgroup_wait<K_WAIT_MAX>(); // We have K_BLOCK_MAX - 1 GMMA instructions pending for this stage, so we can release prior barrier
        if (k_block == K_BLOCK_MAX - 1) {
          pipeline.consumer_release(smem_pipe_release);             // UNLOCK smem_pipe_release, done _computing_ on it
          ++smem_pipe_release;
        }
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1040-1042

```cpp
        if (k_block == 0) {
          barrier_token = pipeline.consumer_try_wait(smem_pipe_read);
        }
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1044-1047

```cpp
        if (k_block == K_BLOCK_MAX - 1) {
          pipeline.consumer_wait(smem_pipe_read, barrier_token);
          Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
            partitioned_extra_info, copy_partitions_extra_info, 0, smem_pipe_read.index());
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1049-1061

```cpp
          Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
            partitioned_extra_info, copy_partitions_extra_info, 1, smem_pipe_read.index());
          Utils::dequantize_A_kblock(tCrA_load, tCrA_mma, partitioned_extra_info, 0);
        }
        else {
          if (k_block < K_BLOCK_MAX - 2) {
            Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
              partitioned_extra_info, copy_partitions_extra_info, k_block + 2, read_stage);
          }
          Utils::dequantize_A_kblock(tCrA_load, tCrA_mma, partitioned_extra_info, k_block + 1);
        }
      }
      warpgroup_fence_operand(accum);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1063-1063

```cpp
    }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1065-1065

```cpp
    warpgroup_fence_operand(accum);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1067-1070

```cpp
    {
      //
      // Compute on k_tile
      //
```
**EN:** The surrounding comments explain the local purpose of this block:  Compute on k_tile .
**CN:** 周围注释解释了这一段的局部作用： Compute on k_tile 。

### Lines 1072-1072

```cpp
      int read_stage = smem_pipe_read.index();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1074-1074

```cpp
      warpgroup_fence_operand(accum);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1076-1078

```cpp
      // Unroll the K mode manually to set scale D to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < K_BLOCK_MAX; ++k_block) {
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1080-1084

```cpp
        warpgroup_arrive();
        // (V,M) x (V,N) => (V,M,N)
        cute::gemm(tiled_mma, tCrA_mma(_,_,k_block), tCrB(_,_,k_block,read_stage), accum);
        tiled_mma.accumulate_ = GMMA::ScaleOut::One;
        warpgroup_commit_batch();
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1086-1091

```cpp
        warpgroup_wait<K_WAIT_MAX>();
        if (k_block == K_BLOCK_MAX - 1) {
          // release prior barrier
          pipeline.consumer_release(smem_pipe_release);             // UNLOCK smem_pipe_release, done _computing_ on it
          ++smem_pipe_release;
        }
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1093-1101

```cpp
        if (k_block < K_BLOCK_MAX - 2) {
          Utils::copy_tensors_MK(smem_tiled_copy_A, tCsA, tCrA_copy_view,
            partitioned_extra_info, copy_partitions_extra_info, k_block + 2, read_stage);
        }
        if (k_block < K_BLOCK_MAX - 1) {
          Utils::dequantize_A_kblock(tCrA_load, tCrA_mma, partitioned_extra_info, k_block + 1);
        }
      }
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1103-1104

```cpp
    warpgroup_fence_operand(accum);
  }
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1106-1111

```cpp
  /// Perform a Consumer Epilogue to release all buffers
  CUTLASS_DEVICE void
  mma_tail(MainloopPipeline pipeline, PipelineState smem_pipe_release, int k_tile_count) {
    // Prologue GMMAs
    int prologue_mma_count = 1;
    k_tile_count -= prologue_mma_count;
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1113-1113

```cpp
    smem_pipe_release.advance(k_tile_count);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1115-1116

```cpp
    // Wait on all GMMAs to complete
    warpgroup_wait<0>();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1118-1122

```cpp
    for (int count = 0; count < prologue_mma_count; ++count) {
      pipeline.consumer_release(smem_pipe_release);                 // UNLOCK smem_pipe_release, done _computing_ on it
      ++smem_pipe_release;
    }
  }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1124-1133

```cpp
  //
  // Methods to perform different parts of TMA/Tensormap modifications
  //
  CUTLASS_DEVICE auto
  tensormaps_init(
      Params const& mainloop_params,
      TensorMapStorage& shared_tensormaps,
      int32_t sm_count,
      int32_t sm_idx) {
    cute::TmaDescriptor* gmem_tensormap = reinterpret_cast<cute::TmaDescriptor*>(mainloop_params.tensormaps);
```
**EN:** This comment starts a method section where the collective exposes its runtime behavior.
**CN:** 这条注释表示进入方法区，collective 的运行时行为将在这里定义。

### Lines 1135-1138

```cpp
    cute::TmaDescriptor* tma_desc_a = &gmem_tensormap[sm_idx];
    cute::TmaDescriptor* tma_desc_b = &gmem_tensormap[sm_idx + sm_count];
    cute::TmaDescriptor* tma_desc_scale = &gmem_tensormap[sm_idx + 2*sm_count];
    cute::TmaDescriptor* tma_desc_zero = &gmem_tensormap[sm_idx + 3*sm_count];
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1140-1144

```cpp
    // Bringing tensormaps from params to smem for modification later
    Tensor pA_tensormap = make_tensor(mainloop_params.tma_load_a.get_tma_descriptor(), Int<1>{}, Int<1>{});
    Tensor sA_tensormap = make_tensor(make_smem_ptr(&shared_tensormaps.smem_tensormap_A), Int<1>{}, Int<1>{});
    Tensor pB_tensormap = make_tensor(mainloop_params.tma_load_b.get_tma_descriptor(), Int<1>{}, Int<1>{});
    Tensor sB_tensormap = make_tensor(make_smem_ptr(&shared_tensormaps.smem_tensormap_B), Int<1>{}, Int<1>{});
```
**EN:** The surrounding comments explain the local purpose of this block: Bringing tensormaps from params to smem for modification later.
**CN:** 周围注释解释了这一段的局部作用：Bringing tensormaps from params to smem for modification later。

### Lines 1146-1149

```cpp
    if (cute::elect_one_sync()) {
      copy(recast<uint128_t>(pA_tensormap), recast<uint128_t>(sA_tensormap));
      copy(recast<uint128_t>(pB_tensormap), recast<uint128_t>(sB_tensormap));
    }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1151-1167

```cpp
    if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      Tensor pS_tensormap = make_tensor(mainloop_params.tma_load_scale.get_tma_descriptor(), Int<1>{}, Int<1>{});
      Tensor sS_tensormap = make_tensor(make_smem_ptr(&shared_tensormaps.smem_tensormap_scale), Int<1>{}, Int<1>{});
      if (cute::elect_one_sync()) {
        copy(recast<uint128_t>(pS_tensormap), recast<uint128_t>(sS_tensormap));
      }
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      Tensor pZ_tensormap = make_tensor(mainloop_params.tma_load_zero.get_tma_descriptor(), Int<1>{}, Int<1>{});
      Tensor sZ_tensormap = make_tensor(make_smem_ptr(&shared_tensormaps.smem_tensormap_zero), Int<1>{}, Int<1>{});
      if (cute::elect_one_sync()) {
        copy(recast<uint128_t>(pZ_tensormap), recast<uint128_t>(sZ_tensormap));
      }
    }
    else if constexpr (KernelConversionMode != ConversionMode::DirectConvert){
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in tensormaps_init.");
    }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1169-1169

```cpp
    syncwarp();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1171-1183

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return cute::make_tuple(tma_desc_a, tma_desc_b);
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      return cute::make_tuple(tma_desc_a, tma_desc_b, tma_desc_scale);
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      return cute::make_tuple(tma_desc_a, tma_desc_b, tma_desc_scale, tma_desc_zero);
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in tensormaps_init.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1185-1208

```cpp
  // Replace address for the global tensor (to be done by single thread)
  CUTLASS_DEVICE
  void
  tensormaps_replace_global_address(
      TensorMapStorage& shared_tensormaps,
      Params const& mainloop_params,
      int32_t next_batch) {
    // Replacing global_address for the next batch
    cute::tma_descriptor_replace_addr_in_shared_mem(shared_tensormaps.smem_tensormap_A,
                                                    mainloop_params.ptr_A[next_batch]);
    cute::tma_descriptor_replace_addr_in_shared_mem(shared_tensormaps.smem_tensormap_B,
                                                    mainloop_params.ptr_B[next_batch]);
    if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      cute::tma_descriptor_replace_addr_in_shared_mem(shared_tensormaps.smem_tensormap_scale,
                                                    mainloop_params.ptr_S[next_batch]);
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      cute::tma_descriptor_replace_addr_in_shared_mem(shared_tensormaps.smem_tensormap_zero,
                                                    mainloop_params.ptr_Z[next_batch]);
    }
    else if constexpr (KernelConversionMode != ConversionMode::DirectConvert){
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in tensormaps_replace_global_address.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1210-1221

```cpp
  // Replace dim and strides for the global tensor - used only for Grouped GEMM (to be done by single thread)
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE
  void
  tensormaps_replace_global_tensor_properties(
      TensorMapStorage& shared_tensormaps,
      Params const& mainloop_params,
      int32_t next_group,
      ProblemShape_MNKL problem_shape_mnkl) {
    const uint32_t M = (SwapAB? get<1>(problem_shape_mnkl) : get<0>(problem_shape_mnkl));
    const uint32_t N = (SwapAB? get<0>(problem_shape_mnkl) : get<1>(problem_shape_mnkl));
    const uint32_t K = get<2>(problem_shape_mnkl);
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。

### Lines 1223-1232

```cpp
    // Replace all dims for consistency
    constexpr int MaxTensorRank = 5;
    cute::array<uint32_t, MaxTensorRank> prob_shape_A  = {1,1,1,1,1};
    cute::array<uint64_t, MaxTensorRank> prob_stride_A = {0,0,0,0,0};
    cute::array<uint32_t, MaxTensorRank> prob_shape_B  = {1,1,1,1,1};
    cute::array<uint64_t, MaxTensorRank> prob_stride_B = {0,0,0,0,0};
    cute::array<uint32_t, MaxTensorRank> prob_shape_scale  = {1,1,1,1,1};
    cute::array<uint64_t, MaxTensorRank> prob_stride_scale = {0,0,0,0,0};
    cute::array<uint32_t, MaxTensorRank> prob_shape_zero   = {1,1,1,1,1};
    cute::array<uint64_t, MaxTensorRank> prob_stride_zero  = {0,0,0,0,0};
```
**EN:** The surrounding comments explain the local purpose of this block: Replace all dims for consistency.
**CN:** 周围注释解释了这一段的局部作用：Replace all dims for consistency。

### Lines 1234-1235

```cpp
    SwappedElementA const* ptr_A = nullptr;
    Tensor tensor_a = make_tensor(ptr_A, detail::get_gmem_layout(make_shape(M,K,Int<1>{}), mainloop_params.ptr_dA[next_group]));
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 1237-1238

```cpp
    SwappedElementB const* ptr_B = nullptr;
    Tensor tensor_b = make_tensor(ptr_B, detail::get_gmem_layout(make_shape(N,K,Int<1>{}), mainloop_params.ptr_dB[next_group]));
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 1240-1243

```cpp
    cute::detail::fill_tma_gmem_shape_stride(mainloop_params.tma_load_a, tensor_a,
                                             prob_shape_A, prob_stride_A);
    cute::detail::fill_tma_gmem_shape_stride(mainloop_params.tma_load_b, tensor_b,
                                             prob_shape_B, prob_stride_B);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1245-1261

```cpp
    if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      NonVoidElementScale const* ptr_S = nullptr;
      auto scale_k = ceil_div(K, mainloop_params.chunk_size);
      Tensor tensor_scale = make_tensor(detail::get_logical_ptr(ptr_S), make_shape(M,scale_k,Int<1>{}), mainloop_params.dS[next_group]);
      cute::detail::fill_tma_gmem_shape_stride(mainloop_params.tma_load_scale, tensor_scale,
                                             prob_shape_scale, prob_stride_scale);
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      ElementZero const* ptr_Z = nullptr;
      auto scale_k = ceil_div(K, mainloop_params.chunk_size);
      Tensor tensor_zero = make_tensor(detail::get_logical_ptr(ptr_Z), make_shape(M,scale_k,Int<1>{}), mainloop_params.dS[next_group]);
      cute::detail::fill_tma_gmem_shape_stride(mainloop_params.tma_load_zero, tensor_zero,
                                               prob_shape_zero, prob_stride_zero);
    }
    else if constexpr (KernelConversionMode != ConversionMode::DirectConvert){
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in tensormaps_replace_global_tensor_properties.");
    }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1263-1275

```cpp
    // Convert strides to byte strides
    for (uint64_t& stride : prob_stride_A) {
      stride = (stride * sizeof_bits_v<SwappedElementA>) / 8;
    }
    for (uint64_t& stride : prob_stride_B) {
      stride = (stride * sizeof_bits_v<SwappedElementB>) / 8;
    }
    for (uint64_t& stride : prob_stride_scale) {
      stride = (stride * sizeof_bits_v<NonVoidElementScale>) / 8;
    }
    for (uint64_t& stride : prob_stride_zero) {
      stride = (stride * sizeof_bits_v<NonVoidElementScale>) / 8;
    }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1278-1283

```cpp
    cute::tma_descriptor_replace_dims_strides_in_shared_mem(shared_tensormaps.smem_tensormap_A,
                                                            prob_shape_A,
                                                            prob_stride_A);
    cute::tma_descriptor_replace_dims_strides_in_shared_mem(shared_tensormaps.smem_tensormap_B,
                                                            prob_shape_B,
                                                            prob_stride_B);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1285-1298

```cpp
    if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      cute::tma_descriptor_replace_dims_strides_in_shared_mem(shared_tensormaps.smem_tensormap_scale,
                                                            prob_shape_scale,
                                                            prob_stride_scale);
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      cute::tma_descriptor_replace_dims_strides_in_shared_mem(shared_tensormaps.smem_tensormap_zero,
                                                            prob_shape_zero,
                                                            prob_stride_zero);
    }
    else if constexpr (KernelConversionMode != ConversionMode::DirectConvert){
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in tensormaps_replace_global_tensor_properties.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1300-1311

```cpp
  template <class... TMs, class ProblemShape_MNKL>
  CUTLASS_DEVICE
  void
  tensormaps_perform_update(
      TensorMapStorage& shared_tensormaps,
      Params const& mainloop_params,
      cute::tuple<TMs...> const& input_tensormaps,
      ProblemShape_MNKL problem_shape_mnkl,
      int32_t next_batch) {
    if (cute::elect_one_sync()) {
      // Replacing global_address for the next batch
      tensormaps_replace_global_address(shared_tensormaps, mainloop_params, next_batch);
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1313-1319

```cpp
      if constexpr (IsGroupedGemmKernel) {
        // Replacing global dims and strides for the next batch
        tensormaps_replace_global_tensor_properties(shared_tensormaps,
          mainloop_params, next_batch, problem_shape_mnkl);
      }
    }
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1321-1343

```cpp
  template <class... TMs>
  CUTLASS_DEVICE
  void
  tensormaps_cp_fence_release (
      TensorMapStorage& shared_tensormaps,
      cute::tuple<TMs...> const& input_tensormaps) {
    if (cute::elect_one_sync()) {
      cute::tma_desc_commit_group();
      cute::tma_desc_wait_group();
    }
    // Entire warp must do this (i.e. it's aligned)
    tma_descriptor_cp_fence_release(get<0>(input_tensormaps), shared_tensormaps.smem_tensormap_A);
    tma_descriptor_cp_fence_release(get<1>(input_tensormaps), shared_tensormaps.smem_tensormap_B);
    if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      tma_descriptor_cp_fence_release(get<2>(input_tensormaps), shared_tensormaps.smem_tensormap_scale);
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      tma_descriptor_cp_fence_release(get<3>(input_tensormaps), shared_tensormaps.smem_tensormap_zero);
    }
    else if constexpr (KernelConversionMode != ConversionMode::DirectConvert){
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in tensormaps_cp_fence_release.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1345-1361

```cpp
  // The entire warp must call this function collectively (that is, the instructions are aligned)
  template <class... TMs>
  CUTLASS_DEVICE
  void
  tensormaps_fence_acquire(cute::tuple<TMs...> const& input_tensormaps) {
    cute::tma_descriptor_fence_acquire(get<0>(input_tensormaps));
    cute::tma_descriptor_fence_acquire(get<1>(input_tensormaps));
    if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      cute::tma_descriptor_fence_acquire(get<2>(input_tensormaps));
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      cute::tma_descriptor_fence_acquire(get<3>(input_tensormaps));
    }
    else if constexpr (KernelConversionMode != ConversionMode::DirectConvert){
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in tensormaps_fence_acquire.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1363-1372

```cpp
  template <class InputTensors, class ProblemShape_MNKL>
  CUTLASS_DEVICE
  InputTensors
  tensors_perform_update(
      InputTensors const& input_tensors,
      [[maybe_unused]] Params const& mainloop_params,
      [[maybe_unused]] ProblemShape_MNKL problem_shape_mnkl,
      [[maybe_unused]] int32_t next_batch) {
    return input_tensors;
  }
```
**EN:** This block introduces `InputTensors` and groups related declarations around that symbol. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这一段引入了 `InputTensors`，并围绕该符号组织相关声明。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1374-1374

```cpp
};
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1376-1376

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 1378-1378

```cpp
} // namespace cutlass::gemm::collective
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1380-1380

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

## Key Concepts / 关键概念

- **EN:** TMA data movement  
  **CN:** TMA 数据搬运
- **EN:** GMMA tensor-core instructions  
  **CN:** GMMA 张量核指令
- **EN:** warp-specialized scheduling  
  **CN:** warp-specialized 调度
- **EN:** mixed-input operands  
  **CN:** 混合输入操作数
- **EN:** array-style MMA tiling  
  **CN:** 数组式 MMA 分块
- **EN:** Intel Xe subgroup execution  
  **CN:** Intel Xe 子组执行
- **EN:** software pipelines  
  **CN:** 软件流水线
- **EN:** compile-time validation  
  **CN:** 编译期校验
- **EN:** collective GEMM mainloop specialization  
  **CN:** collective GEMM 主循环特化

## Dependencies / 依赖关系

- **Direct includes / 直接包含:**
  - `cutlass/cutlass.h`
  - `cutlass/gemm/dispatch_policy.hpp`
  - `cutlass/numeric_types.h`
  - `cutlass/pipeline/pipeline.hpp`
  - `cutlass/trace.h`
  - `cutlass/cuda_host_adapter.hpp`
  - `cutlass/detail/collective/mixed_input_utils.hpp`
  - `cute/arch/cluster_sm90.hpp`
  - `cute/arch/copy_sm90.hpp`
  - `cute/algorithm/functional.hpp`
  - `cute/atom/mma_atom.hpp`
  - `cute/algorithm/gemm.hpp`
  - `cute/numeric/arithmetic_tuple.hpp`
- **Primary symbols / 主要符号:**
  - `ClusterShape`
  - `KernelSchedule_`
  - `TileShape_`
  - `ElementAOptionalTuple`
  - `StrideA_`
  - `ElementBOptionalTuple`
  - `StrideB_`
  - `TiledMma_`
  - `GmemTiledCopyA_`
  - `SmemLayoutAtomA_`
  - `SmemCopyAtomA_`
  - `TransformA_`
  - `GmemTiledCopyB_`
  - `SmemLayoutAtomB_`
  - `SmemCopyAtomB_`
  - `TransformB_`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。