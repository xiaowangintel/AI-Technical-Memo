# sm100_mma_warpspecialized_mixed_input.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/sm100_mma_warpspecialized_mixed_input.hpp`
- **Purpose (EN):** Implements an architecture-specific `CollectiveMma` specialization for SM100, covering mixed input types, warp-specialized scheduling, Intel Xe subgroup execution.
- **用途 (CN):** 为 SM100 实现架构特化的 `CollectiveMma` 特化版本，重点覆盖 混合输入类型、warp-specialized 调度、Intel Xe 子组执行。
- **Lines / 行数:** 1298

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

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
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 32-35

```cpp
#pragma once
#if !defined(CUTLASS_ENABLE_SYCL)
#include <cuda_bf16.h>
#endif
```
**EN:** This include block imports cuda_bf16.h. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cuda_bf16.h。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 37-46

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/pipeline/pipeline.hpp"
#include "cutlass/numeric_conversion.h"
#include "cutlass/detail/sm100_tmem_helper.hpp"
#include "cutlass/detail/cluster.hpp"
#include "cutlass/detail/collective/mixed_input_utils.hpp"
#include "cutlass/detail/sm100_mixed_dtype_blockwise_layout.hpp"
#include "cutlass/detail/blockwise_scale_layout.hpp"
```
**EN:** This include block imports cutlass.h, gemm.h, dispatch_policy.hpp, pipeline.hpp, and 6 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cutlass.h、gemm.h、dispatch_policy.hpp、pipeline.hpp 等 10 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 48-55

```cpp
#include "cute/algorithm/functional.hpp"
#include "cute/arch/cluster_sm90.hpp"
#include "cute/atom/mma_atom.hpp"
#include "cute/atom/copy_atom.hpp"
#include "cute/algorithm/gemm.hpp"
#include "cute/arch/mma_sm100.hpp"
#include "cutlass/trace.h"
#include "cutlass/kernel_hardware_info.hpp"
```
**EN:** This include block imports functional.hpp, cluster_sm90.hpp, mma_atom.hpp, copy_atom.hpp, and 4 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 functional.hpp、cluster_sm90.hpp、mma_atom.hpp、copy_atom.hpp 等 8 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 57-57

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 59-60

```cpp
namespace cutlass::gemm::collective {
using namespace cute;
```
**EN:** This short block both opens the collective namespace and pulls CuTe symbols into scope, setting up a concise vocabulary for the rest of the file.
**CN:** 这个短代码块既打开了 collective 命名空间，也把 CuTe 符号引入作用域，为后续代码建立更简洁的表达方式。

### Lines 62-62

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 64-110

```cpp
// WarpSpecialized Mainloop for Mixed Input Kernels
template <
  int Load2TransformPipelineStageCount_,
  int Transform2MmaPipelineStageCount_,
  int SchedulerPipelineStageCount_,
  int AccumulatorPipelineStageCount_,
  class ClusterShape,
  class TileShape_,
  class ElementAOptionalTuple_,
  class StridePairA_,
  class ElementBOptionalTuple_,
  class StrideB_,
  class TiledMma_,
  class GmemTiledCopyA_,
  class SmemLayoutAtomsA_,
  class CopyAtomsA_,
  class TransformA_,
  class GmemTiledCopyB_,
  class SmemLayoutAtomsB_,
  class CopyAtomsB_,
  class TransformB_>
struct CollectiveMma<
    MainloopSm100TmaUmmaWarpSpecializedMixedInput<
      Load2TransformPipelineStageCount_,
      Transform2MmaPipelineStageCount_,
      SchedulerPipelineStageCount_,
      AccumulatorPipelineStageCount_,
      ClusterShape>,
    TileShape_,
    ElementAOptionalTuple_,
    StridePairA_,
    ElementBOptionalTuple_,
    StrideB_,
    TiledMma_,
    GmemTiledCopyA_,
    SmemLayoutAtomsA_,
    CopyAtomsA_,
    TransformA_,
    GmemTiledCopyB_,
    SmemLayoutAtomsB_,
    CopyAtomsB_,
    TransformB_>
{
public:
  //
  // Type Aliases
  //
```
**EN:** This template block declares or specializes `CollectiveMma`, the mainloop object that coordinates tile movement and matrix-multiply work.
**CN:** 这个模板块声明或特化了 `CollectiveMma`，它负责协调 tile 搬运与矩阵乘主循环。

### Lines 112-127

```cpp
  using ConversionMode = cutlass::detail::ConversionMode;
  // Determine MMA type: MMA_1SM vs MMA_2SM
  using AtomThrShapeMNK = Shape<decltype(shape<0>(typename TiledMma_::ThrLayoutVMNK{})), _1, _1>;
  using DispatchPolicy = MainloopSm100TmaUmmaWarpSpecializedMixedInput<
                            Load2TransformPipelineStageCount_,
                            Transform2MmaPipelineStageCount_,
                            SchedulerPipelineStageCount_,
                            AccumulatorPipelineStageCount_,
                            ClusterShape>;
  using TileShape = TileShape_;
  using TiledMma = TiledMma_;
  using KernelSchedule = typename DispatchPolicy::Schedule;
  static constexpr bool IsDynamicCluster = not cute::is_static_v<ClusterShape>;
  using CtaShape_MNK = decltype(shape_div(TileShape{}, AtomThrShapeMNK{}));
  using ElementAOptionalTuple = ElementAOptionalTuple_;
  using ElementBOptionalTuple = ElementBOptionalTuple_;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 129-129

```cpp
private:
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 131-140

```cpp
  template<class T> friend struct detail::MixedInputUtils;
  using CollectiveType = CollectiveMma<DispatchPolicy, TileShape_, 
                                       ElementAOptionalTuple, StridePairA_, 
                                       ElementBOptionalTuple, StrideB_,
                                       TiledMma_, 
                                       GmemTiledCopyA_, SmemLayoutAtomsA_, CopyAtomsA_,
                                       TransformA_,
                                       GmemTiledCopyB_, SmemLayoutAtomsB_, CopyAtomsB_,
                                       TransformB_>;
  using Utils = detail::MixedInputUtils<CollectiveType>;
```
**EN:** This block introduces `T` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `T`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 142-145

```cpp
  using ElementScaleA = detail::deduce_mixed_width_dtype_t<1, ElementAOptionalTuple_>;
  using ElementScaleB = detail::deduce_mixed_width_dtype_t<1, ElementBOptionalTuple>;
  using ElementZeroA = detail::deduce_mixed_width_dtype_t<2, ElementAOptionalTuple>;
  using ElementZeroB = detail::deduce_mixed_width_dtype_t<2, ElementBOptionalTuple>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 147-150

```cpp
public:
  static_assert(cute::is_tuple<ElementAOptionalTuple>::value ^ cute::is_tuple<ElementBOptionalTuple>::value, 
    "Either A OR B must be a tuple. It must take the from {ElementOperand, [ElementScale],"
    "[ElementZero]}. Inputs in [] are optional.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 152-159

```cpp
  using ElementA = detail::deduce_mixed_width_dtype_t<0, ElementAOptionalTuple>;
  using ElementB = detail::deduce_mixed_width_dtype_t<0, ElementBOptionalTuple>;
  static constexpr bool IsATransformed = cute::is_tuple<ElementAOptionalTuple>::value;
  using ElementScale = cute::conditional_t<IsATransformed, ElementScaleA, ElementScaleB>;
  using ElementZero = cute::conditional_t<IsATransformed, ElementZeroA, ElementZeroB>;
  // For cases where we can't have a void type, we can use this to allow the code to compile when the scale / zero is void.
  using NonVoidElementScale = cute::conditional_t<cute::is_void_v<ElementScale>, float, ElementScale>;
  using NonVoidElementZero = cute::conditional_t<cute::is_void_v<ElementZero>, float, ElementZero>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 161-165

```cpp
  using StrideA = cute::remove_cvref_t<decltype(get<0>(StridePairA_{}))>;
  using LayoutScale = cute::remove_cvref_t<decltype(get<1>(StridePairA_{}))>;
  using InternalStrideA = cute::remove_pointer_t<StrideA>;
  using StrideB = StrideB_;
  using InternalStrideB = cute::remove_pointer_t<StrideB>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 167-169

```cpp
  static_assert((IsATransformed && cutlass::gemm::detail::is_k_major<StrideA>()) || 
                (!IsATransformed && cutlass::gemm::detail::is_k_major<StrideB>()),
                "The transformed type must be K-major.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 171-175

```cpp
  static_assert(( IsATransformed && (sizeof(ElementB) == 2)) ||
                (!IsATransformed && (sizeof(ElementA) == 2)) ||
                (cutlass::gemm::detail::is_k_major<StrideA>() && 
                 cutlass::gemm::detail::is_k_major<StrideB>()), 
                "The unscaled element must be 2 bytes OR both inputs must be K-major");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 177-179

```cpp
  // Define A and B block shapes for reduced size TMA_LOADs
  using CtaShapeA_MK = decltype(partition_shape_A(TiledMma{}, make_shape(size<0>(TileShape{}), size<2>(TileShape{}))));
  using CtaShapeB_NK = decltype(partition_shape_B(TiledMma{}, make_shape(size<1>(TileShape{}), size<2>(TileShape{}))));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 181-182

```cpp
  using ElementAMma = typename TiledMma::ValTypeA;
  using ElementBMma = typename TiledMma::ValTypeB;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 184-184

```cpp
  using ElementAccumulator = typename TiledMma::ValTypeC;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 186-188

```cpp
  using GmemTiledCopyA = GmemTiledCopyA_;
  using GmemTiledCopyB = GmemTiledCopyB_;
  using GmemTiledCopyScale = GmemTiledCopyA_;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 190-194

```cpp
  using SmemLayoutAtomsA = SmemLayoutAtomsA_;
  using SmemLayoutAtomsB = SmemLayoutAtomsB_;
  using CopyAtomsA = CopyAtomsA_;
  using CopyAtomsB = CopyAtomsB_;
  using SmemCopyAtomScale = Copy_Atom<cute::AutoVectorizingCopy, NonVoidElementScale>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 196-199

```cpp
  using SmemLayoutAtomA = typename SmemLayoutAtomsA::InputLayoutAtom;
  using SmemLayoutAtomACompute = typename SmemLayoutAtomsA::ComputeLayoutAtom;
  using SmemLayoutAtomB = typename SmemLayoutAtomsB::InputLayoutAtom;
  using SmemLayoutAtomBCompute = typename SmemLayoutAtomsB::ComputeLayoutAtom;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 201-204

```cpp
  using InputCopyAtomA = typename CopyAtomsA::InputCopyAtom;
  using ComputeCopyAtomA = typename CopyAtomsA::ComputeCopyAtom;
  using InputCopyAtomB = typename CopyAtomsB::InputCopyAtom;
  using ComputeCopyAtomB = typename CopyAtomsB::ComputeCopyAtom;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 206-211

```cpp
  // We must ensure the type to be scaled goes to RF
  static constexpr bool SwapAB = !IsATransformed;
  using InternalSmemLayoutAtomA = cute::conditional_t<!SwapAB, SmemLayoutAtomA, SmemLayoutAtomB>;
  using InternalSmemLayoutAtomB = cute::conditional_t<!SwapAB, SmemLayoutAtomB, SmemLayoutAtomA>;
  using InternalSmemLayoutAtomACompute = cute::conditional_t<!SwapAB, SmemLayoutAtomACompute, SmemLayoutAtomBCompute>;
  using InternalSmemLayoutAtomBCompute = cute::conditional_t<!SwapAB, SmemLayoutAtomBCompute, SmemLayoutAtomACompute>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 213-216

```cpp
  using InternalInputCopyAtomA   = cute::conditional_t<!SwapAB, InputCopyAtomA, InputCopyAtomB>;
  using InternalInputCopyAtomB   = cute::conditional_t<!SwapAB, InputCopyAtomB, InputCopyAtomA>;
  using InternalComputeCopyAtomA   = cute::conditional_t<!SwapAB, ComputeCopyAtomA, ComputeCopyAtomB>;
  using InternalComputeCopyAtomB   = cute::conditional_t<!SwapAB, ComputeCopyAtomB, ComputeCopyAtomA>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 218-231

```cpp
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
  using SwappedStrideA = cute::conditional_t<!SwapAB, StrideA, StrideB>;
  using SwappedStrideB = cute::conditional_t<!SwapAB, StrideB, StrideA>;
  using InternalSwappedStrideA = cute::conditional_t<!SwapAB, InternalStrideA, InternalStrideB>;
  using InternalSwappedStrideB = cute::conditional_t<!SwapAB, InternalStrideB, InternalStrideA>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 233-236

```cpp
  using TransformA = TransformA_;
  using TransformB = TransformB_;
  using InternalTransformA  = cute::conditional_t<!SwapAB, TransformA, TransformB>;
  using InternalTransformB  = cute::conditional_t<!SwapAB, TransformB, TransformA>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 238-240

```cpp
  static constexpr int IsSubbyteA = cute::sizeof_bits_v<SwappedElementA> < 8;
  using TmaElementA = cute::conditional_t<IsSubbyteA, uint8_t, SwappedElementA>;
  using TmaElementScale = uint_bit_t<sizeof_bits_v<NonVoidElementScale> >; // in case we have array. translating to uint to satisfy tma descriptor's specialization
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 242-244

```cpp
  using ArchTag = typename DispatchPolicy::ArchTag;
  static_assert(cute::is_same_v<ElementAMma, cutlass::bfloat16_t> || cute::is_same_v<ElementAMma, cutlass::half_t> || cute::is_same_v<ElementAMma, cutlass::float_e4m3_t>, 
         "Compute type A should be cutlass::bfloat16_t or cutlass::half_t or cutlass::float_e4m3_t");
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 246-249

```cpp
  using Load2TransformPipeline = cutlass::PipelineTmaTransformAsync<
                             DispatchPolicy::Load2TransformPipelineStageCount,
                             AtomThrShapeMNK>;
  using Load2TransformPipelineState = typename Load2TransformPipeline::PipelineState;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 251-255

```cpp
  using Load2MmaPipeline = cutlass::PipelineTmaUmmaAsync<
                             DispatchPolicy::Load2TransformPipelineStageCount,
                             ClusterShape,
                             AtomThrShapeMNK>;
  using Load2MmaPipelineState = typename Load2MmaPipeline::PipelineState;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 257-260

```cpp
  using Transform2MmaPipeline = cutlass::PipelineUmmaConsumerAsync<
                              DispatchPolicy::Transform2MmaPipelineStageCount,
                              AtomThrShapeMNK>;
  using Transform2MmaPipelineState = typename Transform2MmaPipeline::PipelineState;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 262-265

```cpp
  using Mma2AccumPipeline =  cutlass::PipelineUmmaAsync<
                              DispatchPolicy::Schedule::AccumulatorPipelineStageCount,
                              AtomThrShapeMNK>;
  using Mma2AccumPipelineState = typename Mma2AccumPipeline::PipelineState;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 268-272

```cpp
  static constexpr int ScaleGranularityMN = size<0,0>(LayoutScale{});
  static constexpr int ScaleGranularityK = size<1,0>(LayoutScale{});
  using ScaleConfig = cutlass::detail::Sm100MixedInputBlockwiseScaleConfig<
      ScaleGranularityMN, 
      ScaleGranularityK>; 
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 274-276

```cpp
  using ScaleTileShape = cute::conditional_t<!SwapAB, 
          decltype(make_shape(size<0>(TileShape{}), size<2>(TileShape{}))), 
          decltype(make_shape(size<1>(TileShape{}), size<2>(TileShape{})))>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 278-278

```cpp
  static constexpr int ScaleTileShape_MN = get<0>(ScaleTileShape{});
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 280-280

```cpp
  static constexpr int ScaleK = get<1>(ScaleTileShape{}) / ScaleGranularityK;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 282-282

```cpp
  using SmemLayoutAtomScale = decltype(ScaleConfig::smem_atom_layout_scale(ScaleTileShape{})); 
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 284-286

```cpp
  static_assert(cute::rank(InternalSmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(InternalSmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(InternalSmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 288-290

```cpp
  static_assert(cute::rank(InternalSmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<1>(TileShape{}) % size<0>(InternalSmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(InternalSmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 292-294

```cpp
  static_assert(cute::rank(SmemLayoutAtomScale{}) == 2, "SmemLayoutAtomScale must be rank 2");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomScale{})) == 0, "SmemLayoutAtomScale must equal the tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomScale{})) == 0, "SmemLayoutAtomScale must evenly divide tile k shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 296-298

```cpp
  // Thread Counts
  static constexpr uint32_t NumTransformationThreads = 128;
  static constexpr uint32_t NumAccumThreads = 128; //Maintains compatibility with input_transform kernel
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 300-302

```cpp
  // Get the Algorithm parameters
  constexpr static int AccumulatorPipelineStageCount = DispatchPolicy::Schedule::AccumulatorPipelineStageCount;
  constexpr static int StagesPerTile = size<2>(CtaShapeA_MK{});
```
**EN:** The surrounding comments explain the local purpose of this block: Get the Algorithm parameters.
**CN:** 周围注释解释了这一段的局部作用：Get the Algorithm parameters。

### Lines 304-306

```cpp
  static_assert(rank(SmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert(((size<0,0>(CtaShapeA_MK{}) * size<1>(CtaShapeA_MK{})) % size<0>(SmemLayoutAtomACompute{})) == 0, "SmemLayoutAtomCompute must evenly divide tile shape.");
  static_assert(((size<0,1>(CtaShapeA_MK{}) * size<2>(CtaShapeA_MK{})) % size<1>(SmemLayoutAtomACompute{})) == 0, "SmemLayoutAtomCompute must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 308-310

```cpp
  static_assert(rank(SmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert(((size<0,0>(CtaShapeB_NK{}) * size<1>(CtaShapeB_NK{})) % size<0>(SmemLayoutAtomBCompute{})) == 0, "SmemLayoutAtomCompute must evenly divide tile shape.");
  static_assert(((size<0,1>(CtaShapeB_NK{}) * size<2>(CtaShapeB_NK{})) % size<1>(SmemLayoutAtomBCompute{})) == 0, "SmemLayoutAtomCompute must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 312-317

```cpp
  // Tile along K mode first before tiling over MN. PIPE mode last as usual.
  // This maximizes TMA boxes due to better smem-K vectorization, reducing total issued TMAs.
  using SmemLayoutA = decltype(UMMA::tile_to_mma_shape(
      SmemLayoutAtomA{},
      append(CtaShapeA_MK{}, Int<DispatchPolicy::Load2TransformPipelineStageCount>{}),
             (cute::conditional_t<cutlass::gemm::detail::is_mn_major<StrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{})));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 319-322

```cpp
  using SmemLayoutACompute = decltype(UMMA::tile_to_mma_shape(
      SmemLayoutAtomACompute{},
      append(CtaShapeA_MK{}, Int<DispatchPolicy::Transform2MmaPipelineStageCount>{}),
             (cute::conditional_t<cutlass::gemm::detail::is_mn_major<StrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{})));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 324-327

```cpp
  using SmemLayoutB = decltype(UMMA::tile_to_mma_shape(
      SmemLayoutAtomB{},
      append(CtaShapeB_NK{}, Int<DispatchPolicy::Load2TransformPipelineStageCount>{}),
             (cute::conditional_t<cutlass::gemm::detail::is_mn_major<StrideB>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{})));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 329-332

```cpp
  using SmemLayoutScale = decltype(make_layout(
    append(shape(SmemLayoutAtomScale{}), Int<DispatchPolicy::Load2TransformPipelineStageCount>{}),
    append(stride(SmemLayoutAtomScale{}), size(filter_zeros(SmemLayoutAtomScale{})))
  ));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 334-341

```cpp
  static_assert(DispatchPolicy::Load2TransformPipelineStageCount >= 2 && DispatchPolicy::Load2TransformPipelineStageCount >= 2,
                "Specialization requires Stages set to value 2 or more.");
  static_assert((cute::is_base_of<cute::UMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value ||
                 cute::is_base_of<cute::UMMA::tmem_frg_base,      typename TiledMma::FrgTypeA>::value  ) &&
                 cute::is_base_of<cute::UMMA::DescriptorIterator, typename TiledMma::FrgTypeB>::value,
                 "MMA atom must A operand from SMEM or TMEM and B operand from SMEM for this mainloop.");
  static_assert((cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>),
                 "GmemTiledCopyA - invalid TMA copy atom specified.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 343-355

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

### Lines 357-363

```cpp
public:
  static constexpr ConversionMode KernelConversionMode = get_conversion_mode();
  static constexpr bool ModeHasScales = KernelConversionMode == ConversionMode::ConvertAndScale ||
                                        KernelConversionMode == ConversionMode::ConvertAndScaleWithZero;
  static constexpr bool UseScaleLookupTable = KernelConversionMode == ConversionMode::ConvertAndScale &&
                                              cutlass::detail::is_Array_v<ElementScale>;
  static constexpr size_t SmemAlignmentA = cutlass::detail::alignment_for_swizzle(SmemLayoutA{}); 
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 365-365

```cpp
  static constexpr size_t SmemAlignmentB = cutlass::detail::alignment_for_swizzle(SmemLayoutB{});
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 367-368

```cpp
  // Just pick the max alignment of A and B since it is required to be at least 128B
  static constexpr size_t SmemAlignmentScale = cute::max(SmemAlignmentA, SmemAlignmentB);
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 370-370

```cpp
  static_assert(SmemAlignmentA >= 128 and SmemAlignmentB >= 128, "Require at least 128B alignment");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 372-381

```cpp
  struct PipelineStorage {
    using Load2TransformPipelineStorage = typename Load2TransformPipeline::SharedStorage;
    alignas(16) Load2TransformPipelineStorage load2transform_pipeline;
    using Load2MmaPipelineStorage = typename Load2MmaPipeline::SharedStorage;
    alignas(16) Load2MmaPipelineStorage load2mma_pipeline;
    using Transform2MmaPipelineStorage = typename Transform2MmaPipeline::SharedStorage;
    alignas(16) Transform2MmaPipelineStorage transform2mma_pipeline;
    using Mma2AccumPipelineStorage = typename Mma2AccumPipeline::SharedStorage;
    alignas(16) Mma2AccumPipelineStorage mma2accum_pipeline;
  };
```
**EN:** This block introduces `PipelineStorage` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `PipelineStorage`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 383-386

```cpp
  struct SharedStorage {
    static constexpr int scale_elements = Utils::elements_per_smem_scale();
    static constexpr int zero_elements = Utils::elements_per_smem_zero();
    struct TensorStorage : cute::aligned_struct<128, _0> {
```
**EN:** This block introduces `SharedStorage` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `SharedStorage`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 388-393

```cpp
      struct TensorStorageUntransformed {
        alignas(512) cute::ArrayEngine<ElementA, cute::cosize_v<SmemLayoutA>> smem_A;
        alignas(1024) cute::ArrayEngine<ElementB, cute::cosize_v<SmemLayoutB>> smem_B;
        cute::ArrayEngine<NonVoidElementScale, scale_elements> smem_scale;
        cute::ArrayEngine<NonVoidElementZero, zero_elements> smem_zero;
      };
```
**EN:** This block introduces `TensorStorageUntransformed` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorStorageUntransformed`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 395-402

```cpp
      struct TensorStorageTransformedAinSmem {
        // We require alignas(1024) here because the smem_ACompute may not be aligned to 1024 by default.
        // We need 1024B alignment of smem_ACompute because we are using Swizzle<3,4,3> here.
        // The Swizzle<3,4,3> aligns with 1024B. If we don't align the data, the compiler cannot deduce
        // the base pointer of the data.
        // This alignment allows us to perform the function swizzle(layout(i) * base_ptr).
        alignas(1024) cute::ArrayEngine<ElementAMma, cute::cosize_v<SmemLayoutACompute>> smem_ACompute;
      };
```
**EN:** This block introduces `TensorStorageTransformedAinSmem` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorStorageTransformedAinSmem`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 404-406

```cpp
      union TensorStorageTransformedAinTmem {
        cute::ArrayEngine<ElementAMma, 1> smem_ACompute;  // No smem_ACompute
      };
```
**EN:** This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 408-411

```cpp
      using TensorStorageTransformed = cute::conditional_t<
                                      cute::is_base_of<cute::UMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value,
                                      TensorStorageTransformedAinSmem,
                                      TensorStorageTransformedAinTmem>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 413-415

```cpp
      TensorStorageUntransformed input;
      TensorStorageTransformed compute;
    } tensors;
```
**EN:** This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 417-419

```cpp
    PipelineStorage pipeline;
  };
  using TensorStorage = typename SharedStorage::TensorStorage;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 421-425

```cpp
  // Different from other GEMM kernels, both CTAs should be aware of loads. Both CTAs will work on
  // loaded input A and B matrices to convert the data type
  static constexpr uint32_t TmaTransactionBytes_A = cutlass::bits_to_bytes(cosize(take<0,3>(SmemLayoutA{})) * cute::sizeof_bits_v<ElementA>) + Utils::compute_tma_transaction_bytes_extra_transform();
  static constexpr uint32_t TmaTransactionBytes_B = cutlass::bits_to_bytes(size(AtomThrShapeMNK{}) * cosize(take<0,3>(SmemLayoutB{})) * cute::sizeof_bits_v<ElementB>);
  static constexpr uint32_t TmaTransactionBytes = TmaTransactionBytes_A + TmaTransactionBytes_B;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 427-436

```cpp
  // Host side kernel arguments
  struct Arguments {
    ElementA const* ptr_A{nullptr};
    StrideA dA{};
    ElementB const* ptr_B{nullptr};
    StrideB dB{};
    ElementScale const* ptr_S{nullptr};
    LayoutScale layout_S{};
    ElementZero const* ptr_Z{nullptr};
  };
```
**EN:** This block introduces `Arguments` and groups related declarations around that symbol. `Arguments` is the host-side bundle: callers fill it with pointers, strides, and other launch-time values before parameter lowering.
**CN:** 这一段引入了 `Arguments`，并围绕该符号组织相关声明。 `Arguments` 是主机侧参数包：调用者在参数下沉前把指针、步长及其他启动期数据填入其中。

### Lines 438-440

```cpp
  struct TMAScaleParams {
    using ClusterLayout_VMNK = decltype(tiled_divide(make_layout(conditional_return<IsDynamicCluster>(make_shape(uint32_t(0), uint32_t(0), Int<1>{}), ClusterShape{})),
                              make_tile(typename TiledMma::AtomThrID{})));
```
**EN:** This block introduces `TMAScaleParams` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `TMAScaleParams`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 442-448

```cpp
    using TMA_Scale = decltype(make_tma_atom(
        GmemTiledCopyScale{},
        make_tensor(static_cast<NonVoidElementScale const*>(nullptr), LayoutScale{}),
        SmemLayoutScale{}(_,_,cute::Int<0>{}),
        ScaleTileShape{},
        size<2>(ClusterLayout_VMNK{}))
    );
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 450-451

```cpp
    TMA_Scale tma_load_scale;
    TMA_Scale tma_load_zero;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 453-453

```cpp
  };
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 455-455

```cpp
  struct EmptyScaleParams {};
```
**EN:** This block introduces `EmptyScaleParams` and groups related declarations around that symbol.
**CN:** 这一段引入了 `EmptyScaleParams`，并围绕该符号组织相关声明。

### Lines 457-458

```cpp
  // Device side kernel params
  struct Params : public cute::conditional_t<ModeHasScales, TMAScaleParams, EmptyScaleParams>  {
```
**EN:** This block introduces `Params` and groups related declarations around that symbol. `Params` stores the lowered device-facing state such as tensor descriptors, precomputed copy objects, or runtime datatype flags.
**CN:** 这一段引入了 `Params`，并围绕该符号组织相关声明。 `Params` 保存下沉后的设备侧状态，例如张量描述符、预计算的拷贝对象或运行时数据类型标记。

### Lines 460-461

```cpp
    using ClusterLayout_VMNK = decltype(tiled_divide(make_layout(conditional_return<IsDynamicCluster>(make_shape(uint32_t(0), uint32_t(0), Int<1>{}), ClusterShape{})),
                                                     make_tile(typename TiledMma::AtomThrID{})));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 463-470

```cpp
    using TMA_A = decltype(make_tma_atom_A_sm100<TmaElementA>(
        GmemTiledCopyA{},
        make_tensor(static_cast<ElementA const*>(nullptr), repeat_like(StrideA{}, int32_t(0)), StrideA{}),
        SmemLayoutA{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        ClusterLayout_VMNK{})
      );
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 472-479

```cpp
    using TMA_B = decltype(make_tma_atom_B_sm100<ElementB>(
        GmemTiledCopyB{},
        make_tensor(static_cast<ElementB const*>(nullptr), repeat_like(StrideB{}, int32_t(0)), StrideB{}),
        SmemLayoutB{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        ClusterLayout_VMNK{})
    );
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 481-485

```cpp
    TMA_A tma_load_a;
    TMA_B tma_load_b;
    TMA_A tma_load_a_fallback;
    TMA_B tma_load_b_fallback;
    dim3 cluster_shape_fallback;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 487-490

```cpp
    uint32_t tma_transaction_bytes{TmaTransactionBytes};
    SwappedStrideA dA{};
    SwappedStrideB dB{};
  };
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 492-506

```cpp
  CUTLASS_DEVICE
  CollectiveMma(Params const& params, ClusterShape cluster_shape, uint32_t block_rank_in_cluster)
    : cluster_shape_(cluster_shape)
    , block_rank_in_cluster_(block_rank_in_cluster) {
    if constexpr (IsDynamicCluster) {
      const bool is_fallback_cluster = (cute::size<0>(cluster_shape_) == params.cluster_shape_fallback.x &&
                                        cute::size<1>(cluster_shape_) == params.cluster_shape_fallback.y);
      observed_tma_load_a_ = is_fallback_cluster ? &params.tma_load_a_fallback : &params.tma_load_a;
      observed_tma_load_b_ = is_fallback_cluster ? &params.tma_load_b_fallback : &params.tma_load_b;
    }
    else {
      observed_tma_load_a_ = &params.tma_load_a;
      observed_tma_load_b_ = &params.tma_load_b;
    }
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 508-515

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(
    ProblemShape const& problem_shape, 
    Arguments const& args, 
    void* workspace, 
    cutlass::KernelHardwareInfo const& hw_info = cutlass::KernelHardwareInfo{}) {
    (void) workspace;
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 517-519

```cpp
    // Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M,N,K,L] = problem_shape_MNKL;
```
**EN:** The surrounding comments explain the local purpose of this block: Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK).
**CN:** 周围注释解释了这一段的局部作用：Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)。

### Lines 521-522

```cpp
    Tensor tensor_a = make_tensor(args.ptr_A, make_layout(make_shape(M,K,L), args.dA));
    Tensor tensor_b = make_tensor(args.ptr_B, make_layout(make_shape(N,K,L), args.dB));
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 524-526

```cpp
    auto cluster_shape = cutlass::detail::select_cluster_shape(ClusterShape{}, hw_info.cluster_shape);
    // Cluster layout for TMA construction
    auto cluster_layout_vmnk = tiled_divide(make_layout(cluster_shape), make_tile(typename TiledMma::AtomThrID{}));
```
**EN:** The surrounding comments explain the local purpose of this block: Cluster layout for TMA construction.
**CN:** 周围注释解释了这一段的局部作用：Cluster layout for TMA construction。

### Lines 528-530

```cpp
    auto cluster_shape_fallback = cutlass::detail::select_cluster_shape(ClusterShape{}, hw_info.cluster_shape_fallback);
    // Cluster layout for TMA construction
    auto cluster_layout_vmnk_fallback = tiled_divide(make_layout(cluster_shape_fallback), make_tile(typename TiledMma::AtomThrID{}));
```
**EN:** The surrounding comments explain the local purpose of this block: Cluster layout for TMA construction.
**CN:** 周围注释解释了这一段的局部作用：Cluster layout for TMA construction。

### Lines 532-538

```cpp
    typename Params::TMA_A tma_load_a = make_tma_atom_A_sm100<TmaElementA>(
        GmemTiledCopyA{},
        tensor_a,
        SmemLayoutA{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        cluster_layout_vmnk);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 540-546

```cpp
    typename Params::TMA_B tma_load_b = make_tma_atom_B_sm100<ElementB>(
        GmemTiledCopyB{},
        tensor_b,
        SmemLayoutB{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        cluster_layout_vmnk);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 548-554

```cpp
    typename Params::TMA_A tma_load_a_fallback = make_tma_atom_A_sm100<TmaElementA>(
        GmemTiledCopyA{},
        tensor_a,
        SmemLayoutA{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        cluster_layout_vmnk_fallback);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 556-562

```cpp
    typename Params::TMA_B tma_load_b_fallback = make_tma_atom_B_sm100<ElementB>(
        GmemTiledCopyB{},
        tensor_b,
        SmemLayoutB{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        cluster_layout_vmnk_fallback);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 564-564

```cpp
    uint32_t tma_transaction_bytes = TmaTransactionBytes;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 566-578

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return { 
        {},
        tma_load_a, 
        tma_load_b, 
        tma_load_a_fallback, 
        tma_load_b_fallback, 
        hw_info.cluster_shape_fallback, 
        tma_transaction_bytes, 
        args.dA, args.dB };
    } 
    else if constexpr (ModeHasScales) {
      ElementScale const* ptr_S = args.ptr_S;
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 580-587

```cpp
      Tensor tensor_scale = make_tensor(detail::get_logical_ptr(ptr_S), args.layout_S);
      typename Params::TMA_Scale tma_load_scale = make_tma_atom(
          GmemTiledCopyScale{},
          tensor_scale,
          SmemLayoutScale{}(_,_,cute::Int<0>{}),
          ScaleTileShape{},
          size<2>(cluster_layout_vmnk)
      );
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 589-608

```cpp
      if constexpr(KernelConversionMode == ConversionMode::ConvertAndScale) {
        typename Params::TMAScaleParams scale_params{tma_load_scale, {}};
        return { 
          scale_params,
          tma_load_a, 
          tma_load_b, 
          tma_load_a_fallback, 
          tma_load_b_fallback, 
          hw_info.cluster_shape_fallback, 
          tma_transaction_bytes, 
          args.dA, args.dB };
      }
      else if constexpr(KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        Tensor tensor_zero = make_tensor(detail::get_logical_ptr(args.ptr_Z), args.layout_S);
        typename Params::TMA_Scale tma_load_zero = make_tma_atom(
            GmemTiledCopyScale{},
            tensor_zero,
            SmemLayoutScale{}(_,_,cute::Int<0>{}),
            ScaleTileShape{},
            size<2>(cluster_layout_vmnk));
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 610-628

```cpp
        typename Params::TMAScaleParams scale_params{tma_load_scale, tma_load_zero};
        return { 
          scale_params,
          tma_load_a, 
          tma_load_b, 
          tma_load_a_fallback, 
          tma_load_b_fallback, 
          hw_info.cluster_shape_fallback, 
          tma_transaction_bytes, 
          args.dA, args.dB };
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

### Lines 630-634

```cpp
  template<class ProblemShape>
  static bool
  can_implement(
      ProblemShape const& problem_shape,
      [[maybe_unused]] Arguments const& args) {
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. This method performs runtime feasibility checks, usually validating alignment, layout assumptions, or shape constraints before launching the kernel.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这个方法执行运行时可实现性检查，通常会在启动内核前验证对齐、布局假设或形状约束。

### Lines 636-637

```cpp
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M,N,K,L] = problem_shape_MNKL;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 639-641

```cpp
    constexpr int tma_alignment_bits_A = cutlass::detail::get_input_alignment_bits<ElementA>();
    constexpr int tma_alignment_bits_B = cutlass::detail::get_input_alignment_bits<ElementB>();
    constexpr int tma_alignment_bits_S = cutlass::detail::get_input_alignment_bits<NonVoidElementScale>();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 643-646

```cpp
    constexpr int min_tma_aligned_elements_A = tma_alignment_bits_A / cutlass::sizeof_bits<ElementA>::value;
    bool check_aligned_A = cutlass::detail::check_alignment<min_tma_aligned_elements_A>(cute::make_shape(M,K,L), StrideA{});
    constexpr int min_tma_aligned_elements_B = tma_alignment_bits_B / cutlass::sizeof_bits<ElementB>::value;
    bool check_aligned_B = cutlass::detail::check_alignment<min_tma_aligned_elements_B>(cute::make_shape(N,K,L), StrideB{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 648-650

```cpp
    bool check_aligned_S = true;
    bool check_aligned_Z = true;
    bool check_mode_args = true;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 652-659

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      check_mode_args = check_mode_args && (args.ptr_S == nullptr);
      check_mode_args = check_mode_args && (args.ptr_Z == nullptr);
    } 
    else if constexpr (ModeHasScales) {
      constexpr int min_tma_aligned_elements_scale = tma_alignment_bits_S / cutlass::sizeof_bits<ElementScale>::value;
      check_aligned_S = cutlass::detail::check_alignment<min_tma_aligned_elements_scale>(args.layout_S);
      check_mode_args = check_mode_args && (args.ptr_S != nullptr);
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 661-675

```cpp
      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        check_mode_args = check_mode_args && (args.ptr_Z == nullptr);
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        constexpr int min_tma_aligned_elements_zero = tma_alignment_bits_S / cutlass::sizeof_bits<ElementZero>::value;
        check_aligned_Z = cutlass::detail::check_alignment<min_tma_aligned_elements_zero>(args.layout_S);
        check_mode_args = check_mode_args && (args.ptr_Z != nullptr);
      } 
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in can_implement.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in can_implement.");
    }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. This method performs runtime feasibility checks, usually validating alignment, layout assumptions, or shape constraints before launching the kernel.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这个方法执行运行时可实现性检查，通常会在启动内核前验证对齐、布局假设或形状约束。

### Lines 677-691

```cpp
    if (!check_mode_args) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Invalid arguments for the selected conversion mode.\n");
    }
    if (!check_aligned_A) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Tensor A meet the minimum alignment requirements for TMA.\n");
    }
    if (!check_aligned_B) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Tensor B meet the minimum alignment requirements for TMA.\n");
    }
    if (!check_aligned_S) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Tensor S (scale) meet the minimum alignment requirements for TMA.\n");
    }
    if (!check_aligned_Z) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Tensor Z (zeros) meet the minimum alignment requirements for TMA.\n");
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 693-694

```cpp
    return check_mode_args && check_aligned_A && check_aligned_B && check_aligned_S && check_aligned_Z;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 696-714

```cpp
  /// Issue Tma Descriptor Prefetch -- ideally from a single thread for best performance
  CUTLASS_DEVICE static void
  prefetch_tma_descriptors(Params const& params) {
    if constexpr (IsDynamicCluster) {
      dim3 cs = cute::cluster_shape();
      const bool is_fallback_cluster = (cs.x == params.cluster_shape_fallback.x && cs.y == params.cluster_shape_fallback.y);
      if (is_fallback_cluster) {
        cute::prefetch_tma_descriptor(params.tma_load_a_fallback.get_tma_descriptor());
        cute::prefetch_tma_descriptor(params.tma_load_b_fallback.get_tma_descriptor());
      }
      else {
        cute::prefetch_tma_descriptor(params.tma_load_a.get_tma_descriptor());
        cute::prefetch_tma_descriptor(params.tma_load_b.get_tma_descriptor());
      }
    }
    else {
      cute::prefetch_tma_descriptor(params.tma_load_a.get_tma_descriptor());
      cute::prefetch_tma_descriptor(params.tma_load_b.get_tma_descriptor());
    }
```
**EN:** The code prefetches descriptor state early so later TMA or copy operations can start with lower latency. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里会提前预取描述符状态，从而让后续 TMA 或拷贝操作以更低延迟启动。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 716-727

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert);
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      cute::prefetch_tma_descriptor(params.tma_load_scale.get_tma_descriptor());
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      cute::prefetch_tma_descriptor(params.tma_load_scale.get_tma_descriptor());
      cute::prefetch_tma_descriptor(params.tma_load_zero.get_tma_descriptor());
    }  
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in TMA prefetch.");
    }
  }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. The code prefetches descriptor state early so later TMA or copy operations can start with lower latency.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这里会提前预取描述符状态，从而让后续 TMA 或拷贝操作以更低延迟启动。

### Lines 729-732

```cpp
  /// Construct A Single Stage's Accumulator Shape
  CUTLASS_DEVICE auto
  partition_accumulator_shape() {
    auto acc_shape = partition_shape_C(TiledMma{}, take<0,2>(TileShape{}));  // ((MMA_TILE_M,MMA_TILE_N),MMA_M,MMA_N)
```
**EN:** This logic sets up accumulator fragments or TMEM-backed storage so the mainloop and epilogue agree on where partial sums live.
**CN:** 这些逻辑用于设置累加器片段或基于 TMEM 的存储，使主循环与 epilogue 对部分和的存放位置达成一致。

### Lines 734-735

```cpp
    return acc_shape;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 737-757

```cpp
  /// Produce the inputs to the transform threads by loading inputs from gmem -> smem
  template <
    class GTensorA, class GTensorB,
    class GTensorPartitionedA, class GTensorPartitionedB,
    class STensorA, class STensorB,
    class TileCoordMNKL,
    class KTileIterator,
    class... Ts
  >
  CUTLASS_DEVICE auto
  load_A(
      Params const& params,
      Load2TransformPipeline load2xform_pipeline,
      Load2TransformPipelineState load2xform_pipeline_state,
      cute::tuple<GTensorA, GTensorB,
                  GTensorPartitionedA, GTensorPartitionedB,
                  STensorA, STensorB,
                  uint16_t, uint16_t,
                  cute::tuple<Ts...>> const& load_inputs,
      TileCoordMNKL const& cta_coord_mnkl,
      KTileIterator k_tile_iter, int k_tile_count) {
```
**EN:** This block introduces `GTensorA` and groups related declarations around that symbol. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段引入了 `GTensorA`，并围绕该符号组织相关声明。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 759-761

```cpp
    auto [unused_gA, unused_gB,
          tAgA_mkl, tBgB_nkl, tAsA, tBsB,
          mcast_mask_a, mcast_mask_b, extra_input_partitions] = load_inputs;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 763-764

```cpp
    // slice out the work coord from tiled tensors
    Tensor tAgA = tAgA_mkl(_, get<0>(cta_coord_mnkl) / size(typename TiledMma::AtomThrID{}), _, get<3>(cta_coord_mnkl));
```
**EN:** The surrounding comments explain the local purpose of this block: slice out the work coord from tiled tensors.
**CN:** 周围注释解释了这一段的局部作用：slice out the work coord from tiled tensors。

### Lines 766-767

```cpp
    uint32_t skip_wait = (k_tile_count <= 0);
    auto load2xform_pipeline_flag = load2xform_pipeline.producer_try_acquire(load2xform_pipeline_state, skip_wait);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 769-770

```cpp
    //Load2Mma and Load2Transform pipelines both have the same ProducerBarrierType
    using BarrierType = typename Load2TransformPipeline::ProducerBarrierType;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 772-774

```cpp
    // Issue the Mainloop loads
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 776-777

```cpp
      // LOCK mainloop_load2xform_pipeline_state for _writing_
      load2xform_pipeline.producer_acquire(load2xform_pipeline_state, load2xform_pipeline_flag);
```
**EN:** This comment marks the mainloop logic that repeatedly loads tiles and issues MMA operations.
**CN:** 这条注释标记了主循环逻辑：重复加载 tile 并发起 MMA 运算。

### Lines 779-779

```cpp
      int tile_A_write_stage = load2xform_pipeline_state.index();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 781-781

```cpp
      BarrierType* load2xform_tma_barrier = load2xform_pipeline.producer_get_barrier(load2xform_pipeline_state);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 783-784

```cpp
      // Advance mainloop load2transform pipeline
      ++load2xform_pipeline_state;
```
**EN:** This comment marks the mainloop logic that repeatedly loads tiles and issues MMA operations.
**CN:** 这条注释标记了主循环逻辑：重复加载 tile 并发起 MMA 运算。

### Lines 786-787

```cpp
      skip_wait = (k_tile_count <= 1);
      load2xform_pipeline_flag = load2xform_pipeline.producer_try_acquire(load2xform_pipeline_state, skip_wait);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 789-790

```cpp
      // TMA load for A k_tile
      copy(observed_tma_load_a_->with(*load2xform_tma_barrier, mcast_mask_a), tAgA(_,*k_tile_iter), tAsA(_,tile_A_write_stage));
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 792-796

```cpp
      if constexpr (ModeHasScales) {
        auto tSgS_mkl = get<0>(extra_input_partitions);
        auto tSgS = tSgS_mkl(_, get<0>(cta_coord_mnkl) / size(typename TiledMma::AtomThrID{}), _, get<3>(cta_coord_mnkl));
        auto tSsS = get<1>(extra_input_partitions);
        copy(params.tma_load_scale.with(*load2xform_tma_barrier, mcast_mask_a), tSgS(_,*k_tile_iter), tSsS(_,tile_A_write_stage));
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 798-808

```cpp
        if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
          auto tZgZ_mkl = get<2>(extra_input_partitions);
          auto tZgZ = tZgZ_mkl(_, get<0>(cta_coord_mnkl) / size(typename TiledMma::AtomThrID{}), _, get<3>(cta_coord_mnkl));
          auto tZsZ = get<3>(extra_input_partitions);
          copy(params.tma_load_zero.with(*load2xform_tma_barrier, mcast_mask_a), tZgZ(_,*k_tile_iter), tZsZ(_,tile_A_write_stage));
        }
      } 
      else {
        if constexpr (KernelConversionMode == ConversionMode::DirectConvert);
        else static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled for TMA copy op.");
      }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 810-811

```cpp
      ++k_tile_iter;
    }
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 814-814

```cpp
    return cute::make_tuple(load2xform_pipeline_state, k_tile_iter);
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 816-816

```cpp
  }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 818-838

```cpp
  /// Produce the inputs to the transform threads by loading inputs from gmem -> smem
  template <
    class GTensorA, class GTensorB,
    class GTensorPartitionedA, class GTensorPartitionedB,
    class STensorA, class STensorB,
    class TileCoordMNKL,
    class KTileIterator,
    class... Ts
  >
  CUTLASS_DEVICE auto
  load_B(
      Params const& params,
      Load2MmaPipeline load2mma_pipeline,
      Load2MmaPipelineState load2mma_pipeline_state,
      cute::tuple<GTensorA, GTensorB,
                  GTensorPartitionedA, GTensorPartitionedB,
                  STensorA, STensorB,
                  uint16_t, uint16_t,
                  cute::tuple<Ts...>> const& load_inputs,
      TileCoordMNKL const& cta_coord_mnkl,
      KTileIterator k_tile_iter, int k_tile_count) {
```
**EN:** This block introduces `GTensorA` and groups related declarations around that symbol. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段引入了 `GTensorA`，并围绕该符号组织相关声明。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 840-842

```cpp
    auto [unused_gA, unused_gB,
          tAgA_mkl, tBgB_nkl, tAsA, tBsB,
          mcast_mask_a, mcast_mask_b, extra_input_partitions] = load_inputs;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 844-845

```cpp
    // slice out the work coord from tiled tensors
    Tensor tBgB = tBgB_nkl(_, get<1>(cta_coord_mnkl), _, get<3>(cta_coord_mnkl));
```
**EN:** The surrounding comments explain the local purpose of this block: slice out the work coord from tiled tensors.
**CN:** 周围注释解释了这一段的局部作用：slice out the work coord from tiled tensors。

### Lines 847-848

```cpp
    uint32_t skip_wait = (k_tile_count <= 0);
    auto load2mma_pipeline_flag = load2mma_pipeline.producer_try_acquire(load2mma_pipeline_state, skip_wait);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 850-851

```cpp
    //Load2Mma and Load2Transform pipelines both have the same ProducerBarrierType
    using BarrierType = typename Load2TransformPipeline::ProducerBarrierType;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 853-855

```cpp
    // Issue the Mainloop loads
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 857-858

```cpp
      // LOCK mainloop_load2mma_pipeline_state for _writing_
      load2mma_pipeline.producer_acquire(load2mma_pipeline_state, load2mma_pipeline_flag);
```
**EN:** This comment marks the mainloop logic that repeatedly loads tiles and issues MMA operations.
**CN:** 这条注释标记了主循环逻辑：重复加载 tile 并发起 MMA 运算。

### Lines 860-860

```cpp
      int tile_B_write_stage = load2mma_pipeline_state.index();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 862-862

```cpp
      BarrierType* load2mma_tma_barrier = load2mma_pipeline.producer_get_barrier(load2mma_pipeline_state);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 864-865

```cpp
      // Advance mainloop load2mma pipeline
      ++load2mma_pipeline_state;
```
**EN:** This comment marks the mainloop logic that repeatedly loads tiles and issues MMA operations.
**CN:** 这条注释标记了主循环逻辑：重复加载 tile 并发起 MMA 运算。

### Lines 867-868

```cpp
      skip_wait = (k_tile_count <= 1);
      load2mma_pipeline_flag = load2mma_pipeline.producer_try_acquire(load2mma_pipeline_state, skip_wait);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 870-871

```cpp
      // TMA load for B k_tile
      copy(observed_tma_load_b_->with(*load2mma_tma_barrier, mcast_mask_b), tBgB(_,*k_tile_iter), tBsB(_,tile_B_write_stage));
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 873-874

```cpp
      ++k_tile_iter;
    }
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 876-876

```cpp
    return cute::make_tuple(load2mma_pipeline_state, k_tile_iter);
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 878-878

```cpp
  }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 880-891

```cpp
  /// Set up the data needed by this collective for load.
  /// Returned tuple must contain at least two elements, with the first two elements being:
  /// gA_mkl - The tiled tensor for input A
  /// gB_nkl - The tiled tensor for input B
  // Other inputs needed for load(): partitioned AB tensors for gmem and smem, and mcast masks
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE auto
  load_init(
      ProblemShape_MNKL const& problem_shape_MNKL,
      Params const& params,
      TensorStorage& shared_storage) const {
    auto [gA_mkl, gB_nkl] = tile_input_tensors(params, problem_shape_MNKL);
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 893-893

```cpp
    ThrMMA cta_mma = TiledMma{}.get_slice(BlockIdxX() % size(typename TiledMma::AtomThrID{}));
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 895-896

```cpp
    Tensor tCgA_mkl = cta_mma.partition_A(gA_mkl);          // (MMA, MMA_M, MMA_K, m, k, l)
    Tensor tCgB_nkl = cta_mma.partition_B(gB_nkl);          // (MMA, MMA_N, MMA_K, n, k, l)
```
**EN:** This block prepares the load path by slicing global tensors, partitioning work for the current CTA/subgroup, and binding shared-memory destinations.
**CN:** 这一段为加载路径做准备：切分全局张量、为当前 CTA/子组划分工作，并绑定共享内存目标。

### Lines 898-899

```cpp
    Tensor sA = make_tensor(make_smem_ptr(shared_storage.input.smem_A.begin()), SmemLayoutA{});  // (MMA,MMA_M,MMA_K,PIPE)
    Tensor sB = make_tensor(make_smem_ptr(shared_storage.input.smem_B.begin()), SmemLayoutB{});  // (MMA,MMA_N,MMA_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 901-904

```cpp
    // Define the CTA-in-cluster Layout and Coord
    Layout cta_layout_mnk  = make_layout(cluster_shape_);
    Layout cta_layout_vmnk = tiled_divide(cta_layout_mnk, make_tile(typename TiledMma::AtomThrID{}));
    auto cta_coord_vmnk  = cta_layout_vmnk.get_flat_coord(block_rank_in_cluster_);
```
**EN:** The surrounding comments explain the local purpose of this block: Define the CTA-in-cluster Layout and Coord.
**CN:** 周围注释解释了这一段的局部作用：Define the CTA-in-cluster Layout and Coord。

### Lines 906-909

```cpp
    // Project the cta_layout for tma_a along the n-modes
    auto [tAgA_mkl, tAsA] = tma_partition(*observed_tma_load_a_,
                                      get<2>(cta_coord_vmnk), make_layout(size<2>(cta_layout_vmnk)),
                                      group_modes<0,3>(sA), group_modes<0,3>(tCgA_mkl));
```
**EN:** The surrounding comments explain the local purpose of this block: Project the cta_layout for tma_a along the n-modes.
**CN:** 周围注释解释了这一段的局部作用：Project the cta_layout for tma_a along the n-modes。

### Lines 911-914

```cpp
    // Project the cta_layout for tma_b along the m-modes
    auto [tBgB_nkl, tBsB] = tma_partition(*observed_tma_load_b_,
                                      get<1>(cta_coord_vmnk), make_layout(size<1>(cta_layout_vmnk)),
                                      group_modes<0,3>(sB), group_modes<0,3>(tCgB_nkl));
```
**EN:** The surrounding comments explain the local purpose of this block: Project the cta_layout for tma_b along the m-modes.
**CN:** 周围注释解释了这一段的局部作用：Project the cta_layout for tma_b along the m-modes。

### Lines 916-918

```cpp
    // TMA Multicast Masks
    uint16_t mcast_mask_a = create_tma_multicast_mask<2>(cta_layout_vmnk, cta_coord_vmnk);
    uint16_t mcast_mask_b = create_tma_multicast_mask<1>(cta_layout_vmnk, cta_coord_vmnk);
```
**EN:** The surrounding comments explain the local purpose of this block: TMA Multicast Masks.
**CN:** 周围注释解释了这一段的局部作用：TMA Multicast Masks。

### Lines 920-929

```cpp
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return cute::make_tuple(
          gA_mkl, gB_nkl,                        // for scheduler
          tAgA_mkl, tBgB_nkl, tAsA, tBsB,        // for input tensor values
          mcast_mask_a, mcast_mask_b,            // multicast masks
          cute::make_tuple());           
    }
    else if constexpr (ModeHasScales) {
      // Separate out problem shape for convenience
      auto [M,N,K,L] = problem_shape_MNKL;
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 931-932

```cpp
      Tensor mS_mkl = params.tma_load_scale.get_tma_tensor(shape(LayoutScale{}));
      Tensor gS_mkl = local_tile(mS_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 934-934

```cpp
      Tensor sS  = make_tensor(make_smem_ptr(shared_storage.input.smem_scale.begin()), SmemLayoutScale{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 936-937

```cpp
      Tensor tCgS_mkl = cta_mma.partition_A(gS_mkl);          // (MMA, MMA_M, MMA_K, m, k, l)
      Tensor tCsS = cta_mma.partition_A(sS);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 939-942

```cpp
      // Project the cta_layout for tma_scale along the n-modes
      auto [tSgS_mkl, tSsS] = tma_partition(params.tma_load_scale,
                                      get<2>(cta_coord_vmnk), make_layout(size<2>(cta_layout_vmnk)),
                                      group_modes<0,3>(tCsS), group_modes<0,3>(tCgS_mkl));
```
**EN:** The surrounding comments explain the local purpose of this block: Project the cta_layout for tma_scale along the n-modes.
**CN:** 周围注释解释了这一段的局部作用：Project the cta_layout for tma_scale along the n-modes。

### Lines 944-954

```cpp
      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return cute::make_tuple(
          gA_mkl, gB_nkl,                        // for scheduler
          tAgA_mkl, tBgB_nkl, tAsA, tBsB,        // for input tensor values
          mcast_mask_a, mcast_mask_b,            // multicast masks
          cute::make_tuple(tSgS_mkl, tSsS));
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        Tensor mZ_mkl = params.tma_load_scale.get_tma_tensor(shape(LayoutScale{}));
        Tensor gZ_mkl = local_tile(mS_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});
        Tensor sZ  = make_tensor(make_smem_ptr(shared_storage.input.smem_zero.begin()), SmemLayoutScale{});
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 956-956

```cpp
        Tensor tCgZ_mkl = cta_mma.partition_A(gZ_mkl);          // (MMA, MMA_M, MMA_K, m, k, l)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 958-975

```cpp
        Tensor tCsZ = cta_mma.partition_A(sZ);
        // Project the cta_layout for tma_scale along the n-modes
        auto [tZgZ_mkl, tZsZ] = tma_partition(params.tma_load_zero,
                                          get<2>(cta_coord_vmnk), make_layout(size<2>(cta_layout_vmnk)),
                                          group_modes<0,3>(tCsZ), group_modes<0,3>(tCgZ_mkl));
        return cute::make_tuple(
          gA_mkl, gB_nkl,                        // for scheduler
          tAgA_mkl, tBgB_nkl, tAsA, tBsB,        // for input tensor values
          mcast_mask_a, mcast_mask_b,            // multicast masks
          cute::make_tuple(tSgS_mkl, tSsS, tZgZ_mkl, tZsZ));
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in load_init.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in load_init.");
    }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. This block prepares the load path by slicing global tensors, partitioning work for the current CTA/subgroup, and binding shared-memory destinations.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这一段为加载路径做准备：切分全局张量、为当前 CTA/子组划分工作，并绑定共享内存目标。

### Lines 977-977

```cpp
  }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 979-993

```cpp
  template<
    class KTileIterator, class Accumulator,
    class GTensorA, class DstCopyA, class SrcTensorA, class DstTensorA,
    class... Ts
  >
  CUTLASS_DEVICE auto
  transform(
      Load2TransformPipeline load2transform_pipeline,
      Load2TransformPipelineState load2transform_pipeline_consumer_state,
      Transform2MmaPipeline transform2mma_pipeline,
      Transform2MmaPipelineState transform2mma_pipeline_producer_state,
      Accumulator accumulators,
      cute::tuple<GTensorA, DstCopyA, SrcTensorA, DstTensorA,
                  cute::tuple<Ts...>> input_operands,
      KTileIterator k_tile_iter, int k_tile_count) {
```
**EN:** This block introduces `KTileIterator` and groups related declarations around that symbol. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段引入了 `KTileIterator`，并围绕该符号组织相关声明。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 995-996

```cpp
    static_assert(cute::is_same_v<ElementAMma, ElementBMma>, "ElementAMma and ElementBMma types should be the same.");
    cutlass::arch::NamedBarrier transform_bar(NumTransformationThreads, cutlass::arch::ReservedNamedBarriers::TransformBarrier);
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 998-1001

```cpp
    // tAsA : (Copy,#Copy),MMA_Rest,MMA_M_Rest,MMA_K_Rest, SmemStages (In SMEM)
    // tAsACompute : (Copy,#Copy),MMA_Rest,MMA_M_Rest,MMA_K_Rest, SmemStages (In SMEM or TMEM)
    auto [unused_tAgA, dst_copy_A, tAsA, tAsACompute,
          partitioned_extra_info] = input_operands;
```
**EN:** The surrounding comments explain the local purpose of this block: tAsA : (Copy,#Copy),MMA_Rest,MMA_M_Rest,MMA_K_Rest, SmemStages (In SMEM) tAsACompute : (Copy,#Copy),MMA_Rest,MMA_M_Rest,MMA_K_Rest, SmemStages (In SMEM or TMEM).
**CN:** 周围注释解释了这一段的局部作用：tAsA : (Copy,#Copy),MMA_Rest,MMA_M_Rest,MMA_K_Rest, SmemStages (In SMEM) tAsACompute : (Copy,#Copy),MMA_Rest,MMA_M_Rest,MMA_K_Rest, SmemStages (In SMEM or TMEM)。

### Lines 1003-1006

```cpp
    // Create the tensors in registers
    auto tArA = make_tensor<ElementA>(tAsA(_,_,_,_,0).shape());  //(Copy,#Copy),MMA_Rest,MMA_M_Rest,MMA_K_Rest (Register)
    auto tArACompute = make_tensor<ElementAMma>(tAsA(_,_,_,_,0).shape());
    constexpr int K_BLOCK_MAX = size<3>(tArA);
```
**EN:** The surrounding comments explain the local purpose of this block: Create the tensors in registers.
**CN:** 周围注释解释了这一段的局部作用：Create the tensors in registers。

### Lines 1008-1010

```cpp
    uint32_t skip_wait = (k_tile_count <= 0);
    auto load2transform_flag = load2transform_pipeline.consumer_try_wait(load2transform_pipeline_consumer_state, skip_wait);
    auto transform2mma_flag = transform2mma_pipeline.producer_try_acquire(transform2mma_pipeline_producer_state, skip_wait);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1012-1013

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1015-1015

```cpp
      load2transform_pipeline.consumer_wait(load2transform_pipeline_consumer_state, load2transform_flag);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1017-1017

```cpp
      transform2mma_pipeline.producer_acquire(transform2mma_pipeline_producer_state, transform2mma_flag);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1019-1020

```cpp
      int load2transform_consumer_index = load2transform_pipeline_consumer_state.index(); // read stage
      int transform2mma_producer_index = transform2mma_pipeline_producer_state.index(); //write stage
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1022-1022

```cpp
      auto curr_load2transform_pipeline_consumer_state = load2transform_pipeline_consumer_state;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1024-1027

```cpp
      // Copy the input A matrix from SMEM
      copy(AutoVectorizingCopy{}, tAsA(_,_,_,_,load2transform_consumer_index), tArA);
      // Copy scale/zero vector from SMEM
      Utils::copy_scale_zeros_for_transform(partitioned_extra_info, load2transform_consumer_index);
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1029-1031

```cpp
      // Loads from SMEM are done. Signal the mainloop load as early as possible
      transform_bar.sync();
      load2transform_pipeline.consumer_release(curr_load2transform_pipeline_consumer_state);
```
**EN:** This comment marks the mainloop logic that repeatedly loads tiles and issues MMA operations.
**CN:** 这条注释标记了主循环逻辑：重复加载 tile 并发起 MMA 运算。

### Lines 1033-1033

```cpp
      auto curr_transform2mma_pipeline_producer_state = transform2mma_pipeline_producer_state;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1035-1039

```cpp
      // Dequantize A with scale/zero in RF
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < K_BLOCK_MAX; k_block ++){
        Utils::dequantize_A_kblock_for_transform(tArA, tArACompute, partitioned_extra_info, k_block);
      }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1041-1042

```cpp
      // Dequantized A is stored into either Smem or Tmem
      copy(dst_copy_A, tArACompute, tAsACompute(_,_,_,_,transform2mma_producer_index));
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1044-1049

```cpp
      // fence for SMEM writes
      cutlass::arch::fence_view_async_shared();
      if constexpr (is_tmem<decltype(tAsACompute)>::value) {
        // fence for TMEM writes if A operand is coming from TMEM
        cutlass::arch::fence_view_async_tmem_store();
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1051-1055

```cpp
      // Let the MMA know we are done transforming
      transform2mma_pipeline.producer_commit(curr_transform2mma_pipeline_producer_state);
      // Next pipeline stage
      ++load2transform_pipeline_consumer_state;
      ++transform2mma_pipeline_producer_state;
```
**EN:** The surrounding comments explain the local purpose of this block: Let the MMA know we are done transforming Next pipeline stage.
**CN:** 周围注释解释了这一段的局部作用：Let the MMA know we are done transforming Next pipeline stage。

### Lines 1057-1063

```cpp
      skip_wait = (k_tile_count <= 1);
      // Peek the next pipeline stage's barriers
      load2transform_flag = load2transform_pipeline.consumer_try_wait(load2transform_pipeline_consumer_state, skip_wait);
      transform2mma_flag = transform2mma_pipeline.producer_try_acquire(transform2mma_pipeline_producer_state, skip_wait);
    }
    return cute::make_tuple(load2transform_pipeline_consumer_state, transform2mma_pipeline_producer_state);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1065-1071

```cpp
  template<class ProblemShape_MNKL, class Accumulator>
  CUTLASS_DEVICE auto
  transform_init(
      Params const& params,
      ProblemShape_MNKL const& problem_shape_MNKL,
      Accumulator accumulators,
      TensorStorage& shared_storage) {
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 1073-1073

```cpp
    auto [gA_mkl, gB_nkl] = tile_input_tensors(params, problem_shape_MNKL);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1075-1077

```cpp
    Tensor sA_orig = make_tensor(make_smem_ptr(shared_storage.input.smem_A.begin()), SmemLayoutA{});
    Tensor sA = as_position_independent_swizzle_tensor(sA_orig);
    Tensor sACompute = make_tensor(make_smem_ptr(shared_storage.compute.smem_ACompute.begin()), SmemLayoutACompute{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1079-1080

```cpp
    Tensor sS = make_tensor(make_smem_ptr(shared_storage.input.smem_scale.begin()), SmemLayoutScale{}); 
    Tensor sZ = make_tensor(make_smem_ptr(shared_storage.input.smem_zero.begin()), SmemLayoutScale{}); 
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1082-1106

```cpp
    // Map input, compute, and fragment tensors to
    //   Copy strategies and partitioned tensors. These will become the input
    //   operands of the transform function. Depending on MMA atom type, the
    //   operands can reside in SMEM or TMEM
    auto setup_copy_ops = [&] (
        auto tensor_input,
        auto input_copy_atom,
        auto tensor_compute,
        auto make_fragment,
        auto compute_copy_atom) constexpr {
      auto fragment_compute = make_fragment(tensor_compute);
      if constexpr (cute::is_tmem<cute::remove_cvref_t<decltype(fragment_compute)>>::value) {
        // For M=128 with 2CTA MMA atoms, the TMEM tensor for A has a duplicated allocation.
        // Instead of allocation a 64x16 TMEM tensor, we have a 128x16 allocation
        // See: TmemAllocMode::Duplicated.
        Tensor tensor_input2x = [&] () constexpr {
        if constexpr (decltype(size<0,0>(fragment_compute) == Int<128>{} && size<0,0>(tensor_input) == Int<64>{})::value) {
          return make_tensor(tensor_input.data(),
                             logical_product(tensor_input.layout(),
                                             make_tile(make_tile(Layout<_2,_0>{},_),_,_,_))); 
          }
          else {
            return tensor_input;
          }
        }();
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1108-1113

```cpp
        fragment_compute.data() = accumulators.data().get() + cutlass::detail::find_tmem_tensor_col_offset(accumulators);
        // If operand comes from TMEM, create the TMEM_STORE based copy
        auto r2t_tiled_copy = make_tmem_copy(compute_copy_atom, fragment_compute(_,_,_,0));
        auto thr_r2t_tiled_copy = r2t_tiled_copy.get_slice(ThreadIdxX() % NumTransformationThreads);
        auto partitioned_tensor_input = thr_r2t_tiled_copy.partition_S(tensor_input2x); //(TMEM_STORE, TMEM_STORE_M, TMEM_STORE_N)
        auto partitioned_tensor_compute = thr_r2t_tiled_copy.partition_D(fragment_compute); //(TMEM_STORE, TMEM_STORE_M, TMEM_STORE_N)
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1115-1122

```cpp
        // Source copy is based on the source operand of TMEM_STORE copy.
        auto smem2reg_tiled_copy = make_tiled_copy_S(Copy_Atom<DefaultCopy, ElementA>{}, r2t_tiled_copy);
        return cute::make_tuple(smem2reg_tiled_copy, r2t_tiled_copy, partitioned_tensor_input, partitioned_tensor_compute);
      }
      else {
        auto tensor_compute_ind_sw = as_position_independent_swizzle_tensor(tensor_compute);
        auto r2s_tiled_copy = make_cotiled_copy(compute_copy_atom, Layout<Shape <_128,_8>, Stride<  _8,_1>>{},
                                                     tensor_compute(_,_,_,0).layout());
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1124-1126

```cpp
        auto smem2reg_tiled_copy = make_tiled_copy_S(input_copy_atom, r2s_tiled_copy);
        auto thr_r2s_tiled_copy = r2s_tiled_copy.get_slice(ThreadIdxX() % NumTransformationThreads);
        auto partitioned_tensor_input = thr_r2s_tiled_copy.partition_S(tensor_input); //(SMEM_STORE, SMEM_STORE_M, SMEM_STORE_N)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1128-1128

```cpp
        auto partitioned_tensor_compute = thr_r2s_tiled_copy.partition_D(tensor_compute_ind_sw);//(SMEM_STORE, SMEM_STORE_M, SMEM_STORE_N)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1131-1133

```cpp
        return cute::make_tuple(smem2reg_tiled_copy, AutoVectorizingCopy{}, partitioned_tensor_input, partitioned_tensor_compute);
      }
    };
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1135-1136

```cpp
    auto [src_copy_A, dst_copy_A, tAsA, tAsACompute] =
        setup_copy_ops(sA, InputCopyAtomA{}, sACompute, [&](auto &arg) {return TiledMma::make_fragment_A(arg);}, ComputeCopyAtomA{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1138-1142

```cpp
    // Partition of thread -> shared and thread -> RF
    auto fragment_compute = TiledMma::make_fragment_A(sACompute);
    fragment_compute.data() = accumulators.data().get() + cutlass::detail::find_tmem_tensor_col_offset(accumulators);
    auto r2t_tiled_copy = make_tmem_copy(ComputeCopyAtomA{}, fragment_compute(_,_,_,0));
    auto src_copy_scale = make_tiled_copy_S(Copy_Atom<DefaultCopy, ElementScale>{}, r2t_tiled_copy);
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1144-1144

```cpp
    auto partitioned_extra_info = Utils::partition_extra_transform_info(TiledMma{}, src_copy_scale, shared_storage);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1146-1148

```cpp
    return cute::make_tuple(gA_mkl, dst_copy_A, tAsA, tAsACompute,
                            partitioned_extra_info);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1150-1168

```cpp
  /// Perform a collective-scoped matrix multiply-accumulate
  /// Consumer Perspective
  template <
    class FrgEngine, class FrgLayout,
    class TensorA, class TensorB
  >
  CUTLASS_DEVICE auto
  mma(
      Load2MmaPipeline load2mma_pipeline,
      Load2MmaPipelineState load2mma_pipeline_consumer_state,
      Transform2MmaPipeline transform2mma_pipeline,
      Transform2MmaPipelineState transform2mma_pipeline_consumer_state,
      Mma2AccumPipeline mma2accum_pipeline,
      Mma2AccumPipelineState mma2accum_pipeline_producer_state,
      cute::Tensor<FrgEngine, FrgLayout> const& accumulators,
      cute::tuple<TensorA, TensorB> const& input_operands,
      int k_tile_count
  ) {
    TiledMma tiled_mma;
```
**EN:** This block introduces `FrgEngine` and groups related declarations around that symbol. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段引入了 `FrgEngine`，并围绕该符号组织相关声明。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1170-1171

```cpp
    auto curr_load2mma_pipeline_consumer_state = load2mma_pipeline_consumer_state;
    auto next_load2mma_pipeline_consumer_state = load2mma_pipeline_consumer_state;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1173-1174

```cpp
    auto curr_transform2mma_pipeline_consumer_state = transform2mma_pipeline_consumer_state;
    auto next_transform2mma_pipeline_consumer_state = transform2mma_pipeline_consumer_state;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1176-1180

```cpp
    uint32_t skip_wait = (k_tile_count <= 0);
    auto transform2mma_flag = transform2mma_pipeline.consumer_try_wait(next_transform2mma_pipeline_consumer_state, skip_wait);
    auto load2mma_flag = load2mma_pipeline.consumer_try_wait(next_load2mma_pipeline_consumer_state, skip_wait);
    ++next_transform2mma_pipeline_consumer_state;
    ++next_load2mma_pipeline_consumer_state;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1183-1186

```cpp
    // tCrA : (MMA), MMA_M, MMA_K, SmemStage  (In SMEM or TMEM)
    //      We use SMEM stages to match #buffers in Load <-> Convert
    // tCrB : (MMA), MMA_N, MMA_K, SmemStages (In SMEM)
    auto const [tCrA, tCrB] = input_operands;
```
**EN:** The surrounding comments explain the local purpose of this block: tCrA : (MMA), MMA_M, MMA_K, SmemStage  (In SMEM or TMEM) We use SMEM stages to match #buffers in Load <-> Convert tCrB : (MMA), MMA_N, MMA_K, SmemStages (In SMEM).
**CN:** 周围注释解释了这一段的局部作用：tCrA : (MMA), MMA_M, MMA_K, SmemStage  (In SMEM or TMEM) We use SMEM stages to match #buffers in Load <-> Convert tCrB : (MMA), MMA_N, MMA_K, SmemStages (In SMEM)。

### Lines 1188-1188

```cpp
    mma2accum_pipeline.producer_acquire(mma2accum_pipeline_producer_state);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1190-1193

```cpp
    int mma2accum_pipeline_producer_state_index = mma2accum_pipeline_producer_state.index();
    auto tCtC = accumulators(_,_,_,mma2accum_pipeline_producer_state_index);
    auto curr_mma2accum_pipeline_producer_state = mma2accum_pipeline_producer_state;
    ++mma2accum_pipeline_producer_state;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1195-1199

```cpp
    //
    // PIPELINED MAIN LOOP
    //
    // Clear the accumulator
    tiled_mma.accumulate_ = UMMA::ScaleOut::Zero;
```
**EN:** The surrounding comments explain the local purpose of this block:  PIPELINED MAIN LOOP  Clear the accumulator.
**CN:** 周围注释解释了这一段的局部作用： PIPELINED MAIN LOOP  Clear the accumulator。

### Lines 1201-1202

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1204-1205

```cpp
      load2mma_pipeline.consumer_wait(curr_load2mma_pipeline_consumer_state, load2mma_flag);
      transform2mma_pipeline.consumer_wait(curr_transform2mma_pipeline_consumer_state, transform2mma_flag);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1207-1208

```cpp
      int load2mma_pipeline_consumer_state_index = curr_load2mma_pipeline_consumer_state.index(); //read_stage
      int transform2mma_pipeline_consumer_state_index = curr_transform2mma_pipeline_consumer_state.index(); //read_stage
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1210-1211

```cpp
      auto tCrA0 = tCrA(_,_,_,transform2mma_pipeline_consumer_state_index);
      auto tCrB0 = tCrB(_,_,_,load2mma_pipeline_consumer_state_index);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1213-1217

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < size<2>(tCrA); k_block ++) {
        cute::gemm(tiled_mma, tCrA0(_,_,k_block), tCrB0(_,_,k_block), tCtC);               // A[0]*B[0]
        tiled_mma.accumulate_ = UMMA::ScaleOut::One;
      }
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine. Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1219-1220

```cpp
      load2mma_pipeline.consumer_release(curr_load2mma_pipeline_consumer_state);
      transform2mma_pipeline.consumer_release(curr_transform2mma_pipeline_consumer_state);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1222-1224

```cpp
      skip_wait = (k_tile_count <= 1);
      load2mma_flag = load2mma_pipeline.consumer_try_wait(next_load2mma_pipeline_consumer_state, skip_wait);
      transform2mma_flag = transform2mma_pipeline.consumer_try_wait(next_transform2mma_pipeline_consumer_state, skip_wait);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1226-1227

```cpp
      curr_load2mma_pipeline_consumer_state = next_load2mma_pipeline_consumer_state;
      curr_transform2mma_pipeline_consumer_state = next_transform2mma_pipeline_consumer_state;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1229-1231

```cpp
      ++next_load2mma_pipeline_consumer_state;
      ++next_transform2mma_pipeline_consumer_state;
    }
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1233-1233

```cpp
    mma2accum_pipeline.producer_commit(curr_mma2accum_pipeline_producer_state);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1235-1236

```cpp
    return cute::make_tuple(curr_load2mma_pipeline_consumer_state, curr_transform2mma_pipeline_consumer_state, mma2accum_pipeline_producer_state);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1238-1241

```cpp
  template<class FrgEngine, class FrgLayout>
  CUTLASS_DEVICE auto
  mma_init(cute::Tensor<FrgEngine, FrgLayout> const& accumulators, TensorStorage& shared_storage) const {
    TiledMma tiled_mma;
```
**EN:** This block introduces `FrgEngine` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `FrgEngine`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 1243-1253

```cpp
    auto get_tCrA = [&] () constexpr {
      if constexpr (cute::is_base_of<cute::UMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value) {
        Tensor sACompute = make_tensor(make_smem_ptr(shared_storage.compute.smem_ACompute.begin()), SmemLayoutACompute{});
        return tiled_mma.make_fragment_A(sACompute);
      }
      else {
        auto tCrA = tiled_mma.make_fragment_A(shape(SmemLayoutACompute{}));
        tCrA.data() = accumulators.data().get() + cutlass::detail::find_tmem_tensor_col_offset(accumulators);
        return tCrA;
      }
    };
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1255-1259

```cpp
    Tensor tCrA = get_tCrA();
    Tensor sB = make_tensor(make_smem_ptr(shared_storage.input.smem_B.begin()), SmemLayoutB{});
    Tensor tCrB = tiled_mma.make_fragment_B(sB);
    return cute::make_tuple(tCrA, tCrB);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1261-1265

```cpp
  template<class FrgEngine, class FrgLayout, class TmemCopyAtom, class EpilogueTile>
  CUTLASS_DEVICE auto
  accum_init(cute::Tensor<FrgEngine, FrgLayout> const& accumulators, TmemCopyAtom tmem_cp_atom, EpilogueTile epilogue_tile) {
    return accumulators;
  }
```
**EN:** This block introduces `FrgEngine` and groups related declarations around that symbol. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这一段引入了 `FrgEngine`，并围绕该符号组织相关声明。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1267-1274

```cpp
private:
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE
  constexpr auto
  tile_input_tensors(Params const& params, ProblemShape_MNKL const& problem_shape_MNKL) const {
    using X = cute::Underscore;
    // Separate out problem shape for convenience
    auto [M,N,K,L] = problem_shape_MNKL;
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 1276-1278

```cpp
    // Represent the full tensors -- get these from TMA
    Tensor mA_mkl = observed_tma_load_a_->get_tma_tensor(make_shape(M,K,L));
    Tensor mB_nkl = observed_tma_load_b_->get_tma_tensor(make_shape(N,K,L));
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 1280-1282

```cpp
    // Tile the tensors and defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});
    Tensor gB_nkl = local_tile(mB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensors and defer the slice.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensors and defer the slice。

### Lines 1284-1285

```cpp
    return cute::make_tuple(gA_mkl, gB_nkl);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1287-1288

```cpp
  typename Params::TMA_A const* observed_tma_load_a_ = nullptr;
  typename Params::TMA_B const* observed_tma_load_b_ = nullptr;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1290-1292

```cpp
  ClusterShape cluster_shape_;
  uint32_t block_rank_in_cluster_;
};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1294-1294

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 1296-1296

```cpp
} // namespace cutlass::gemm::collective
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1298-1298

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

## Key Concepts / 关键概念

- **EN:** TMA data movement  
  **CN:** TMA 数据搬运
- **EN:** UMMA tensor-core instructions  
  **CN:** UMMA 张量核指令
- **EN:** warp-specialized scheduling  
  **CN:** warp-specialized 调度
- **EN:** blockwise scaling  
  **CN:** 块级缩放
- **EN:** mixed-input operands  
  **CN:** 混合输入操作数
- **EN:** array-style MMA tiling  
  **CN:** 数组式 MMA 分块
- **EN:** Intel Xe subgroup execution  
  **CN:** Intel Xe 子组执行
- **EN:** SYCL portability hooks  
  **CN:** SYCL 可移植性钩子
- **EN:** software pipelines  
  **CN:** 软件流水线
- **EN:** compile-time validation  
  **CN:** 编译期校验

## Dependencies / 依赖关系

- **Direct includes / 直接包含:**
  - `cuda_bf16.h`
  - `cutlass/cutlass.h`
  - `cutlass/gemm/gemm.h`
  - `cutlass/gemm/dispatch_policy.hpp`
  - `cutlass/pipeline/pipeline.hpp`
  - `cutlass/numeric_conversion.h`
  - `cutlass/detail/sm100_tmem_helper.hpp`
  - `cutlass/detail/cluster.hpp`
  - `cutlass/detail/collective/mixed_input_utils.hpp`
  - `cutlass/detail/sm100_mixed_dtype_blockwise_layout.hpp`
  - `cutlass/detail/blockwise_scale_layout.hpp`
  - `cute/algorithm/functional.hpp`
  - `cute/arch/cluster_sm90.hpp`
  - `cute/atom/mma_atom.hpp`
  - `cute/atom/copy_atom.hpp`
  - `cute/algorithm/gemm.hpp`
  - `cute/arch/mma_sm100.hpp`
  - `cutlass/trace.h`
  - `cutlass/kernel_hardware_info.hpp`
- **Primary symbols / 主要符号:**
  - `ClusterShape`
  - `TileShape_`
  - `ElementAOptionalTuple_`
  - `StridePairA_`
  - `ElementBOptionalTuple_`
  - `StrideB_`
  - `TiledMma_`
  - `GmemTiledCopyA_`
  - `SmemLayoutAtomsA_`
  - `CopyAtomsA_`
  - `TransformA_`
  - `GmemTiledCopyB_`
  - `SmemLayoutAtomsB_`
  - `CopyAtomsB_`
  - `TransformB_`
  - `CollectiveMma`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。