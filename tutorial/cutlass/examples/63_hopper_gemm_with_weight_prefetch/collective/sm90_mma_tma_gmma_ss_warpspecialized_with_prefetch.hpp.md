# sm90_mma_tma_gmma_ss_warpspecialized_with_prefetch.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/63_hopper_gemm_with_weight_prefetch/collective/sm90_mma_tma_gmma_ss_warpspecialized_with_prefetch.hpp`  
**Purpose / 用途**: Defines a Hopper SM90 CUTLASS collective mainloop that combines TMA global-to-shared transfers, GMMA warpgroup math, warp-specialized producer/consumer roles, and an auxiliary weight-prefetch pipeline. / 定义 Hopper SM90 的 CUTLASS collective mainloop，把 TMA 全局到共享内存搬运、GMMA warpgroup 计算、warp-specialized 生产者/消费者分工，以及额外的权重预取流水线组合在一起。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-71

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#include "cutlass/cutlass.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/numeric_types.h"
#include "cutlass/pipeline/pipeline.hpp"
#include "cutlass/trace.h"

#include "cute/arch/cluster_sm90.hpp"
#include "cute/arch/copy_sm90.hpp"
#include "cute/algorithm/functional.hpp"
#include "cute/atom/mma_atom.hpp"
#include "cute/algorithm/gemm.hpp"
#include "cute/numeric/arithmetic_tuple.hpp"
#include "cutlass/arch/grid_dependency_control.h"

#include "dispatch_policy_extra.hpp"

