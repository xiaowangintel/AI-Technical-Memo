# dual_mma_multistage.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/45_dual_gemm/threadblock/dual_mma_multistage.h`  
**Purpose / 用途**: Multistage fused threadblock MMA pipeline This header contains the deepest performance-critical logic of the example. It is a dual-stream extension of CUTLASS multistage tensor-op MMA: one A stream is shared, two B streams are staged independently, and two warp MMA operators consume the same transformed A fragments to accumulate two output tensors in parallel. / 多阶段融合 threadblock MMA 流水线 这个头文件包含了示例中最深、最关键的性能路径。它可以视作 CUTLASS 多阶段 tensor-op MMA 的双流扩展：A 只有一条共享数据流，B0/B1 则各自独立 staging，两个 warp MMA 算子复用同一份变换后的 A fragment，并行累积两个输出张量。

---

## Line-by-Line Analysis / 逐行分析

### Logical Block 1 / 逻辑块 1 — lines 31-47

```cpp
/*! \file
    \brief Template for a double-buffered threadblock-scoped GEMM kernel.
*/

#pragma once

#include "cutlass/aligned_buffer.h"
#include "cutlass/arch/memory.h"
#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/numeric_types.h"

#include "cutlass/gemm/threadblock/mma_base.h"
#include "dual_mma_base.h"
```

**EN**: The opening comment identifies the file as a double-buffered threadblock GEMM kernel. In the dual-GEMM context, that means one CTA is responsible for advancing two right-hand operand streams through the same K pipeline.
**CN**: 开头注释把该文件描述为双缓冲 threadblock GEMM kernel。在 dual-GEMM 场景下，这意味着一个 CTA 要在同一条 K 向流水线中同时推进两条右操作数数据流。

### Logical Block 2 / 逻辑块 2 — lines 58-170

```cpp
template <
    /// Size of the Gemm problem - concept: gemm::GemmShape<>
    typename Shape_,
    /// Iterates over tiles of A operand in global memory
    //  (concept: ReadableTileIterator | ForwardTileIterator |
    //  MaskedTileIterator)
    typename IteratorA_,
    /// Iterates over tiles of A operand in shared memory
    /// (concept: WriteableTileIterator | RandomAccessTileIterator)
    typename SmemIteratorA_,
    /// Cache operation for operand A
    cutlass::arch::CacheOperation::Kind CacheOpA,
    /// Iterates over tiles of B0 operand in global memory
    //  (concept: ReadableTileIterator | ForwardTileIterator |
    //  MaskedTileIterator)
    typename IteratorB0_,
    /// Iterates over tiles of B0 operand in shared memory
    /// (concept: WriteableTileIterator | RandomAccessTileIterator)
    typename SmemIteratorB0_,
    /// Cache operation for operand B
    cutlass::arch::CacheOperation::Kind CacheOpB,
    /// Iterates over tiles of B1 operand in global memory
    //  (concept: ReadableTileIterator | ForwardTileIterator |
    //  MaskedTileIterator)
    typename IteratorB1_,
    /// Iterates over tiles of B1 operand in shared memory
    /// (concept: WriteableTileIterator | RandomAccessTileIterator)
    typename SmemIteratorB1_,
    /// Data type of accumulator matrix
    typename ElementC_,
    /// Data type of accumulator matrix
    typename LayoutC_,
    /// Policy describing tuning details (concept: MmaPolicy)
    typename Policy0_,
    /// B1-specific version of the policy (concept: MmaPolicy)
    typename Policy1_,
    /// Number of stages,
    int Stages,
    /// Use zfill or predicate for out-of-bound cp.async
    SharedMemoryClearOption SharedMemoryClear = SharedMemoryClearOption::kNone,
    /// Used for partial specialization
    typename Enable = bool>
class DualMmaMultistage : 
  public DualMmaBase<Shape_, Policy0_, Policy1_, Stages> {
public:
  ///< Base class
  using Base = DualMmaBase<Shape_, Policy0_, Policy1_, Stages>;
  ///< Size of the Gemm problem - concept: gemm::GemmShape<>
  using Shape = Shape_;
  ///< Iterates over tiles of A operand in global memory
  using IteratorA = IteratorA_;
  ///< Iterates over tiles of B0 operand in global memory
  using IteratorB0 = IteratorB0_;
  ///< Iterates over tiles of B1 operand in global memory
  using IteratorB1 = IteratorB1_;
  ///< Data type of accumulator matrix
  using ElementC = ElementC_;
  ///< Layout of accumulator matrix
  using LayoutC = LayoutC_;
  ///< Policy describing tuning details
  using Policy0 = Policy0_;
  using Policy1 = Policy1_;

  using SmemIteratorA = SmemIteratorA_;
  using SmemIteratorB0 = SmemIteratorB0_;
  using SmemIteratorB1 = SmemIteratorB1_;

  static cutlass::arch::CacheOperation::Kind const kCacheOpA = CacheOpA;
  static cutlass::arch::CacheOperation::Kind const kCacheOpB = CacheOpB;

  //
  // Dependent types
  //

  /// Fragment of accumulator tile
  using FragmentC = typename Policy0::Operator::FragmentC;

  /// Warp-level Mma
  using Operator0 = typename Policy0::Operator;
  using Operator1 = typename Policy1::Operator;

  /// Minimum architecture is Sm80 to support cp.async
  using ArchTag = arch::Sm80;
  
  /// Complex transform on A operand
  static ComplexTransform const kTransformA = Operator0::kTransformA;

  /// Complex transform on B operand
  static ComplexTransform const kTransformB0 = Operator0::kTransformB;
  static ComplexTransform const kTransformB1 = Operator1::kTransformB;

  /// Internal structure exposed for introspection.
  struct Detail {

    /// Number of cp.async instructions to load one stage of operand A
    static int const AsyncCopyIterationsPerStageA =
        IteratorA::ThreadMap::Iterations::kCount;

    /// Number of cp.async instructions to load one stage of operand B
    static int const AsyncCopyIterationsPerStageB =
        IteratorB0::ThreadMap::Iterations::kCount;

    /// Number of stages
    static int const kStages = Stages;

    /// Number of cp.async instructions to load on group of operand A
    static int const kAccessesPerGroupA =
        (AsyncCopyIterationsPerStageA + Base::kWarpGemmIterations - 1) / Base::kWarpGemmIterations;

    /// Number of cp.async instructions to load on group of operand B
    static int const kAccessesPerGroupB =
        (AsyncCopyIterationsPerStageB + Base::kWarpGemmIterations - 1) / Base::kWarpGemmIterations;
  };
```

