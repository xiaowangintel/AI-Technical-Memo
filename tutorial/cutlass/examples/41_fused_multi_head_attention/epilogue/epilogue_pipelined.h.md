# epilogue_pipelined.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/epilogue/epilogue_pipelined.h`
**Purpose / 用途**: A CUTLASS epilogue adapted for FMHA. It writes MM1 accumulators to global memory while optionally reading previous output fragments and forwarding a logical row id into the output operator for row-wise rescaling. / 这是一个为 FMHA 定制的 CUTLASS epilogue：它负责把 MM1 累加器写回全局内存，并在需要时读取已有输出 fragment，同时把逻辑行号传给输出算子，以支持按行重标定。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-66 / 第 1-66 行
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
  \brief Epilogue for threadblock scoped GEMMs using Tensor Ops.

  File copied from "cutlass/epilogue/threadblock/epilogue.h"
  then modified to:
  (1) load 2 source fragments at the same time (pipelining)
  (2) support reading from a different dtype
  (3) pass the row id to the OutputOp if it takes it
    (see MemoryEfficientAttentionNormalize)
  Note that in general the fragment passed to the OutputOp could
  span multiple rows but it does not happen with the configurations we have
*/
#pragma once

#include "cutlass/aligned_buffer.h"
#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include CUDA_STD_HEADER(cassert)
#include "cutlass/functional.h"
#include "cutlass/layout/tensor.h"
#include "cutlass/layout/vector.h"
#include "cutlass/numeric_types.h"
#include "cutlass/tensor_coord.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/transform/pitch_linear_thread_map.h"
#include "cutlass/transform/threadblock/regular_tile_iterator.h"
#include "cutlass/epilogue/threadblock/epilogue_base.h"
#include "cutlass/epilogue/threadblock/predicated_tile_iterator.h"
#include "cutlass/numeric_types.h"

