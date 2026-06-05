# scatter_epilogue.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/52_hopper_gather_scatter_fusion/scatter_epilogue.hpp`  
**Purpose / 用途**: Implements a custom CUTLASS epilogue collective that optionally gathers source tensor C, applies a thread-level epilogue operator, and scatters results into tensor D. / 实现一个自定义 CUTLASS epilogue collective：可选地 gather 源张量 C，执行线程级 epilogue 运算，并把结果 scatter 写入张量 D。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-47

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
/*! \file
  \brief Functor performing elementwise operations used by epilogues.
*/

#pragma once

#include "cutlass/cutlass.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/epilogue/collective/detail.hpp"

#include "cute/tensor.hpp"
#include "cute/numeric/numeric_types.hpp"

#include "gather_tensor.hpp"

namespace cutlass::epilogue::collective {

```

**EN**: The header comment and include list establish that this file is a lightweight epilogue extension rather than a full kernel. It depends on generic CUTLASS epilogue plumbing plus `gather_tensor.hpp`, which is the key utility that turns ordinary global-memory tensors into coordinate-remapped views.
**CN**: 文件注释和头文件列表表明：这不是一个完整 kernel，而是一个轻量级 epilogue 扩展。它依赖通用的 CUTLASS epilogue 基础设施以及 `gather_tensor.hpp`，后者是把普通全局内存张量包装成“坐标可重映射视图”的关键工具。

### Lines 48-107

```cpp
/// Applies an element wise operation to all elements within the fragment
/// and scatter-writes them out to destination storage.
/// GatherC and ScatterD are types of user-defined functions that apply the
/// transoformation of the strided coordinate (e.g. through an index array).
template <
  class StrideC_,
  class StrideD_,
  class ThreadEpilogueOp_,
  class EpilogueSchedule_,
  class GatherC_,
  class ScatterD_
>
class EpilogueGatherScatter {
public:
  //
  // Type Aliases
  //
  using EpilogueSchedule = EpilogueSchedule_;
  
  // derived types of output thread level operator
  using ThreadEpilogueOp = ThreadEpilogueOp_;
  using ElementOutput = typename ThreadEpilogueOp::ElementOutput;
  using ElementAccumulator = typename ThreadEpilogueOp::ElementAccumulator;
  using ElementCompute = typename ThreadEpilogueOp::ElementCompute;
  using ElementScalar = ElementCompute;
  using ElementC = typename ThreadEpilogueOp::ElementC;
  using StrideC = StrideC_;
  using ElementD = typename ThreadEpilogueOp::ElementD;
  using StrideD = StrideD_;

  // Every epilogue needs these two GmemTiledCopy{C,D} aliases.
  // If you don't know what they should be, just use void.
  using GmemTiledCopyC = void;
  using GmemTiledCopyD = void;

  using GatherC = GatherC_;
  using ScatterD = ScatterD_;

  static const int kOutputAlignment = ThreadEpilogueOp::kCount;
  using AlignmentType = typename cute::uint_bit<sizeof_bits<ElementOutput>::value * kOutputAlignment>::type;

  static_assert(cute::rank(StrideC{}) == 3, "StrideCD must be rank-3: [M, N, L]");
  static_assert(cute::rank(StrideD{}) == 3, "StrideCD must be rank-3: [M, N, L]");

  struct SharedStorage { };

  // Host side epilogue arguments
  struct Arguments {
    typename ThreadEpilogueOp::Params thread_params{};
    ElementC const* ptr_C = nullptr;
    StrideC dC{};
    ElementD* ptr_D = nullptr;
    StrideD dD{};
    GatherC gather_C{};
    ScatterD scatter_D{};
  };

  // Device side epilogue params
  using Params = Arguments;

```

**EN**: `EpilogueGatherScatter` is parameterized by source/destination strides, a thread-level epilogue operator, the epilogue schedule tag, and the gather/scatter functor types. The alias section surfaces all derived element types, defines placeholder `GmemTiledCopyC/D` aliases required by higher-level adapters, and packages host/device arguments as raw pointers, strides, and functors.
**CN**: `EpilogueGatherScatter` 以源/目标步长、线程级 epilogue 操作符、epilogue schedule 标签，以及 gather/scatter functor 类型为模板参数。别名部分导出了所有派生元素类型，定义了上层 adapter 所要求的占位 `GmemTiledCopyC/D` 别名，并把主机/设备参数封装为原始指针、步长与 functor。

### Lines 108-131

```cpp
  //
  // Methods
  //

  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(
      [[maybe_unused]] ProblemShape const& _,
      Arguments const& args,
      [[maybe_unused]] void* workspace) {
    return args;
  }

  template<class ProblemShape>
  static bool
  can_implement(
      [[maybe_unused]] ProblemShape const& problem_shape,
      [[maybe_unused]] Arguments const& args) {
    return true;
  }

