# sm90_mma_array_tma_gmma_ss_warpspecialized_fp8_blockwise_scaling.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/sm90_mma_array_tma_gmma_ss_warpspecialized_fp8_blockwise_scaling.hpp`
- **Purpose (EN):** Implements an architecture-specific `CollectiveMma` specialization for SM90, covering FP8 accumulation paths, blockwise scaling rules, TMA transfers, GMMA instructions, warp-specialized scheduling, and 1 more.
- **用途 (CN):** 为 SM90 实现架构特化的 `CollectiveMma` 特化版本，重点覆盖 FP8 累加路径、块级缩放规则、TMA 传输、GMMA 指令、warp-specialized 调度 等 6 项。
- **Lines / 行数:** 1245

## Line-by-Line Analysis / 逐行分析

### Lines 1-31

```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 33-38

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/numeric_types.h"
#include "cutlass/pipeline/pipeline.hpp"
#include "cutlass/trace.h"
#include "cutlass/cuda_host_adapter.hpp"
```
**EN:** This include block imports cutlass.h, dispatch_policy.hpp, numeric_types.h, pipeline.hpp, and 2 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cutlass.h、dispatch_policy.hpp、numeric_types.h、pipeline.hpp 等 6 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 40-46

```cpp
#include "cute/arch/cluster_sm90.hpp"
#include "cute/arch/copy_sm80.hpp"
#include "cute/arch/copy_sm90.hpp"
#include "cute/algorithm/functional.hpp"
#include "cute/atom/mma_atom.hpp"
#include "cute/algorithm/gemm.hpp"
#include "cute/numeric/arithmetic_tuple.hpp"
```
**EN:** This include block imports cluster_sm90.hpp, copy_sm80.hpp, copy_sm90.hpp, functional.hpp, and 3 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cluster_sm90.hpp、copy_sm80.hpp、copy_sm90.hpp、functional.hpp 等 7 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 48-48

```cpp
#include "cutlass/detail/blockwise_scale_layout.hpp"
```
**EN:** This include block imports blockwise_scale_layout.hpp. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 blockwise_scale_layout.hpp。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 50-50

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 52-53

```cpp
namespace cutlass::gemm::collective {
using namespace cute;
```
**EN:** This short block both opens the collective namespace and pulls CuTe symbols into scope, setting up a concise vocabulary for the rest of the file.
**CN:** 这个短代码块既打开了 collective 命名空间，也把 CuTe 符号引入作用域，为后续代码建立更简洁的表达方式。

### Lines 55-118

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
// WarpSpecialized Mainloop
template <
  int Stages,
  class ClusterShape,
  class KernelSchedule,
  class TileShape_,
  class ElementA_,
  class StridePairA_,
  class ElementB_,
  class StridePairB_,
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
    MainloopSm90ArrayTmaGmmaWarpSpecializedBlockwise<Stages, ClusterShape, KernelSchedule>,
    TileShape_,
    ElementA_,
    StridePairA_,
    ElementB_,
    StridePairB_,
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
  //
  // Type Aliases
  //
  using DispatchPolicy = MainloopSm90ArrayTmaGmmaWarpSpecializedBlockwise<Stages, ClusterShape, KernelSchedule>;
  using TileShape = TileShape_;
  using ElementA = ElementA_;
  using StrideA = cute::tuple_element_t<0,StridePairA_>;
  using LayoutSFA = cute::tuple_element_t<1,StridePairA_>;
  using InternalStrideA = cute::remove_pointer_t<StrideA>;
  using InternalLayoutSFA = cute::remove_pointer_t<LayoutSFA>;
  using ElementB = ElementB_;
  using StrideB = cute::tuple_element_t<0,StridePairB_>;
  using LayoutSFB = cute::tuple_element_t<1,StridePairB_>;
  using InternalStrideB = cute::remove_pointer_t<StrideB>;
  using InternalLayoutSFB = cute::remove_pointer_t<LayoutSFB>;
  using TiledMma = TiledMma_;
  using ElementAccumulator = typename TiledMma::ValTypeC;
  using ElementBlockScale = ElementAccumulator;
  using GmemTiledCopyA = GmemTiledCopyA_;
  using GmemTiledCopyB = GmemTiledCopyB_;
  using SmemLayoutAtomA = SmemLayoutAtomA_;
  using SmemLayoutAtomB = SmemLayoutAtomB_;
  using SmemCopyAtomA = SmemCopyAtomA_;
  using SmemCopyAtomB = SmemCopyAtomB_;
  using TransformA = TransformA_;
  using TransformB = TransformB_;
  using ArchTag = typename DispatchPolicy::ArchTag;
```
**EN:** This template block declares or specializes `CollectiveMma`, the mainloop object that coordinates tile movement and matrix-multiply work.
**CN:** 这个模板块声明或特化了 `CollectiveMma`，它负责协调 tile 搬运与矩阵乘主循环。

### Lines 120-121

```cpp
  using MainloopPipeline = cutlass::PipelineTmaAsync<DispatchPolicy::Stages>;
  using PipelineState = cutlass::PipelineState<DispatchPolicy::Stages>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 123-124

```cpp
  using PipelineParams = typename MainloopPipeline::Params;
  using CtaShape_MNK = decltype(shape_div(TileShape{}, ClusterShape{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 126-126

```cpp
  static constexpr int NumProducerThreadEvents = 33;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 128-130

```cpp
  static constexpr int ScaleGranularityM = size<0,0>(InternalLayoutSFA{});
  static constexpr int ScaleGranularityN = size<0,0>(InternalLayoutSFB{});
  static constexpr int ScaleGranularityK = size<1,0>(InternalLayoutSFA{});
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 132-133

```cpp
  static_assert(size<2>(TileShape{}) % ScaleGranularityK == 0);
  static_assert(ScaleGranularityK % size<2>(typename TiledMma::AtomShape_MNK{}) == 0);
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 135-140

```cpp
  static constexpr int ScalePromotionInterval = ScaleGranularityK / size<2>(typename TiledMma::AtomShape_MNK{});
  static_assert(ScalePromotionInterval % 4 == 0, "ScalePromotionInterval must be a multiple of 4.");
  static_assert(ScalePromotionInterval >= size<2>(TileShape{}) / tile_size<2>(TiledMma{}),
    "ScalePromotionInterval must be greater than or equal to the number of stages of the MMA atom.");
  static_assert(ScalePromotionInterval % (size<2>(TileShape{}) / tile_size<2>(TiledMma{})) == 0,
    "ScalePromotionInterval must be a multiple of the number of stages of the MMA atom.");
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 142-143

```cpp
  static constexpr int ScaleMsPerTile = size<0>(TileShape{}) / ScaleGranularityM;
  static constexpr int ScaleNsPerTile = size<1>(TileShape{}) / ScaleGranularityN;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 145-147

