# builder.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/63_hopper_gemm_with_weight_prefetch/collective/builder.hpp`  
**Purpose / 用途**: Specializes CUTLASS’s generic SM90 collective builder so that specific custom kernel schedule tags instantiate a prefetch-capable Hopper FP8 `CollectiveMma`, including a stage-count formula that reserves shared memory for prefetch state. / 对 CUTLASS 通用的 SM90 collective builder 做特化，使特定的自定义 kernel schedule tag 能实例化出支持预取的 Hopper FP8 `CollectiveMma`，并通过一个阶段数公式为预取状态预留共享内存。

---

## Line-by-Line Analysis / 逐行分析

### Custom builder entrypoint and required helper headers (Lines 32-39)

```cpp
#pragma once

#include "cutlass/gemm/collective/collective_builder.hpp"

#include "dispatch_policy_extra.hpp"
#include "sm90_mma_tma_gmma_ss_warpspecialized_with_prefetch.hpp"
#include "../pipeline/prefetch_pipeline_sm90.hpp"
```

**EN**: This header starts by pulling in the standard collective builder plus three local helpers: extra dispatch-policy tags, the custom SM90 TMA/GMMA mainloop implementation, and the prefetch pipeline shared-storage definition. Together they provide the building blocks needed to intercept CUTLASS’s normal builder flow and substitute a prefetch-aware collective.

**CN**: 该头文件先包含标准 collective builder，再引入三个本地辅助头：额外的 dispatch-policy tag、自定义的 SM90 TMA/GMMA 主循环实现，以及预取 pipeline 的共享存储定义。三者合在一起，构成了拦截 CUTLASS 常规 builder 流程并替换成“感知预取”的 collective 所需的基础积木。

### Stage-count helper that reserves space for prefetch storage (Lines 44-64)

```cpp
// Returns the maximum number of smem tiles that can be used with a given smem capacity, or overrides with manual count. 
template<int CapacityBytes, class ElementA, class ElementB, class TileShapeMNK, int stages>
constexpr int
compute_stage_count_or_override_prefetch(StageCount<stages> stage_count) {
  return stages;
}

// Returns the maximum number of smem tiles that can be used with a given smem capacity, or overrides with manual count. 
template<int CapacityBytes, class ElementA, class ElementB, class TileShapeMNK, int carveout_bytes>
constexpr int
compute_stage_count_or_override_prefetch(StageCountAutoCarveout<carveout_bytes> stage_count) {
  constexpr auto mainloop_pipeline_bytes = sizeof(typename cutlass::PipelineTmaAsync<1>::SharedStorage);
  constexpr auto prefetch_pipeline_bytes = sizeof(typename cutlass::detail::PrefetcherPipelineSharedStorage<PrefetchStages>);
  constexpr auto a_bits = cute::sizeof_bits_v<ElementA>;
  constexpr auto b_bits = cute::sizeof_bits_v<ElementB>;
  constexpr int MK_bytes = cutlass::bits_to_bytes(a_bits * size<0>(TileShapeMNK{}) * size<2>(TileShapeMNK{})); //also the prefetch smem size
  constexpr int NK_bytes = cutlass::bits_to_bytes(b_bits * size<1>(TileShapeMNK{}) * size<2>(TileShapeMNK{}));
  constexpr int stage_bytes = MK_bytes + NK_bytes + static_cast<int>(mainloop_pipeline_bytes);

  return (CapacityBytes - carveout_bytes - MK_bytes * PrefetchStagesActual - prefetch_pipeline_bytes) / stage_bytes;
}
```

**EN**: The overloaded `compute_stage_count_or_override_prefetch()` function is the core builder policy tweak. If the user passes an explicit `StageCount<N>`, the override is honored directly. If the user passes `StageCountAutoCarveout<carveout_bytes>`, the helper computes how many pipeline stages fit after subtracting epilogue carveout, the extra shared-memory tiles reserved for prefetched A data, and the shared storage consumed by the prefetch pipeline itself. This is the precise place where the builder pattern adapts automatic stage selection to the weight-prefetch design.

**CN**: 重载的 `compute_stage_count_or_override_prefetch()` 是这个 builder 的核心策略改动。如果用户显式给出 `StageCount<N>`，就直接采用；如果给的是 `StageCountAutoCarveout<carveout_bytes>`，辅助函数就会在扣除 epilogue carveout、为预取 A 数据保留的额外共享内存 tile，以及预取 pipeline 自身占用的共享存储之后，再计算还能容纳多少主循环阶段。这正是 builder 模式把“自动 stage 数选择”适配到权重预取设计的关键位置。