////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace epilogue {
namespace threadblock {
```
**EN**: License, includes, and FMHA-specific comment explaining the row-id modification to the stock CUTLASS epilogue.
**CN**: 许可证、依赖头文件以及 FMHA 定制说明：相比标准 CUTLASS epilogue，额外把 row id 传入输出算子。

### Lines 67-82 / 第 67-82 行
```cpp
template <typename Op>
struct ApplyEpilogueOp {
  static CUTLASS_DEVICE typename Op::FragmentOutput apply(
      Op const& output_op,
      int row_id,
      typename Op::FragmentAccumulator const& accum,
      typename Op::FragmentOutput const& source) {
    return output_op(accum, source);
  }
  static CUTLASS_DEVICE typename Op::FragmentOutput apply(
      Op const& output_op,
      int row_id,
      typename Op::FragmentAccumulator const& accum) {
    return output_op(accum);
  }
};
```
**EN**: Default `ApplyEpilogueOp` adapter for output operators that do not need row-aware specialization.
**CN**: 默认 `ApplyEpilogueOp` 适配器，服务于不需要特殊按行逻辑的输出算子。

### Lines 83-223 / 第 83-223 行
```cpp

////////////////////////////////////////////////////////////////////////////////

/// Epilogue operator
template <
    typename Shape_, ///< Shape of threadblock tile (concept: GemmShape)
    typename WarpMmaOperator_, ///< Warp-level MMA operator (concept:
                               ///< gemm::warp::MmaTensorOp)
    int PartitionsK, ///< Number of partitions of the K dimension
    typename OutputTileIterator_, ///< Tile iterator writing output tensors
    typename AccumulatorFragmentIterator_, ///< Fragment iterator selecting
                                           ///< accumulators
    typename WarpTileIterator_, ///< Warp-scoped tile iterator writing
                                ///< accumulators to SMEM
    typename SharedLoadIterator_, ///< Threadblock-scoped tile iterator loading
                                  ///< from SMEM
    typename OutputOp_, ///< Output operator
    typename Padding_, ///< Padding added to SMEM allocation to avoid bank
                       ///< conflicts (concept: MatrixShape)
    int FragmentsPerPartition =
        1, ///< Used to coarsten the epilogue granularity
    int IterationsUnroll = ///< Used to reduce binary size when epilogue op is
                           ///< large
    (!IsEpilogueFunctorHeavy<OutputOp_>::value),
    typename OutputTileSourceIterator_ =
        OutputTileIterator_ ///< Tile iterator reading tensors
    >
class EpiloguePipelined : public EpilogueBase<
                              Shape_,
                              typename WarpMmaOperator_::Shape,
                              PartitionsK,
                              AccumulatorFragmentIterator_,
                              WarpTileIterator_,
                              Padding_,
                              FragmentsPerPartition> {
 public:
  using Base = EpilogueBase<
      Shape_,
      typename WarpMmaOperator_::Shape,
      PartitionsK,
      AccumulatorFragmentIterator_,
      WarpTileIterator_,
      Padding_,
      FragmentsPerPartition>;

  using Shape = Shape_;
  using WarpMmaOperator = WarpMmaOperator_;
  static int const kPartitionsK = PartitionsK;
  using OutputTileIterator = OutputTileIterator_;
  using OutputTileSourceIterator = OutputTileSourceIterator_;
  using AccumulatorFragmentIterator = AccumulatorFragmentIterator_;
  using WarpTileIterator = WarpTileIterator_;
  using SharedLoadIterator = SharedLoadIterator_;
  using OutputOp = OutputOp_;
  using Padding = Padding_;

  using Layout = layout::RowMajor;
  using LongIndex = typename Layout::LongIndex;

  /// The complete warp-level accumulator tile
  using AccumulatorTile = typename Base::AccumulatorTile;

  /// Accumulator element
  using ElementAccumulator = typename WarpTileIterator::Element;

  /// Output element
  using ElementOutput = typename OutputTileIterator::Element;
  using ElementSource = typename OutputTileSourceIterator::Element;

  /// Output access size
  static int const kElementsPerAccess = OutputTileIterator::kElementsPerAccess;

  /// Tensor reference to destination tensor
  using TensorRef = typename OutputTileIterator::TensorRef;

  /// Tensor reference to sync tensor
  using SyncTensorRef =
      typename cutlass::TensorRef<int, cutlass::layout::PackedVectorLayout>;

  /// Const tensor reference to source tensor
  using ConstTensorRef = typename OutputTileIterator::ConstTensorRef;

  /// Array type used to output
  using OutputAccessType = Array<
      typename OutputTileIterator::Element,
      OutputTileIterator::kElementsPerAccess>;
  using SourceAccessType = Array<
      typename OutputTileSourceIterator::Element,
      OutputTileSourceIterator::kElementsPerAccess>;

  /// Array type used by output functor
  using AccumulatorAccessType = Array<
      typename WarpTileIterator::Element,
      OutputTileIterator::kElementsPerAccess>;

  /// Number of warps
  using WarpCount = typename Base::WarpCount;

  static int constexpr kSmemTiles = Base::kFragmentsPerIteration > 1
      ? Base::kFragmentsPerIteration
      : kPartitionsK;
  static int constexpr kSmemPointerOffset =
      Base::SharedStorage::StorageShape::kCount / kSmemTiles;

 public:
  static_assert(
      OutputTileSourceIterator::Fragment::kElements ==
          OutputTileIterator::Fragment::kElements,
      "Mismatch between input tile and output tile iterator (kElements)");
  static_assert(
      OutputTileSourceIterator::kIterations == OutputTileIterator::kIterations,
      "Mismatch between input tile and output tile iterator (kIterations)");
  static_assert(
      SharedLoadIterator::Fragment::kElements ==
          OutputTileIterator::Fragment::kElements,
      "Mismatch between shared load iterator and output tile iterator.");

  static_assert(
      OutputTileIterator::kElementsPerAccess,
      "OutputTileIterator::kElementsPerAccess must not be zero.");

  static_assert(
      !(OutputTileIterator::Fragment::kElements %
        OutputTileIterator::kElementsPerAccess),
      "Divisibility");

 private:
  /// Loads fragment from shared memory aligned with output tensor
  SharedLoadIterator shared_load_iterator_;

 public:
  /// Constructor
  CUTLASS_DEVICE
  EpiloguePipelined(
      typename Base::SharedStorage& shared_storage, ///< Shared storage object
      int thread_idx, ///< ID of a thread within the threadblock
      int warp_idx, ///< ID of warp within threadblock
      int lane_idx ///< Id of thread within warp
      )
      : Base(shared_storage, thread_idx, warp_idx, lane_idx),
        shared_load_iterator_(shared_storage.reference(), thread_idx) {}
```
**EN**: `EpiloguePipelined` template declaration, type aliases, static traits, and constructor.
**CN**: `EpiloguePipelined` 模板声明、类型别名、静态 traits 与构造函数。

### Lines 224-224 / 第 224-224 行
```cpp

```
**EN**: Blank separator between neighboring logical sections.
**CN**: 相邻逻辑段之间的空行分隔。

### Lines 225-253 / 第 225-253 行
```cpp
  /// Streams the result to global memory
  CUTLASS_DEVICE
  void operator()(
      OutputOp const& output_op, ///< Output operator
      OutputTileIterator
          destination_iterator, ///< Tile iterator for destination
      AccumulatorTile const&
          accumulators, ///< Complete warp-level accumulator tile
      OutputTileSourceIterator
          source_iterator) { ///< Threadblock tile coordinate in GEMM (in units
                             ///< of threadblock tiles)

    if (!output_op.is_source_needed()) {
      compute_source_not_needed_(output_op, destination_iterator, accumulators);
    } else {
      compute_source_needed_(
          output_op, destination_iterator, accumulators, source_iterator);
    }
  }
  CUTLASS_DEVICE
  void operator()(
      OutputOp const& output_op, ///< Output operator
      OutputTileIterator
          destination_iterator, ///< Tile iterator for destination
      AccumulatorTile const&
          accumulators) { ///< Complete warp-level accumulator tile
    compute_source_not_needed_(output_op, destination_iterator, accumulators);
  }
```
**EN**: Public `operator()` overloads that dispatch to source-needed or source-not-needed paths.
**CN**: 公共 `operator()` 重载，根据是否需要 source 数据分派到不同实现。

### Lines 254-397 / 第 254-397 行
```cpp
 private:
  template <class Seq>
  struct acc2smem_source_not_needed;

  template <size_t... Seq>
  struct acc2smem_source_not_needed<cutlass::index_sequence<Seq...>> {
    template <int Advance>
    CUTLASS_DEVICE static void helper(
        AccumulatorFragmentIterator accum_fragment_iterator,
        WarpTileIterator& warp_tile_iterator) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < Advance; i++) {
        ++accum_fragment_iterator;
      }

      CUTLASS_PRAGMA_UNROLL
      for (int p = 0; p < Base::kFragmentsPerIteration; ++p) {
        typename AccumulatorFragmentIterator::Fragment accum_fragment;

        accum_fragment_iterator.load(accum_fragment);
        ++accum_fragment_iterator;

        warp_tile_iterator.store(accum_fragment);
        if (p < Base::kFragmentsPerIteration - 1) {
          warp_tile_iterator.add_pointer_offset(kSmemPointerOffset);
        }
      }

      if (Base::kFragmentsPerIteration > 1) {
        warp_tile_iterator.add_pointer_offset(
            kSmemPointerOffset * (1 - Base::kFragmentsPerIteration));
      }
    }

