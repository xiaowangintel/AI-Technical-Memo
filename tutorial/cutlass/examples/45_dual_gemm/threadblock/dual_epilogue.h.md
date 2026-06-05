# dual_epilogue.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/45_dual_gemm/threadblock/dual_epilogue.h`  
**Purpose / 用途**: Threadblock epilogue that emits D0, D1, and fused D2 This header is the “writeback and final fusion” stage of the fused kernel. It receives two accumulator tiles from the mainloop, converts them through the ordinary per-GEMM epilogues, and then applies a third output operator to those already-epilogued fragments to produce the final fused tensor `D2`. / 输出 D0、D1 与融合 D2 的 threadblock epilogue 这个头文件实现了融合 kernel 的“写回与最终融合”阶段。它从 mainloop 接收两份累加器 tile，先分别走普通的单 GEMM epilogue，再对这些已经过 epilogue 的结果应用第三个输出算子，生成最终融合张量 `D2`。

---

## Line-by-Line Analysis / 逐行分析

### Logical Block 1 / 逻辑块 1 — lines 31-57

```cpp
/*! \file
  \brief Epilogue for threadblock scoped GEMMs using Tensor Ops.

  The epilogue rearranges the result of a matrix product through shared memory to match canonical
  tensor layouts in global memory. Epilogues support conversion and reduction operations.

*/

#pragma once
#include "cutlass/array.h"
#include CUDA_STD_HEADER(cassert)
#include "cutlass/cutlass.h"
#include "cutlass/numeric_types.h"
#include "cutlass/layout/vector.h"
#include "cutlass/layout/tensor.h"
#include "cutlass/tensor_coord.h"
#include "cutlass/aligned_buffer.h"
#include "cutlass/functional.h"

#include "cutlass/gemm/gemm.h"

#include "cutlass/transform/pitch_linear_thread_map.h"
#include "cutlass/transform/threadblock/regular_tile_iterator.h"

#include "cutlass/epilogue/threadblock/epilogue_base.h"
#include "cutlass/epilogue/threadblock/predicated_tile_iterator.h"
#include "cutlass/numeric_types.h"
```

**EN**: The opening comments summarize the standard CUTLASS epilogue mission: rearrange accumulator fragments through shared memory into a layout suitable for global-memory stores, while allowing conversion and reduction. The custom dual version keeps that mission but duplicates it for two output streams.
**CN**: 开头注释概括了标准 CUTLASS epilogue 的任务：把累加器 fragment 通过共享内存重排成适合写回全局内存的布局，同时允许类型转换与归约。这个自定义的双路版本保留了这一使命，只是把流程扩展到了两条输出流。

### Logical Block 2 / 逻辑块 2 — lines 68-177