```cpp
  static_assert(rank(SmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 149-151

```cpp
  static_assert(rank(SmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<1>(TileShape{}) % size<0>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 153-154

```cpp
  static_assert((size<0>(TileShape{}) % ScaleGranularityM) == 0, "FP8 scaling granularity must evenly divide tile shape along M.");
  static_assert((size<1>(TileShape{}) % ScaleGranularityN) == 0, "FP8 scaling granularity must evenly divide tile shape along N.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 156-157

```cpp
  static constexpr bool MMajorSFA = size<0,1>(InternalLayoutSFA{}.stride()) == 1;
  static constexpr bool NMajorSFB = size<0,1>(InternalLayoutSFB{}.stride()) == 1;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 159-166

```cpp
  using ScaleConfig = ::cutlass::detail::Sm90BlockwiseScaleConfig<
      ScaleGranularityM, 
      ScaleGranularityN, 
      ScaleGranularityK, 
      MMajorSFA ? cute::GMMA::Major::MN : cute::GMMA::Major::K, 
      NMajorSFB ? cute::GMMA::Major::MN : cute::GMMA::Major::K>;
  using SmemLayoutAtomSFA = decltype(ScaleConfig::smem_atom_layoutSFA(TileShape{}));
  using SmemLayoutAtomSFB = decltype(ScaleConfig::smem_atom_layoutSFB(TileShape{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 168-176

```cpp
  // Tile along modes in a way that maximizes the TMA box size.
  using SmemLayoutA = decltype(tile_to_shape(
      SmemLayoutAtomA{},
      make_shape(shape<0>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::Stages>{}),
      cute::conditional_t< ::cutlass::gemm::detail::is_major<0,StrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  using SmemLayoutB = decltype(tile_to_shape(
      SmemLayoutAtomB{},
      make_shape(shape<1>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::Stages>{}),
      cute::conditional_t< ::cutlass::gemm::detail::is_major<0,StrideB>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 178-181

```cpp
  // Block scaling gmem-to-smem copy atom
  //  we can have partial tiles in M or N, so don't vectorize those loads
  using CopyAtomSFA = Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<ElementBlockScale>, ElementBlockScale>;
  using CopyAtomSFB = Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<ElementBlockScale>, ElementBlockScale>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 183-184

```cpp
  static constexpr int AlignmentSFA = 1;
  static constexpr int AlignmentSFB = 1;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 186-194

```cpp
  // Block scaling smem layout
  using SmemLayoutSFA = decltype(make_layout(
    append(shape(SmemLayoutAtomSFA{}), Int<DispatchPolicy::Stages>{}),
    append(stride(SmemLayoutAtomSFA{}), size(filter_zeros(SmemLayoutAtomSFA{})))
  ));
  using SmemLayoutSFB = decltype(make_layout(
    append(shape(SmemLayoutAtomSFB{}), Int<DispatchPolicy::Stages>{}),
    append(stride(SmemLayoutAtomSFB{}), size(filter_zeros(SmemLayoutAtomSFB{})))
  ));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 197-204

```cpp
  static_assert(DispatchPolicy::Stages >= 2, "Specialization requires Stages set to value 2 or more.");
  static_assert(cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value &&
                cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeB>::value,
                "MMA atom must source both A and B operand from smem_desc for this mainloop.");
  static_assert(cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>,
      "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
  static_assert(cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD_MULTICAST>,
      "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 206-211

```cpp
  // TMA converts f32 input to tf32 when copying from GMEM to SMEM
  // For all other types, cast to size equivalent uint type to avoid any rounding by TMA.
  static constexpr bool ConvertF32toTF32A = cute::is_same_v<float, ElementA>;
  static constexpr bool ConvertF32toTF32B = cute::is_same_v<float, ElementB>;
  using InternalElementA = cute::conditional_t<ConvertF32toTF32A, tfloat32_t, uint_bit_t<sizeof_bits_v<ElementA>>>;
  using InternalElementB = cute::conditional_t<ConvertF32toTF32B, tfloat32_t, uint_bit_t<sizeof_bits_v<ElementB>>>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 213-214

```cpp
  static_assert(cute::is_same_v<ElementAccumulator, ElementBlockScale>,
             "ElementAccumulator and ElementBlockScale should be same datatype");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 216-222

```cpp
  struct SharedStorage {
    struct TensorStorage : cute::aligned_struct<128, _0> {
      cute::array_aligned<typename TiledMma::ValTypeA, cute::cosize_v<SmemLayoutA>> smem_A;
      cute::array_aligned<typename TiledMma::ValTypeB, cute::cosize_v<SmemLayoutB>> smem_B;
      cute::array_aligned<ElementBlockScale, cute::cosize_v<SmemLayoutSFA>> smem_SFA;
      cute::array_aligned<ElementBlockScale, cute::cosize_v<SmemLayoutSFB>> smem_SFB;
    } tensors;
```
**EN:** This block introduces `SharedStorage` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `SharedStorage`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 224-227

```cpp
    struct TensorMapStorage : cute::aligned_struct<128, _0> {
      cute::TmaDescriptor smem_tensormap_A;
      cute::TmaDescriptor smem_tensormap_B;
    } tensormaps;
```
**EN:** This block introduces `TensorMapStorage` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TensorMapStorage`，并围绕该符号组织相关声明。

### Lines 229-234

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

### Lines 236-236

```cpp
  static constexpr bool IsGroupedGemmKernel = !cute::is_same_v<InternalStrideA, StrideA>;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 238-248

```cpp
  // Host side kernel arguments
  struct Arguments {
    ElementA const** ptr_A;
    StrideA dA;
    ElementB const** ptr_B;
    StrideB dB;
    ElementBlockScale const** ptr_SFA;
    LayoutSFA layout_SFA;
    ElementBlockScale const** ptr_SFB;
    LayoutSFB layout_SFB;
  };
```
**EN:** This block introduces `Arguments` and groups related declarations around that symbol. `Arguments` is the host-side bundle: callers fill it with pointers, strides, and other launch-time values before parameter lowering.
**CN:** 这一段引入了 `Arguments`，并围绕该符号组织相关声明。 `Arguments` 是主机侧参数包：调用者在参数下沉前把指针、步长及其他启动期数据填入其中。

### Lines 250-279

```cpp
  // Device side kernel params
  struct Params {
    // Assumption: StrideA is congruent with Problem_MK
    using TMA_A = decltype(make_tma_copy_A_sm90(
        GmemTiledCopyA{},
        make_tensor(static_cast<InternalElementA const*>(nullptr), repeat_like(InternalStrideA{}, int32_t(0)), InternalStrideA{}),
        SmemLayoutA{}(_,_,cute::Int<0>{}),
        TileShape{},
        ClusterShape{}));
    // Assumption: StrideB is congruent with Problem_NK
    using TMA_B = decltype(make_tma_copy_B_sm90(
        GmemTiledCopyB{},
        make_tensor(static_cast<InternalElementB const*>(nullptr), repeat_like(InternalStrideB{}, int32_t(0)), InternalStrideB{}),
        SmemLayoutB{}(_,_,cute::Int<0>{}),
        TileShape{},
        ClusterShape{}));
    TMA_A tma_load_a;
    TMA_B tma_load_b;
    uint32_t tma_transaction_bytes = TmaTransactionBytes;
    void* tensormaps;
    InternalElementA const** ptr_A;
    StrideA dA;
    InternalElementB const** ptr_B;
    StrideB dB;
    // Block scaling factors for A and B
    ElementBlockScale const** ptr_SFA;
    LayoutSFA layout_SFA;
    ElementBlockScale const** ptr_SFB;
    LayoutSFB layout_SFB;
  };
```
**EN:** This block introduces `Params` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `Params`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 281-283

```cpp
  //
  // Methods
  //
```
**EN:** This comment starts a method section where the collective exposes its runtime behavior.
**CN:** 这条注释表示进入方法区，collective 的运行时行为将在这里定义。

### Lines 285-301

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(
      ProblemShape problem_shapes,
      Arguments const& args,
      void* workspace) {
    // These tensor shapes (only applicable for grouped gemm) and pointers are only used to create tensormap/tma desc.
    // These will be replaced with correct values before the initial tma load.
    auto init_shape = repeat_like(typename ProblemShape::UnderlyingProblemShape{}, int32_t(1));
    auto init_M = get<0>(init_shape);
    auto init_N = get<1>(init_shape);
    auto init_K = get<2>(init_shape);
    // Batches/Groups are managed by using appropriate pointers to input matrices
    const uint32_t init_L = 1;
    // NOTE: Since TMA desc creation with nullptr not possible until 12.6, we use an initial address even when tensor addresses are on device. This address is never used.
    InternalElementA const* ptr_A_first_batch = reinterpret_cast<InternalElementA const*>(reinterpret_cast<uint64_t>(args.ptr_A) & 0xFFFFFFFFFFFFFFF0);  // Address must be 16B-aligned
    InternalElementB const* ptr_B_first_batch = reinterpret_cast<InternalElementB const*>(reinterpret_cast<uint64_t>(args.ptr_B) & 0xFFFFFFFFFFFFFFF0);  // Address must be 16B-aligned
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 303-315

```cpp
    InternalStrideA stride_a;
    InternalStrideB stride_b;
    if constexpr (IsGroupedGemmKernel) {
      // Strides for Grouped Gemm will be replaced prior to the first access regardless.
      stride_a = InternalStrideA{};
      stride_b = InternalStrideB{};
    }
    else {
      // Tensor shapes for Ptr-Array are initialized correctly only here.
      auto problem_shape_MNK = problem_shapes.get_host_problem_shape(0);
      init_M = get<0>(problem_shape_MNK);
      init_N = get<1>(problem_shape_MNK);
      init_K = get<2>(problem_shape_MNK);
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 317-333

```cpp
      stride_a = args.dA;
      stride_b = args.dB;
    }
    Tensor tensor_a = make_tensor(ptr_A_first_batch, make_layout(make_shape(init_M,init_K,init_L), stride_a));
    Tensor tensor_b = make_tensor(ptr_B_first_batch, make_layout(make_shape(init_N,init_K,init_L), stride_b));
    auto tma_load_a = make_tma_copy(
         GmemTiledCopyA{},
         tensor_a,
         SmemLayoutA{}(_,_,cute::Int<0>{}),
         make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
         size<1>(ClusterShape{})); // mcast along N mode for this M load, if any
    auto tma_load_b = make_tma_copy(
         GmemTiledCopyB{},
         tensor_b,
         SmemLayoutB{}(_,_,cute::Int<0>{}),
         make_shape(shape<1>(TileShape{}), shape<2>(TileShape{})),
         size<0>(ClusterShape{})); // mcast along M mode for this N load, if any
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 335-335

```cpp
    void* tensormaps = workspace;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 337-351

```cpp
    return {
      tma_load_a,
      tma_load_b,
      TmaTransactionBytes,
      tensormaps,
      reinterpret_cast<InternalElementA const**>(args.ptr_A),
      args.dA,
      reinterpret_cast<InternalElementB const**>(args.ptr_B),
      args.dB,
      args.ptr_SFA,
      args.layout_SFA,
      args.ptr_SFB,
      args.layout_SFB
    };
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 353-360

```cpp
  template <class ProblemShape>
  static size_t
  get_workspace_size(ProblemShape const& problem_shape, Arguments const& args, int sm_count) {
    constexpr uint32_t NumInputTensors = 2;
    constexpr size_t SizeOfCuTensorMap = sizeof(cute::TmaDescriptor);
    // Allocate gmem space for input tensormaps per each SM, A tensormap copies followed by B tensormap copies
    return (NumInputTensors * SizeOfCuTensorMap * sm_count);
  }
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 362-366

```cpp
  template <class ProblemShape>
  static cutlass::Status
  initialize_workspace(ProblemShape const& problem_shape, Arguments const& args, void* workspace, cudaStream_t stream, CudaHostAdapter* cuda_adapter = nullptr) {
    return cutlass::Status::kSuccess;
  }
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 368-376

```cpp
  template<class ProblemShape>
  static bool
  can_implement(
      ProblemShape problem_shapes,
      Arguments const& args) {
    bool implementable = true;
    constexpr int tma_alignment_bits = 128;
    constexpr int min_tma_aligned_elements_A = tma_alignment_bits / cutlass::sizeof_bits<ElementA>::value;
    constexpr int min_tma_aligned_elements_B = tma_alignment_bits / cutlass::sizeof_bits<ElementB>::value;
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. This method performs runtime feasibility checks, usually validating alignment, layout assumptions, or shape constraints before launching the kernel.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这个方法执行运行时可实现性检查，通常会在启动内核前验证对齐、布局假设或形状约束。

### Lines 378-386

```cpp
    if (problem_shapes.is_host_problem_shape_available()) {
      // Check alignment for all problem sizes
      for (int i = 0; i < problem_shapes.groups(); i++) {
        auto problem_shape_MNKL = append<4>(problem_shapes.get_host_problem_shape(i), 1);
        auto [M,N,K,L] = problem_shape_MNKL;
        implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_A>(cute::make_shape(M,K,L), InternalStrideA{});
        implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_B>(cute::make_shape(N,K,L), InternalStrideB{});
      }
    }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 388-392

```cpp
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA.\n");
    }
    return implementable;
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 394-398

```cpp
  static constexpr int K_PIPE_MAX = DispatchPolicy::Stages;
  static constexpr int K_PIPE_MMAS = DispatchPolicy::PipelineAsyncMmaStages;
  static constexpr uint32_t TmaTransactionBytes =
        cutlass::bits_to_bytes(size<0>(SmemLayoutA{}) * size<1>(SmemLayoutA{}) * static_cast<uint32_t>(sizeof_bits<ElementA>::value))+
        cutlass::bits_to_bytes(size<0>(SmemLayoutB{}) * size<1>(SmemLayoutB{}) * static_cast<uint32_t>(sizeof_bits<ElementB>::value));
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 400-413

```cpp
  // Set up the data needed by this collective for load and mma.
  // Returns a tuple of tensors. The collective and the kernel layer have the contract that the
  // returned tuple must contain at least two elements, with the first two elements being:
  // gA_mkl - The tma tensor, A after a local tile so it has shape  (BLK_M,BLK_K,m,k,l)
  // gB_nkl - The tma tensor, B after a local tile so it has shape  (BLK_N,BLK_K,n,k,l)
  // The rest of the tensors can be specified as needed by this collective.
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE auto
  load_init(
    ProblemShape_MNKL const& problem_shape_MNKL,
    Params const& mainloop_params,
    ElementBlockScale const* ptr_SFA = nullptr,
    ElementBlockScale const* ptr_SFB = nullptr
  ) const {
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. This block prepares the load path by slicing global tensors, partitioning work for the current CTA/subgroup, and binding shared-memory destinations.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 这一段为加载路径做准备：切分全局张量、为当前 CTA/子组划分工作，并绑定共享内存目标。

### Lines 415-418

```cpp
    using X = Underscore;
    // Separate out problem shape for convenience
    auto [M,N,K,L] = problem_shape_MNKL;
    const int32_t init_L = 1;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 420-423

```cpp
    // TMA requires special handling of strides to deal with coord codomain mapping
    // Represent the full tensors -- get these from TMA
    Tensor mA_mkl = mainloop_params.tma_load_a.get_tma_tensor(make_shape(M,K,init_L));                        // (m,k,l)
    Tensor mB_nkl = mainloop_params.tma_load_b.get_tma_tensor(make_shape(N,K,init_L));                        // (n,k,l)
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 425-427

```cpp
    // Make tiled views, defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});         // (BLK_M,BLK_K,m,k,l)
    Tensor gB_nkl = local_tile(mB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});         // (BLK_N,BLK_K,n,k,l)
```
**EN:** The surrounding comments explain the local purpose of this block: Make tiled views, defer the slice.
**CN:** 周围注释解释了这一段的局部作用：Make tiled views, defer the slice。

### Lines 429-429

```cpp
    // Make the tiled views of scale tensors
```
**EN:** This comment-only block labels or explains the following section: Make the tiled views of scale tensors.
**CN:** 这个纯注释块用于标记或解释后续区域：Make the tiled views of scale tensors。

### Lines 431-434

```cpp
    Tensor mSFA_mkl = make_tensor(make_gmem_ptr(ptr_SFA),
        ScaleConfig::tile_atom_to_shape_SFA(make_shape(M, N, K, init_L)));                              // (scale_m,k,l)
    Tensor mSFB_nkl = make_tensor(make_gmem_ptr(ptr_SFB),
        ScaleConfig::tile_atom_to_shape_SFB(make_shape(M, N, K, init_L)));                              // (scale_n,k,l)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 436-436

```cpp
    return cute::make_tuple(gA_mkl, gB_nkl, mSFA_mkl, mSFB_nkl);
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 438-438

```cpp
  }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 440-468

```cpp
  // Perform a collective-scoped matrix multiply-accumulate
  // Producer Perspective
  template <
    class TensorA, class TensorB,
    class TensorMapA, class TensorMapB,
    class TensorScaleA, class TensorScaleB,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load(
      Params const& mainloop_params,
      MainloopPipeline pipeline,
      PipelineState smem_pipe_write,
      cute::tuple<TensorA, TensorB, TensorScaleA, TensorScaleB> const& load_inputs,
      cute::tuple<TensorMapA, TensorMapB> const& input_tensormaps,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
    int lane_predicate = cute::elect_one_sync();
    // Blockscaling: Tma loads for load_input and CpAsync for load_scale
    if (lane_predicate) {
      Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.data()), SmemLayoutA{});         // (BLK_M,BLK_K,PIPE)
      Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.data()), SmemLayoutB{});         // (BLK_N,BLK_K,PIPE)
      Tensor sSFA = make_tensor(cute::make_smem_ptr(shared_tensors.smem_SFA.data()),
          SmemLayoutSFA{});                                                                           // (BLK_M,BLK_K,P)
      Tensor sSFB = make_tensor(cute::make_smem_ptr(shared_tensors.smem_SFB.data()),
          SmemLayoutSFB{});                                                                           // (BLK_N,BLK_K,P)
```
**EN:** This block introduces `TensorA` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorA`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 470-472

```cpp
      //
      // Prepare the TMA loads for A and B
      //
```
**EN:** This comment-only block labels or explains the following section:  Prepare the TMA loads for A and B .
**CN:** 这个纯注释块用于标记或解释后续区域： Prepare the TMA loads for A and B 。

### Lines 474-475

```cpp
      constexpr uint32_t cluster_shape_x = get<0>(ClusterShape());
      uint2 cluster_local_block_id = {block_rank_in_cluster % cluster_shape_x, block_rank_in_cluster / cluster_shape_x};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 477-478

```cpp
      Tensor gA_mkl = get<0>(load_inputs);
      Tensor gB_nkl = get<1>(load_inputs);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 480-481

```cpp
      auto block_tma_a = mainloop_params.tma_load_a.get_slice(cluster_local_block_id.y);
      auto block_tma_b = mainloop_params.tma_load_b.get_slice(cluster_local_block_id.x);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 483-486

```cpp
      // Partition the inputs based on the current block coordinates.
      auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
      Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                     // (BLK_M,BLK_K,k)
      Tensor gB = gB_nkl(_,_,n_coord,_,l_coord);                                                     // (BLK_N,BLK_K,k)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition the inputs based on the current block coordinates..
**CN:** 周围注释解释了这一段的局部作用：Partition the inputs based on the current block coordinates.。

### Lines 488-490

```cpp
      // Block scaling: load_scale has scaling tensors in global memory which are not tiled
      Tensor mSFA_mkl = get<2>(load_inputs);
      Tensor mSFB_nkl = get<3>(load_inputs);
```
**EN:** The surrounding comments explain the local purpose of this block: Block scaling: load_scale has scaling tensors in global memory which are not tiled.
**CN:** 周围注释解释了这一段的局部作用：Block scaling: load_scale has scaling tensors in global memory which are not tiled。

### Lines 492-493

```cpp
      Tensor gSFA_mkl = local_tile(mSFA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});   // (BLK_M,BLK_K,m,k,l)
      Tensor gSFB_nkl = local_tile(mSFB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});   // (BLK_N,BLK_K,n,k,l)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 495-496

```cpp
      Tensor gSFA_k = gSFA_mkl(_,_,m_coord,_,l_coord);
      Tensor gSFB_k = gSFB_nkl(_,_,n_coord,_,l_coord);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 498-499

```cpp
      TiledCopy scale_copy_a = make_tiled_copy(CopyAtomSFA{}, Layout<Shape<_1>>{}, Layout<Shape<_1>>{});
      TiledCopy scale_copy_b = make_tiled_copy(CopyAtomSFB{}, Layout<Shape<_1>>{}, Layout<Shape<_1>>{});
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 501-502

```cpp
      ThrCopy thr_scale_copy_a = scale_copy_a.get_slice(_0{});
      ThrCopy thr_scale_copy_b = scale_copy_b.get_slice(_0{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 504-505

```cpp
      Tensor tSFAgSFA_k = thr_scale_copy_a.partition_S(gSFA_k);
      Tensor tSFAsSFA   = thr_scale_copy_a.partition_D(sSFA);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 507-508

```cpp
      Tensor tSFBgSFB_k = thr_scale_copy_b.partition_S(gSFB_k);
      Tensor tSFBsSFB   = thr_scale_copy_b.partition_D(sSFB);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 510-512

```cpp
      // Applies the mapping from block_tma_a
      Tensor tAgA = block_tma_a.partition_S(gA);                                                 // (TMA,TMA_M,TMA_K,k)
      Tensor tAsA = block_tma_a.partition_D(sA);                                              // (TMA,TMA_M,TMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Applies the mapping from block_tma_a.
**CN:** 周围注释解释了这一段的局部作用：Applies the mapping from block_tma_a。

### Lines 514-515

```cpp
      Tensor tBgB = block_tma_b.partition_S(gB);                                                 // (TMA,TMA_N,TMA_K,k)
      Tensor tBsB = block_tma_b.partition_D(sB);                                              // (TMA,TMA_N,TMA_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 517-518

```cpp
      uint16_t mcast_mask_a = 0;
      uint16_t mcast_mask_b = 0;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 520-527

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

### Lines 529-534

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

### Lines 536-540

```cpp
      // Mainloop
      CUTLASS_PRAGMA_NO_UNROLL
      for ( ; k_tile_count > 0; --k_tile_count) {
        // LOCK smem_pipe_write for _writing_
        pipeline.producer_acquire(smem_pipe_write);
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 542-544

```cpp
        //
        // Copy gmem to smem for *k_tile_iter
        //
```
**EN:** This comment-only block labels or explains the following section:  Copy gmem to smem for *k_tile_iter .
**CN:** 这个纯注释块用于标记或解释后续区域： Copy gmem to smem for *k_tile_iter 。

### Lines 546-547

```cpp
        using BarrierType = typename MainloopPipeline::ProducerBarrierType;
        BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 549-551

```cpp
        int write_stage = smem_pipe_write.index();
        copy(mainloop_params.tma_load_a.with(get<0>(input_tensormaps), *tma_barrier, mcast_mask_a), tAgA(_,_,_,*k_tile_iter), tAsA(_,_,_,write_stage));
        copy(mainloop_params.tma_load_b.with(get<1>(input_tensormaps), *tma_barrier, mcast_mask_b), tBgB(_,_,_,*k_tile_iter), tBsB(_,_,_,write_stage));
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 553-553

```cpp
        ++k_tile_iter;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 555-559

```cpp
        // Advance smem_pipe_write
        ++smem_pipe_write;
      }
    }
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Advance smem_pipe_write.
**CN:** 周围注释解释了这一段的局部作用：Advance smem_pipe_write。

### Lines 561-564

```cpp
  // Perform a Producer Epilogue to prevent early exit of blocks in a Cluster
  CUTLASS_DEVICE void
  load_tail(MainloopPipeline pipeline, PipelineState smem_pipe_write) {
    int lane_predicate = cute::elect_one_sync();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 566-575

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

### Lines 577-602

```cpp
  // Perform a collective-scoped matrix multiply-accumulate
  // Producer Perspective
  template <
    class TensorA, class TensorB,
    class TensorSFA, class TensorSFB,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load_auxiliary(
      Params const& mainloop_params,
      MainloopPipeline pipeline,
      PipelineState smem_pipe_write,
      cute::tuple<TensorA,
                  TensorB,
                  TensorSFA,
                  TensorSFB> const& load_inputs,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
    int lane_predicate = cute::elect_one_sync();
    Tensor sSFA = make_tensor(cute::make_smem_ptr(shared_tensors.smem_SFA.data()),
        SmemLayoutSFA{});                                                                             // (BLK_M,BLK_K,P)
    Tensor sSFB = make_tensor(cute::make_smem_ptr(shared_tensors.smem_SFB.data()),
        SmemLayoutSFB{});                                                                             // (BLK_N,BLK_K,P)
```
**EN:** This block introduces `TensorA` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorA`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 604-605

```cpp
    // Partition the inputs based on the current block coordinates.
    auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
```
**EN:** The surrounding comments explain the local purpose of this block: Partition the inputs based on the current block coordinates..
**CN:** 周围注释解释了这一段的局部作用：Partition the inputs based on the current block coordinates.。

### Lines 607-611

```cpp
    // Block scaling: load_scale has scaling tensors in global memory which are not tiled
    Tensor mSFA_mkl  = get<2>(load_inputs);
    Tensor mSFB_nkl  = get<3>(load_inputs);
    Layout layoutSFA = mSFA_mkl.layout();
    Layout layoutSFB = mSFB_nkl.layout();
```
**EN:** The surrounding comments explain the local purpose of this block: Block scaling: load_scale has scaling tensors in global memory which are not tiled.
**CN:** 周围注释解释了这一段的局部作用：Block scaling: load_scale has scaling tensors in global memory which are not tiled。

### Lines 613-614

```cpp
    Tensor iSFA_mkl = make_identity_tensor(shape(layoutSFA));                                // (m,k,l)
    Tensor iSFB_nkl = make_identity_tensor(shape(layoutSFB));                                // (n,k,l)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 617-620

```cpp
    Tensor gSFA_mkl = local_tile(mSFA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});     // (BLK_M,BLK_K,m,k,l)
    Tensor cSFA_mkl = local_tile(iSFA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});     // (BLK_M,BLK_K,m,k,l)
    Tensor gSFB_nkl = local_tile(mSFB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});     // (BLK_N,BLK_K,n,k,l)
    Tensor cSFB_nkl = local_tile(iSFB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});     // (BLK_N,BLK_K,n,k,l)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 622-625

```cpp
    Tensor gSFA_k = gSFA_mkl(_,_,m_coord,_,l_coord);
    Tensor cSFA_k = cSFA_mkl(_,_,m_coord,_,l_coord);
    Tensor gSFB_k = gSFB_nkl(_,_,n_coord,_,l_coord);
    Tensor cSFB_k = cSFB_nkl(_,_,n_coord,_,l_coord);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 627-628

```cpp
    TiledCopy scale_copy_a = make_tiled_copy(CopyAtomSFA{}, Layout<Shape<_32>>{}, Layout<Shape<_1>>{});
    TiledCopy scale_copy_b = make_tiled_copy(CopyAtomSFB{}, Layout<Shape<_32>>{}, Layout<Shape<_1>>{});
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 630-631

```cpp
    ThrCopy thr_scale_copy_a = scale_copy_a.get_slice(thread_idx);
    ThrCopy thr_scale_copy_b = scale_copy_b.get_slice(thread_idx);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 633-635

```cpp
    Tensor tSFAgSFA_k = thr_scale_copy_a.partition_S(gSFA_k);
    Tensor tSFAcSFA_k = thr_scale_copy_a.partition_S(cSFA_k);
    Tensor tSFAsSFA   = thr_scale_copy_a.partition_D(sSFA);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 637-639

```cpp
    Tensor tSFBgSFB_k = thr_scale_copy_b.partition_S(gSFB_k);
    Tensor tSFBcSFB_k = thr_scale_copy_b.partition_S(cSFB_k);
    Tensor tSFBsSFB   = thr_scale_copy_b.partition_D(sSFB);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 641-642

```cpp
    Tensor tSFApSFA = make_tensor<bool>(shape(filter_zeros(tSFAsSFA(_,_,_,_0{}))));                 // (CPY,CPY_M,CPY_K)
    Tensor tSFBpSFB = make_tensor<bool>(shape(filter_zeros(tSFBsSFB(_,_,_,_0{}))));                 // (CPY,CPY_N,CPY_K)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 644-645

```cpp
    auto SFA_shape = shape(layoutSFA);
    auto SFB_shape = shape(layoutSFB);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 647-651

```cpp
    // Mainloop
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
      // LOCK smem_pipe_write for _writing_
      pipeline.producer_acquire(smem_pipe_write);
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 653-660

```cpp
      // Since scale granularity K is multiple of BLK_K we do not have to consider if that is OOB
      bool load_sfa = thread_idx < ScaleMsPerTile;
      Tensor tSFAcSFA = tSFAcSFA_k(_,_,_,*k_tile_iter);
      Tensor tSFAcSFA_compact = filter_zeros(tSFAcSFA);
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(tSFApSFA); ++i) {
        tSFApSFA(i) = load_sfa && elem_less(tSFAcSFA_compact(i), SFA_shape);
      }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 662-668

```cpp
      bool load_sfb = thread_idx < ScaleNsPerTile;
      Tensor tSFBcSFB = tSFBcSFB_k(_,_,_,*k_tile_iter);
      Tensor tSFBcSFB_compact = filter_zeros(tSFBcSFB);
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(tSFBpSFB); ++i) {
        tSFBpSFB(i) = load_sfb && elem_less(tSFBcSFB_compact(i), SFB_shape);
      }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 670-673

```cpp
      //
      // Copy gmem to smem for *k_tile_iter
      //
      int write_stage = smem_pipe_write.index();
```
**EN:** The surrounding comments explain the local purpose of this block:  Copy gmem to smem for *k_tile_iter .
**CN:** 周围注释解释了这一段的局部作用： Copy gmem to smem for *k_tile_iter 。

### Lines 675-677

```cpp
      // Copy scale tensors from global memory to shared memory
      copy_if(scale_copy_a, tSFApSFA, filter_zeros(tSFAgSFA_k(_,_,_,*k_tile_iter)), filter_zeros(tSFAsSFA(_,_,_,write_stage)));
      copy_if(scale_copy_b, tSFBpSFB, filter_zeros(tSFBgSFB_k(_,_,_,*k_tile_iter)), filter_zeros(tSFBsSFB(_,_,_,write_stage)));
```
**EN:** The surrounding comments explain the local purpose of this block: Copy scale tensors from global memory to shared memory.
**CN:** 周围注释解释了这一段的局部作用：Copy scale tensors from global memory to shared memory。

### Lines 679-679

```cpp
      pipeline.producer_commit(smem_pipe_write, cutlass::arch::cpasync_barrier_arrive_noinc);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 681-681

```cpp
      ++k_tile_iter;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 683-686

```cpp
      // Advance smem_pipe_write
      ++smem_pipe_write;
    }
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Advance smem_pipe_write.
**CN:** 周围注释解释了这一段的局部作用：Advance smem_pipe_write。

### Lines 689-703

```cpp
  template<
    class EngineAccum,
    class LayoutAccum,
    class ScaleFactor
  >
  CUTLASS_DEVICE
  void scale_if_needed(GmmaFP8Accumulation<EngineAccum, LayoutAccum>& accumulation, ScaleFactor scaleFactor) {
    if constexpr (ScalePromotionInterval != 4) {
      accumulation.scale_if_needed(scaleFactor);
    }
    else {
      // avoid unnecessary tests when granularity is the finnest
      accumulation.scale(scaleFactor);
    }
  }
```
**EN:** This block introduces `EngineAccum` and groups related declarations around that symbol. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段引入了 `EngineAccum`，并围绕该符号组织相关声明。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 705-720

```cpp
  template<
    class EngineAccum,
    class LayoutAccum,
    class ScaleFactor1,
    class ScaleFactor2
  >
  CUTLASS_DEVICE
  void scale_if_needed(GmmaFP8Accumulation<EngineAccum, LayoutAccum>& accumulation, ScaleFactor1 scaleFactor1, ScaleFactor2 scaleFactor2) {
    if constexpr (ScalePromotionInterval != 4) {
      accumulation.scale_if_needed(scaleFactor1, scaleFactor2);
    }
    else {
      // avoid unnecessary tests when granularity is the finnest
      accumulation.scale(scaleFactor1, scaleFactor2);
    }
  }
```
**EN:** This block introduces `EngineAccum` and groups related declarations around that symbol. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段引入了 `EngineAccum`，并围绕该符号组织相关声明。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 723-742

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
    static_assert(is_rmem<FrgTensorC>::value, "C tensor must be rmem resident.");
    static_assert(rank(SmemLayoutA{}) == 3, "Smem layout must be rank 3.");
    static_assert(rank(SmemLayoutB{}) == 3, "Smem layout must be rank 3.");
    static_assert(cute::is_void_v<SmemCopyAtomA>,
      "SM90 GMMA mainloops cannot have a non-void copy atom for smem sourced instructions.");
    static_assert(cute::is_void_v<SmemCopyAtomB>,
      "SM90 GMMA mainloops cannot have a non-void copy atom for smem sourced instructions.");
```
**EN:** This block introduces `FrgTensorC` and groups related declarations around that symbol. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这一段引入了 `FrgTensorC`，并围绕该符号组织相关声明。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 744-745

```cpp
    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.data()), SmemLayoutA{});           // (BLK_M,BLK_K,PIPE)
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.data()), SmemLayoutB{});           // (BLK_N,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 747-766