    CUTLASS_DEVICE
    static void push(
        size_t pos,
        AccumulatorFragmentIterator const& iterator_begin,
        WarpTileIterator& warp_tile_iterator) {
      int dummy[] = {
          (pos == (Seq * Base::kFragmentsPerIteration)) &&
          (helper<Seq * Base::kFragmentsPerIteration>(
               iterator_begin, warp_tile_iterator),
           0)...};

      CUTLASS_UNUSED(dummy[0]);
    }
  };

  static_assert(
      kPartitionsK == 1 || Base::kFragmentsPerIteration == 1,
      "One of these must be exactly 1.");

  /// Streams the result to global memory
  CUTLASS_DEVICE
  void compute_source_not_needed_(
      OutputOp const& output_op, ///< Output operator
      OutputTileIterator
          destination_iterator, ///< Tile iterator for destination
      AccumulatorTile const&
          accumulators ///< Complete warp-level accumulator tile
  ) {
    //
    // Iterator over warp-level accumulator fragment
    //

    AccumulatorFragmentIterator accum_fragment_iterator(accumulators);

    //
    // Iterate over accumulator tile
    //

#pragma unroll(                                                          \
    IterationsUnroll                                                     \
        ? OutputTileIterator::kIterations / Base::kFragmentsPerIteration \
        : 1)
    for (int iter = 0; iter < OutputTileIterator::kIterations;
         iter += Base::kFragmentsPerIteration) {
      //
      // Convert and store fragment
      //

      __syncthreads();

      acc2smem_source_not_needed<cutlass::make_index_sequence<
          OutputTileIterator::kIterations / Base::kFragmentsPerIteration>>::
          push(iter, accum_fragment_iterator, this->warp_tile_iterator_);

      __syncthreads();

      //
      // Load fragments from shared memory
      //

      CUTLASS_PRAGMA_UNROLL
      for (int p = 0; p < Base::kFragmentsPerIteration; ++p) {
        typename SharedLoadIterator::Fragment
            aligned_accum_fragment[kPartitionsK];

        shared_load_iterator_.load(aligned_accum_fragment[0]);

        if (p < Base::kFragmentsPerIteration - 1) {
          shared_load_iterator_.add_pointer_offset(kSmemPointerOffset);
        } else if (kPartitionsK > 1) {
          plus<typename SharedLoadIterator::Fragment> add_fragments;

          CUTLASS_PRAGMA_UNROLL
          for (int i = 1; i < kPartitionsK; ++i) {
            shared_load_iterator_.add_pointer_offset(kSmemPointerOffset);
            shared_load_iterator_.load(aligned_accum_fragment[i]);
            aligned_accum_fragment[0] = add_fragments(
                aligned_accum_fragment[0], aligned_accum_fragment[i]);
          }

          shared_load_iterator_.add_pointer_offset(
              (1 - kPartitionsK) * kSmemPointerOffset);
        }

        //
        // Compute the output result
        //

        typename OutputTileIterator::Fragment output_fragment;

        apply_output_operator_source_not_needed_(
            destination_iterator.thread_start_row(),
            output_fragment,
            output_op,
            aligned_accum_fragment[0]);

        //
        // Store the final result
        //

        destination_iterator.store(output_fragment);
        ++destination_iterator;
      }

      if (Base::kFragmentsPerIteration > 1) {
        shared_load_iterator_.add_pointer_offset(
            kSmemPointerOffset * (1 - Base::kFragmentsPerIteration));
      }
    }
  }