```cpp
template <
  typename Shape_,                          ///< Shape of threadblock tile (concept: GemmShape)
  typename WarpMmaOperator_,                ///< Warp-level MMA operator (concept: gemm::warp::MmaTensorOp)
  int PartitionsK,                          ///< Number of partitions of the K dimension
  typename OutputTileIterator_,             ///< Tile iterator reading and writing output tensors
  typename AccumulatorFragmentIterator_,    ///< Fragment iterator selecting accumulators
  typename WarpTileIterator_,               ///< Warp-scoped tile iterator writing accumulators to SMEM
  typename SharedLoadIterator_,             ///< Threadblock-scoped tile iterator loading from SMEM
  ///< Output operator
  typename OutputOp0_,
  typename OutputOp1_,
  typename OutputOp2_,
  typename Padding_,                        ///< Padding added to SMEM allocation to avoid bank conflicts (concept: MatrixShape)
  bool StoreD0 = true,
  bool StoreD1 = true,
  int FragmentsPerPartition = 1,            ///< Used to coarsten the epilogue granularity
  int IterationsUnroll =                    ///< Used to reduce binary size when epilogue op is large
    (!IsEpilogueFunctorHeavy<OutputOp0_>::value)
>
class DualEpilogue {

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
  static bool constexpr kStoreD0 = StoreD0;
  static bool constexpr kStoreD1 = StoreD1;
  using OutputTileIterator = OutputTileIterator_;
  using AccumulatorFragmentIterator = AccumulatorFragmentIterator_;
  using WarpTileIterator = WarpTileIterator_;
  using SharedLoadIterator = SharedLoadIterator_;
  using OutputOp0 = OutputOp0_;
  using OutputOp1 = OutputOp1_;
  using OutputOp2 = OutputOp2_;
  using Padding = Padding_;

  using Layout = layout::RowMajor;
  using LongIndex = typename Layout::LongIndex;

  /// The complete warp-level accumulator tile
  using AccumulatorTile = typename Base::AccumulatorTile;

  /// Accumulator element
  using ElementAccumulator = typename WarpTileIterator::Element;

  /// Output element
  using ElementOutput = typename OutputTileIterator::Element;

  /// Output access size
  static int const kElementsPerAccess = OutputTileIterator::kElementsPerAccess;

  /// Tensor reference to destination tensor
  using TensorRef = typename OutputTileIterator::TensorRef;

  /// Tensor reference to sync tensor
  using SyncTensorRef = typename cutlass::TensorRef<int, cutlass::layout::PackedVectorLayout>;

  /// Const tensor reference to source tensor
  using ConstTensorRef = typename OutputTileIterator::ConstTensorRef;

  /// Array type used to output
  using OutputAccessType = Array<
    typename OutputTileIterator::Element, OutputTileIterator::kElementsPerAccess>;

  /// Array type used by output functor
  using AccumulatorAccessType = Array<typename WarpTileIterator::Element, OutputTileIterator::kElementsPerAccess>; 
  
  /// Number of warps
  using WarpCount = typename Base::WarpCount;

  struct SharedStorage {
    using Element = typename WarpTileIterator::Element;

    /// Tensor reference to shared memory allocation
    using TensorRef = typename WarpTileIterator::TensorRef;

    /// Logical shape of the shared memory tile written to by all warps.
    using Shape = typename Base::Shape;

    /// Shape of the shared memory allocation for the epilogue    
    using StorageShape = typename Base::SharedStorage::StorageShape;

    //
    // Data members
    //

    AlignedBuffer<Element, StorageShape::kCount> storage[2];

    //
    // Methods
    //

    /// Returns a tensor reference to the shared memory buffer
    CUTLASS_DEVICE
    TensorRef reference(int i) {
      return TensorRef(
        storage[i].data(), 
        Layout::packed({StorageShape::kRow, StorageShape::kColumn}));
    }
  };
```

**EN**: The template parameters spell out everything the epilogue needs from the surrounding kernel: tile shape, warp MMA operator, K partition count, output iterators, accumulator iterators, shared-load iterators, two ordinary output ops, one fused output op, padding policy, and storage flags. `SharedStorage` then allocates two aligned shared-memory tiles, one for the D0 stream and one for the D1 stream.
**CN**: 模板参数明确列出了 epilogue 依赖的全部外围信息：tile 形状、warp MMA 算子、K 分区数、输出迭代器、累加器迭代器、共享内存加载迭代器、两个普通输出算子、一个融合输出算子、padding 策略以及存储标志。随后 `SharedStorage` 分配两块对齐的共享内存 tile，分别服务于 D0 与 D1 两条输出流。

### Logical Block 3 / 逻辑块 3 — lines 184-226