```cpp
    // Block scaling
    Tensor sSFA = make_tensor(cute::make_smem_ptr(shared_tensors.smem_SFA.data()), make_layout(
        make_shape(shape<0>(SmemLayoutSFA{}),
                   get<1>(TileShape{}),
                   make_shape(shape<1>(SmemLayoutSFA{}),
                              shape<2>(SmemLayoutSFA{}))),
        make_stride(stride<0>(SmemLayoutSFA{}), _0{},
                    make_stride(stride<1>(SmemLayoutSFA{}),
                                stride<2>(SmemLayoutSFA{})))
      ));                                                                                     // (BLK_M,BLK_N,(BLK_K,P))
    Tensor sSFB = make_tensor(cute::make_smem_ptr(shared_tensors.smem_SFB.data()), make_layout(
        make_shape(get<0>(TileShape{}),
                   shape<0>(SmemLayoutSFB{}),
                   make_shape(shape<1>(SmemLayoutSFB{}),
                              shape<2>(SmemLayoutSFB{}))),
        make_stride(_0{},
                    stride<0>(SmemLayoutSFB{}),
                    make_stride(stride<1>(SmemLayoutSFB{}),
                                stride<2>(SmemLayoutSFB{})))
      ));                                                                                     // (BLK_M,BLK_N,(BLK_K,P))
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 769-771

```cpp
    //
    // Define C accumulators and A/B partitioning
    //
