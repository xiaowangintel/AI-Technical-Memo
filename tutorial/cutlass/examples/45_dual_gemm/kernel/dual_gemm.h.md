# dual_gemm.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/45_dual_gemm/kernel/dual_gemm.h`  
**Purpose / 用途**: CTA kernel wrapper for fused DualGemm execution This header is where the fused operator becomes a concrete CUDA threadblock program. It does not choose the math policy itself—that was already done by the device wrapper—but it owns CTA tile coordinates, pointer offsets, iterator construction, split-K synchronization, and epilogue invocation. / 融合 DualGemm 执行的 CTA kernel 包装层 这个头文件让融合算子真正落地为具体的 CUDA threadblock 程序。它本身不负责选择数学策略——这些在 device wrapper 中已经决定好了——但它负责 CTA 坐标、指针偏移、迭代器构造、split-K 同步以及 epilogue 调用。

---

## Line-by-Line Analysis / 逐行分析

### Logical Block 1 / 逻辑块 1 — lines 31-45

```cpp
/*! \file
    \brief Template for a pipelined GEMM kernel. Does not compute batching or support split-K.
*/

#pragma once

#include "cutlass/cutlass.h"

#include "cutlass/gemm/gemm.h"
#include "cutlass/matrix_coord.h"
#include "cutlass/semaphore.h"

#include "../threadblock/dual_mma_multistage.h"
#include "../threadblock/dual_epilogue.h"
#include "../dual_gemm_common.h"
```

**EN**: The includes are intentionally minimal: only GEMM coordinates, semaphore support, and the two custom threadblock components are needed. This tells you the file is an orchestration layer rather than a policy-definition layer.
**CN**: 这里的包含项刻意保持精简：只需要 GEMM 坐标、semaphore 支持以及两个自定义 threadblock 组件。这说明该文件本质上是编排层，而不是策略定义层。

### Logical Block 2 / 逻辑块 2 — lines 55-95

```cpp
template <
  typename DualMma_,               ///! Threadblock-scoped matrix multiply-accumulate 
  typename Epilogue0_,             ///! Epilogue
  typename Epilogue1_,             ///! Epilogue
  typename OutputOp2_,             ///! Epilogue
  typename ThreadblockSwizzle_,   ///! Threadblock swizzling function
  bool SplitKSerial,              ///! If true, code supporting split-K via serial reduction is enabled.
  bool StoreD0,
  bool StoreD1
>
struct DualGemm {

  using DualMma = DualMma_;

  using Epilogue0 = Epilogue0_;
  using Epilogue1 = Epilogue1_;
  using OutputOp0 = typename Epilogue0::OutputOp;
  using OutputOp1 = typename Epilogue1::OutputOp;
  using OutputOp2 = OutputOp2_;
  using ThreadblockSwizzle = ThreadblockSwizzle_;
  static constexpr bool kStoreD0 = StoreD0;
  static constexpr bool kStoreD1 = StoreD1;

  using DualEpilogue = cutlass::epilogue::threadblock::DualEpilogue<
      typename Epilogue0::Shape,
      typename Epilogue0::WarpMmaOperator,
      Epilogue0::kPartitionsK,
      typename Epilogue0::OutputTileIterator,
      typename Epilogue0::AccumulatorFragmentIterator,
      typename Epilogue0::WarpTileIterator,
      typename Epilogue0::SharedLoadIterator,
      OutputOp0,
      OutputOp1,
      OutputOp2,
      typename Epilogue0::Padding,
      kStoreD0,
      kStoreD1,
      Epilogue0::kFragmentsPerIteration,
      true // IterationsUnroll
  >;
```

**EN**: The template binds together the already-chosen fused mainloop (`DualMma`), two standard epilogues, the custom output functor for `D2`, threadblock swizzle, and storage/split-K policy flags. `DualEpilogue` is then synthesized from the epilogue pieces so it can consume two accumulator tiles and emit three logical outputs.
**CN**: 模板把之前已经选好的融合 mainloop（`DualMma`）、两个标准 epilogue、用于 `D2` 的自定义输出 functor、threadblock swizzle，以及存储/split-K 策略标志拼接在一起。随后又基于这些片段合成 `DualEpilogue`，使它能够接收两份累加器 tile 并输出三个逻辑结果。

### Logical Block 3 / 逻辑块 3 — lines 109-217