### CollectiveBuilder specialization for prefetch-only schedule tags (Lines 68-152)

```cpp
// GMMA_TMA_WS_FP8_FAST_ACCUM_SS + prefetch
template <
  class ElementA,
  class GmemLayoutATag,
  int AlignmentA,
  class ElementB,
  class GmemLayoutBTag,
  int AlignmentB,
  class ElementAccumulator,
  class TileShape_MNK,
  class ClusterShape_MNK,
  class StageCountType,
  class KernelScheduleType
>
struct CollectiveBuilder<
    arch::Sm90,
    arch::OpClassTensorOp,
    ElementA,
    GmemLayoutATag,
    AlignmentA,
    ElementB,
    GmemLayoutBTag,
    AlignmentB,
    ElementAccumulator,
    TileShape_MNK,
    ClusterShape_MNK,
    StageCountType,
    KernelScheduleType,
    cute::enable_if_t<
      cute::is_same_v<KernelScheduleType, KernelTmaWarpSpecializedFP8FastAccumWithPrefetch>>
> {
  static_assert(is_static<TileShape_MNK>::value);
  static_assert(is_static<ClusterShape_MNK>::value);
  static_assert(detail::is_aligned<ElementA, AlignmentA, ElementB, AlignmentB, detail::tma_alignment_bytes>(),
                "Not meet TMA alignment requirement yet\n");
  static_assert(detail::is_input_fp8<ElementA, ElementB>(),
                "Only FP8 datatypes are compatible with these kernel schedules\n");
  // Dispatch TN fp8 kernels only to TMA warp specialized FP8 builder
  static_assert(!detail::is_use_rmem_A<ElementA, GmemLayoutATag, ElementB, GmemLayoutBTag>(),
                 "Not supported for fp8 non-TN warp specialized kernels yet\n");
#ifndef CUTLASS_SM90_COLLECTIVE_BUILDER_SUPPORTED
  static_assert(cutlass::detail::dependent_false<ElementA>, "Unsupported Toolkit for SM90 Collective Builder\n");
#endif

  static constexpr cute::GMMA::Major GmmaMajorA = detail::gmma_ss_tag_to_major_A<ElementA, GmemLayoutATag>();
  static constexpr cute::GMMA::Major GmmaMajorB = detail::gmma_ss_tag_to_major_B<ElementB, GmemLayoutBTag>();

  using AtomLayoutMNK = Layout<Shape<_1,_1,_1>>;

  using TiledMma = decltype(cute::make_tiled_mma(cute::GMMA::ss_op_selector<
      ElementA, ElementB, ElementAccumulator, TileShape_MNK, GmmaMajorA, GmmaMajorB>(), AtomLayoutMNK{}));

  using GmemTiledCopyA = decltype(detail::sm90_cluster_shape_to_tma_atom(shape<1>(ClusterShape_MNK{})));
  using GmemTiledCopyB = decltype(detail::sm90_cluster_shape_to_tma_atom(shape<0>(ClusterShape_MNK{})));

  using SmemLayoutAtomA = decltype(detail::ss_smem_selector<
      GmmaMajorA, ElementA, decltype(cute::get<0>(TileShape_MNK{})), decltype(cute::get<2>(TileShape_MNK{}))>());
  using SmemLayoutAtomB = decltype(detail::ss_smem_selector<
      GmmaMajorB, ElementB, decltype(cute::get<1>(TileShape_MNK{})), decltype(cute::get<2>(TileShape_MNK{}))>());

  static constexpr int PipelineStages = detail::compute_stage_count_or_override_prefetch<detail::sm90_smem_capacity_bytes,
      ElementA, ElementB, TileShape_MNK>(StageCountType{});
  using DispatchPolicy = MainloopSm90TmaGmmaWarpSpecializedWithPrefetch<PipelineStages, ClusterShape_MNK, KernelScheduleType>;

  using SmemCopyAtomA = void;
  using SmemCopyAtomB = void;

  using CollectiveOp = CollectiveMma<
      DispatchPolicy,
      TileShape_MNK,
      ElementA,
      TagToStrideA_t<GmemLayoutATag>,
      ElementB,
      TagToStrideB_t<GmemLayoutBTag>,
      TiledMma,
      GmemTiledCopyA,
      SmemLayoutAtomA,
      SmemCopyAtomA,
      cute::identity,
      GmemTiledCopyB,
      SmemLayoutAtomB,
      SmemCopyAtomB,
      cute::identity
    >;
};
```