```
**EN**: `compute_source_not_needed_()` and helper machinery for the pure accumulator-to-output path.
**CN**: `compute_source_not_needed_()` 及其辅助逻辑：处理“只看累加器”的输出路径。

### Lines 398-428 / 第 398-428 行
```cpp

  template <class Seq>
  struct acc2smem_source_needed;

  template <size_t... Seq>
  struct acc2smem_source_needed<cutlass::index_sequence<Seq...>> {
    template <int Advance>
    CUTLASS_DEVICE static void helper(
        AccumulatorFragmentIterator accum_fragment_iterator,
        WarpTileIterator& warp_tile_iterator) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < Advance; i++) {
        ++accum_fragment_iterator;
      }

      typename AccumulatorFragmentIterator::Fragment accum_fragment;
      accum_fragment_iterator.load(accum_fragment);
      warp_tile_iterator.store(accum_fragment);
    }

    CUTLASS_DEVICE
    static void push(
        size_t pos,
        AccumulatorFragmentIterator const& iterator_begin,
        WarpTileIterator& warp_tile_iterator) {
      int dummy[] = {
          (pos == Seq) &&
          (helper<Seq>(iterator_begin, warp_tile_iterator), 0)...};
    }
  };
```
**EN**: Accumulator-to-shared-memory helper for the source-needed path.
**CN**: 面向 source-needed 路径的“累加器写共享内存”辅助逻辑。

### Lines 429-523 / 第 429-523 行
```cpp
  /// Streams the result to global memory
  CUTLASS_DEVICE
  void compute_source_needed_(
      OutputOp const& output_op, ///< Output operator
      OutputTileIterator
          destination_iterator, ///< Tile iterator for destination
      AccumulatorTile const&
          accumulators, ///< Complete warp-level accumulator tile
      OutputTileSourceIterator
          source_iterator ///< Threadblock tile coordinate in GEMM (in units of
                          ///< threadblock tiles)
  ) {
    typename OutputTileSourceIterator::Fragment source_fragment[2];

    source_fragment[0].clear();
    source_iterator.load(source_fragment[0]);
    ++source_iterator;
    source_fragment[1].clear();

    //
    // Iterator over warp-level accumulator fragment
    //

    AccumulatorFragmentIterator accum_fragment_iterator(accumulators);

    //
    // Iterate over accumulator tile
    //

#pragma unroll(IterationsUnroll ? OutputTileIterator::kIterations : 1)
    for (int iter = 0; iter < OutputTileIterator::kIterations; ++iter) {
      if (iter > 0) {
        __syncthreads();
      }
      //
      // Load the source for next iteration (pipelining)
      //

      if (iter + 1 < OutputTileIterator::kIterations) {
        source_iterator.load(source_fragment[(iter + 1) % 2]);
      }
      ++source_iterator;
      acc2smem_source_needed<
          cutlass::make_index_sequence<OutputTileIterator::kIterations>>::
          push(iter, accum_fragment_iterator, this->warp_tile_iterator_);

      __syncthreads();

      //
      // Load fragments from shared memory
      //

      typename SharedLoadIterator::Fragment
          aligned_accum_fragment[kPartitionsK];

      shared_load_iterator_.load(aligned_accum_fragment[0]);

      // If the number of k-slices is > 1 - perform a reduction amongst the
      // k-slices
      if (kPartitionsK > 1) {
        plus<typename SharedLoadIterator::Fragment> add_fragments;

        CUTLASS_PRAGMA_UNROLL
        for (int i = 1; i < kPartitionsK; ++i) {
          shared_load_iterator_.add_pointer_offset(kSmemPointerOffset);
          shared_load_iterator_.load(aligned_accum_fragment[i]);
          aligned_accum_fragment[0] = add_fragments(
              aligned_accum_fragment[0], aligned_accum_fragment[i]);
        }

        shared_load_iterator_.add_pointer_offset(
            (1 - kPartitionsK) * kSmemPointerOffset);
      }

      //
      // Compute the output result
      //

      typename OutputTileIterator::Fragment output_fragment;

      apply_output_operator_(
          destination_iterator.thread_start_row(),
          output_fragment,
          output_op,
          aligned_accum_fragment[0],
          source_fragment[iter % 2]);

      //
      // Store the final result
      //

      destination_iterator.store(output_fragment);
      ++destination_iterator;
    }
  }