```cpp
  static_assert(SharedLoadIterator::Fragment::kElements == OutputTileIterator::Fragment::kElements,
    "Mismatch between shared load iterator and output tile iterator.");

  static_assert(OutputTileIterator::kElementsPerAccess, "OutputTileIterator::kElementsPerAccess must not be zero.");

  static_assert(!(OutputTileIterator::Fragment::kElements % OutputTileIterator::kElementsPerAccess), 
    "Divisibility");

private:

  /// Loads fragment from shared memory aligned with output tensor
  SharedLoadIterator shared_load_iterator0_;
  SharedLoadIterator shared_load_iterator1_;

  /// Stores a warp's fragment of accumulators to SMEM
  WarpTileIterator warp_tile_iterator0_;
  WarpTileIterator warp_tile_iterator1_;

public:

  /// Constructor
  CUTLASS_DEVICE
  DualEpilogue(
    SharedStorage &shared_storage,    ///< Shared storage object    
    int thread_idx,                   ///< ID of a thread within the threadblock
    int warp_idx,                     ///< ID of warp within threadblock
    int lane_idx                     ///< Id of thread within warp
  ):
    shared_load_iterator0_(shared_storage.reference(0), thread_idx),
    shared_load_iterator1_(shared_storage.reference(1), thread_idx),
    warp_tile_iterator0_(shared_storage.reference(0), lane_idx),
    warp_tile_iterator1_(shared_storage.reference(1), lane_idx)
  {
    int warp_k = warp_idx / (WarpCount::kM * WarpCount::kN);
    int warp_mn = warp_idx % (WarpCount::kM * WarpCount::kN);
    int warp_m = warp_mn % WarpCount::kM;
    int warp_n = warp_mn / WarpCount::kM;

    MatrixCoord warp_offset{warp_k * WarpCount::kM + warp_m, warp_n};

    warp_tile_iterator0_.add_tile_offset(warp_offset);
    warp_tile_iterator1_.add_tile_offset(warp_offset);
  }
```

**EN**: The static assertions guarantee iterator compatibility, while the constructor maps each warp to its shared-memory tile offset. Because the dual epilogue owns two warp tile iterators and two shared-load iterators, the warp-coordinate math must stay perfectly synchronized across both output paths.
**CN**: 静态断言保证了各类迭代器之间的兼容性，而构造函数则把每个 warp 映射到对应的共享内存 tile 偏移。由于 dual epilogue 同时维护两套 warp tile iterator 和两套 shared-load iterator，因此 warp 坐标计算必须在两条输出路径上保持完全同步。

### Logical Block 4 / 逻辑块 4 — lines 230-343

```cpp
  void operator()(
    OutputOp0 const &output_op0,
    OutputOp1 const &output_op1,
    OutputOp2 const &output_op2,
    OutputTileIterator dest0,
    OutputTileIterator dest1,
    OutputTileIterator dest2,
    AccumulatorTile const &accumulator0,
    AccumulatorTile const &accumulator1,
    OutputTileIterator source_iterator[2],
    bool writeToD2 // true if it's the final split-k
  ) {
    // TODO: Implement when no source is needed

    typename OutputTileIterator::Fragment source_fragment[2];
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < 2; ++i) {
      source_fragment[i].clear();
    }

    //
    // Iterator over warp-level accumulator fragment
    //

    AccumulatorFragmentIterator accum_fragment_iterator[2] = {accumulator0, accumulator1};

    //
    // Iterate over accumulator tile
    // 

    #pragma unroll(IterationsUnroll ? OutputTileIterator::kIterations : 1)
    for (int iter = 0; iter < OutputTileIterator::kIterations; ++iter) {

      //
      // Load the source
      //

      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < 2; ++i) {
        source_iterator[i].load(source_fragment[i]);
        ++source_iterator[i];
      }

      //
      // Convert and store fragment
      //
      
      __syncthreads();

      acc2smem_source_needed<cutlass::make_index_sequence<OutputTileIterator::kIterations>>::push(
          iter, accum_fragment_iterator[0], this->warp_tile_iterator0_);
      acc2smem_source_needed<cutlass::make_index_sequence<OutputTileIterator::kIterations>>::push(
          iter, accum_fragment_iterator[1], this->warp_tile_iterator1_);

      __syncthreads();

      //
      // Load fragments from shared memory
      //

      typename SharedLoadIterator::Fragment aligned_accum_fragment0[kPartitionsK];
      typename SharedLoadIterator::Fragment aligned_accum_fragment1[kPartitionsK];

      shared_load_iterator0_.load(aligned_accum_fragment0[0]);
      shared_load_iterator1_.load(aligned_accum_fragment1[0]);

      // If the number of k-slices is > 1 - perform a reduction amongst the k-slices
      if (kPartitionsK > 1) {

        plus <typename SharedLoadIterator::Fragment> add_fragments;

        CUTLASS_PRAGMA_UNROLL
        for ( int i = 1; i < kPartitionsK; ++i) {
          shared_load_iterator0_.add_pointer_offset(kSmemPointerOffset);
          shared_load_iterator1_.add_pointer_offset(kSmemPointerOffset);
          shared_load_iterator0_.load(aligned_accum_fragment0[i]);
          shared_load_iterator1_.load(aligned_accum_fragment1[i]);
          aligned_accum_fragment0[0] = add_fragments(aligned_accum_fragment0[0], aligned_accum_fragment0[i]);
          aligned_accum_fragment1[0] = add_fragments(aligned_accum_fragment1[0], aligned_accum_fragment1[i]);
        }

        shared_load_iterator0_.add_pointer_offset((1 - kPartitionsK) * kSmemPointerOffset);
        shared_load_iterator1_.add_pointer_offset((1 - kPartitionsK) * kSmemPointerOffset);
      }

      //
      // Compute the output result
      //
     
      typename OutputTileIterator::Fragment output_fragment[3];

      apply_output_operator_(output_fragment,
        output_op0, output_op1, output_op2,
        aligned_accum_fragment0[0], aligned_accum_fragment1[0],
        source_fragment);


      //
      // Store the final result
      //

      if (kStoreD0) {
        dest0.store(output_fragment[0]);
        ++dest0;
      }
      if (kStoreD1) {
        dest1.store(output_fragment[1]);
        ++dest1;
      }
      if (writeToD2) {
        dest2.store(output_fragment[2]);
        ++dest2;
      }
    }
```