**EN**: The template surface is broad because it needs three families of iterators: A global/shared iterators, B0 global/shared iterators, and B1 global/shared iterators. The nested `Detail` struct precomputes how many `cp.async` instructions are needed per stage and how they should be grouped relative to the number of warp-level GEMM steps. Those constants drive the overlap between memory movement and math.
**CN**: 模板接口很宽，因为它需要三大类迭代器：A 的全局/共享迭代器、B0 的全局/共享迭代器，以及 B1 的全局/共享迭代器。嵌套的 `Detail` 结构体会预先计算每个 stage 需要多少条 `cp.async` 指令，以及它们应如何相对 warp 级 GEMM 步数分组。这些常量共同决定了内存搬运与计算重叠的节奏。

### Logical Block 3 / 逻辑块 3 — lines 187-232

```cpp
  /// Construct from tensor references
  CUTLASS_DEVICE
  DualMmaMultistage(
      ///< Shared storage needed for internal use by threadblock-scoped GEMM
      typename Base::SharedStorage &shared_storage,
      ///< ID within the threadblock
      int thread_idx,
      ///< ID of warp
      int warp_idx,
      ///< ID of each thread within a warp
      int lane_idx
    ):
      Base(shared_storage, thread_idx, warp_idx, lane_idx),
      smem_iterator_A_(shared_storage.operand_A_ref(), thread_idx),
      smem_iterator_B0_(shared_storage.operand_B0_ref(), thread_idx),
      smem_iterator_B1_(shared_storage.operand_B1_ref(), thread_idx)
  {
    // Compute warp location within threadblock tile by mapping the warp_id to
    // three coordinates:
    //   _m: the warp's position within the threadblock along the M dimension
    //   _n: the warp's position within the threadblock along the N dimension
    //   _k: the warp's position within the threadblock along the K dimension

    int warp_idx_mn = warp_idx % (Base::WarpCount::kM * Base::WarpCount::kN);
    int warp_idx_k = warp_idx / (Base::WarpCount::kM * Base::WarpCount::kN);

    int warp_idx_m = warp_idx_mn % Base::WarpCount::kM;
    int warp_idx_n = warp_idx_mn / Base::WarpCount::kM;

    // Add per-warp offsets in units of warp-level tiles
    this->warp_tile_iterator_A_.add_tile_offset(
        {warp_idx_m, Base::kWarpGemmIterations * warp_idx_k});
    this->warp_tile_iterator_B0_.add_tile_offset(
        {Base::kWarpGemmIterations * warp_idx_k, warp_idx_n});
    this->warp_tile_iterator_B1_.add_tile_offset(
        {Base::kWarpGemmIterations * warp_idx_k, warp_idx_n});
  }
```