```cpp
  struct Params {
    DualGemmMode mode;
    cutlass::gemm::GemmCoord problem_size;
    cutlass::gemm::GemmCoord grid_tiled_shape;
    int swizzle_log_tile;

    // Mma0
    typename DualMma::IteratorA::Params params_A0;
    typename DualMma::IteratorA::TensorRef ref_A0;
    typename DualMma::IteratorB0::Params params_B0;
    typename DualMma::IteratorB0::TensorRef ref_B0;
    typename Epilogue0::OutputTileIterator::Params params_C0;
    typename Epilogue0::OutputTileIterator::TensorRef ref_C0;
    typename Epilogue0::OutputTileIterator::Params params_D0;
    typename Epilogue0::OutputTileIterator::TensorRef ref_D0;
    typename OutputOp0::Params output_op_0;

    // Mma1
    typename DualMma::IteratorB1::Params params_B1;
    typename DualMma::IteratorB1::TensorRef ref_B1;
    typename Epilogue1::OutputTileIterator::Params params_C1;
    typename Epilogue1::OutputTileIterator::TensorRef ref_C1;
    typename Epilogue1::OutputTileIterator::Params params_D1;
    typename Epilogue1::OutputTileIterator::TensorRef ref_D1;
    typename OutputOp1::Params output_op_1;

    typename Epilogue1::OutputTileIterator::Params params_D2;
    typename Epilogue1::OutputTileIterator::TensorRef ref_D2;
    typename OutputOp2::Params output_op_2;

    int *semaphore;
    int gemm_k_size;

    int64_t batch_stride_A;
    int64_t batch_stride_B0;
    int64_t batch_stride_B1;
    int64_t batch_stride_C;
    int64_t batch_stride_D;

    //
    // Methods
    //

    CUTLASS_HOST_DEVICE
    Params(): swizzle_log_tile(0), semaphore(0), gemm_k_size(0) { }

    CUTLASS_HOST_DEVICE
    Params(
      DualGemmMode mode,
      cutlass::gemm::GemmCoord const & problem_size,
      cutlass::gemm::GemmCoord const & grid_tiled_shape,
      // Mma0: D0 = A @ B0 + C0
      typename DualMma::IteratorA::TensorRef ref_A0,
      typename DualMma::IteratorB0::TensorRef ref_B0,
      typename Epilogue0::OutputTileIterator::TensorRef ref_C0,
      typename Epilogue0::OutputTileIterator::TensorRef ref_D0,
      // Mma1: D1 = A @ B1 + C1
      typename DualMma::IteratorB1::TensorRef ref_B1,
      typename Epilogue1::OutputTileIterator::TensorRef ref_C1,
      typename Epilogue1::OutputTileIterator::TensorRef ref_D1,

      typename Epilogue1::OutputTileIterator::TensorRef ref_D2,
      typename OutputOp0::Params output_op_0 = typename OutputOp0::Params(),
      typename OutputOp1::Params output_op_1 = typename OutputOp1::Params(),
      typename OutputOp2::Params output_op_2 = typename OutputOp2::Params(),
      int *workspace = nullptr,
      int64_t batch_stride_A = 1,
      int64_t batch_stride_B0 = 1,
      int64_t batch_stride_B1 = 1,
      int64_t batch_stride_C = 1,
      int64_t batch_stride_D = 1
    ):
      mode(mode),
      problem_size(problem_size),
      grid_tiled_shape(grid_tiled_shape),
      swizzle_log_tile(ThreadblockSwizzle().get_log_tile(grid_tiled_shape)),
      // Mma0
      params_A0(ref_A0.layout()),
      ref_A0(ref_A0),
      params_B0(ref_B0.layout()),
      ref_B0(ref_B0),
      params_C0(ref_C0.layout()),
      ref_C0(ref_C0),
      params_D0(ref_D0.layout()),
      ref_D0(ref_D0),
      // Mma1
      params_B1(ref_B1.layout()),
      ref_B1(ref_B1),
      params_C1(ref_C1.layout()),
      ref_C1(ref_C1),
      params_D1(ref_D1.layout()),
      ref_D1(ref_D1),
      params_D2(ref_D2.layout()),
      ref_D2(ref_D2),
      output_op_0(output_op_0),
      output_op_1(output_op_1),
      output_op_2(output_op_2),
      batch_stride_A(batch_stride_A),
      batch_stride_B0(batch_stride_B0),
      batch_stride_B1(batch_stride_B1),
      batch_stride_C(batch_stride_C),
      batch_stride_D(batch_stride_D) {

      int total_gemm_k_iterations = (problem_size.k() + DualMma::Shape::kK - 1) / DualMma::Shape::kK;
      int gemm_k_iterations = (total_gemm_k_iterations + grid_tiled_shape.k() - 1) / grid_tiled_shape.k();
      gemm_k_size = gemm_k_iterations * DualMma::Shape::kK;

      semaphore = workspace;
    }
```