```
**EN**: `compute_source_needed_()` with pipelined source loads, shared-memory reads, optional K-partition reduction, output-op application, and stores.
**CN**: `compute_source_needed_()`：包含 source 预取、共享内存读取、可选 K-partition 归约、输出算子应用与最终存储。

### Lines 524-580 / 第 524-580 行
```cpp

  /// Helper to invoke the output functor over each vector of output
  CUTLASS_DEVICE
  void apply_output_operator_(
      int begin_row,
      typename OutputTileIterator::Fragment& output_fragment,
      OutputOp const& output_op, ///< Output operator
      typename SharedLoadIterator::Fragment const& aligned_accum_fragment,
      typename OutputTileSourceIterator::Fragment const& source_fragment) {
    OutputAccessType* output_frag_ptr =
        reinterpret_cast<OutputAccessType*>(&output_fragment);

    AccumulatorAccessType const* compute_frag_ptr =
        reinterpret_cast<AccumulatorAccessType const*>(&aligned_accum_fragment);

    SourceAccessType const* source_frag_ptr =
        reinterpret_cast<SourceAccessType const*>(&source_fragment);

    int const kOutputOpIterations = OutputTileIterator::Fragment::kElements /
        OutputTileIterator::kElementsPerAccess;

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kOutputOpIterations; ++i) {
      // Call the output operator
      output_frag_ptr[i] = ApplyEpilogueOp<OutputOp>::apply(
          output_op,
          begin_row + getRowOffset(i * OutputTileIterator::kElementsPerAccess),
          compute_frag_ptr[i],
          source_frag_ptr[i]);
    }
  }

  /// Helper to invoke the output functor over each vector of output
  CUTLASS_DEVICE
  void apply_output_operator_source_not_needed_(
      int begin_row,
      typename OutputTileIterator::Fragment& output_fragment,
      OutputOp const& output_op, ///< Output operator
      typename SharedLoadIterator::Fragment const& aligned_accum_fragment) {
    OutputAccessType* output_frag_ptr =
        reinterpret_cast<OutputAccessType*>(&output_fragment);

    AccumulatorAccessType const* compute_frag_ptr =
        reinterpret_cast<AccumulatorAccessType const*>(&aligned_accum_fragment);

    int const kOutputOpIterations = OutputTileIterator::Fragment::kElements /
        OutputTileIterator::kElementsPerAccess;

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kOutputOpIterations; ++i) {
      // Call the output operator
      output_frag_ptr[i] = ApplyEpilogueOp<OutputOp>::apply(
          output_op,
          begin_row + getRowOffset(i * OutputTileIterator::kElementsPerAccess),
          compute_frag_ptr[i]);
    }
  }