**EN**: The constructor wires the shared-memory iterators and maps each warp into CTA-local `(m, n, k)` coordinates. The same warp offset is then applied to the A iterator and both B iterators, ensuring that the two GEMM streams look at matching tiles for the same CTA position.
**CN**: 构造函数先绑定共享内存迭代器，再把每个 warp 映射到 CTA 内部的 `(m, n, k)` 坐标。随后把对应偏移同时施加到 A 迭代器和两份 B 迭代器上，保证两个 GEMM 流在同一个 CTA 位置观察到匹配的 tile。

### Logical Block 4 / 逻辑块 4 — lines 235-337

```cpp
  void copy_tiles_and_advance(IteratorA &iterator_A, IteratorB0 &iterator_B0, IteratorB1 &iterator_B1,
                              int group_start_A = 0, int group_start_B = 0) {
    iterator_A.set_iteration_index(group_start_A *
                                   IteratorA::kAccessesPerVector);
    this->smem_iterator_A_.set_iteration_index(group_start_A);

    // Async Copy for operand A
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < Detail::kAccessesPerGroupA; ++j) {
      if (group_start_A + j < Detail::AsyncCopyIterationsPerStageA) {
        typename IteratorA::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorA::AccessType *>(
                this->smem_iterator_A_.get());

        int const kSrcBytes = sizeof_bits<typename IteratorA::Element>::value *
                              IteratorA::ThreadMap::kElementsPerAccess /
                              IteratorA::kAccessesPerVector / 8;

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorA::kAccessesPerVector; ++v) {
          auto gmem_ptr = iterator_A.get();

          if (SharedMemoryClear == SharedMemoryClearOption::kZfill) {
            cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpA>(
                dst_ptr + v, gmem_ptr, iterator_A.valid());
          } else {
            cutlass::arch::cp_async<kSrcBytes, kCacheOpA>(
                dst_ptr + v, gmem_ptr, iterator_A.valid());
          }

          ++iterator_A;
        }

        ++this->smem_iterator_A_;
      }
    }

    iterator_B0.set_iteration_index(group_start_B *
                                   IteratorB0::kAccessesPerVector);
    iterator_B1.set_iteration_index(group_start_B *
                                   IteratorB1::kAccessesPerVector);
    this->smem_iterator_B0_.set_iteration_index(group_start_B);
    this->smem_iterator_B1_.set_iteration_index(group_start_B);

    // Async Copy for operand B0
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < Detail::kAccessesPerGroupB; ++j) {
      if (group_start_B + j < Detail::AsyncCopyIterationsPerStageB) {
        typename IteratorB0::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorB0::AccessType *>(
                this->smem_iterator_B0_.get());

        int const kSrcBytes = sizeof_bits<typename IteratorB0::Element>::value *
                              IteratorB0::ThreadMap::kElementsPerAccess /
                              IteratorB0::kAccessesPerVector / 8;

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorB0::kAccessesPerVector; ++v) {
          auto gmem_ptr = iterator_B0.get();

          if (SharedMemoryClear == SharedMemoryClearOption::kZfill) {
            cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB>(
                dst_ptr + v, gmem_ptr, iterator_B0.valid());
          } else {
            cutlass::arch::cp_async<kSrcBytes, kCacheOpB>(
                dst_ptr + v, gmem_ptr, iterator_B0.valid());
          }

          ++iterator_B0;
        }
        ++this->smem_iterator_B0_;
      }
    }
    // Async Copy for operand B1
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < Detail::kAccessesPerGroupB; ++j) {
      if (group_start_B + j < Detail::AsyncCopyIterationsPerStageB) {
        typename IteratorB1::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorB1::AccessType *>(
                this->smem_iterator_B1_.get());

        int const kSrcBytes = sizeof_bits<typename IteratorB1::Element>::value *
                              IteratorB1::ThreadMap::kElementsPerAccess /
                              IteratorB1::kAccessesPerVector / 8;

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorB1::kAccessesPerVector; ++v) {
          auto gmem_ptr = iterator_B1.get();

          if (SharedMemoryClear == SharedMemoryClearOption::kZfill) {
            cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB>(
                dst_ptr + v, gmem_ptr, iterator_B1.valid());
          } else {
            cutlass::arch::cp_async<kSrcBytes, kCacheOpB>(
                dst_ptr + v, gmem_ptr, iterator_B1.valid());
          }

          ++iterator_B1;
        }
        ++this->smem_iterator_B1_;
      }
    }
  }
```