**EN**: The main `operator()` loads source fragments for both outputs, writes both accumulator tiles into shared memory, reloads them in output-aligned form, optionally reduces across K partitions, applies output operators, and conditionally stores D0, D1, and D2. The sequencing matters: `OutputOp2` is applied *after* `OutputOp0` and `OutputOp1`, so D2 is defined in terms of the finalized D0/D1 values rather than raw accumulators.
**CN**: 主 `operator()` 会先读取两路源 fragment，把两份累加器 tile 都写入共享内存，再以输出对齐的形式回读，必要时在 K 分区之间完成归约，应用输出算子，并按条件存储 D0、D1、D2。这里的顺序非常关键：`OutputOp2` 是在 `OutputOp0` 和 `OutputOp1` 之后应用的，因此 D2 的定义依赖的是最终版 D0/D1，而不是原始累加器。

### Logical Block 5 / 逻辑块 5 — lines 348-415

```cpp
  static_assert(kPartitionsK == 1 || Base::kFragmentsPerIteration == 1, "One of these must be exactly 1.");

  template<class Seq>
  struct acc2smem_source_needed;

  template <size_t... Seq>
  struct acc2smem_source_needed<cutlass::index_sequence<Seq...>> {
    template<int Advance>
    CUTLASS_DEVICE
    static void helper(AccumulatorFragmentIterator accum_fragment_iterator,
                       WarpTileIterator &warp_tile_iterator) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < Advance; i++) {
        ++accum_fragment_iterator;
      }

      typename AccumulatorFragmentIterator::Fragment accum_fragment;
      accum_fragment_iterator.load(accum_fragment);
      warp_tile_iterator.store(accum_fragment);
    }

    CUTLASS_DEVICE
    static void push(size_t pos,
                     AccumulatorFragmentIterator const &iterator_begin,
                     WarpTileIterator &warp_tile_iterator) {
      int dummy[] = {(pos == Seq) && (helper<Seq>(iterator_begin, warp_tile_iterator), 0)...};
    }
  };

  /// Helper to invoke the output functor over each vector of output
  CUTLASS_DEVICE
  void apply_output_operator_(
    typename OutputTileIterator::Fragment (&output_fragment)[3],
    OutputOp0 const &output_op0,
    OutputOp1 const &output_op1,
    OutputOp2 const &output_op2,
    typename SharedLoadIterator::Fragment const& aligned_accum_fragment0,
    typename SharedLoadIterator::Fragment const& aligned_accum_fragment1,
    typename OutputTileIterator::Fragment const (&source_fragment)[2]) {
      
    OutputAccessType* output_frag_ptr[3] = {
      reinterpret_cast<OutputAccessType *>(&output_fragment[0]),
      reinterpret_cast<OutputAccessType *>(&output_fragment[1]),
      reinterpret_cast<OutputAccessType *>(&output_fragment[2])
    };

    AccumulatorAccessType const *compute_frag_ptr[2] = {
      reinterpret_cast<AccumulatorAccessType const *>(&aligned_accum_fragment0),
      reinterpret_cast<AccumulatorAccessType const *>(&aligned_accum_fragment1)
    };

    OutputAccessType const *source_frag_ptr[2] = {
      reinterpret_cast<OutputAccessType const *>(&source_fragment[0]),
      reinterpret_cast<OutputAccessType const *>(&source_fragment[1])
    };

    int const kOutputOpIterations = 
      OutputTileIterator::Fragment::kElements / OutputTileIterator::kElementsPerAccess;

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kOutputOpIterations; ++i) {

      // Call the output operators
      output_frag_ptr[0][i] = output_op0(compute_frag_ptr[0][i], source_frag_ptr[0][i]);
      output_frag_ptr[1][i] = output_op1(compute_frag_ptr[1][i], source_frag_ptr[1][i]);
      output_frag_ptr[2][i] = output_op2(output_frag_ptr[0][i], output_frag_ptr[1][i]);
    }
  }
```