**EN**: This partial specialization activates only when the architecture is SM90 tensor-op and the kernel schedule tag is `KernelTmaWarpSpecializedFP8FastAccumWithPrefetch`. The body performs a series of compile-time checks (static tile/cluster shape, TMA alignment, FP8-only inputs, TN-only path), derives GMMA major modes, selects TMA copy atoms and shared-memory layout atoms, computes `PipelineStages` with the custom helper, and finally assembles a `CollectiveMma` whose dispatch policy is `MainloopSm90TmaGmmaWarpSpecializedWithPrefetch`. In other words, the builder pattern is used here as a type-level factory for a custom mainloop.

**CN**: 这个偏特化只会在“架构为 SM90 TensorOp 且 kernel schedule tag 为 `KernelTmaWarpSpecializedFP8FastAccumWithPrefetch`”时生效。其主体会执行一系列编译期检查（静态 tile/cluster 形状、TMA 对齐、仅支持 FP8 输入、仅支持 TN 路径），推导 GMMA 主序模式，选择 TMA copy atom 与共享内存布局 atom，用前面的自定义辅助函数计算 `PipelineStages`，最后组装出一个 dispatch policy 为 `MainloopSm90TmaGmmaWarpSpecializedWithPrefetch` 的 `CollectiveMma`。换句话说，这里把 builder 模式用成了一个类型级工厂，用于生成自定义主循环。

### Parallel specialization for split-DMA prefetch schedule tags (Lines 154-238)

```cpp
// GMMA_TMA_WS_FP8_FAST_ACCUM_SS + prefetch and split DMA warps
template <
  class ElementA,
  class GmemLayoutATag,
  int AlignmentA,
  class ElementB,
  class GmemLayoutBTag,
  int AlignmentB,
  class ElementAccumulator,
  class TileShape_MNK,
  class ClusterShape_MNK,
  class StageCountType,
  class KernelScheduleType
>
struct CollectiveBuilder<
    arch::Sm90,
    arch::OpClassTensorOp,
    ElementA,
    GmemLayoutATag,
    AlignmentA,
    ElementB,
    GmemLayoutBTag,
    AlignmentB,
    ElementAccumulator,
    TileShape_MNK,
    ClusterShape_MNK,
    StageCountType,
    KernelScheduleType,
    cute::enable_if_t<
      cute::is_same_v<KernelScheduleType, KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA>>
> {
  static_assert(is_static<TileShape_MNK>::value);
  static_assert(is_static<ClusterShape_MNK>::value);
  static_assert(detail::is_aligned<ElementA, AlignmentA, ElementB, AlignmentB, detail::tma_alignment_bytes>(),
                "Not meet TMA alignment requirement yet\n");
  static_assert(detail::is_input_fp8<ElementA, ElementB>(),
                "Only FP8 datatypes are compatible with these kernel schedules\n");
  // Dispatch TN fp8 kernels only to TMA warp specialized FP8 builder
  static_assert(!detail::is_use_rmem_A<ElementA, GmemLayoutATag, ElementB, GmemLayoutBTag>(),
                 "Not supported for fp8 non-TN warp specialized kernels yet\n");
#ifndef CUTLASS_SM90_COLLECTIVE_BUILDER_SUPPORTED
  static_assert(cutlass::detail::dependent_false<ElementA>, "Unsupported Toolkit for SM90 Collective Builder\n");
#endif

  static constexpr cute::GMMA::Major GmmaMajorA = detail::gmma_ss_tag_to_major_A<ElementA, GmemLayoutATag>();
  static constexpr cute::GMMA::Major GmmaMajorB = detail::gmma_ss_tag_to_major_B<ElementB, GmemLayoutBTag>();

  using AtomLayoutMNK = Layout<Shape<_1,_1,_1>>;

  using TiledMma = decltype(cute::make_tiled_mma(cute::GMMA::ss_op_selector<
      ElementA, ElementB, ElementAccumulator, TileShape_MNK, GmmaMajorA, GmmaMajorB>(), AtomLayoutMNK{}));

  using GmemTiledCopyA = decltype(detail::sm90_cluster_shape_to_tma_atom(shape<1>(ClusterShape_MNK{})));
  using GmemTiledCopyB = decltype(detail::sm90_cluster_shape_to_tma_atom(shape<0>(ClusterShape_MNK{})));

  using SmemLayoutAtomA = decltype(detail::ss_smem_selector<
      GmmaMajorA, ElementA, decltype(cute::get<0>(TileShape_MNK{})), decltype(cute::get<2>(TileShape_MNK{}))>());
  using SmemLayoutAtomB = decltype(detail::ss_smem_selector<
      GmmaMajorB, ElementB, decltype(cute::get<1>(TileShape_MNK{})), decltype(cute::get<2>(TileShape_MNK{}))>());

  static constexpr int PipelineStages = detail::compute_stage_count_or_override_prefetch<detail::sm90_smem_capacity_bytes,
      ElementA, ElementB, TileShape_MNK>(StageCountType{});
  using DispatchPolicy = MainloopSm90TmaGmmaWarpSpecializedWithPrefetch<PipelineStages, ClusterShape_MNK, KernelScheduleType>;

  using SmemCopyAtomA = void;
  using SmemCopyAtomB = void;

  using CollectiveOp = CollectiveMma<
      DispatchPolicy,
      TileShape_MNK,
      ElementA,
      TagToStrideA_t<GmemLayoutATag>,
      ElementB,
      TagToStrideB_t<GmemLayoutBTag>,
      TiledMma,
      GmemTiledCopyA,
      SmemLayoutAtomA,
      SmemCopyAtomA,
      cute::identity,
      GmemTiledCopyB,
      SmemLayoutAtomB,
      SmemCopyAtomB,
      cute::identity
    >;
};
```