**EN**: `copy_tiles_and_advance()` is the reusable async copy routine. It stages A, B0, and B1 in lockstep: reset iteration indices, compute destination pointers in shared memory, issue `cp.async` or `cp_async_zfill` for each vector access, and then advance both the global and shared-memory iterators. This is the first place where the dual-stream nature is explicit: B0 and B1 are copied independently but follow the same scheduling skeleton.
**CN**: `copy_tiles_and_advance()` 是可复用的异步拷贝例程。它会锁步地 staging A、B0 和 B1：重置迭代索引，计算共享内存目标地址，对每个向量访问发出 `cp.async` 或 `cp_async_zfill`，然后同步推进全局与共享内存迭代器。这里第一次非常直接地体现了“双流”本质：B0 与 B1 各自独立拷贝，但遵循同一套调度骨架。

### Logical Block 5 / 逻辑块 5 — lines 341-459

```cpp
  void operator()(
      ///< problem size of GEMM
      int gemm_k_iterations,
      ///< destination accumulator tile
      FragmentC &accum0,
      FragmentC &accum1,
      ///< iterator over A operand in global memory
      IteratorA iterator_A,
      ///< iterator over B operand in global memory
      IteratorB0 iterator_B0,
      IteratorB1 iterator_B1,
      ///< initial value of accumulator
      FragmentC const &src_accum0,
      FragmentC const &src_accum1
    ) {

    //
    // Prologue
    //

    // Issue several complete stages
    CUTLASS_PRAGMA_UNROLL
    for (int stage = 0; stage < Base::kStages - 1;
         ++stage, --gemm_k_iterations) {

      iterator_A.clear_mask(gemm_k_iterations == 0);
      iterator_B0.clear_mask(gemm_k_iterations == 0);
      iterator_B1.clear_mask(gemm_k_iterations == 0);

      iterator_A.set_iteration_index(0);
      this->smem_iterator_A_.set_iteration_index(0);

      // Async Copy for operand A
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageA; ++j) {
        typename IteratorA::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorA::AccessType *>(
                this->smem_iterator_A_.get());

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorA::kAccessesPerVector; ++v) {
          int const kSrcBytes =
              sizeof_bits<typename IteratorA::Element>::value *
              IteratorA::ThreadMap::kElementsPerAccess /
              IteratorA::kAccessesPerVector / 8;

          int src_bytes = (iterator_A.valid() ? kSrcBytes : 0);

          cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpA>(
              dst_ptr + v, iterator_A.get(), iterator_A.valid());

          ++iterator_A;
        }

        ++this->smem_iterator_A_;
      }

      iterator_B0.set_iteration_index(0);
      iterator_B1.set_iteration_index(0);
      this->smem_iterator_B0_.set_iteration_index(0);
      this->smem_iterator_B1_.set_iteration_index(0);

      // Async Copy for operand B0
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageB; ++j) {
        typename IteratorB0::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorB0::AccessType *>(
                this->smem_iterator_B0_.get());

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorB0::kAccessesPerVector; ++v) {
          int const kSrcBytes =
              sizeof_bits<typename IteratorB0::Element>::value *
              IteratorB0::ThreadMap::kElementsPerAccess /
              IteratorB0::kAccessesPerVector / 8;

          cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB>(
              dst_ptr + v, iterator_B0.get(), iterator_B0.valid());

          ++iterator_B0;
        }

        ++this->smem_iterator_B0_;
      }
      // Async Copy for operand B1
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageB; ++j) {
        typename IteratorB1::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorB1::AccessType *>(
                this->smem_iterator_B1_.get());

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorB1::kAccessesPerVector; ++v) {
          int const kSrcBytes =
              sizeof_bits<typename IteratorB1::Element>::value *
              IteratorB1::ThreadMap::kElementsPerAccess /
              IteratorB1::kAccessesPerVector / 8;

          cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB>(
              dst_ptr + v, iterator_B1.get(), iterator_B1.valid());

          ++iterator_B1;
        }

        ++this->smem_iterator_B1_;
      }

      // Move to the next stage
      iterator_A.add_tile_offset({0, 1});
      iterator_B0.add_tile_offset({1, 0});
      iterator_B1.add_tile_offset({1, 0});

      this->smem_iterator_A_.add_tile_offset({0, 1});
      this->smem_iterator_B0_.add_tile_offset({1, 0});
      this->smem_iterator_B1_.add_tile_offset({1, 0});

      // Defines the boundary of a stage of cp.async.
      cutlass::arch::cp_async_fence();
    }
```