#include "../pipeline/prefetch_pipeline_sm90.hpp"

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace cutlass::gemm::collective {
using namespace cute;

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace detail {

constexpr int PrefetchStages = 4;
constexpr int PrefetchInitialStages = 1;
// This determines how much shmem we set aside for prefetch.
// We don't reuse anything loaded by prefetcher, so we can keep
// loading into the same place -- there will be a conflict when
// writing, but it doesn't affect performance as much as the doors
// that this opens.
constexpr int PrefetchStagesActual = 1;

} // namespace detail

```

**EN**: The file starts with the legal header, pulls in CUTLASS/CuTe building blocks, and introduces a small detail namespace for prefetch constants. `PrefetchStages`, `PrefetchInitialStages`, and `PrefetchStagesActual` describe the side pipeline depth and show that the prefetch path intentionally reserves only one actual shared-memory stage while keeping a larger logical queue for overlap control.
**CN**: 文件先给出版权声明并引入 CUTLASS/CuTe 组件，然后在 `detail` 命名空间中定义预取常量。`PrefetchStages`、`PrefetchInitialStages` 与 `PrefetchStagesActual` 描述了旁路预取流水线的深度，并表明该预取路径只保留一个真实的共享内存阶段，而用更大的逻辑队列来管理重叠执行。

### Lines 72-124

```cpp
// WarpSpecialized Mainloop
template <
  int Stages,
  class ClusterShape,
  class KernelSchedule,
  class TileShape_,
  class ElementA_,
  class StrideA_,
  class ElementB_,
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
    MainloopSm90TmaGmmaWarpSpecializedWithPrefetch<Stages, ClusterShape, KernelSchedule>,
    TileShape_,
    ElementA_,
    StrideA_,
    ElementB_,
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
  //
  // Type Aliases
  //
  using DispatchPolicy = MainloopSm90TmaGmmaWarpSpecializedWithPrefetch<Stages, ClusterShape, KernelSchedule>;
  using TileShape = TileShape_;
  using ElementA = ElementA_;
  using StrideA = StrideA_;
  using ElementB = ElementB_;
  using StrideB = StrideB_;
  using TiledMma = TiledMma_;
  using ElementAccumulator = typename TiledMma::ValTypeC;
  using GmemTiledCopyA = GmemTiledCopyA_;
  using GmemTiledCopyB = GmemTiledCopyB_;
  using SmemLayoutAtomA = SmemLayoutAtomA_;
  using SmemLayoutAtomB = SmemLayoutAtomB_;
  using SmemCopyAtomA = SmemCopyAtomA_;
  using SmemCopyAtomB = SmemCopyAtomB_;
```

**EN**: This is a partial specialization of `CollectiveMma` for `MainloopSm90TmaGmmaWarpSpecializedWithPrefetch`. The template parameters expose all compile-time choices—pipeline stage count, cluster shape, kernel schedule, tile shape, operand element/stride types, GMMA tile, TMA copy atoms, shared-memory layout atoms, and transforms—so the generated kernel is fully selected at compile time.
**CN**: 这里是面向 `MainloopSm90TmaGmmaWarpSpecializedWithPrefetch` 的 `CollectiveMma` 偏特化。模板参数把所有编译期选择都显式展开：流水线阶段数、cluster 形状、kernel schedule、tile 形状、操作数元素与步长类型、GMMA tile、TMA copy atom、共享内存布局 atom 以及变换器，因此最终生成的 kernel 在编译期就被完全确定。

### Lines 125-198

```cpp
  using TransformA = TransformA_;
  using TransformB = TransformB_;
  using ArchTag = typename DispatchPolicy::ArchTag;

  static_assert(size<1>(ClusterShape{}) == 1, "Cluster shape N must be 1");
  using CtaShape_MNK = decltype(shape_div(TileShape{}, ClusterShape{}));

  using PrefetcherPipeline = cutlass::PrefetchPipeline<detail::PrefetchStages>;

  using MainloopPipeline = cutlass::PipelineTmaAsync<DispatchPolicy::Stages>;
  using PipelineState = cutlass::PipelineState<DispatchPolicy::Stages>;
  using PipelineParams = typename MainloopPipeline::Params;

  static_assert(cute::rank(SmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");

  static_assert(cute::rank(SmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<1>(TileShape{}) % size<0>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");

  // Tile along modes in a way that maximizes the TMA box size.
  using SmemLayoutA = decltype(tile_to_shape(
      SmemLayoutAtomA{},
      make_shape(shape<0>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::Stages>{}),
      cute::conditional_t< ::cutlass::gemm::detail::is_major<0,StrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  using SmemLayoutB = decltype(tile_to_shape(
      SmemLayoutAtomB{},
      make_shape(shape<1>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::Stages>{}),
      cute::conditional_t< ::cutlass::gemm::detail::is_major<0,StrideB>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));

  static_assert(rank(SmemLayoutA{}) == 3 && size<2>(SmemLayoutA{}) == DispatchPolicy::Stages);
  static_assert(rank(SmemLayoutB{}) == 3 && size<2>(SmemLayoutB{}) == DispatchPolicy::Stages);

  using PrefetchSmemLayoutA = decltype(make_layout(make_shape(
    cute::Int<size<0>(SmemLayoutA{})>{},
    cute::Int<size<1>(SmemLayoutA{})>{},
    cute::Int<detail::PrefetchStagesActual>{})));

  static constexpr auto prefetch_smem_size = cute::cosize_v<PrefetchSmemLayoutA>;

  static_assert(DispatchPolicy::Stages >= 2, "Specialization requires Stages set to value 2 or more.");
  static_assert(cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value &&
                cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeB>::value,
                "MMA atom must source both A and B operand from smem_desc for this mainloop.");
  static_assert(cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>,
      "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
  static_assert(cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD_MULTICAST>,
      "GmemTiledCopy - invalid SM90 TMA copy atom specified.");

  // TMA converts f32 input to tf32 when copying from GMEM to SMEM
  // For all other types, cast to size equivalent uint type to avoid any rounding by TMA.
  static constexpr bool ConvertF32toTF32A = cute::is_same_v<float, ElementA>;
  static constexpr bool ConvertF32toTF32B = cute::is_same_v<float, ElementB>;
  using InternalElementA = cute::conditional_t<ConvertF32toTF32A, tfloat32_t, uint_bit_t<sizeof_bits_v<ElementA>>>;
  using InternalElementB = cute::conditional_t<ConvertF32toTF32B, tfloat32_t, uint_bit_t<sizeof_bits_v<ElementB>>>;

  // Defined outside the class where it's used, to work around MSVC issues
  using PrefetcherPipelineStorage = ::cutlass::detail::PrefetcherPipelineSharedStorage<detail::PrefetchStages>;

  struct SharedStorage {
    struct TensorStorage : cute::aligned_struct<128, _0> {
      cute::array_aligned<typename TiledMma::ValTypeA, cute::cosize_v<SmemLayoutA>> smem_A;
      cute::array_aligned<typename TiledMma::ValTypeB, cute::cosize_v<SmemLayoutB>> smem_B;
      cute::array_aligned<typename TiledMma::ValTypeA, prefetch_smem_size> smem_prefetch;
    } tensors;

    using PipelineStorage = typename MainloopPipeline::SharedStorage;
    PipelineStorage pipeline;
    PrefetcherPipelineStorage prefetcher_pipeline;
  };
  using TensorStorage = typename SharedStorage::TensorStorage;
  using PipelineStorage = typename SharedStorage::PipelineStorage;

```

**EN**: The next section derives internal layouts and pipeline types. Static assertions enforce cluster and tile invariants, `tile_to_shape` builds stage-major SMEM layouts for A/B, FP32 inputs can be converted to TF32 through `InternalElementA/B`, and `SharedStorage` packs the A/B stage buffers, a dedicated prefetch buffer, and both the mainloop and prefetch pipeline state objects.
**CN**: 这一段推导内部布局与流水线类型。静态断言保证 cluster 与 tile 的约束成立，`tile_to_shape` 为 A/B 构造按 stage 分层的共享内存布局，FP32 输入可通过 `InternalElementA/B` 在搬运时转成 TF32，而 `SharedStorage` 则把 A/B 的 stage 缓冲区、独立的预取缓冲区，以及主流水线与预取流水线状态对象一起打包。

### Lines 199-234

```cpp
  // Host side kernel arguments
  struct Arguments {
    ElementA const* ptr_A;
    StrideA dA;
    ElementB const* ptr_B;
    StrideB dB;
    uint32_t mma_promotion_interval = 4;
    float overlap_ratio = 0.5;
    float prefetch_ratio = -1.0;
  };

  // Device side kernel params
  struct Params {
    // Assumption: StrideA is congruent with Problem_MK
    using TMA_A = decltype(make_tma_copy_A_sm90(
        GmemTiledCopyA{},
        make_tensor(static_cast<InternalElementA const*>(nullptr), repeat_like(StrideA{}, int32_t(0)), StrideA{}),
        SmemLayoutA{}(_,_,cute::Int<0>{}),
        TileShape{},
        ClusterShape{}));
    // Assumption: StrideB is congruent with Problem_NK
    using TMA_B = decltype(make_tma_copy_B_sm90(
        GmemTiledCopyB{},
        make_tensor(static_cast<InternalElementB const*>(nullptr), repeat_like(StrideB{}, int32_t(0)), StrideB{}),
        SmemLayoutB{}(_,_,cute::Int<0>{}),
        TileShape{},
        ClusterShape{}));

    TMA_A tma_load_a;
    TMA_B tma_load_b;
    uint32_t tma_transaction_bytes = TmaTransactionBytesMK + TmaTransactionBytesNK;
    uint32_t tma_transaction_bytes_mk = TmaTransactionBytesMK;
    uint32_t tma_transaction_bytes_nk = TmaTransactionBytesNK;
    float overlap_ratio = 0.5;
    float prefetch_ratio = -1.0;
  };
```

**EN**: Host-side `Arguments` keep raw pointers, logical strides, and two runtime overlap knobs. Device-side `Params` materialize concrete TMA descriptor types (`TMA_A`, `TMA_B`) from the compile-time traits and cache the total and per-operand transaction byte counts, which are later fed into barrier accounting and launch-control policy.
**CN**: 主机侧 `Arguments` 保存原始指针、逻辑步长以及两个运行时重叠参数。设备侧 `Params` 根据编译期 traits 实例化出具体的 TMA 描述符类型（`TMA_A`、`TMA_B`），并缓存总传输字节数与各操作数字节数，这些信息随后会被用于 barrier 记账与 launch-control 策略。

### Lines 235-320

```cpp

  //
  // Methods
  //

  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(ProblemShape const& problem_shape, Arguments const& args, void* workspace) {
    (void) workspace;

    // Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M,N,K,L] = problem_shape_MNKL;

    auto ptr_A = reinterpret_cast<InternalElementA const*>(args.ptr_A);
    auto ptr_B = reinterpret_cast<InternalElementB const*>(args.ptr_B);

    Tensor tensor_a = make_tensor(ptr_A, make_layout(make_shape(M,K,L), args.dA));
    Tensor tensor_b = make_tensor(ptr_B, make_layout(make_shape(N,K,L), args.dB));

    typename Params::TMA_A tma_load_a = make_tma_copy_A_sm90(
        GmemTiledCopyA{},
        tensor_a,
        SmemLayoutA{}(_,_,cute::Int<0>{}),
        TileShape{},
        ClusterShape{});
    typename Params::TMA_B tma_load_b = make_tma_copy_B_sm90(
        GmemTiledCopyB{},
        tensor_b,
        SmemLayoutB{}(_,_,cute::Int<0>{}),
        TileShape{},
        ClusterShape{});
    uint32_t transaction_bytes_mk = TmaTransactionBytesMK;
    uint32_t transaction_bytes_nk = TmaTransactionBytesNK;
    uint32_t transaction_bytes = transaction_bytes_mk + transaction_bytes_nk;

    return {
      tma_load_a,
      tma_load_b,
      transaction_bytes,
      transaction_bytes_mk,
      transaction_bytes_nk,
      args.overlap_ratio,
      args.prefetch_ratio
    };
  }

  template<class ProblemShape>
  static bool
  can_implement(
      ProblemShape const& problem_shape,
      [[maybe_unused]] Arguments const& args) {
    constexpr int tma_alignment_bits = 128;
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M,N,K,L] = problem_shape_MNKL;

    constexpr int min_tma_aligned_elements_A = tma_alignment_bits / cutlass::sizeof_bits<ElementA>::value;
    bool implementable = cutlass::detail::check_alignment<min_tma_aligned_elements_A>(cute::make_shape(M,K,L), StrideA{});
    constexpr int min_tma_aligned_elements_B = tma_alignment_bits / cutlass::sizeof_bits<ElementB>::value;
    implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_B>(cute::make_shape(N,K,L), StrideB{});

    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA.\n");
      return false;
    }

    if (args.overlap_ratio > 1.0) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: `overlap_ratio` must be either negative (disabled) or in [0, 1].\n");
      return false;
    }

    if (args.prefetch_ratio > 1.0) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: `prefetch_ratio` must be either negative (disabled) or in [0, 1].\n");
      return false;
    }

    return true;
  }

  static constexpr int K_PIPE_MAX = DispatchPolicy::Stages;
  static constexpr int K_PIPE_MMAS = 1;
  static constexpr uint32_t TmaTransactionBytesMK =
        cutlass::bits_to_bytes(size<0>(SmemLayoutA{}) * size<1>(SmemLayoutA{}) * static_cast<uint32_t>(sizeof_bits<ElementA>::value));
  static constexpr uint32_t TmaTransactionBytesNK =
        cutlass::bits_to_bytes(size<0>(SmemLayoutB{}) * size<1>(SmemLayoutB{}) * static_cast<uint32_t>(sizeof_bits<ElementB>::value));

```

**EN**: `to_underlying_arguments` lowers a logical problem shape to rank-4 MNKL form, creates tensor views, and constructs TMA copies for A and B. `can_implement` then checks 128-bit TMA alignment and validates `overlap_ratio`/`prefetch_ratio`, while the `K_PIPE_MAX`, `K_PIPE_MMAS`, and transaction-byte constants define how many GMMA groups may be in flight and how the async pipeline is dimensioned.
**CN**: `to_underlying_arguments` 把逻辑问题形状降成 4 维 MNKL，创建张量视图，并为 A/B 构造 TMA copy 描述符。随后 `can_implement` 检查 128-bit TMA 对齐并验证 `overlap_ratio`/`prefetch_ratio` 的取值范围，而 `K_PIPE_MAX`、`K_PIPE_MMAS` 与传输字节常量则定义了 GMMA 可并行飞行的数量以及异步流水线的规模。

### Lines 321-352

```cpp
  /// Issue Tma Descriptor Prefetch -- ideally from a single thread for best performance
  CUTLASS_DEVICE
  static void prefetch_tma_descriptors(Params const& mainloop_params) {
    cute::prefetch_tma_descriptor(mainloop_params.tma_load_a.get_tma_descriptor());
    cute::prefetch_tma_descriptor(mainloop_params.tma_load_b.get_tma_descriptor());
  }

  /// Set up the data needed by this collective for load and mma.
  /// Returns a tuple of tensors. The collective and the kernel layer have the contract
  /// Returned tuple must contain at least two elements, with the first two elements being:
  /// gA_mkl - The tma tensor, A after a local tile so it has shape  (BLK_M,BLK_K,m,k,l)
  /// gB_nkl - The tma tensor, B after a local tile so it has shape  (BLK_N,BLK_K,n,k,l)
  /// The rest of the tensors can be specified as needed by this collective.
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE auto
  load_init(ProblemShape_MNKL const& problem_shape_MNKL, Params const& mainloop_params) const {
    using X = Underscore;
    // Separate out problem shape for convenience
    auto [M,N,K,L] = problem_shape_MNKL;

    // TMA requires special handling of strides to deal with coord codomain mapping
    // Represent the full tensors -- get these from TMA
    Tensor mA_mkl = mainloop_params.tma_load_a.get_tma_tensor(make_shape(M,K,L));                            // (m,k,l)
    Tensor mB_nkl = mainloop_params.tma_load_b.get_tma_tensor(make_shape(N,K,L));                            // (n,k,l)

    // Make tiled views, defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});        // (BLK_M,BLK_K,m,k,l)
    Tensor gB_nkl = local_tile(mB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});        // (BLK_N,BLK_K,n,k,l)

    return cute::make_tuple(gA_mkl, gB_nkl);
  }

```

**EN**: `prefetch_tma_descriptors` explicitly warms the TMA descriptors, and `load_init` converts those descriptors into global tensor views before applying `local_tile`. That means later producer code can work with CTA-local `(BLK_M, BLK_K, m, k, l)` and `(BLK_N, BLK_K, n, k, l)` tiles instead of rebuilding addressing logic on every iteration.
**CN**: `prefetch_tma_descriptors` 会显式预热 TMA 描述符，`load_init` 则把这些描述符转换成全局张量视图并再套用 `local_tile`。这样后续生产者代码就能直接处理 CTA 局部的 `(BLK_M, BLK_K, m, k, l)` 与 `(BLK_N, BLK_K, n, k, l)` tile，而不必在每轮迭代中重复构造寻址逻辑。

### Lines 353-460

```cpp
  template <
    class TensorA, class TensorB,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load(
      Params const& mainloop_params,
      MainloopPipeline pipeline,
      PrefetcherPipeline prefetcher_pipeline,
      PipelineState smem_pipe_write,
      TensorA const& gA_mkl,
      TensorB const& gB_nkl,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
    int lane_predicate = cute::elect_one_sync();

    if (lane_predicate) {
      bool disable_gdc = mainloop_params.overlap_ratio < 0.0;
      float overlap_ratio = mainloop_params.overlap_ratio;
      int launch_dep_grids_threshold = static_cast<int>(static_cast<float>(k_tile_count - 1) * overlap_ratio);

      Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.data()), SmemLayoutA{});        // (BLK_M,BLK_K,PIPE)
      Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.data()), SmemLayoutB{});        // (BLK_N,BLK_K,PIPE)

      //
      // Prepare the TMA loads for A
      //

      constexpr uint32_t cluster_shape_x = get<0>(typename DispatchPolicy::ClusterShape());
      uint2 cluster_local_block_id = {block_rank_in_cluster % cluster_shape_x, block_rank_in_cluster / cluster_shape_x};

      auto cta_tma_a = mainloop_params.tma_load_a.get_slice(cluster_local_block_id.y);
      auto cta_tma_b = mainloop_params.tma_load_b.get_slice(cluster_local_block_id.x);

      // Partition the inputs based on the current block coordinates.
      auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
      Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                     // (BLK_M,BLK_K,k)
      Tensor gB = gB_nkl(_,_,n_coord,_,l_coord);                                                     // (BLK_N,BLK_K,k)

      // Applies the mapping from cta_tma_a
      Tensor tAgA = cta_tma_a.partition_S(gA);                                                   // (TMA,TMA_M,TMA_K,k)
      Tensor tAsA = cta_tma_a.partition_D(sA);                                                // (TMA,TMA_M,TMA_K,PIPE)

      // Applies the mapping from cta_tma_b
      Tensor tBgB = cta_tma_b.partition_S(gB);                                                   // (TMA,TMA_N,TMA_K,k)
      Tensor tBsB = cta_tma_b.partition_D(sB);                                                // (TMA,TMA_N,TMA_K,PIPE)

      uint16_t mcast_mask_a = 0;
      uint16_t mcast_mask_b = 0;

      // Issue TmaLoads
      // Maps the tile -> block, value
      if constexpr (cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>) {
        auto block_layout = Layout<typename DispatchPolicy::ClusterShape>{}; // (m,n) -> block_id
        for (int n = 0; n < size<1>(block_layout); ++n) {
          mcast_mask_a |= (uint16_t(1) << block_layout(cluster_local_block_id.x,n,Int<0>{}));
        }
      }

      if constexpr (cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD_MULTICAST>) {
        auto block_layout = Layout<typename DispatchPolicy::ClusterShape>{};                       // (m,n) -> block_id
        for (int m = 0; m < size<0>(block_layout); ++m) {
          mcast_mask_b |= (uint16_t(1) << block_layout(m,cluster_local_block_id.y,Int<0>{}));
        }
      }

      // We have to wait on dependent grids because of B.
      cutlass::arch::wait_on_dependent_grids();

      // Signal prefetcher to stop
      prefetcher_pipeline.producer_arrive();

      bool launch_dep_grids = false;
      // Mainloop
      CUTLASS_PRAGMA_NO_UNROLL
      for (int cnt=0 ; k_tile_count > 0; --k_tile_count, ++cnt) {
        // LOCK smem_pipe_write for _writing_
        pipeline.producer_acquire(smem_pipe_write);

        //
        // Copy gmem to smem for *k_tile_iter
        //

        using BarrierType = typename MainloopPipeline::ProducerBarrierType;
        BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);

        int write_stage = smem_pipe_write.index();
        copy(mainloop_params.tma_load_a.with(*tma_barrier, mcast_mask_a, cute::TMA::CacheHintSm90::EVICT_FIRST), tAgA(_,_,_,*k_tile_iter), tAsA(_,_,_,write_stage));
        copy(mainloop_params.tma_load_b.with(*tma_barrier, mcast_mask_b, cute::TMA::CacheHintSm90::EVICT_LAST), tBgB(_,_,_,*k_tile_iter), tBsB(_,_,_,write_stage));
        ++k_tile_iter;

        if (!disable_gdc && cnt >= launch_dep_grids_threshold && !launch_dep_grids) {
          launch_dep_grids = true;
          cutlass::arch::launch_dependent_grids();
        }

        // Advance smem_pipe_write
        ++smem_pipe_write;
      }
      if (!disable_gdc && !launch_dep_grids) {
        cutlass::arch::launch_dependent_grids();
      }
    }
  }