  CUTLASS_HOST_DEVICE
  EpilogueGatherScatter(Params const& params_) : params(params_) { }

```

**EN**: `to_underlying_arguments` is just a pass-through because this custom epilogue does not need extra lowering, `can_implement` accepts everything, and the constructor simply stores `Params`. That simplicity is intentional: all structural scheduling decisions are already made by the surrounding SM90 epilogue adapter, while this class focuses purely on per-element mapping and fusion semantics.
**CN**: `to_underlying_arguments` 只是直接透传，因为这个自定义 epilogue 不需要额外降参；`can_implement` 也始终返回真；构造函数仅保存 `Params`。这种简洁是刻意设计的：结构性的调度决策已经由外层 SM90 epilogue adapter 完成，而本类只专注于逐元素的坐标映射与融合语义。

### Lines 132-181

```cpp
  template<
    class ProblemShapeMNKL,
    class BlockShapeMNK,
    class BlockCoordMNKL,
    class FrgEngine, class FrgLayout,
    class TiledMma,
    class ResidueMNK
  >
  CUTLASS_DEVICE void
  operator()(
      ProblemShapeMNKL problem_shape_mnkl,
      BlockShapeMNK blk_shape_MNK,
      BlockCoordMNKL blk_coord_mnkl,
      cute::Tensor<FrgEngine, FrgLayout> const& accumulators,
      TiledMma tiled_mma,
      ResidueMNK residue_mnk,
      int thread_idx,
      char* smem_buf)
  {
    using namespace cute;
    using X = Underscore;

    static_assert(cute::rank(ProblemShapeMNKL{}) == 4, "ProblemShapeMNKL must be rank 4");
    static_assert(is_static<BlockShapeMNK>::value, "ThreadBlock tile shape must be static");
    static_assert(cute::rank(BlockShapeMNK{}) == 3, "BlockShapeMNK must be rank 3");
    static_assert(cute::rank(BlockCoordMNKL{}) == 4, "BlockCoordMNKL must be rank 3");

    (void) smem_buf;
    ThreadEpilogueOp epilogue_op{params.thread_params};

    // Separate out problem shape for convenience
    auto M = get<0>(problem_shape_mnkl);
    auto N = get<1>(problem_shape_mnkl);
    auto L = get<3>(problem_shape_mnkl);

    auto stride_c = detail::get_epilogue_stride<EpilogueSchedule>(params.dC);
    auto stride_d = detail::get_epilogue_stride<EpilogueSchedule>(params.dD);

    // Represent the full output tensor
    Tensor mC_mnl = make_gather_tensor(make_gmem_ptr(params.ptr_C), make_shape(M,N,L), stride_c, params.gather_C);  // (m,n,l)
    Tensor mD_mnl = make_gather_tensor(make_gmem_ptr(params.ptr_D), make_shape(M,N,L), stride_d, params.scatter_D); // (m,n,l)

    Tensor gC_mnl = local_tile(mC_mnl, blk_shape_MNK, make_coord(_,_,_), Step<_1,_1, X>{});    // (BLK_M,BLK_N,m,n,l)
    Tensor gD_mnl = local_tile(mD_mnl, blk_shape_MNK, make_coord(_,_,_), Step<_1,_1, X>{});    // (BLK_M,BLK_N,m,n,l)

    // Slice to get the tile this CTA is responsible for
    auto [m_coord, n_coord, k_coord, l_coord] = blk_coord_mnkl;
    Tensor gC = gC_mnl(_,_,m_coord,n_coord,l_coord);                                                 // (BLK_M,BLK_N)
    Tensor gD = gD_mnl(_,_,m_coord,n_coord,l_coord);                                                 // (BLK_M,BLK_N)

```

**EN**: The call operator receives the problem shape, block shape, block coordinates, accumulator fragments, MMA tiler, residue, and thread index. It converts C and D into gather/scatter-aware global tensors with `make_gather_tensor`, slices them to the CTA tile via `local_tile`, and then selects the exact tile that the current CTA owns. This is the point where logical gather/scatter semantics are fused into the epilogue address generation.
**CN**: 调用运算符接收问题形状、block 形状、block 坐标、累加器 fragment、MMA 切分器、尾块残差以及线程索引。它通过 `make_gather_tensor` 把 C 和 D 转成带 gather/scatter 语义的全局张量，再借助 `local_tile` 切到 CTA 级 tile，最后选出当前 CTA 真正负责的那一块。这正是把逻辑 gather/scatter 语义融合进 epilogue 地址生成的地方。

### Lines 182-219

```cpp
    // Partition source and destination tiles to match the accumulator partitioning
    auto thr_mma = tiled_mma.get_thread_slice(thread_idx);
    Tensor tCgD = thr_mma.partition_C(gD);                                       // (VEC,THR_M,THR_N)
    Tensor tCgC = thr_mma.partition_C(gC);                                       // (VEC,THR_M,THR_N)

    static_assert(is_static<FrgLayout>::value, "Accumulator layout must be static");
    CUTE_STATIC_ASSERT_V(size(tCgC) == size(tCgD),
        "Source and destination must have the same number of elements.");
    CUTE_STATIC_ASSERT_V(size(tCgD) == size(accumulators),
        "Accumulator count must have the same destination element count.");

    // Make an identity coordinate tensor for predicating our output MN tile
    auto cD = make_identity_tensor(make_shape(unwrap(shape<0>(gD)), unwrap(shape<1>(gD))));
    Tensor tCcD = thr_mma.partition_C(cD);

    // source is needed
    if (epilogue_op.is_source_needed()) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(accumulators); ++i) {
        if (elem_less(tCcD(i), make_coord(get<0>(residue_mnk), get<1>(residue_mnk)))) {
          tCgD(i) = epilogue_op(accumulators(i), tCgC(i));
        }
      }
    }
    // source is not needed, avoid load
    else {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(accumulators); ++i) {
        if (elem_less(tCcD(i), make_coord(get<0>(residue_mnk), get<1>(residue_mnk)))) {
          tCgD(i) = epilogue_op(accumulators(i));
        }
      }
    }
  }