**EN**: The prologue fills the pipeline by issuing `Base::kStages - 1` complete async-copy stages for A/B0/B1 before any math begins. It masks off invalid iterations near the tail, advances tile offsets stage by stage, and closes each stage with `cp_async_fence()`. When the loop ends, the pipeline has enough prefetched data to start overlapping math with future global-memory loads.
**CN**: 前导阶段会在正式计算之前，先为 A/B0/B1 发出 `Base::kStages - 1` 个完整 async-copy stage，把流水线填满。它会在尾部清除无效迭代的 mask，逐 stage 推进 tile 偏移，并用 `cp_async_fence()` 为每个 stage 收口。循环结束时，流水线中已经预取了足够多的数据，可以开始把计算与后续全局内存加载重叠起来。

### Logical Block 6 / 逻辑块 6 — lines 470-528

```cpp
    if (SharedMemoryClear == SharedMemoryClearOption::kClearLastStage) {

      /// Iterator to write threadblock-scoped tile of A operand to shared memory
      SmemIteratorA last_smem_iterator_A(this->smem_iterator_A_);

      typename IteratorA::AccessType zero_A;
      zero_A.clear();

      last_smem_iterator_A.set_iteration_index(0);

      // Async Copy for operand A
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageA; ++j) {

        typename IteratorA::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorA::AccessType *>(
                last_smem_iterator_A.get());

        *dst_ptr = zero_A;

        ++last_smem_iterator_A;
      }

      typename IteratorB0::AccessType zero_B;
      zero_B.clear();

      /// Iterator to write threadblock-scoped tile of B0 operand to shared memory
      SmemIteratorB0 last_smem_iterator_B0(this->smem_iterator_B0_);
      last_smem_iterator_B0.set_iteration_index(0);

      // Async Copy for operand B0
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageB; ++j) {
        typename IteratorB0::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorB0::AccessType *>(
                last_smem_iterator_B0.get());

        *dst_ptr = zero_B;

        ++last_smem_iterator_B0;
      }

      /// Iterator to write threadblock-scoped tile of B1 operand to shared memory
      SmemIteratorB1 last_smem_iterator_B1(this->smem_iterator_B1_);
      last_smem_iterator_B1.set_iteration_index(0);

      // Async Copy for operand B1
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageB; ++j) {

        typename IteratorB1::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorB1::AccessType *>(
                last_smem_iterator_B1.get());

        *dst_ptr = zero_B;

        ++last_smem_iterator_B1;
      }
    }
```

**EN**: Some specializations need the last shared-memory stage to be explicitly zeroed so out-of-footprint accumulator lanes stay harmless. This optional block writes zeros into the pending A/B0/B1 shared tiles when `SharedMemoryClearOption::kClearLastStage` is selected.
**CN**: 某些特化要求把最后一个共享内存 stage 显式清零，这样超出 GEMM 有效范围的累加器 lane 就不会污染结果。当选择 `SharedMemoryClearOption::kClearLastStage` 时，这段代码会把待使用的 A/B0/B1 共享 tile 全部置零。

### Logical Block 7 / 逻辑块 7 — lines 530-585