```

**EN**: `load` is the combined producer path for A and B. A single elected lane derives cluster-local multicast masks, waits on dependent grids because B may observe producer-consumer ordering with a previous grid, signals the prefetcher to stop once the real mainloop starts, issues asynchronous TMA copies into the staged SMEM buffers, and optionally calls `launch_dependent_grids()` after the configured overlap threshold to overlap with the next grid.
**CN**: `load` 是同时搬运 A 和 B 的统一生产者路径。单个被选中的 lane 会计算 cluster 内 multicast mask，由于 B 可能与前一个 grid 存在生产/消费顺序关系，因此先等待 dependent grids，再在真正主循环启动后通知预取器停止，然后异步发起 TMA 拷贝把数据写入分阶段的共享内存缓冲区，并在达到设定的重叠阈值后按需调用 `launch_dependent_grids()` 以便与下一个 grid 重叠。

### Lines 461-548

```cpp
  template <
    class TensorA,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load_MK(
      Params const& mainloop_params,
      MainloopPipeline pipeline,
      PrefetcherPipeline prefetcher_pipeline,
      PipelineState smem_pipe_write,
      TensorA const& gA_mkl,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
    int lane_predicate = cute::elect_one_sync();

    if (lane_predicate) {
      bool disable_gdc = mainloop_params.overlap_ratio < 0.0;
      float overlap_ratio = mainloop_params.overlap_ratio;
      int launch_dep_grids_threshold = static_cast<int>(static_cast<float>(k_tile_count - 1) * overlap_ratio);

      Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.data()), SmemLayoutA{});        // (BLK_M,BLK_K,PIPE)

      //
      // Prepare the TMA loads for A
      //

      constexpr uint32_t cluster_shape_x = get<0>(typename DispatchPolicy::ClusterShape());
      uint2 cluster_local_block_id = {block_rank_in_cluster % cluster_shape_x, block_rank_in_cluster / cluster_shape_x};

      auto cta_tma_a = mainloop_params.tma_load_a.get_slice(cluster_local_block_id.y);

      // Partition the inputs based on the current block coordinates.
      auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
      Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                     // (BLK_M,BLK_K,k)

      // Applies the mapping from cta_tma_a
      Tensor tAgA = cta_tma_a.partition_S(gA);                                                   // (TMA,TMA_M,TMA_K,k)
      Tensor tAsA = cta_tma_a.partition_D(sA);                                                // (TMA,TMA_M,TMA_K,PIPE)

      uint16_t mcast_mask_a = 0;

      // Issue TmaLoads
      // Maps the tile -> block, value
      if constexpr (cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>) {
        auto block_layout = Layout<typename DispatchPolicy::ClusterShape>{}; // (m,n) -> block_id
        for (int n = 0; n < size<1>(block_layout); ++n) {
          mcast_mask_a |= (uint16_t(1) << block_layout(cluster_local_block_id.x,n,Int<0>{}));
        }
      }

      // Don't wait on dependent grids when loading `A`, because
      // we assume `A` (weights) are static.

      bool launch_dep_grids = false;
      // Mainloop
      CUTLASS_PRAGMA_NO_UNROLL
      for (int cnt=0 ; k_tile_count > 0; --k_tile_count, ++cnt) {
        // LOCK smem_pipe_write for _writing_
        pipeline.producer_acquire(smem_pipe_write);

        //
        // Copy gmem to smem for *k_tile_iter
        //

        using BarrierType = typename MainloopPipeline::ProducerBarrierType;
        BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);

        int write_stage = smem_pipe_write.index();
        copy(mainloop_params.tma_load_a.with(*tma_barrier, mcast_mask_a, cute::TMA::CacheHintSm90::EVICT_FIRST), tAgA(_,_,_,*k_tile_iter), tAsA(_,_,_,write_stage));
        ++k_tile_iter;

        if (!disable_gdc && cnt >= launch_dep_grids_threshold && !launch_dep_grids) {
          launch_dep_grids = true;
          cutlass::arch::launch_dependent_grids();
        }

        // Advance smem_pipe_write
        ++smem_pipe_write;
      }
      if (!disable_gdc && !launch_dep_grids) {
        cutlass::arch::launch_dependent_grids();
      }
    }
  }