```
**EN:** This comment-only block labels or explains the following section:  Define C accumulators and A/B partitioning .
**CN:** 这个纯注释块用于标记或解释后续区域： Define C accumulators and A/B partitioning 。

### Lines 773-773

```cpp
    // Layout of warp group to thread mapping
```
**EN:** This comment highlights a warp-specialized design choice for the following implementation.
**CN:** 这条注释强调后续实现采用了 warp-specialized 的设计。

### Lines 775-779

```cpp
    static_assert(stride<0>(typename TiledMma::ALayout{}) == 0 and
                  stride<0>(typename TiledMma::BLayout{}) == 0 and
                  size<0>(typename TiledMma::ALayout{}) == NumThreadsPerWarpGroup and
                  size<0>(typename TiledMma::BLayout{}) == NumThreadsPerWarpGroup,
                  "Stride of the first mode must be 0 and the size of the mode must be NumThreadsPerWarpGroup");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 781-783

```cpp
    constexpr int MmaWarpGroups = size(TiledMma{}) / NumThreadsPerWarpGroup;
    Layout warp_group_thread_layout = make_layout(Int<MmaWarpGroups>{},
                                                  Int<NumThreadsPerWarpGroup>{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 785-785

```cpp
    int warp_group_idx = shfl_sync(0xFFFFFFFF, thread_idx / NumThreadsPerWarpGroup, 0);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 787-788