```cpp
    // Waits until stages up to the previous (kStages-2)th stage have committed.
    cutlass::arch::cp_async_wait<Base::kStages - 2>();
    __syncthreads();

    // Pair of fragments used to overlap shared memory loads and math
    // instructions
    WarpLoadedFragmentA warp_loaded_frag_A[2];
    WarpLoadedFragmentB0 warp_loaded_frag_B0[2];
    WarpLoadedFragmentB1 warp_loaded_frag_B1[2];
    WarpTransformedFragmentA warp_transformed_frag_A[2];
    WarpTransformedFragmentB0 warp_transformed_frag_B0[2];
    WarpTransformedFragmentB1 warp_transformed_frag_B1[2];

    Operator0 warp_mma0;
    Operator1 warp_mma1;

    this->warp_tile_iterator_A_.set_kgroup_index(0);
    this->warp_tile_iterator_B0_.set_kgroup_index(0);
    this->warp_tile_iterator_B1_.set_kgroup_index(0);

    this->warp_tile_iterator_A_.load(warp_loaded_frag_A[0]);
    this->warp_tile_iterator_B0_.load(warp_loaded_frag_B0[0]);
    this->warp_tile_iterator_B1_.load(warp_loaded_frag_B1[0]);

    ++this->warp_tile_iterator_A_;
    ++this->warp_tile_iterator_B0_;
    ++this->warp_tile_iterator_B1_;

    iterator_A.clear_mask(gemm_k_iterations == 0);
    iterator_B0.clear_mask(gemm_k_iterations == 0);
    iterator_B1.clear_mask(gemm_k_iterations == 0);

    int smem_write_stage_idx = Base::kStages - 1;
    int smem_read_stage_idx = 0;

    warp_mma0.transform(warp_transformed_frag_A[0], warp_transformed_frag_B0[0],
                        warp_loaded_frag_A[0], warp_loaded_frag_B0[0]);
    warp_mma1.transform(warp_transformed_frag_A[0], warp_transformed_frag_B1[0],
                        warp_loaded_frag_A[0], warp_loaded_frag_B1[0]);

    // tf32x3 kernels use staging accumulation. warp_mma uses a temporary
    // accumulator and this temporary accumulator is added to the final
    // accumulator once in every mainloop iteration.
    plus<FragmentC> plus_accum;

    FragmentC tmp_accum0, tmp_accum1;

    if (platform::is_same<typename Operator0::MathOperator,
                          arch::OpMultiplyAddFastF32>::value
      || platform::is_same<typename Operator0::MathOperator,
                           arch::OpMultiplyAddComplexFastF32>::value) {

      tmp_accum0.clear();
      tmp_accum1.clear();
    }
```

**EN**: After waiting for the prefetched stages to commit, the code allocates double-buffered warp fragments and loads the first A/B0/B1 tiles. A single transformed A fragment is then paired separately with transformed B0 and B1 fragments. For fast-F32 math operators, the code also introduces temporary accumulators so staging accumulation semantics are preserved correctly.
**CN**: 等待预取 stage 提交之后，代码会分配双缓冲 warp fragment，并加载第一组 A/B0/B1 tile。随后同一份变换后的 A fragment 会分别与 B0、B1 的变换后 fragment 配对。对于 fast-F32 数学算子，代码还额外引入临时累加器，以保证 staged accumulation 的语义正确。

### Logical Block 8 / 逻辑块 8 — lines 590-766