```

**EN**: `load_MK` is the A-only variant. Its logic mirrors the unified loader, but it deliberately skips the initial dependent-grid wait because the file assumes A represents reusable weights; that lets the kernel overlap static-weight fetch with downstream work more aggressively. The cache hint also keeps A on the `EVICT_FIRST` policy.
**CN**: `load_MK` 是只加载 A 的版本。它的流程与统一加载器类似，但刻意跳过了初始的 dependent-grid 等待，因为这里默认 A 表示可复用的权重；这样 kernel 就能更激进地把静态权重读取与后续工作重叠。对应的缓存 hint 也保持为 A 使用 `EVICT_FIRST` 策略。

### Lines 549-645

```cpp
  template <
    class TensorB,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load_NK(
      Params const& mainloop_params,
      MainloopPipeline pipeline,
      PrefetcherPipeline prefetcher_pipeline,
      PipelineState smem_pipe_write,
      TensorB const& gB_nkl,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
    int lane_predicate = cute::elect_one_sync();

    if (lane_predicate) {
      Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.data()), SmemLayoutB{});        // (BLK_N,BLK_K,PIPE)

      //
      // Prepare the TMA loads for B
      //

      constexpr uint32_t cluster_shape_x = get<0>(typename DispatchPolicy::ClusterShape());
      uint2 cluster_local_block_id = {block_rank_in_cluster % cluster_shape_x, block_rank_in_cluster / cluster_shape_x};

      auto cta_tma_b = mainloop_params.tma_load_b.get_slice(cluster_local_block_id.x);

      // Partition the inputs based on the current block coordinates.
      auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
      Tensor gB = gB_nkl(_,_,n_coord,_,l_coord);                                                     // (BLK_N,BLK_K,k)

      // Applies the mapping from cta_tma_b
      Tensor tBgB = cta_tma_b.partition_S(gB);                                                   // (TMA,TMA_N,TMA_K,k)
      Tensor tBsB = cta_tma_b.partition_D(sB);                                                // (TMA,TMA_N,TMA_K,PIPE)

      uint16_t mcast_mask_b = 0;

      // Issue TmaLoads
      // Maps the tile -> block, value
      if constexpr (cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD_MULTICAST>) {
        auto block_layout = Layout<typename DispatchPolicy::ClusterShape>{};                       // (m,n) -> block_id
        for (int m = 0; m < size<0>(block_layout); ++m) {
          mcast_mask_b |= (uint16_t(1) << block_layout(m,cluster_local_block_id.y,Int<0>{}));
        }
      }

      // Ensure that the prefetched kernel does not touch
      // unflushed global memory prior to this instruction
      cutlass::arch::wait_on_dependent_grids();

      // Signal prefetcher to stop
      prefetcher_pipeline.producer_arrive();

      // Mainloop
      CUTLASS_PRAGMA_NO_UNROLL
      for (; k_tile_count > 0; --k_tile_count) {
        // LOCK smem_pipe_write for _writing_
        pipeline.producer_acquire(smem_pipe_write);

        //
        // Copy gmem to smem for *k_tile_iter
        //

        using BarrierType = typename MainloopPipeline::ProducerBarrierType;
        BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);

        int write_stage = smem_pipe_write.index();
        copy(mainloop_params.tma_load_b.with(*tma_barrier, mcast_mask_b, cute::TMA::CacheHintSm90::EVICT_LAST), tBgB(_,_,_,*k_tile_iter), tBsB(_,_,_,write_stage));
        ++k_tile_iter;

        // Advance smem_pipe_write
        ++smem_pipe_write;
      }
    }
  }

  /// Perform a Producer Epilogue to prevent early exit of blocks in a Cluster
  CUTLASS_DEVICE void
  load_tail(MainloopPipeline pipeline, PipelineState smem_pipe_write) {
    int lane_predicate = cute::elect_one_sync();

    // Issue the epilogue waits
    if (lane_predicate) {
      /* This helps avoid early exit of blocks in Cluster
       * Waits for all stages to either be released (all
       * Consumer UNLOCKs), or if the stage was never used
       * then would just be acquired since the phase was
       * still inverted from make_producer_start_state
       */
      pipeline.producer_tail(smem_pipe_write);
    }
  }