```
**EN**: Helpers that vectorize the output-operator call over fragment lanes, optionally including source fragments.
**CN**: 辅助函数：按向量访问宽度遍历 fragment，并调用输出算子；可选择是否携带 source fragment。

### Lines 581-613 / 第 581-613 行
```cpp

  // This should be constexpr, but it's only supported on c++14
  static int CUTLASS_HOST_DEVICE getRowOffset(int i) {
    using ThreadMap = typename OutputTileIterator::ThreadMap;

    CUTLASS_PRAGMA_UNROLL
    for (int cluster = 0; cluster < ThreadMap::Iterations::kCluster;
         ++cluster) {
      CUTLASS_PRAGMA_UNROLL
      for (int group = 0; group < ThreadMap::Iterations::kGroup; ++group) {
        CUTLASS_PRAGMA_UNROLL
        for (int row = 0; row < ThreadMap::Iterations::kRow; ++row) {
          int row_offset = row * ThreadMap::Delta::kRow +
              group * ThreadMap::Delta::kGroup +
              cluster * ThreadMap::Delta::kCluster;
          int frag_row_idx =
              (row +
               ThreadMap::Iterations::kRow *
                   (group + ThreadMap::Iterations::kGroup * cluster));
          CUTLASS_PRAGMA_UNROLL
          for (int column = 0; column < ThreadMap::Iterations::kColumn;
               ++column) {
            int frag_idx = ThreadMap::kElementsPerAccess *
                (frag_row_idx * ThreadMap::Iterations::kColumn + column);
            if (i < frag_idx + ThreadMap::kElementsPerAccess) {
              return row_offset;
            }
          }
        }
      }
    }
    return -1;
  }
```
**EN**: `getRowOffset()` reconstructs logical row offsets from the thread map.
**CN**: `getRowOffset()` 根据 thread map 反推出逻辑行偏移。

### Lines 614-622 / 第 614-622 行
```cpp
};

////////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace epilogue
} // namespace cutlass