```cpp
    CUTLASS_GEMM_LOOP
    for (; gemm_k_iterations > (-Base::kStages + 1);) {
      //
      // Loop over GEMM K dimension
      //

      // Computes a warp-level GEMM on data held in shared memory
      // Each "warp_mma_k" refers to a warp-level matrix multiply-accumulate
      CUTLASS_PRAGMA_UNROLL
      for (int warp_mma_k = 0; warp_mma_k < Base::kWarpGemmIterations;
           ++warp_mma_k) {

        // Load warp-level tiles from shared memory, wrapping to k offset if
        // this is the last group as the case may be.

        this->warp_tile_iterator_A_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations);
        this->warp_tile_iterator_B0_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations);
        this->warp_tile_iterator_B1_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations);
        
        this->warp_tile_iterator_A_.load(warp_loaded_frag_A[(warp_mma_k + 1) % 2]);
        this->warp_tile_iterator_B0_.load(warp_loaded_frag_B0[(warp_mma_k + 1) % 2]);
        this->warp_tile_iterator_B1_.load(warp_loaded_frag_B1[(warp_mma_k + 1) % 2]);

        ++this->warp_tile_iterator_A_;
        ++this->warp_tile_iterator_B0_;
        ++this->warp_tile_iterator_B1_;

        if (warp_mma_k > 0) {
          warp_mma0.transform(warp_transformed_frag_A[warp_mma_k % 2],
                              warp_transformed_frag_B0[warp_mma_k % 2],
                              warp_loaded_frag_A[warp_mma_k % 2],
                              warp_loaded_frag_B0[warp_mma_k % 2]);
          warp_mma1.transform(warp_transformed_frag_A[warp_mma_k % 2],
                              warp_transformed_frag_B1[warp_mma_k % 2],
                              warp_loaded_frag_A[warp_mma_k % 2],
                              warp_loaded_frag_B1[warp_mma_k % 2]);
        }

        if (platform::is_same<typename Operator0::MathOperator,
                              arch::OpMultiplyAddFastF32>::value
          || platform::is_same<typename Operator0::MathOperator,
                               arch::OpMultiplyAddComplexFastF32>::value) {

          warp_mma0(
            tmp_accum0,
            warp_transformed_frag_A[warp_mma_k % 2],
            warp_transformed_frag_B0[warp_mma_k % 2], 
            tmp_accum0
          );
          warp_mma1(
            tmp_accum1,
            warp_transformed_frag_A[warp_mma_k % 2],
            warp_transformed_frag_B1[warp_mma_k % 2], 
            tmp_accum1
          );

          if (warp_mma_k == 0) {
            accum0 = plus_accum(accum0, tmp_accum0);
            accum1 = plus_accum(accum1, tmp_accum1);
            tmp_accum0.clear();
            tmp_accum1.clear();
          }
        } else {
          warp_mma0(
            accum0,
            warp_transformed_frag_A[warp_mma_k % 2],
            warp_transformed_frag_B0[warp_mma_k % 2],
            accum0
          );
          warp_mma1(
            accum1,
            warp_transformed_frag_A[warp_mma_k % 2],
            warp_transformed_frag_B1[warp_mma_k % 2],
            accum1
          );
        }

        // Issue global->shared copies for the this stage
        if (warp_mma_k < Base::kWarpGemmIterations - 1) {
          int group_start_iteration_A, group_start_iteration_B;

          group_start_iteration_A = warp_mma_k * Detail::kAccessesPerGroupA;
          group_start_iteration_B = warp_mma_k * Detail::kAccessesPerGroupB;

          copy_tiles_and_advance(iterator_A, iterator_B0, iterator_B1, group_start_iteration_A, 
                               group_start_iteration_B);
        }

        if (warp_mma_k + 2 == Base::kWarpGemmIterations) {
          int group_start_iteration_A, group_start_iteration_B;
          group_start_iteration_A =
              (warp_mma_k + 1) * Detail::kAccessesPerGroupA;
          group_start_iteration_B =
              (warp_mma_k + 1) * Detail::kAccessesPerGroupB;

          copy_tiles_and_advance(iterator_A, iterator_B0, iterator_B1, group_start_iteration_A, 
                               group_start_iteration_B);

          // Inserts a memory fence between stages of cp.async instructions.
          cutlass::arch::cp_async_fence();

          // Waits until stages up to the previous (kStages-2)th stage have committed.
          arch::cp_async_wait<Base::kStages - 2>();
          __syncthreads();

          // Move to the next stage
          iterator_A.add_tile_offset({0, 1});
          iterator_B0.add_tile_offset({1, 0});
          iterator_B1.add_tile_offset({1, 0});

          this->smem_iterator_A_.add_tile_offset({0, 1});
          this->smem_iterator_B0_.add_tile_offset({1, 0});
          this->smem_iterator_B1_.add_tile_offset({1, 0});

          // Add negative offsets to return iterators to the 'start' of the
          // circular buffer in shared memory
          if (smem_write_stage_idx == (Base::kStages - 1)) {
            this->smem_iterator_A_.add_tile_offset({0, -Base::kStages});
            this->smem_iterator_B0_.add_tile_offset({-Base::kStages, 0});
            this->smem_iterator_B1_.add_tile_offset({-Base::kStages, 0});
            smem_write_stage_idx = 0;
          } else {
            ++smem_write_stage_idx;
          }

          if (smem_read_stage_idx == (Base::kStages - 1)) {
            this->warp_tile_iterator_A_.add_tile_offset(
                {0, -Base::kStages * Policy0::kPartitionsK *
                        Base::kWarpGemmIterations});
            this->warp_tile_iterator_B0_.add_tile_offset(
                {-Base::kStages * Policy0::kPartitionsK *
                     Base::kWarpGemmIterations,
                 0});
            this->warp_tile_iterator_B1_.add_tile_offset(
                {-Base::kStages * Policy1::kPartitionsK *
                     Base::kWarpGemmIterations,
                 0});
            smem_read_stage_idx = 0;
          } else {
            ++smem_read_stage_idx;
          }

          --gemm_k_iterations;
          iterator_A.clear_mask(gemm_k_iterations == 0);
          iterator_B0.clear_mask(gemm_k_iterations == 0);
          iterator_B1.clear_mask(gemm_k_iterations == 0);
        }

        // Do any conversions feeding the first stage at the end of the loop so
        // we can start right away on mma instructions
        if (warp_mma_k + 1 == Base::kWarpGemmIterations) {
          warp_mma0.transform(warp_transformed_frag_A[(warp_mma_k + 1) % 2],
                              warp_transformed_frag_B0[(warp_mma_k + 1) % 2],
                              warp_loaded_frag_A[(warp_mma_k + 1) % 2],
                              warp_loaded_frag_B0[(warp_mma_k + 1) % 2]);
          warp_mma1.transform(warp_transformed_frag_A[(warp_mma_k + 1) % 2],
                              warp_transformed_frag_B1[(warp_mma_k + 1) % 2],
                              warp_loaded_frag_A[(warp_mma_k + 1) % 2],
                              warp_loaded_frag_B1[(warp_mma_k + 1) % 2]);
        }
      }

    }

    if (platform::is_same<typename Operator0::MathOperator,
                          arch::OpMultiplyAddFastF32>::value
      || platform::is_same<typename Operator0::MathOperator,
                           arch::OpMultiplyAddComplexFastF32>::value) {
      accum0 = plus_accum(accum0, tmp_accum0); 
      accum1 = plus_accum(accum1, tmp_accum1); 
    }
 
    // commit and drain all pending and predicated cp.async pnz from the GEMM mainloop
    cutlass::arch::cp_async_fence();
    cutlass::arch::cp_async_wait<0>();
    __syncthreads();
  }
```