```

**EN**: `load_NK` is the B-only path, and it does perform `wait_on_dependent_grids()` because B is treated as data that may depend on prior kernels. After that it stops the prefetcher, issues staged TMA copies for B, and `load_tail` drains producer ownership at the end so no block in the Hopper cluster exits early while peers still rely on the shared pipeline state.
**CN**: `load_NK` 是只加载 B 的路径，它会执行 `wait_on_dependent_grids()`，因为 B 被视为可能依赖先前 kernel 的数据。之后它通知预取器停止、为 B 发出分阶段的 TMA 拷贝，而 `load_tail` 会在结尾把生产者端的所有阶段排空，防止 Hopper cluster 中某个 block 提前退出，而其他 block 仍依赖共享的流水线状态。

### Lines 646-723

```cpp
  template <
    class TensorA,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  prefetch_MK(
      Params const& mainloop_params,
      PrefetcherPipeline prefetcher_pipeline,
      PipelineState smem_pipe_write,
      TensorA const& gA_mkl,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
    int lane_predicate = cute::elect_one_sync();

    if (lane_predicate) {
      bool do_best_effort_prefetch = mainloop_params.prefetch_ratio < 0;
      float prefetch_ratio = do_best_effort_prefetch ? 1.0 : mainloop_params.prefetch_ratio;
      int prefetch_iters = static_cast<int>(static_cast<float>(k_tile_count) * 0.5 * prefetch_ratio);
      prefetch_iters = min(k_tile_count, ((prefetch_iters + detail::PrefetchStages - 1) / detail::PrefetchStages) * detail::PrefetchStages);

      Tensor sA = make_tensor(
          make_smem_ptr(shared_tensors.smem_prefetch.data()), PrefetchSmemLayoutA{});             // (BLK_M,BLK_K,PIPE)

      //
      // Prepare the TMA loads for A
      //

      constexpr uint32_t cluster_shape_x = get<0>(typename DispatchPolicy::ClusterShape());
      uint2 cluster_local_block_id = {block_rank_in_cluster % cluster_shape_x, block_rank_in_cluster / cluster_shape_x};

      auto cta_tma_a = mainloop_params.tma_load_a.get_slice(cluster_local_block_id.y);

      // Partition the inputs based on the current block coordinates.
      auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
      Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                     // (BLK_M,BLK_K,k)

      // Applies the mapping from cta_tma_a
      Tensor tAgA = cta_tma_a.partition_S(gA);                                                   // (TMA,TMA_M,TMA_K,k)
      Tensor tAsA = cta_tma_a.partition_D(sA);                                                // (TMA,TMA_M,TMA_K,PIPE)

      uint16_t mcast_mask_a = 0;

      // Issue TmaLoads
      // Maps the tile -> block, value
      if constexpr (cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>) {
        auto block_layout = Layout<typename DispatchPolicy::ClusterShape>{};                       // (m,n) -> block_id
        for (int n = 0; n < size<1>(block_layout); ++n) {
          mcast_mask_a |= (uint16_t(1) << block_layout(cluster_local_block_id.x,n,Int<0>{}));
        }
      }

      uint32_t prefetcher_stage = 0;
      uint32_t prefetcher_phase = 0;
      CUTLASS_PRAGMA_NO_UNROLL
      for (int cnt = 0 ; cnt < prefetch_iters; ++cnt) {

        if (do_best_effort_prefetch && prefetcher_pipeline.have_producers_arrived()) {
          break;
        }

        prefetcher_pipeline.prefetcher_acquire(prefetcher_stage, prefetcher_phase, cnt >= detail::PrefetchStages);
        using BarrierType = typename PrefetcherPipeline::PrefetcherBarrierType;
        BarrierType* tma_barrier = prefetcher_pipeline.prefetcher_get_barrier(prefetcher_stage);

        int write_stage = 0;
        copy(mainloop_params.tma_load_a.with(*tma_barrier, mcast_mask_a, cute::TMA::CacheHintSm90::EVICT_FIRST), tAgA(_,_,_,*k_tile_iter), tAsA(_,_,_,write_stage));
        ++k_tile_iter;
        ++k_tile_iter;

        prefetcher_pipeline.advance_prefetcher_state(prefetcher_stage, prefetcher_phase);
      }
      prefetcher_pipeline.prefetcher_tail(prefetcher_stage, prefetcher_phase);
    }
  }