```cpp
    TiledMma tiled_mma;
    auto thread_mma = tiled_mma.get_slice(warp_group_thread_layout(warp_group_idx));
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 790-791

```cpp
    Tensor tCsSFA = tiled_mma.get_slice(thread_idx).partition_C(sSFA);                 // (MMA,MMA_M,MMA_N,(MMA_K,PIPE))
    Tensor tCsSFB = tiled_mma.get_slice(thread_idx).partition_C(sSFB);                 // (MMA,MMA_M,MMA_N,(MMA_K,PIPE))
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 794-795

```cpp
    Tensor tCsA = thread_mma.partition_A(sA);                                                 // (MMA,MMA_M,MMA_K,PIPE)
    Tensor tCsB = thread_mma.partition_B(sB);                                                 // (MMA,MMA_N,MMA_K,PIPE)
```
**EN:** This block prepares the load path by slicing global tensors, partitioning work for the current CTA/subgroup, and binding shared-memory destinations.
**CN:** 这一段为加载路径做准备：切分全局张量、为当前 CTA/子组划分工作，并绑定共享内存目标。

### Lines 797-799

```cpp
    // Allocate "fragments/descriptors"
    Tensor tCrA = thread_mma.make_fragment_A(tCsA);                                           // (MMA,MMA_M,MMA_K,PIPE)
    Tensor tCrB = thread_mma.make_fragment_B(tCsB);                                           // (MMA,MMA_N,MMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Allocate "fragments/descriptors".
**CN:** 周围注释解释了这一段的局部作用：Allocate "fragments/descriptors"。

### Lines 801-806

```cpp
    CUTE_STATIC_ASSERT_V(size<1>(tCsA) == size<1>(accum));                                                         // M
    CUTE_STATIC_ASSERT_V(size<1>(tCsB) == size<2>(accum));                                                         // N
    CUTE_STATIC_ASSERT_V(size<2>(tCsA) == size<2>(tCsB));                                                          // K
    CUTE_STATIC_ASSERT_V(size<3>(tCsA) == size<3>(tCsB));                                                       // PIPE
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<2>(sA));                                         // PIPE
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<2>(sB));                                         // PIPE
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 808-812