**EN**: The `Params` structure is the fully expanded runtime state consumed by the kernel. It stores per-iterator layout params and tensor refs for A/B0/C0/D0, B1/C1/D1, plus D2 and the three output-operator params. It also precomputes `gemm_k_size`, which is the per-slice K span used when mode is ordinary `kGemm` with split-K style partitioning.
**CN**: `Params` 是 kernel 实际消费的完整运行时状态。它保存了 A/B0/C0/D0、B1/C1/D1、D2 对应的迭代器布局参数与 tensor ref，以及三个输出算子的参数。同时还预先计算了 `gemm_k_size`，它表示普通 `kGemm` 模式下每个 split-K 切片负责的 K 方向跨度。

### Logical Block 4 / 逻辑块 4 — lines 233-281

```cpp
  /// Determines whether kernel satisfies alignment
    static Status can_implement(
      cutlass::gemm::GemmCoord const & problem_size,
      typename DualMma::IteratorA::TensorRef ref_A0,
      typename DualMma::IteratorB0::TensorRef ref_B0,
      typename Epilogue0::OutputTileIterator::TensorRef ref_C0,
      typename Epilogue0::OutputTileIterator::TensorRef ref_D0,
      typename DualMma::IteratorB1::TensorRef ref_B1,
      typename Epilogue1::OutputTileIterator::TensorRef ref_C1,
      typename Epilogue1::OutputTileIterator::TensorRef ref_D1,
      typename Epilogue1::OutputTileIterator::TensorRef ref_D2) {

    static int const kAlignmentA = DualMma::IteratorA::AccessType::kElements;
    static int const kAlignmentB = DualMma::IteratorB0::AccessType::kElements;
    static int const kAlignmentC = Epilogue0::OutputTileIterator::kElementsPerAccess;

    if (!TensorRef_aligned(ref_A0, kAlignmentA)) {
      return Status::kErrorMisalignedOperand;
    }

    if (!TensorRef_aligned(ref_B0, kAlignmentB)) {
      return Status::kErrorMisalignedOperand;
    }

    if (!TensorRef_aligned(ref_C0, kAlignmentC)) {
      return Status::kErrorMisalignedOperand;
    }

    if (!TensorRef_aligned(ref_D0, kAlignmentC)) {
      return Status::kErrorMisalignedOperand;
    }

    if (!TensorRef_aligned(ref_B1, kAlignmentB)) {
      return Status::kErrorMisalignedOperand;
    }

    if (!TensorRef_aligned(ref_C1, kAlignmentC)) {
      return Status::kErrorMisalignedOperand;
    }

    if (!TensorRef_aligned(ref_D1, kAlignmentC)) {
      return Status::kErrorMisalignedOperand;
    }

    if (!TensorRef_aligned(ref_D2, kAlignmentC)) {
      return Status::kErrorMisalignedOperand;
    }

    return Status::kSuccess;
```

**EN**: Alignment validation happens here at the lowest host-callable layer before launch. The checks reuse iterator vector widths for A/B and epilogue access width for C/D. Because the fused kernel writes three outputs, `D2` is validated explicitly as well.
**CN**: 发射前最后一层可由主机调用的对齐校验就在这里完成。A/B 的检查复用了迭代器访问向量宽度，C/D 的检查使用 epilogue 访问宽度。由于融合 kernel 会写三个输出，因此 `D2` 也要单独检查。

### Logical Block 5 / 逻辑块 5 — lines 286-391