////////////////////////////////////////////////////////////////////////////////
```
**EN**: Namespace closure.
**CN**: 命名空间收尾。

---
## Key Concepts / 关键概念
- **EN:** FMHA uses online normalization, so the epilogue is not just a final cast/store stage; it may need previous output and row-wise normalization factors.
  **CN:** FMHA 使用在线归一化，因此 epilogue 不只是最终的类型转换与写回阶段；它还可能需要读取旧输出以及按行的归一化系数。
- **EN:** The source-needed path overlaps source loading with accumulator staging, which is why the file is called “pipelined”.
  **CN:** source-needed 路径会把 source 读取与累加器搬运重叠起来，这正是文件名中 “pipelined” 的来源。
- **EN:** `getRowOffset()` is the FMHA-critical customization: row-wise `m'` and `s'` values only make sense if each vector lane is mapped back to the correct logical query row.
  **CN:** `getRowOffset()` 是 FMHA 里的关键定制点：只有把每个向量 lane 映射回正确的逻辑 query 行，按行维护的 `m'` 与 `s'` 才有意义。
- **EN:** The class preserves standard CUTLASS iterator composition: accumulator fragment iterator → warp tile iterator → shared-load iterator → output tile iterator.
  **CN:** 这个类保留了标准 CUTLASS 的 iterator 组合方式：累加器 fragment iterator → warp tile iterator → shared-load iterator → 输出 tile iterator。
- **EN:** `fmha_grouped.h` instantiates this epilogue around MM1 so each value-aggregation tile can be rescaled and written correctly, whether the tile is first, intermediate, or last in the K sweep.
  **CN:** `fmha_grouped.h` 会围绕 MM1 实例化该 epilogue，使每个 value 聚合 tile 都能在 K 维扫描的首块、中间块或尾块情况下被正确重标定并写回。
- **EN:** It is the bridge between CUTLASS GEMM output fragments and FMHA-specific row-wise normalization semantics.
  **CN:** 它是 CUTLASS GEMM 输出 fragment 与 FMHA 按行归一化语义之间的桥梁。
## Dependencies / 依赖项
- `"cutlass/aligned_buffer.h"` — aligned shared-memory/storage buffers / 对齐的共享内存 / 存储缓冲区
- `"cutlass/array.h"` — fixed-size array containers used for fragments / 用于 fragment 的定长数组容器
- `"cutlass/cutlass.h"` — CUTLASS core definitions and portability macros / CUTLASS 核心定义与可移植性宏
- `CUDA_STD_HEADER(cassert)` — assert support routed through CUTLASS CUDA portability macros / 通过 CUTLASS CUDA 可移植宏引入的断言支持
- `"cutlass/functional.h"` — numeric functors and elementwise operator helpers / 数值 functor 与逐元素算子辅助
- `"cutlass/layout/tensor.h"` — tensor layout primitives used by epilogue iterators / epilogue iterator 使用的张量布局原语
- `"cutlass/layout/vector.h"` — vector layout helpers for epilogue fragments / epilogue fragment 使用的向量布局辅助
- `"cutlass/numeric_types.h"` — CUTLASS scalar and numeric utility types / CUTLASS 标量与数值工具类型
- `"cutlass/tensor_coord.h"` — tensor coordinate types for tile addressing / 用于 tile 寻址的张量坐标类型
- `"cutlass/gemm/gemm.h"` — CUTLASS GEMM shapes, coordinates, and core GEMM types / CUTLASS GEMM 形状、坐标与核心 GEMM 类型
- `"cutlass/transform/pitch_linear_thread_map.h"` — thread maps that distribute vector accesses across threads / 在线程间分配向量访问的 thread map
- `"cutlass/transform/threadblock/regular_tile_iterator.h"` — threadblock iterators for regular shared-memory tile access / 用于常规共享内存 tile 访问的 threadblock iterator
- `"cutlass/epilogue/threadblock/epilogue_base.h"` — base epilogue scaffolding for threadblock output pipelines / 线程块输出流水的 epilogue 基类框架
- `"cutlass/epilogue/threadblock/predicated_tile_iterator.h"` — predicated output iterators used during epilogue writeback / epilogue 写回阶段使用的 predicated 输出 iterator