```cpp
    //
    // PIPELINED MAIN LOOP
    //
    static_assert((0 <= K_PIPE_MMAS) && (K_PIPE_MMAS <  K_PIPE_MAX),
        "ERROR : Incorrect number of MMAs in flight");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 814-815

```cpp
    // We release buffers to producer warps(dma load) with some mmas in flight
    PipelineState smem_pipe_release = smem_pipe_read;
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 817-820

```cpp
    // Per block scale values for operand A and B
    // Since scale factors always broadcast across MMA_K we slice that away
    Tensor tCrSFA = make_tensor_like<ElementBlockScale>(tCsSFA(_, _, _, _0{}));                     // (MMA,MMA_M,MMA_N)
    Tensor tCrSFB = make_tensor_like<ElementBlockScale>(tCsSFB(_, _, _, _0{}));                     // (MMA,MMA_M,MMA_N)
```
**EN:** The surrounding comments explain the local purpose of this block: Per block scale values for operand A and B Since scale factors always broadcast across MMA_K we slice that away.
**CN:** 周围注释解释了这一段的局部作用：Per block scale values for operand A and B Since scale factors always broadcast across MMA_K we slice that away。

### Lines 822-823

```cpp
    // Prologue GMMAs
    tiled_mma.accumulate_ = GMMA::ScaleOut::Zero;
```
**EN:** The surrounding comments explain the local purpose of this block: Prologue GMMAs.
**CN:** 周围注释解释了这一段的局部作用：Prologue GMMAs。

### Lines 825-827

```cpp
    auto barrier_token = pipeline.consumer_try_wait(smem_pipe_read);
    GmmaFP8Accumulation accumulation(accum, ScalePromotionInterval, size<2>(tCrA));
    warpgroup_fence_operand(accumulation());
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 829-831

```cpp
    if (k_tile_count > 0) {
      // WAIT on smem_pipe_read until its data are available (phase bit flips from rdPhaseBit value)
      pipeline.consumer_wait(smem_pipe_read, barrier_token);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 833-836

```cpp
      int read_stage = smem_pipe_read.index();
      // Load per block scale values from shared memory to registers
      copy(tCsSFA(_,_,_,make_coord(_0{},read_stage)), tCrSFA);
      copy(tCsSFB(_,_,_,make_coord(_0{},read_stage)), tCrSFB);
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 838-848

```cpp
      warpgroup_fence_operand(accumulation());
      warpgroup_arrive();
      // Unroll the K mode manually to set scale D to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < size<2>(tCrA); ++k_block) {
        // (V,M) x (V,N) => (V,M,N)
        cute::gemm(tiled_mma, tCrA(_,_,k_block,read_stage), tCrB(_,_,k_block,read_stage), accumulation());
        tiled_mma.accumulate_ = GMMA::ScaleOut::One;
      }
      warpgroup_commit_batch();
      warpgroup_fence_operand(accumulation());
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 851-867

```cpp
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        tCrSFA(_0{}) = tCrSFA(_0{}) * tCrSFB(_0{});
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        ElementBlockScale scale_b = tCrSFB(_0{});
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(filter_zeros(tCrSFA)); i++) {
          filter_zeros(tCrSFA)(i) = filter_zeros(tCrSFA)(i) * scale_b;
        }
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        ElementBlockScale scale_a = tCrSFA(_0{});
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(filter_zeros(tCrSFB)); i++) {
          filter_zeros(tCrSFB)(i) = filter_zeros(tCrSFB)(i) * scale_a;
        }
      }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 869-871

```cpp
      warpgroup_wait<0>();
      ++smem_pipe_read;
      barrier_token = pipeline.consumer_try_wait(smem_pipe_read);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 873-887

```cpp
      // Block scale the accumulators with reg tensor `tCrSFA` and `tCrSFB`
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        ElementBlockScale scale_ab = tCrSFA(_0{});
        scale_if_needed(accumulation, scale_ab);
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        scale_if_needed(accumulation, tCrSFA);
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        scale_if_needed(accumulation, tCrSFB);
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile  > 1) {
        scale_if_needed(accumulation, tCrSFA, tCrSFB);
      }
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 889-889

```cpp
    warpgroup_fence_operand(accumulation());
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 891-892

```cpp
    // Mainloop GMMAs
    k_tile_count--;
```
**EN:** This comment marks the mainloop logic that repeatedly loads tiles and issues MMA operations.
**CN:** 这条注释标记了主循环逻辑：重复加载 tile 并发起 MMA 运算。

### Lines 894-897

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 1; --k_tile_count)
    {
      pipeline.consumer_wait(smem_pipe_read, barrier_token);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 899-901

```cpp
      //
      // Compute on k_tile
      //
```
**EN:** This comment-only block labels or explains the following section:  Compute on k_tile .
**CN:** 这个纯注释块用于标记或解释后续区域： Compute on k_tile 。

### Lines 903-903

```cpp
      int read_stage = smem_pipe_read.index();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 905-907

```cpp
      // Load per block scale values from shared memory to registers (at most twice per block along M and/or N)
      copy(tCsSFA(_,_,_,make_coord(_0{}, read_stage)), tCrSFA);
      copy(tCsSFB(_,_,_,make_coord(_0{}, read_stage)), tCrSFB);
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 909-917

```cpp
      if constexpr (ScalePromotionInterval != 4) {
        if (accumulation.prepare_if_needed()) {
          tiled_mma.accumulate_ = GMMA::ScaleOut::Zero;
        }
      }
      else {
        // Always zero out the accumulator for finest granularity
        tiled_mma.accumulate_ = GMMA::ScaleOut::Zero;
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 919-928

```cpp
      warpgroup_fence_operand(accumulation());
      warpgroup_arrive();
      // Unroll the K mode manually to set scale D to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < size<2>(tCrA); ++k_block) {
        // (V,M) x (V,N) => (V,M,N)
        cute::gemm(tiled_mma, tCrA(_,_,k_block,read_stage), tCrB(_,_,k_block,read_stage), accumulation());
        tiled_mma.accumulate_ = GMMA::ScaleOut::One;
      }
      warpgroup_commit_batch();
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 930-931