```cpp
  void operator()(Params const &params, SharedStorage &shared_storage) {
    // Compute threadblock location
    ThreadblockSwizzle threadblock_swizzle;

    cutlass::gemm::GemmCoord threadblock_tile_offset =
        threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);

    // Early exit if CTA is out of range
    if (params.grid_tiled_shape.m() <= threadblock_tile_offset.m() ||
      params.grid_tiled_shape.n() <= threadblock_tile_offset.n()) {

      return;
    }

    int offset_k = 0;
    int problem_size_k = params.problem_size.k();

    ElementA *ptr_A0 = static_cast<ElementA *>(params.ref_A0.data());
    ElementB *ptr_B0 = static_cast<ElementB *>(params.ref_B0.data());
    ElementB *ptr_B1 = static_cast<ElementB *>(params.ref_B1.data());

    //
    // Fetch pointers based on mode.
    //
    if (params.mode == DualGemmMode::kGemm) {
      if (threadblock_tile_offset.k() + 1 < params.grid_tiled_shape.k()) {
        problem_size_k = (threadblock_tile_offset.k() + 1) * params.gemm_k_size;
      }

      offset_k = threadblock_tile_offset.k() * params.gemm_k_size;
    }
    else if (params.mode == DualGemmMode::kBatched) {
      ptr_A0 += threadblock_tile_offset.k() * params.batch_stride_A;
      ptr_B0 += threadblock_tile_offset.k() * params.batch_stride_B0;
      ptr_B1 += threadblock_tile_offset.k() * params.batch_stride_B1;
    }

    // Compute initial location in logical coordinates
    cutlass::MatrixCoord tb_offset_A0{
      threadblock_tile_offset.m() * DualMma::Shape::kM,
      offset_k,
    };

    cutlass::MatrixCoord tb_offset_B0{
      offset_k,
      threadblock_tile_offset.n() * DualMma::Shape::kN
    };

    cutlass::MatrixCoord tb_offset_B1{
      offset_k,
      threadblock_tile_offset.n() * DualMma::Shape::kN
    };

    // Compute position within threadblock
    int thread_idx = threadIdx.x;

    // Construct iterators to A and B operands
    typename DualMma::IteratorA iterator_A0(
      params.params_A0,
      ptr_A0,
      {params.problem_size.m(), problem_size_k},
      thread_idx,
      tb_offset_A0);

    typename DualMma::IteratorB0 iterator_B0(
      params.params_B0,
      ptr_B0,
      {problem_size_k, params.problem_size.n()},
      thread_idx,
      tb_offset_B0);

    typename DualMma::IteratorB1 iterator_B1(
      params.params_B1,
      ptr_B1,
      {problem_size_k, params.problem_size.n()},
      thread_idx,
      tb_offset_B1);


    // Broadcast the warp_id computed by lane 0 to ensure dependent code
    // is compiled as warp-uniform.
    int warp_idx = __shfl_sync(0xffffffff, threadIdx.x / 32, 0);
    int lane_idx = threadIdx.x % 32;

    //
    // Main loop
    //


    // Construct thread-scoped matrix multiply
    typename DualMma::FragmentC accum0;
    typename DualMma::FragmentC accum1;
    accum0.clear();
    accum1.clear();

    // Compute threadblock-scoped matrix multiply-add
    int gemm_k_iterations = (problem_size_k - offset_k + DualMma::Shape::kK - 1) / DualMma::Shape::kK;

    DualMma mma(shared_storage.main_loop, thread_idx, warp_idx, lane_idx);
    if (!kSplitKSerial || gemm_k_iterations > 0) {
      // Compute threadblock-scoped matrix multiply-add
      mma(gemm_k_iterations,
        accum0, accum1,
        iterator_A0, iterator_B0, iterator_B1,
        accum0, accum1);
    }
```

**EN**: The first half of `operator()` computes the CTA tile coordinate, exits early for out-of-range CTAs, and then interprets `threadblock_tile_offset.k()` according to `DualGemmMode`. In `kGemm` mode it derives an `offset_k` and a truncated local K extent for split-K slicing; in `kBatched` mode it simply bumps the base pointers by batch strides. After that it builds global-memory iterators for A, B0, and B1 and launches the fused `DualMma` mainloop to accumulate `accum0` and `accum1` in parallel.
**CN**: `operator()` 的前半段会先计算 CTA tile 坐标，越界时直接返回，然后根据 `DualGemmMode` 解释 `threadblock_tile_offset.k()`。在 `kGemm` 模式下，它会推导 `offset_k` 和当前切片的局部 K 范围；在 `kBatched` 模式下，则直接按照 batch stride 平移基地址。随后它构造 A、B0、B1 的全局内存迭代器，并启动融合后的 `DualMma` mainloop，同时累积 `accum0` 与 `accum1`。