```

**EN**: `prefetch_MK` drives the auxiliary weight-prefetch pipeline. It computes how many iterations to prefetch from `prefetch_ratio`, allocates a separate SMEM view over `smem_prefetch`, acquires/releases prefetcher barriers independently from the main pipeline, and issues best-effort or bounded TMA loads for A so a following dependent grid can see weights in a warmer cache state without polluting the mainloop buffer schedule.
**CN**: `prefetch_MK` 负责驱动辅助的权重预取流水线。它根据 `prefetch_ratio` 计算预取轮数，为 `smem_prefetch` 建立独立的共享内存视图，独立于主流水线地获取/推进预取 barrier，并为 A 发出尽力而为或有界的 TMA 加载，使后续 dependent grid 能在更热的缓存状态下读取权重，同时不破坏主流水线缓冲区的阶段调度。

### Lines 724-847

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
    static_assert(cute::rank(SmemLayoutA{}) == 3, "Smem layout must be rank 3.");
    static_assert(cute::rank(SmemLayoutB{}) == 3, "Smem layout must be rank 3.");
    static_assert(cute::is_void_v<SmemCopyAtomA>,
      "SM90 GMMA mainloops cannot have a non-void copy atom for smem sourced instructions.");
    static_assert(cute::is_void_v<SmemCopyAtomB>,
      "SM90 GMMA mainloops cannot have a non-void copy atom for smem sourced instructions.");

    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.data()), SmemLayoutA{});          // (BLK_M,BLK_K,PIPE)
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.data()), SmemLayoutB{});          // (BLK_N,BLK_K,PIPE)

    //
    // Define C accumulators and A/B partitioning
    //

    TiledMma tiled_mma;
    auto thread_mma = tiled_mma.get_thread_slice(thread_idx);

    Tensor tCsA = thread_mma.partition_A(sA);                                                 // (MMA,MMA_M,MMA_K,PIPE)
    Tensor tCsB = thread_mma.partition_B(sB);                                                 // (MMA,MMA_N,MMA_K,PIPE)

    // Allocate "fragments/descriptors"
    Tensor tCrA = thread_mma.make_fragment_A(tCsA);                                           // (MMA,MMA_M,MMA_K,PIPE)
    Tensor tCrB = thread_mma.make_fragment_B(tCsB);                                           // (MMA,MMA_N,MMA_K,PIPE)

    CUTE_STATIC_ASSERT_V(size<1>(tCsA) == size<1>(accum));                                                         // M
    CUTE_STATIC_ASSERT_V(size<1>(tCsB) == size<2>(accum));                                                         // N
    CUTE_STATIC_ASSERT_V(size<2>(tCsA) == size<2>(tCsB));                                                          // K
    CUTE_STATIC_ASSERT_V(size<3>(tCsA) == size<3>(tCsB));                                                       // PIPE
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<2>(sA));                                         // PIPE
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<2>(sB));                                         // PIPE

    //
    // PIPELINED MAIN LOOP
    //
    static_assert((0 <= K_PIPE_MMAS) && (K_PIPE_MMAS <  K_PIPE_MAX),
        "ERROR : Incorrect number of MMAs in flight");

    // We release buffers to producer warps(dma load) with some mmas in flight
    PipelineState smem_pipe_release = smem_pipe_read;

    // Prologue GMMAs
    int prologue_mma_count = min(K_PIPE_MMAS, k_tile_count);

    tiled_mma.accumulate_ = GMMA::ScaleOut::Zero;

    warpgroup_fence_operand(accum);
    CUTLASS_PRAGMA_UNROLL
    for (int k_tile_prologue = prologue_mma_count; k_tile_prologue > 0; --k_tile_prologue)
    {
      // WAIT on smem_pipe_read until its data are available (phase bit flips from rdPhaseBit value)
      auto barrier_token = pipeline.consumer_try_wait(smem_pipe_read);
      pipeline.consumer_wait(smem_pipe_read, barrier_token);

      int read_stage = smem_pipe_read.index();
      warpgroup_arrive();
      // Unroll the K mode manually to set scale D to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < size<2>(tCrA); ++k_block) {
        // (V,M,K) x (V,N,K) => (V,M,N)
        cute::gemm(tiled_mma, tCrA(_,_,k_block,read_stage), tCrB(_,_,k_block,read_stage), accum);
        tiled_mma.accumulate_ = GMMA::ScaleOut::One;
      }

      warpgroup_commit_batch();

      ++smem_pipe_read;
    }

    warpgroup_fence_operand(accum);
    // Mainloop GMMAs
    k_tile_count -= prologue_mma_count;

    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count)
    {
      // WAIT on smem_pipe_read until its data are available (phase bit flips from rdPhaseBit value)
      auto barrier_token = pipeline.consumer_try_wait(smem_pipe_read);
      pipeline.consumer_wait(smem_pipe_read, barrier_token);

      //
      // Compute on k_tile
      //

      int read_stage = smem_pipe_read.index();
      warpgroup_fence_operand(accum);
      warpgroup_arrive();
      // Unroll the K mode manually to set scale D to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < size<2>(tCrA); ++k_block) {
        // (V,M,K) x (V,N,K) => (V,M,N)
        cute::gemm(tiled_mma, tCrA(_,_,k_block,read_stage), tCrB(_,_,k_block,read_stage), accum);
        tiled_mma.accumulate_ = GMMA::ScaleOut::One;
      }
      warpgroup_commit_batch();

      /// Wait on the GMMA barrier for K_PIPE_MMAS (or fewer) outstanding to ensure smem_pipe_write is consumed
      warpgroup_wait<K_PIPE_MMAS>();
      warpgroup_fence_operand(accum);

      // UNLOCK smem_pipe_release, done _computing_ on it
      pipeline.consumer_release(smem_pipe_release);

      // Advance smem_pipe_read and smem_pipe_release
      ++smem_pipe_read;
      ++smem_pipe_release;
    }

    warpgroup_fence_operand(accum);
  }

```