**EN**: The second specialization repeats the same construction pipeline but activates for `KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA`. That duplication is deliberate: the two schedule tags represent different execution policies, yet both want the same broad builder recipe—alignment checks, GMMA selection, TMA atoms, shared-memory layouts, custom stage-count calculation, and a final `CollectiveMma` using the prefetch dispatch policy template. This is a classic builder-pattern extension technique: add new product variants by specializing the selection layer while reusing the composition steps.

**CN**: 第二个偏特化沿用了相同的构造流程，但触发条件变成 `KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA`。这种“看似重复”的实现其实是刻意的：这两个 schedule tag 代表不同的执行策略，但它们都希望复用同一套 builder 配方——对齐检查、GMMA 选择、TMA atom、共享内存布局、自定义 stage 计算，以及最终使用预取 dispatch policy 模板的 `CollectiveMma`。这正是 builder 模式常见的扩展方法：通过特化选择层，增加新的产品变体，同时复用已有的组装步骤。

---

## Key Concepts / 关键概念

- **EN**: This file is a builder-pattern customization layer: it does not implement GEMM math directly, but decides which type composition should be instantiated for certain schedule tags.
  **CN**: 这个文件本质上是 builder 模式的定制层：它并不直接实现 GEMM 数学，而是决定在特定 schedule tag 下应实例化哪种类型组合。
- **EN**: Automatic stage-count selection must be corrected for prefetch because prefetch consumes extra shared memory beyond the normal mainloop pipeline.
  **CN**: 由于预取会占用超出常规主循环 pipeline 的额外共享内存，所以自动 stage 数选择必须针对预取场景做修正。
- **EN**: Compile-time guards ensure the specialization is only used for supported FP8/TN/TMA-aligned Hopper cases.
  **CN**: 编译期保护保证该特化只会用于受支持的 FP8/TN/TMA 对齐的 Hopper 场景。
- **EN**: The two specializations differ mainly in accepted schedule tags; the rest of the collective assembly pipeline is intentionally shared in shape and semantics.
  **CN**: 这两个特化主要区别在于接受的 schedule tag 不同；collective 组装流程的其余部分则在结构和语义上刻意保持一致。

## Dependencies / 依赖项

- **EN**: `cutlass/gemm/collective/collective_builder.hpp` supplies the primary template being specialized.
  **CN**: `cutlass/gemm/collective/collective_builder.hpp` 提供了这里被特化的主模板。
- **EN**: `dispatch_policy_extra.hpp` defines the custom schedule tag types that trigger these specializations.
  **CN**: `dispatch_policy_extra.hpp` 定义了触发这些偏特化的自定义 schedule tag 类型。
- **EN**: `sm90_mma_tma_gmma_ss_warpspecialized_with_prefetch.hpp` provides the concrete prefetch-capable `CollectiveMma` mainloop machinery.
  **CN**: `sm90_mma_tma_gmma_ss_warpspecialized_with_prefetch.hpp` 提供了真正支持预取的 `CollectiveMma` 主循环实现。
- **EN**: `../pipeline/prefetch_pipeline_sm90.hpp` contributes the shared-storage accounting used by the stage-count helper.
  **CN**: `../pipeline/prefetch_pipeline_sm90.hpp` 提供了 stage 数计算中需要计入的预取 pipeline 共享存储信息。