### Logical Block 6 / 逻辑块 6 — lines 397-443

```cpp
    OutputOp0 output_op_0(params.output_op_0);
    OutputOp1 output_op_1(params.output_op_1);
    OutputOp2 output_op_2(params.output_op_2);

    //
    // Masked tile iterators constructed from members
    //

    threadblock_tile_offset =
        threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);

    //assume identity swizzle
    MatrixCoord threadblock_offset(
      threadblock_tile_offset.m() * DualMma::Shape::kM,
      threadblock_tile_offset.n() * DualMma::Shape::kN
    );

    int block_idx = threadblock_tile_offset.m() + threadblock_tile_offset.n() * params.grid_tiled_shape.m();

    ElementC *ptr_C0 = static_cast<ElementC *>(params.ref_C0.data());
    ElementC *ptr_C1 = static_cast<ElementC *>(params.ref_C1.data());
    ElementC *ptr_D0 = static_cast<ElementC *>(params.ref_D0.data());
    ElementC *ptr_D1 = static_cast<ElementC *>(params.ref_D1.data());
    ElementC *ptr_D2 = static_cast<ElementC *>(params.ref_D2.data());

    // Construct the semaphore.
    Semaphore semaphore(params.semaphore + block_idx, thread_idx);

    if (params.mode == DualGemmMode::kGemm) {
      // If performing a reduction via split-K, fetch the initial synchronization
      if (kSplitKSerial && params.grid_tiled_shape.k() > 1) {

        // Fetch the synchronization lock initially but do not block.
        semaphore.fetch();

        // Indicate which position in a serial reduction the output operator is currently updating
        output_op_0.set_k_partition(threadblock_tile_offset.k(), params.grid_tiled_shape.k());
        output_op_1.set_k_partition(threadblock_tile_offset.k(), params.grid_tiled_shape.k());
      }
    }
    else if (params.mode == DualGemmMode::kBatched) {
      ptr_C0 += threadblock_tile_offset.k() * params.batch_stride_C;
      ptr_C1 += threadblock_tile_offset.k() * params.batch_stride_C;
      ptr_D0 += threadblock_tile_offset.k() * params.batch_stride_D;
      ptr_D1 += threadblock_tile_offset.k() * params.batch_stride_D;
      ptr_D2 += threadblock_tile_offset.k() * params.batch_stride_D;
    }
```

**EN**: Before entering the epilogue, the kernel reconstructs output operators, creates a semaphore, and adjusts C/D pointers again depending on the execution mode. In split-K serial mode it also programs the output operators with the current K-partition index so they know whether they are producing an intermediate partial sum or the final result.
**CN**: 进入 epilogue 之前，kernel 会先重建输出算子、创建 semaphore，并根据执行模式再次修正 C/D 指针。在 split-K serial 模式下，它还会把当前 K 分区编号写入输出算子，使后者知道自己是在处理部分和还是最终结果。

### Logical Block 7 / 逻辑块 7 — lines 445-515

```cpp
    // Tile iterator loading from source tensor.
    typename Epilogue0::OutputTileIterator iterator_C0(
      params.params_C0,
      ptr_C0,
      params.problem_size.mn(),
      thread_idx,
      threadblock_offset
    );
    typename Epilogue1::OutputTileIterator iterator_C1(
      params.params_C1,
      ptr_C1,
      params.problem_size.mn(),
      thread_idx,
      threadblock_offset
    );

    // Tile iterator writing to destination tensor.
    typename Epilogue0::OutputTileIterator iterator_D0(
      params.params_D0,
      ptr_D0,
      params.problem_size.mn(),
      thread_idx,
      threadblock_offset
    );
    typename Epilogue1::OutputTileIterator iterator_D1(
      params.params_D1,
      ptr_D1,
      params.problem_size.mn(),
      thread_idx,
      threadblock_offset
    );
    typename Epilogue1::OutputTileIterator iterator_D2(
      params.params_D2,
      ptr_D2,
      params.problem_size.mn(),
      thread_idx,
      threadblock_offset
    );

    DualEpilogue epilogue(
      shared_storage.epilogue,
      thread_idx, 
      warp_idx, 
      lane_idx);

    // Wait on the semaphore - this latency may have been covered by iterator construction
    if (kSplitKSerial && params.grid_tiled_shape.k() > 1) {
        
      // For subsequent threadblocks, the source matrix is held in the 'D' tensor.
      if (threadblock_tile_offset.k()) {
        iterator_C0 = iterator_D0;
        iterator_C1 = iterator_D1;
      }

      semaphore.wait(threadblock_tile_offset.k());

      __threadfence();
    }

    // Execute the epilogue operator to update the destination tensor.
    typename Epilogue0::OutputTileIterator source_iters[] = {
      iterator_C0, iterator_C1
    };
    const bool writeToD2 = (!kSplitKSerial || params.grid_tiled_shape.k() == threadblock_tile_offset.k() + 1);
    epilogue(
      output_op_0, output_op_1, output_op_2,
      iterator_D0, iterator_D1, iterator_D2,
      accum0, accum1,
      source_iters,
      writeToD2
    );
```