**EN**: `mma` is the consumer-side GMMA loop. It partitions stage-resident SMEM tensors into per-thread descriptors, constructs register fragments, runs a prologue to fill the in-flight GMMA window, and then enters the steady-state loop that waits on pipeline tokens, launches warpgroup GMMA batches, fences accumulator visibility, waits for a bounded number of outstanding MMAs, and finally releases consumed stages back to the producer. This is the core TMA+GMMA warp-specialized pipeline in the file.
**CN**: `mma` 是消费者侧的 GMMA 计算循环。它把按 stage 存放的共享内存张量切分成每线程描述符，构造寄存器 fragment，先执行一个前导阶段填满在途 GMMA 窗口，然后进入稳态循环：等待流水线 token、发起 warpgroup GMMA 批次、通过 fence 保证累加器可见性、等待有界数量的在途 MMA 完成，并把已消费阶段释放回生产者。这正是本文件中最核心的 TMA+GMMA warp-specialized 流水线。

### Lines 848-871

```cpp
  /// Perform a Consumer Epilogue to release all buffers
  CUTLASS_DEVICE void
  mma_tail(MainloopPipeline pipeline, PipelineState smem_pipe_release, int k_tile_count) {
    // Prologue GMMAs
    int prologue_mma_count = min(K_PIPE_MMAS, k_tile_count);
    k_tile_count -= prologue_mma_count;

    smem_pipe_release.advance(k_tile_count);

    // Wait on all GMMAs to complete
    warpgroup_wait<0>();

    for (int count = 0; count < prologue_mma_count; ++count) {
      pipeline.consumer_release(smem_pipe_release);                 // UNLOCK smem_pipe_release, done _computing_ on it
      ++smem_pipe_release;
    }
  }
};

/////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace cutlass::gemm::collective

/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: `mma_tail` finishes the consumer side by waiting for all outstanding GMMA operations and releasing any still-owned pipeline stages. The namespace close marks the end of a collective implementation that is almost entirely assembled from compile-time traits but coordinated at runtime by lightweight pipeline state machines.
**CN**: `mma_tail` 通过等待所有剩余的 GMMA 操作完成，并释放仍被消费者持有的流水线阶段，从而收尾消费者端。最后的命名空间结束处表明：这个 collective 的绝大多数结构都由编译期 traits 组装，但运行时仍通过轻量级流水线状态机来协调。

---

## Key Concepts / 关键概念

**EN**: Compile-time trait selection drives everything: tile shapes, cluster shape, TMA atom kind, GMMA descriptor types, shared-memory layout, and scheduling policy are all encoded in the template signature.
  **CN**: 一切都由编译期 trait 选择驱动：tile 形状、cluster 形状、TMA atom 类型、GMMA 描述符类型、共享内存布局以及调度策略都编码在模板签名中。
**EN**: The main async path is a producer/consumer pipeline built on `PipelineTmaAsync`, while a second `PrefetchPipeline` warms A/weight data for dependent grids.
  **CN**: 主异步路径是建立在 `PipelineTmaAsync` 上的生产者/消费者流水线，而第二条 `PrefetchPipeline` 则用于为 dependent grid 预热 A/权重数据。
**EN**: `wait_on_dependent_grids()` and `launch_dependent_grids()` are used as explicit inter-grid ordering/overlap controls; the file treats A and B differently to exploit static-weight behavior.
  **CN**: `wait_on_dependent_grids()` 与 `launch_dependent_grids()` 被用作显式的 grid 间顺序/重叠控制；该文件对 A 与 B 采用不同策略，以利用静态权重这一特性。
**EN**: The consumer side is pure GMMA warpgroup math: TMA fills staged SMEM buffers, then warpgroup GMMA instructions stream through those descriptors while pipeline tokens control reuse safety.
  **CN**: 消费者端是纯粹的 GMMA warpgroup 计算：TMA 先填充分阶段共享内存缓冲区，随后 warpgroup GMMA 指令流式消费这些描述符，而流水线 token 负责保证缓冲区复用的安全。

## Dependencies / 依赖项

**EN**: `cutlass/gemm/dispatch_policy.hpp` and local `dispatch_policy_extra.hpp` define the dispatch-policy type matched by this specialization.
  **CN**: `cutlass/gemm/dispatch_policy.hpp` 与本地的 `dispatch_policy_extra.hpp` 定义了该偏特化要匹配的 dispatch-policy 类型。
**EN**: `cutlass/pipeline/pipeline.hpp` and `../pipeline/prefetch_pipeline_sm90.hpp` provide the main and auxiliary async pipeline primitives.
  **CN**: `cutlass/pipeline/pipeline.hpp` 与 `../pipeline/prefetch_pipeline_sm90.hpp` 提供了主流水线与辅助预取流水线原语。
**EN**: CuTe SM90 headers such as `cute/arch/copy_sm90.hpp` and `cute/algorithm/gemm.hpp` supply TMA copy builders, GMMA descriptor machinery, and tensor algebra helpers.
  **CN**: `cute/arch/copy_sm90.hpp`、`cute/algorithm/gemm.hpp` 等 CuTe SM90 头文件提供了 TMA copy 构造器、GMMA 描述符机制以及张量代数工具。
**EN**: `cutlass/arch/grid_dependency_control.h` is the mechanism for cross-grid wait/launch coordination used by the overlap and prefetch logic.
  **CN**: `cutlass/arch/grid_dependency_control.h` 是重叠控制与预取逻辑中用于 grid 间等待/启动协调的关键机制。