**EN**: The helper template `acc2smem_source_needed` is a compile-time dispatcher that advances an accumulator fragment iterator to the correct logical position, loads that fragment, and stores it through the warp tile iterator. `apply_output_operator_()` then reinterprets fragments as vector-access chunks and performs three operations in lockstep: `D0 = op0(acc0, src0)`, `D1 = op1(acc1, src1)`, `D2 = op2(D0, D1)`.
**CN**: 辅助模板 `acc2smem_source_needed` 是一个编译期分发器：它把累加器 fragment 迭代器推进到正确的逻辑位置，加载该 fragment，再通过 warp tile iterator 写入共享内存。随后 `apply_output_operator_()` 会把 fragment 重新解释为向量访问块，并锁步完成三次操作：`D0 = op0(acc0, src0)`、`D1 = op1(acc1, src1)`、`D2 = op2(D0, D1)`。

### Logical Block 6 / 逻辑块 6 — lines 416-424

```cpp
};

////////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace epilogue
} // namespace cutlass

////////////////////////////////////////////////////////////////////////////////
```

**EN**: The closing block is small, but conceptually it seals the fused writeback stage: once control returns to the kernel wrapper, all three logical outputs have already been materialized according to storage policy.
**CN**: 结尾虽然很短，但从概念上标记了融合写回阶段的结束：控制返回 kernel wrapper 时，三个逻辑输出已经依据存储策略完成了物化。

---

## Key Concepts / 关键概念
**EN**: `kernel::DualGemm` owns the call site, `DualMmaMultistage` produces `accumulator0/1`, and `OutputOp2` defines how the post-epilogue `D0` and `D1` fragments combine.
**CN**: 组合关系：`kernel::DualGemm` 负责调用位置，`DualMmaMultistage` 产生 `accumulator0/1`，而 `OutputOp2` 决定经过 epilogue 后的 `D0`、`D1` fragment 如何组合。

**EN**: Custom epilogue behavior in one line: first finish each GEMM’s own output op, then fuse those two results into `D2`.
**CN**: 自定义 epilogue 的一句话概括：先完成两个 GEMM 各自的输出变换，再把这两个结果融合成 `D2`。

## Dependencies / 依赖项
**EN**: CUTLASS dependency: `cutlass/array.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/array.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/cutlass.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/cutlass.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/numeric_types.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/numeric_types.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/layout/vector.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/layout/vector.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/layout/tensor.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/layout/tensor.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/tensor_coord.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/tensor_coord.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/aligned_buffer.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/aligned_buffer.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/functional.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/functional.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/gemm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/transform/pitch_linear_thread_map.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/transform/pitch_linear_thread_map.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/transform/threadblock/regular_tile_iterator.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/transform/threadblock/regular_tile_iterator.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/epilogue/threadblock/epilogue_base.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/epilogue/threadblock/epilogue_base.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/epilogue/threadblock/predicated_tile_iterator.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/epilogue/threadblock/predicated_tile_iterator.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/numeric_types.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/numeric_types.h` 提供该文件直接使用的库级原语。