**EN**: This block creates source and destination tile iterators, optionally redirects source iterators from `C0/C1` to `D0/D1` for later split-K slices, waits on the semaphore, and calls `DualEpilogue`. The `writeToD2` flag is crucial: only the final split-K slice is allowed to write `D2`, because `D2` is defined from the fully reduced `D0` and `D1`, not from partial fragments.
**CN**: 这一段构造源与目标 tile 迭代器；对于后续 split-K 切片，它还会把源迭代器从 `C0/C1` 重定向到 `D0/D1`；随后等待 semaphore，并调用 `DualEpilogue`。其中 `writeToD2` 尤其关键：只有最后一个 split-K 切片可以写 `D2`，因为 `D2` 必须建立在已经完成归约的 `D0`、`D1` 之上，而不是部分结果之上。

### Logical Block 8 / 逻辑块 8 — lines 521-537

```cpp
    if (kSplitKSerial && params.grid_tiled_shape.k() > 1) {
      
      int lock = 0;
      if (params.grid_tiled_shape.k() == threadblock_tile_offset.k() + 1) {

        // The final threadblock resets the semaphore for subsequent grids.
        lock = 0;
      }
      else {
        // Otherwise, the semaphore is incremented
        lock = threadblock_tile_offset.k() + 1;
      }

      __threadfence();
      semaphore.release(lock);
    }
  }
```

**EN**: After the epilogue, serial split-K mode advances or resets the semaphore. The final slice resets the lock for future grids; earlier slices release the next lock value. This is the synchronization glue that lets multiple CTA slices accumulate into the same logical output tile safely.
**CN**: epilogue 结束后，串行 split-K 模式会推进或重置 semaphore。最后一个切片负责把锁恢复为 0 以供后续网格使用；前面的切片则释放“下一个值”的锁。这正是多个 CTA 切片能够安全归约到同一个逻辑输出 tile 的同步胶水层。

---

## Key Concepts / 关键概念
**EN**: `DualMma` performs the fused threadblock mainloop, `DualEpilogue` writes `D0/D1/D2`, and `DualGemmMode` decides whether the third grid dimension means split-K slice or batch index.
**CN**: 组合关系：`DualMma` 负责融合后的 threadblock mainloop，`DualEpilogue` 负责写出 `D0/D1/D2`，而 `DualGemmMode` 决定网格第三维到底表示 split-K 切片还是 batch 索引。

**EN**: Kernel flow: map CTA → interpret mode → build iterators → run fused MMA for both GEMMs → serialize split-K if needed → emit `D0`, `D1`, and final `D2`.
**CN**: Kernel 流程：计算 CTA 映射 → 解释执行模式 → 构造迭代器 → 为两个 GEMM 运行融合 MMA → 必要时串行完成 split-K → 输出 `D0`、`D1` 和最终 `D2`。

## Dependencies / 依赖项
**EN**: CUTLASS dependency: `cutlass/cutlass.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/cutlass.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/gemm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/matrix_coord.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/matrix_coord.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/semaphore.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/semaphore.h` 提供该文件直接使用的库级原语。

**EN**: Project-local dependency: `../threadblock/dual_mma_multistage.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`../threadblock/dual_mma_multistage.h` 提供该文件直接包含的辅助代码。

**EN**: Project-local dependency: `../threadblock/dual_epilogue.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`../threadblock/dual_epilogue.h` 提供该文件直接包含的辅助代码。

**EN**: Project-local dependency: `../dual_gemm_common.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`../dual_gemm_common.h` 提供该文件直接包含的辅助代码。