private:
  Params params;
};
```

**EN**: Next, the code partitions C and D according to the MMA thread mapping, creates an identity tensor for residue predication, and applies the thread epilogue op elementwise. The important branch is `is_source_needed()`: when the epilogue op needs source C (for example, nonzero beta), it loads from `tCgC`; otherwise it skips that read and applies a source-free form. The write still lands in `tCgD`, so scatter semantics are preserved either way.
**CN**: 接下来，代码按 MMA 的线程映射切分 C 与 D，构造一个恒等坐标张量用于尾块 predication，并逐元素调用线程级 epilogue 操作。最关键的分支是 `is_source_needed()`：当 epilogue 操作需要源张量 C（例如 beta 非零）时，就从 `tCgC` 读取；否则会跳过该读操作，直接采用无源版本。无论哪条分支，最终写回的目标都是 `tCgD`，因此 scatter 语义始终被保留。

### Lines 220-222

```cpp

} // namespace cutlass::epilogue::collective

```

**EN**: The file ends immediately after storing `Params`, which reinforces that this class is a focused customization point plugged into a larger SM90 epilogue pipeline.
**CN**: 文件在保存 `Params` 后很快结束，这也再次说明该类只是插入到更大 SM90 epilogue 流水线中的一个专用定制点。

---

## Key Concepts / 关键概念

**EN**: The class does not build an async pipeline itself; instead it relies on the outer SM90 epilogue adapter for scheduling and only customizes coordinate mapping plus thread-level fusion.
  **CN**: 该类本身并不构建异步流水线；它依赖外层 SM90 epilogue adapter 完成调度，而这里只定制坐标映射与线程级融合。
**EN**: Gather on C and scatter on D are both expressed by wrapping raw pointers in `make_gather_tensor`, so address remapping happens through normal tensor indexing instead of bespoke control flow.
  **CN**: 对 C 的 gather 和对 D 的 scatter 都通过 `make_gather_tensor` 包装原始指针来表达，因此地址重映射是经由常规张量索引完成的，而不是额外的专门控制流。
**EN**: `ThreadEpilogueOp` remains fully generic, so the same class can host linear combination, activation, or other fused operators as long as they expose the CUTLASS thread-op interface.
  **CN**: `ThreadEpilogueOp` 保持完全泛化，因此只要某个操作满足 CUTLASS 线程级接口，这个类就可以承载线性组合、激活函数或其他融合操作。
**EN**: Residue predication is handled by an identity tensor plus `elem_less`, ensuring gather/scatter remapping still respects partial tiles at the edges.
  **CN**: 尾块 predication 借助恒等坐标张量与 `elem_less` 完成，从而保证即使发生 gather/scatter 重映射，边界上的部分 tile 仍能被正确裁剪。

## Dependencies / 依赖项

**EN**: `gather_tensor.hpp` provides the coordinate-remapping tensor wrapper used for both source and destination.
  **CN**: `gather_tensor.hpp` 提供了源张量与目标张量都要用到的坐标重映射张量包装器。
**EN**: `cutlass/epilogue/collective/detail.hpp` provides helper utilities such as `get_epilogue_stride` used to interpret schedule-dependent strides.
  **CN**: `cutlass/epilogue/collective/detail.hpp` 提供了 `get_epilogue_stride` 等辅助工具，用于解释依赖 schedule 的 stride。
**EN**: The surrounding SM90 epilogue adapter expects aliases like `GmemTiledCopyC/D`, `Arguments`, and `Params`, so this class follows that contract.
  **CN**: 外层 SM90 epilogue adapter 会要求 `GmemTiledCopyC/D`、`Arguments` 与 `Params` 等别名，因此该类遵循了这套接口约定。