```cpp
      /// Wait on the GMMA barrier for K_PIPE_MMAS (or fewer) outstanding to ensure smem_pipe_write is consumed
      warpgroup_fence_operand(accumulation());
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 933-949

```cpp
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        tCrSFA(_0{}) = tCrSFA(_0{}) * tCrSFB(_0{});
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        ElementBlockScale scale_b = tCrSFB(_0{});
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(filter_zeros(tCrSFA)); i++) {
          filter_zeros(tCrSFA)(i) = filter_zeros(tCrSFA)(i) * scale_b;
        }
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        ElementBlockScale scale_a = tCrSFA(_0{});
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(filter_zeros(tCrSFB)); i++) {
          filter_zeros(tCrSFB)(i) = filter_zeros(tCrSFB)(i) * scale_a;
        }
      }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 951-954

```cpp
      warpgroup_wait<0>();
      pipeline.consumer_release(smem_pipe_release); // Unlock previous tile
      ++smem_pipe_read;
      barrier_token = pipeline.consumer_try_wait(smem_pipe_read);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 956-969

```cpp
      // Block scale the accumulators with reg tensor `tCrSFA` and `tCrSFB`
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        ElementBlockScale scale_ab = tCrSFA(_0{});
        scale_if_needed(accumulation, scale_ab);
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        scale_if_needed(accumulation, tCrSFA);
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        scale_if_needed(accumulation, tCrSFB);
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile  > 1) {
        scale_if_needed(accumulation, tCrSFA, tCrSFB);
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 971-973

```cpp
      // Advance smem_pipe_read and smem_pipe_release
      ++smem_pipe_release;
    }
```
**EN:** The surrounding comments explain the local purpose of this block: Advance smem_pipe_read and smem_pipe_release.
**CN:** 周围注释解释了这一段的局部作用：Advance smem_pipe_read and smem_pipe_release。

### Lines 975-976

```cpp
    if (k_tile_count > 0) {
      pipeline.consumer_wait(smem_pipe_read, barrier_token);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 978-980

```cpp
      //
      // Compute on k_tile
      //
```
**EN:** This comment-only block labels or explains the following section:  Compute on k_tile .
**CN:** 这个纯注释块用于标记或解释后续区域： Compute on k_tile 。

### Lines 982-985

```cpp
      int read_stage = smem_pipe_read.index();
      // Load per block scale values from shared memory to registers (at most twice per block along M and/or N)
      copy(tCsSFA(_,_,_,make_coord(_0{}, read_stage)), tCrSFA);
      copy(tCsSFB(_,_,_,make_coord(_0{}, read_stage)), tCrSFB);
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 987-995

```cpp
      if constexpr (ScalePromotionInterval != 4) {
        if (accumulation.prepare_if_needed()) {
          tiled_mma.accumulate_ = GMMA::ScaleOut::Zero;
        }
      }
      else {
        // Always zero out the accumulator for finest granularity
        tiled_mma.accumulate_ = GMMA::ScaleOut::Zero;
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 997-1006

```cpp
      warpgroup_fence_operand(accumulation());
      warpgroup_arrive();
      // Unroll the K mode manually to set scale D to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < size<2>(tCrA); ++k_block) {
        // (V,M) x (V,N) => (V,M,N)
        cute::gemm(tiled_mma, tCrA(_,_,k_block,read_stage), tCrB(_,_,k_block,read_stage), accumulation());
        tiled_mma.accumulate_ = GMMA::ScaleOut::One;
      }
      warpgroup_commit_batch();
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1008-1009

```cpp
      /// Wait on the GMMA barrier for K_PIPE_MMAS (or fewer) outstanding to ensure smem_pipe_write is consumed
      warpgroup_fence_operand(accumulation());
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1011-1029

```cpp
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        tCrSFA(_0{}) = tCrSFA(_0{}) * tCrSFB(_0{});
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        ElementBlockScale scale_b = tCrSFB(_0{});
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(filter_zeros(tCrSFA)); i++) {
          filter_zeros(tCrSFA)(i) = filter_zeros(tCrSFA)(i) * scale_b;
        }
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        ElementBlockScale scale_a = tCrSFA(_0{});
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(filter_zeros(tCrSFB)); i++) {
          filter_zeros(tCrSFB)(i) = filter_zeros(tCrSFB)(i) * scale_a;
        }
      }
      warpgroup_wait<0>();
      pipeline.consumer_release(smem_pipe_release); // Unlock previous tile
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1031-1061

```cpp
      // Block scale the accumulators with reg tensor `tCrSFA` and `tCrSFB`
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        ElementBlockScale scale_ab = tCrSFA(_0{});
        scale_if_needed(accumulation, scale_ab);
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        scale_if_needed(accumulation, tCrSFA);
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        scale_if_needed(accumulation, tCrSFB);
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile  > 1) {
        scale_if_needed(accumulation, tCrSFA, tCrSFB);
      }
    }
    if constexpr (ScalePromotionInterval != 4) {
      // residues only exists when granularity is not the finnest
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        ElementBlockScale scale_ab = tCrSFA(_0{});
        accumulation.scale_residue_if_needed(scale_ab);
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        accumulation.scale_residue_if_needed(tCrSFA);
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        accumulation.scale_residue_if_needed(tCrSFB);
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile  > 1) {
        accumulation.scale_residue_if_needed(tCrSFA, tCrSFB);
      }
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1063-1063

```cpp
    warpgroup_fence_operand(accumulation());
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1065-1065

```cpp
  }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1067-1075

```cpp
  /// Perform a Consumer Epilogue to release all buffers
  CUTLASS_DEVICE void
  mma_tail(MainloopPipeline pipeline, PipelineState smem_pipe_release, int k_tile_count) {
    if (k_tile_count > 0) {
      // The pipeline is not released in the first iteration
      smem_pipe_release.advance(k_tile_count - 1);
      pipeline.consumer_release(smem_pipe_release);
    }
  }
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1077-1079

```cpp
  //
  // Methods to perform different parts of TMA/Tensormap modifications
  //
```
**EN:** This comment starts a method section where the collective exposes its runtime behavior.
**CN:** 这条注释表示进入方法区，collective 的运行时行为将在这里定义。

### Lines 1081-1087

```cpp
  CUTLASS_DEVICE auto
  tensormaps_init(
      Params const& mainloop_params,
      TensorMapStorage& shared_tensormaps,
      int32_t sm_count,
      int32_t sm_idx) {
    cute::TmaDescriptor* gmem_tensormap = reinterpret_cast<cute::TmaDescriptor*>(mainloop_params.tensormaps);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1089-1090

```cpp
    cute::TmaDescriptor* tma_desc_a = &gmem_tensormap[sm_idx];
    cute::TmaDescriptor* tma_desc_b = &gmem_tensormap[sm_idx + sm_count];
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1092-1097

```cpp
    if (cute::elect_one_sync()) {
      // Bringing tensormaps from params to smem for modification later
      Tensor pA_tensormap = make_tensor(mainloop_params.tma_load_a.get_tma_descriptor(), Int<1>{}, Int<1>{});
      Tensor sA_tensormap = make_tensor(make_smem_ptr(&shared_tensormaps.smem_tensormap_A), Int<1>{}, Int<1>{});
      Tensor pB_tensormap = make_tensor(mainloop_params.tma_load_b.get_tma_descriptor(), Int<1>{}, Int<1>{});
      Tensor sB_tensormap = make_tensor(make_smem_ptr(&shared_tensormaps.smem_tensormap_B), Int<1>{}, Int<1>{});
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1099-1102

```cpp
      copy(recast<uint128_t>(pA_tensormap), recast<uint128_t>(sA_tensormap));
      copy(recast<uint128_t>(pB_tensormap), recast<uint128_t>(sB_tensormap));
    }
    syncwarp();
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1104-1105