**EN**: This is the true multistage mainloop. For each warp-level K step, it preloads the next shared-memory fragments, performs the necessary transforms, executes `warp_mma0` and `warp_mma1`, opportunistically issues the next A/B0/B1 async copies, advances circular shared-memory stage indices, wraps iterators when the ring buffer rolls over, and updates masks near the tail. Conceptually, this is the dual-GEMM fusion engine: both GEMMs reuse the same A pipeline and differ only in which B iterator/policy they consume.
**CN**: 这部分才是真正的多阶段 mainloop。对于每个 warp 级 K 步，它都会预取下一组共享内存 fragment，完成必要的变换，执行 `warp_mma0` 与 `warp_mma1`，穿插发出下一批 A/B0/B1 async copy，推进循环共享内存 stage 索引，在 ring buffer 回卷时回绕迭代器，并在尾部更新 mask。从概念上说，这就是 dual-GEMM 的融合引擎：两个 GEMM 共用同一条 A 流水线，差别只在于各自消费哪份 B 迭代器/策略。

### Logical Block 9 / 逻辑块 9 — lines 762-775

```cpp
    // commit and drain all pending and predicated cp.async pnz from the GEMM mainloop
    cutlass::arch::cp_async_fence();
    cutlass::arch::cp_async_wait<0>();
    __syncthreads();
  }
};

/////////////////////////////////////////////////////////////////////////////////////////////////

}  // namespace threadblock
}  // namespace gemm
}  // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: The final fence/wait pair drains all pending async copies and synchronizes the CTA before control returns to the kernel. At that point the two accumulator fragments are complete and ready for the dual epilogue.
**CN**: 最后的 fence/wait 组合会排空所有未完成的异步拷贝，并在返回 kernel 之前同步整个 CTA。到这一步，两份累加器 fragment 都已经完整，可以交给 dual epilogue 处理。

---

## Key Concepts / 关键概念
**EN**: the device wrapper synthesizes this class from two `DefaultMma` policies; the kernel passes global-memory iterators into it; the epilogue later consumes the two accumulator fragments it produces.
**CN**: 组合关系：device wrapper 基于两份 `DefaultMma` 策略合成该类；kernel 把全局内存迭代器传给它；后续 epilogue 再消费它产出的两份累加器 fragment。

**EN**: Threadblock MMA pipeline: prefetch A/B0/B1 → overlap cp.async with two warp MMAs → reuse A for both streams → drain pipeline → hand two accumulators to the epilogue.
**CN**: threadblock MMA 流水线：预取 A/B0/B1 → 用两路 warp MMA 与 cp.async 重叠执行 → 让两路计算复用同一份 A → 排空流水线 → 把两份累加器交给 epilogue。

## Dependencies / 依赖项
**EN**: CUTLASS dependency: `cutlass/aligned_buffer.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/aligned_buffer.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/arch/memory.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/arch/memory.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/array.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/array.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/cutlass.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/cutlass.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/gemm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/matrix_shape.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/matrix_shape.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/numeric_types.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/numeric_types.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/threadblock/mma_base.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/threadblock/mma_base.h` 提供该文件直接使用的库级原语。

**EN**: Project-local dependency: `dual_mma_base.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`dual_mma_base.h` 提供该文件直接包含的辅助代码。