```cpp
    return cute::make_tuple(tma_desc_a, tma_desc_b);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 1107-1119

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
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Replace address for the global tensor (to be done by single thread) Replacing global_address for the next batch.
**CN:** 周围注释解释了这一段的局部作用：Replace address for the global tensor (to be done by single thread) Replacing global_address for the next batch。

### Lines 1121-1138

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
    const uint32_t M = get<0>(problem_shape_mnkl);
    const uint32_t N = get<1>(problem_shape_mnkl);
    const uint32_t K = get<2>(problem_shape_mnkl);
    // Replace all dims for consistency
    constexpr int MaxTensorRank = 5;
    cute::array<uint32_t, MaxTensorRank> prob_shape_A  = {1,1,1,1,1};
    cute::array<uint64_t, MaxTensorRank> prob_stride_A = {0,0,0,0,0};
    cute::array<uint32_t, MaxTensorRank> prob_shape_B  = {1,1,1,1,1};
    cute::array<uint64_t, MaxTensorRank> prob_stride_B = {0,0,0,0,0};
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。

### Lines 1140-1141

```cpp
    InternalElementA const* ptr_A = nullptr;
    Tensor tensor_a = make_tensor(ptr_A, make_shape(M,K,Int<1>{}), mainloop_params.dA[next_group]);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1143-1144

```cpp
    InternalElementB const* ptr_B = nullptr;
    Tensor tensor_b = make_tensor(ptr_B, make_shape(N,K,Int<1>{}), mainloop_params.dB[next_group]);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1146-1149

```cpp
    cute::detail::fill_tma_gmem_shape_stride(mainloop_params.tma_load_a, tensor_a,
                                             prob_shape_A, prob_stride_A);
    cute::detail::fill_tma_gmem_shape_stride(mainloop_params.tma_load_b, tensor_b,
                                             prob_shape_B, prob_stride_B);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1151-1157

```cpp
    // Convert strides to byte strides
    for (uint64_t& stride : prob_stride_A) {
      stride = (stride * sizeof_bits_v<InternalElementA>) / 8;
    }
    for (uint64_t& stride : prob_stride_B) {
      stride = (stride * sizeof_bits_v<InternalElementB>) / 8;
    }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 1159-1165

```cpp
    cute::tma_descriptor_replace_dims_strides_in_shared_mem(shared_tensormaps.smem_tensormap_A,
                                                            prob_shape_A,
                                                            prob_stride_A);
    cute::tma_descriptor_replace_dims_strides_in_shared_mem(shared_tensormaps.smem_tensormap_B,
                                                            prob_shape_B,
                                                            prob_stride_B);
  }
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1167-1178

```cpp
  template <class TensorMapA, class TensorMapB, class ProblemShape_MNKL>
  CUTLASS_DEVICE
  void
  tensormaps_perform_update(
      TensorMapStorage& shared_tensormaps,
      Params const& mainloop_params,
      cute::tuple<TensorMapA, TensorMapB> const& input_tensormaps,
      ProblemShape_MNKL problem_shape_mnkl,
      int32_t next_batch) {
    if (cute::elect_one_sync()) {
      // Replacing global_address for the next batch
      tensormaps_replace_global_address(shared_tensormaps, mainloop_params, next_batch);
```
**EN:** This block introduces `TensorMapA` and groups related declarations around that symbol. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段引入了 `TensorMapA`，并围绕该符号组织相关声明。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1180-1186

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

### Lines 1188-1201

```cpp
  template <class TensorMapA, class TensorMapB>
  CUTLASS_DEVICE
  void
  tensormaps_cp_fence_release (
      TensorMapStorage& shared_tensormaps,
      cute::tuple<TensorMapA, TensorMapB> const& input_tensormaps) {
    if (cute::elect_one_sync()) {
      cute::tma_desc_commit_group();
      cute::tma_desc_wait_group();
    }
    // Entire warp must do this (i.e. it's aligned)
    tma_descriptor_cp_fence_release(get<0>(input_tensormaps), shared_tensormaps.smem_tensormap_A);
    tma_descriptor_cp_fence_release(get<1>(input_tensormaps), shared_tensormaps.smem_tensormap_B);
  }
```
**EN:** This block introduces `TensorMapA` and groups related declarations around that symbol. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段引入了 `TensorMapA`，并围绕该符号组织相关声明。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1203-1210

```cpp
  // The entire warp must call this function collectively (that is, the instructions are aligned)
  template <class TensorMapA, class TensorMapB>
  CUTLASS_DEVICE
  void
  tensormaps_fence_acquire(cute::tuple<TensorMapA, TensorMapB> const& input_tensormaps) {
    cute::tma_descriptor_fence_acquire(get<0>(input_tensormaps));
    cute::tma_descriptor_fence_acquire(get<1>(input_tensormaps));
  }
```
**EN:** This block introduces `TensorMapA` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TensorMapA`，并围绕该符号组织相关声明。

### Lines 1212-1219

```cpp
  template <class InputTensors, class ProblemShape_MNKL>
  CUTLASS_DEVICE
  InputTensors
  tensors_perform_update(
      [[maybe_unused]] InputTensors const& input_tensors,
      Params const& mainloop_params,
      [[maybe_unused]] ProblemShape_MNKL problem_shape_mnkl,
      int32_t next_batch) {
```
**EN:** This block introduces `InputTensors` and groups related declarations around that symbol.
**CN:** 这一段引入了 `InputTensors`，并围绕该符号组织相关声明。

### Lines 1221-1229

```cpp
    if constexpr (IsGroupedGemmKernel) {
      return load_init(
        problem_shape_mnkl,
        mainloop_params,
        mainloop_params.ptr_SFA[next_batch],
        mainloop_params.ptr_SFB[next_batch]
      );
    } else {
      auto [gA_mkl, gB_nkl, mScaleA_mkl, mScaleB_nkl] = input_tensors;
```
**EN:** This block prepares the load path by slicing global tensors, partitioning work for the current CTA/subgroup, and binding shared-memory destinations. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段为加载路径做准备：切分全局张量、为当前 CTA/子组划分工作，并绑定共享内存目标。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1231-1232

```cpp
      auto scaleA_layout = mScaleA_mkl.layout();
      auto scaleB_layout = mScaleB_nkl.layout();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1234-1239

```cpp
      mScaleA_mkl = make_tensor(make_gmem_ptr(mainloop_params.ptr_SFA[next_batch]), scaleA_layout); // (m,ScaleMsPerTile,k,l)
      mScaleB_nkl = make_tensor(make_gmem_ptr(mainloop_params.ptr_SFB[next_batch]), scaleB_layout); // (n,ScaleNsPerTile,k,l)
      return cute::make_tuple(gA_mkl, gB_nkl, mScaleA_mkl, mScaleB_nkl);
    }
  }
};
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 1241-1241

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 1243-1243

```cpp
} // namespace cutlass::gemm::collective
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1245-1245

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
- **EN:** blockwise scaling  
  **CN:** 块级缩放
- **EN:** FP8 data paths  
  **CN:** FP8 数据路径
- **EN:** cp.async copy stages  
  **CN:** cp.async 拷贝阶段
- **EN:** array-style MMA tiling  
  **CN:** 数组式 MMA 分块
- **EN:** Intel Xe subgroup execution  
  **CN:** Intel Xe 子组执行
- **EN:** software pipelines  
  **CN:** 软件流水线
- **EN:** compile-time validation  
  **CN:** 编译期校验

## Dependencies / 依赖关系

- **Direct includes / 直接包含:**
  - `cutlass/cutlass.h`
  - `cutlass/gemm/dispatch_policy.hpp`
  - `cutlass/numeric_types.h`
  - `cutlass/pipeline/pipeline.hpp`
  - `cutlass/trace.h`
  - `cutlass/cuda_host_adapter.hpp`
  - `cute/arch/cluster_sm90.hpp`
  - `cute/arch/copy_sm80.hpp`
  - `cute/arch/copy_sm90.hpp`
  - `cute/algorithm/functional.hpp`
  - `cute/atom/mma_atom.hpp`
  - `cute/algorithm/gemm.hpp`
  - `cute/numeric/arithmetic_tuple.hpp`
  - `cutlass/detail/blockwise_scale_layout.hpp`
- **Primary symbols / 主要符号:**
  - `ClusterShape`
  - `KernelSchedule`
  - `TileShape_`
  - `ElementA_`
  - `StridePairA_`
  - `ElementB_`
  - `StridePairB_`
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