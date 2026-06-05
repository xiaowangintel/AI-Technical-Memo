# gemm_with_epilogue_visitor.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/35_gemm_softmax/gemm_with_epilogue_visitor.h`
**Purpose / 用途**: Defines an example-local GEMM kernel wrapper that reuses a CUTLASS MMA mainloop and replaces the normal epilogue with an epilogue visitor that emits partial max/sum statistics for softmax fusion. / 定义了一个示例级 GEMM 内核封装：复用 CUTLASS 的 MMA 主循环，并以 epilogue visitor 替换普通尾处理，在写回时输出 softmax 融合所需的局部最大值与求和统计。
---
## Line-by-Line Analysis / 逐行分析
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
    \brief GEMM kernel to support the epilogue visitor model 
    for customized softmax partial reduction epilogue fusion.

    This source file will likely be moved to `include/cutlass/gemm/kernel/` in the future once
    its usage has been stabilized. For now, it is included in this example to demonstrate
    some basic output fusion options.
*/
```
**EN**: The banner explains that this file is an experimental example kernel for epilogue-visitor fusion. It is intentionally kept beside the example instead of the stable CUTLASS include tree because the API is still being validated.
**CN**: 文件头说明这是一个用于 epilogue visitor 融合的实验性示例内核。它暂时放在示例目录而不是稳定的 CUTLASS 头文件树中，表示该 API 仍在验证与演进。

```cpp
#pragma once

#include "cutlass/cutlass.h"
#include "cutlass/fast_math.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/matrix_coord.h"
#include "cutlass/complex.h"
#include "cutlass/semaphore.h"

#include "cutlass/trace.h"
```
**EN**: These includes pull in the core CUTLASS abstractions needed to build a custom GEMM kernel: tensor references, coordinate math, complex-transform tags, fast math helpers, synchronization primitives, and tracing hooks.
**CN**: 这些头文件引入构建自定义 GEMM 内核所需的核心 CUTLASS 抽象：张量引用、坐标计算、复数变换标记、快速数学工具、同步原语以及调试跟踪接口。

```cpp
namespace cutlass {
namespace gemm {
namespace kernel {
```
**EN**: The implementation lives in cutlass::gemm::kernel, matching CUTLASS’s usual separation between threadblock kernels and higher-level device wrappers.
**CN**: 实现位于 cutlass::gemm::kernel 命名空间中，这与 CUTLASS 按“内核层 / 设备封装层”分层的惯例保持一致。

```cpp
template <
  typename Mma_,                  ///! Threadblock-scoped matrix multiply-accumulate
  typename Epilogue_,             ///! Epilogue
  typename ThreadblockSwizzle_    ///! Threadblock swizzling function
>
struct GemmWithEpilogueVisitor {
public:

  using Mma = Mma_;
  using Epilogue = Epilogue_;
  using EpilogueVisitor = typename Epilogue::Visitor;
  using ThreadblockSwizzle = ThreadblockSwizzle_;

  using ElementA = typename Mma::IteratorA::Element;
  using LayoutA = typename Mma::IteratorA::Layout;
  using TensorRefA = TensorRef<ElementA, LayoutA>;

  using ElementB = typename Mma::IteratorB::Element;
  using LayoutB = typename Mma::IteratorB::Layout;
  using TensorRefB = TensorRef<ElementB, LayoutB>;

  using ElementC = typename EpilogueVisitor::ElementOutput;
  using LayoutC = typename Epilogue::Layout;
  using TensorRefC = TensorRef<ElementC, LayoutC>;

  static ComplexTransform const kTransformA = Mma::kTransformA;
  static ComplexTransform const kTransformB = Mma::kTransformB;
  using Operator = typename Mma::Operator;

  using OperatorClass = typename Mma::Operator::OperatorClass;
  using ThreadblockShape = typename Mma::Shape;
  using WarpShape = typename Mma::Operator::Shape;
  using InstructionShape = typename Mma::Policy::Operator::InstructionShape;
  using ArchTag = typename Mma::ArchTag;

  using ElementNorm = typename EpilogueVisitor::ElementNorm;
  using ElementSum = typename EpilogueVisitor::ElementSum;

  static int const kStages = Mma::kStages;
  static int const kAlignmentA = Mma::IteratorA::AccessType::kElements;
  static int const kAlignmentB = Mma::IteratorB::AccessType::kElements;
  static int const kAlignmentC = EpilogueVisitor::kElementsPerAccess;

  /// Warp count (concept: GemmShape)
  using WarpCount = typename Mma::WarpCount;
  static int const kThreadCount = 32 * WarpCount::kCount;

  /// Split-K preserves splits that are 128b aligned
  static int const kSplitKAlignment = const_max(
    128 / sizeof_bits<ElementA>::value,
    128 / sizeof_bits<ElementB>::value
  );
```
**EN**: The template is parameterized by a threadblock-scoped MMA type, an epilogue type, and a swizzle policy. Most of this block derives element types, layouts, architecture tags, tile shapes, access alignments, and visitor-specific accumulator/output types from nested CUTLASS types, so the wrapper stays generic while still exposing the metadata needed by launch code and alignment checks.
**CN**: 该模板由线程块级 MMA 类型、epilogue 类型以及线程块映射（swizzle）策略参数化。本段大量通过 CUTLASS 的嵌套类型推导元素类型、布局、架构标签、tile 形状、向量访问对齐，以及 visitor 需要的归约/输出类型，使这个封装既保持通用性，又能向启动代码和对齐检查暴露必要元数据。

```cpp
  /// Argument structure
  struct Arguments {

    //
    // Data members
    //

    GemmUniversalMode mode;
    GemmCoord problem_size;
    int batch_count;

    TensorRefA ref_A;
    TensorRefB ref_B;
    TensorRefC ref_C;
    TensorRefC ref_D;

    ElementNorm *ptr_Max; 
    ElementSum  *ptr_Sum;

    int64_t    batch_stride_A;
    int64_t    batch_stride_B;

    typename EpilogueVisitor::Arguments epilogue_visitor;

    //
    // Methods
    //

    Arguments():
      mode(GemmUniversalMode::kGemm),
      batch_count(1)
    { }


    /// constructs an arguments structure
    Arguments(
      GemmUniversalMode mode_,
      GemmCoord problem_size_,
      int batch_count_,
      TensorRefA ref_A_,
      TensorRefB ref_B_,
      TensorRefC ref_C_,
      TensorRefC ref_D_,
      ElementNorm *ptr_Max_,
      ElementSum *ptr_Sum_,
      int64_t batch_stride_A_,
      int64_t batch_stride_B_,
      typename EpilogueVisitor::Arguments epilogue_visitor_
    ):
      mode(mode_),
      problem_size(problem_size_),
      batch_count(batch_count_),
      ref_A(ref_A_),
      ref_B(ref_B_),
      ref_C(ref_C_),
      ref_D(ref_D_),
      ptr_Max(ptr_Max_),
      ptr_Sum(ptr_Sum_),
      batch_stride_A(batch_stride_A_),
      batch_stride_B(batch_stride_B_),
      epilogue_visitor(epilogue_visitor_)
    {

    }
  };
```
**EN**: Arguments is the host-facing runtime interface. It carries the GEMM mode, M/N/K size, batch count, tensor references for A/B/C/D, workspace pointers for partial row maxima and sums, batch strides, and any extra visitor arguments. The constructor simply packages everything so the call site does not need to populate fields one by one.
**CN**: Arguments 是面向主机侧的运行时接口。它携带 GEMM 模式、M/N/K 尺寸、batch 数、A/B/C/D 的 TensorRef、行最大值与行和的临时工作区指针、batch 跨距，以及 visitor 额外参数。构造函数只是把这些信息打包起来，避免调用端逐字段填写。

```cpp
  /// Parameters structure
  struct Params {

    cutlass::gemm::GemmCoord problem_size;
    cutlass::gemm::GemmCoord grid_tiled_shape;
    int swizzle_log_tile;

    typename Mma::IteratorA::Params params_A;
    typename Mma::IteratorB::Params params_B;
    typename EpilogueVisitor::OutputTileIterator::Params params_C;
    typename EpilogueVisitor::OutputTileIterator::Params params_D;

    GemmUniversalMode mode;
    int batch_count;
    int gemm_k_size;

    void * ptr_A;
    void * ptr_B;
    ElementC * ptr_C;
    ElementC * ptr_D;

    ElementNorm * ptr_Max;
    ElementSum * ptr_Sum;

    int64_t batch_stride_A;
    int64_t batch_stride_B;

    typename EpilogueVisitor::Params epilogue_visitor;

    //
    // Methods
    //

    CUTLASS_HOST_DEVICE
    Params():
      swizzle_log_tile(0),
      params_A(0),
      params_B(0),
      params_C(0),
      params_D(0),
      batch_count(0),
      gemm_k_size(0),
      mode(cutlass::gemm::GemmUniversalMode::kGemm),
      ptr_A(nullptr),
      ptr_B(nullptr),
      ptr_C(nullptr),
      ptr_D(nullptr),
      ptr_Max(nullptr),
      ptr_Sum(nullptr),
      batch_stride_A(0),
      batch_stride_B(0)
    { }


    Params(
      Arguments const &args
    ):
      problem_size(args.problem_size),
      swizzle_log_tile(0),
      params_A(args.ref_A.layout()),
      params_B(args.ref_B.layout()),
      params_C(args.ref_C.layout()),
      params_D(args.ref_D.layout()),
      mode(args.mode),
      batch_count(args.batch_count),
      gemm_k_size(args.problem_size.k()),
      ptr_A(args.ref_A.data()),
      ptr_B(args.ref_B.data()),
      ptr_C(args.ref_C.data()),
      ptr_D(args.ref_D.data()),
      ptr_Max(args.ptr_Max),
      ptr_Sum(args.ptr_Sum),
      batch_stride_A(args.batch_stride_A),
      batch_stride_B(args.batch_stride_B),
      epilogue_visitor(args.epilogue_visitor)
    {

      ThreadblockSwizzle threadblock_swizzle;

      grid_tiled_shape = threadblock_swizzle.get_tiled_shape(
        args.problem_size,
        {ThreadblockShape::kM, ThreadblockShape::kN, ThreadblockShape::kK},
        args.batch_count);

      if (args.mode == GemmUniversalMode::kGemm || args.mode == GemmUniversalMode::kGemmSplitKParallel) {

        int const kAlignK = const_max(const_max(128 / sizeof_bits<ElementA>::value, 128 / sizeof_bits<ElementB>::value), 1);

        gemm_k_size = round_up(ceil_div(args.problem_size.k(), args.batch_count), kAlignK);

        if (gemm_k_size) {
          grid_tiled_shape.k() = ceil_div(args.problem_size.k(), gemm_k_size);
        }
      }

      swizzle_log_tile = threadblock_swizzle.get_log_tile(grid_tiled_shape);
    }
  };
```
**EN**: Params is the precomputed device-parameter form of Arguments. It turns layouts into iterator parameter objects, caches raw pointers, computes the tiled launch shape through the swizzle object, and rounds the effective Split-K partition size to a 128-bit-friendly boundary so vectorized accesses remain legal.
**CN**: Params 是 Arguments 的设备侧预计算版本。它把布局对象转换成迭代器参数，缓存原始指针，通过 swizzle 计算 tile 化后的网格形状，并把 Split-K 的有效 K 分块尺寸向 128-bit 友好边界对齐，从而保证向量化访存合法。

```cpp
  /// Shared memory storage structure
  union SharedStorage {

    typename Mma::SharedStorage main_loop;

    struct {
      typename Epilogue::SharedStorage epilogue;
      typename EpilogueVisitor::SharedStorage visitor;
    } epilogue;
  };
```
**EN**: SharedStorage uses a union to reuse the same shared-memory region for the mainloop and the epilogue/visitor phase. This is a common CUTLASS kernel-composition technique: different phases run sequentially, so their scratch space can overlap.
**CN**: SharedStorage 通过 union 复用同一块共享内存：主循环阶段使用一部分结构，epilogue/visitor 阶段使用另一部分结构。这是 CUTLASS 常见的内核组合技巧，因为不同阶段按顺序执行，所以其暂存空间可以重叠。

```cpp
  CUTLASS_DEVICE
  GemmWithEpilogueVisitor() { }
```
**EN**: The kernel object itself is stateless on device; construction is trivial because all real state is passed through Params and SharedStorage.
**CN**: 设备侧内核对象本身几乎无状态；构造函数为空，因为真正的运行状态都通过 Params 与 SharedStorage 传入。

```cpp
  static Status can_implement(
    cutlass::gemm::GemmCoord const & problem_size) {

    CUTLASS_TRACE_HOST("GemmWithEpilogueVisitor::can_implement()");

    static int const kAlignmentA = Mma::IteratorA::AccessType::kElements;
    static int const kAlignmentB = Mma::IteratorB::AccessType::kElements;
    static int const kAlignmentC = Epilogue::OutputTileIterator::kElementsPerAccess;

    bool isAMisaligned = false;
    bool isBMisaligned = false;
    bool isCMisaligned = false;

    if (platform::is_same<LayoutA, layout::RowMajor>::value) {
      isAMisaligned = problem_size.k() % kAlignmentA;
    } else if (platform::is_same<LayoutA, layout::ColumnMajor>::value) {
      isAMisaligned = problem_size.m() % kAlignmentA;
    } else if (platform::is_same<LayoutA, layout::ColumnMajorInterleaved<32>>::value
            || platform::is_same<LayoutA, layout::ColumnMajorInterleaved<64>>::value) {
      isAMisaligned = problem_size.k() % kAlignmentA;
    }

    if (platform::is_same<LayoutB, layout::RowMajor>::value) {
      isBMisaligned = problem_size.n() % kAlignmentB;
    } else if (platform::is_same<LayoutB, layout::ColumnMajor>::value) {
      isBMisaligned = problem_size.k() % kAlignmentB;
    } else if (platform::is_same<LayoutB, layout::RowMajorInterleaved<32>>::value
            || platform::is_same<LayoutB, layout::RowMajorInterleaved<64>>::value) {
      isBMisaligned = problem_size.k() % kAlignmentB;
    }

    if (platform::is_same<LayoutC, layout::RowMajor>::value) {
      isCMisaligned = problem_size.n() % kAlignmentC;
    } else if (platform::is_same<LayoutC, layout::ColumnMajor>::value) {
      isCMisaligned = problem_size.m() % kAlignmentC;
    } else if (platform::is_same<LayoutC, layout::ColumnMajorInterleaved<32>>::value
            || platform::is_same<LayoutC, layout::ColumnMajorInterleaved<64>>::value) {
      isCMisaligned = problem_size.n() % kAlignmentC;
    }

    if (isAMisaligned) {
      CUTLASS_TRACE_HOST("  returning kErrorMisalignedOperand for A operand");
      return Status::kErrorMisalignedOperand;
    }

    if (isBMisaligned) {
      CUTLASS_TRACE_HOST("  returning kErrorMisalignedOperand for B operand");
      return Status::kErrorMisalignedOperand;
    }

    if (isCMisaligned) {
      CUTLASS_TRACE_HOST("  returning kErrorMisalignedOperand for C operand");
      return Status::kErrorMisalignedOperand;
    }

    CUTLASS_TRACE_HOST("  returning kSuccess");

    return Status::kSuccess;
  }

  static Status can_implement(Arguments const &args) {
    return can_implement(args.problem_size);
  }
```
**EN**: can_implement() validates vector-access alignment against the logical problem size. Which dimension must be aligned depends on layout: for row-major A the contiguous dimension is K, for row-major B/C it is N, and for column-major variants it changes accordingly. Rejecting misaligned problems up front protects the iterator types, which assume accesses in units of kAlignmentA/B/C elements.
**CN**: can_implement() 根据问题规模检查向量访存对齐是否满足要求。需要对齐的维度取决于布局：例如 row-major 的 A 连续维度是 K，row-major 的 B/C 连续维度是 N，而 column-major 变体则对应不同维度。提前拒绝未对齐问题，是为了保护后续迭代器，因为它们默认按 kAlignmentA/B/C 个元素为单位访问。

```cpp
  #define SPLIT_K_ENABLED 1

  /// Executes one GEMM
  CUTLASS_DEVICE
  void operator()(Params const &params, SharedStorage &shared_storage) {

    // Compute threadblock location
    ThreadblockSwizzle threadblock_swizzle;

    cutlass::gemm::GemmCoord threadblock_tile_offset = threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);

    // Early exit if CTA is out of range
    if (params.grid_tiled_shape.m() <= threadblock_tile_offset.m() ||
      params.grid_tiled_shape.n() <= threadblock_tile_offset.n()) {

      return;
    }

    int offset_k = 0;
    int problem_size_k = params.problem_size.k();

    ElementA *ptr_A = static_cast<ElementA *>(params.ptr_A);
    ElementB *ptr_B = static_cast<ElementB *>(params.ptr_B);


    #if SPLIT_K_ENABLED
    //
    // Fetch pointers based on mode.
    //
    if (params.mode == GemmUniversalMode::kGemm ||
      params.mode == GemmUniversalMode::kGemmSplitKParallel) {

      if (threadblock_tile_offset.k() + 1 < params.grid_tiled_shape.k()) {

        problem_size_k = (threadblock_tile_offset.k() + 1) * params.gemm_k_size;
      }

      offset_k = threadblock_tile_offset.k() * params.gemm_k_size;
    }
    else if (params.mode == GemmUniversalMode::kBatched) {
      ptr_A += threadblock_tile_offset.k() * params.batch_stride_A;
      ptr_B += threadblock_tile_offset.k() * params.batch_stride_B;
    }
    else if (params.mode == GemmUniversalMode::kArray) {
      ptr_A = static_cast<ElementA * const *>(params.ptr_A)[threadblock_tile_offset.k()];
      ptr_B = static_cast<ElementB * const *>(params.ptr_B)[threadblock_tile_offset.k()];
    }
    #endif
```
**EN**: operator() begins by mapping the CTA to a GEMM tile through ThreadblockSwizzle, exits early when the tile is outside the logical grid, and then adjusts the A/B pointers according to GemmUniversalMode. This unified path supports plain GEMM, Split-K GEMM, strided batched GEMM, and array-of-pointers GEMM without changing the main MMA body.
**CN**: operator() 首先通过 ThreadblockSwizzle 把当前 CTA 映射到某个 GEMM tile；若 tile 超出逻辑网格则直接返回。随后它根据 GemmUniversalMode 调整 A/B 指针，从而在不改动核心 MMA 主体的前提下，同时支持普通 GEMM、Split-K GEMM、步长 batched GEMM 和指针数组 GEMM。

```cpp
    // Compute initial location in logical coordinates
    cutlass::MatrixCoord tb_offset_A{
      threadblock_tile_offset.m() * Mma::Shape::kM,
      offset_k,
    };

    cutlass::MatrixCoord tb_offset_B{
      offset_k,
      threadblock_tile_offset.n() * Mma::Shape::kN
    };

    // Compute position within threadblock
    int thread_idx = threadIdx.x;

    // Construct iterators to A and B operands
    typename Mma::IteratorA iterator_A(
      params.params_A,
      ptr_A,
      {params.problem_size.m(), problem_size_k},
      thread_idx,
      tb_offset_A);

    typename Mma::IteratorB iterator_B(
      params.params_B,
      ptr_B,
      {problem_size_k, params.problem_size.n()},
      thread_idx,
      tb_offset_B);
```
**EN**: This block computes the logical tile offsets for A and B and constructs IteratorA / IteratorB. In CUTLASS, these iterators encapsulate layout-specific address arithmetic, per-thread access patterns, masking, and vector width, so the surrounding kernel logic can stay high level.
**CN**: 这一段计算 A/B tile 的逻辑起始坐标，并构造 IteratorA / IteratorB。在 CUTLASS 中，这些迭代器封装了与布局相关的地址计算、线程级访问模式、边界谓词以及向量宽度，因此外围内核逻辑可以保持较高抽象。

```cpp
    // Broadcast the warp_id computed by lane 0 to ensure dependent code
    // is compiled as warp-uniform.
    int warp_idx = __shfl_sync(0xffffffff, threadIdx.x / 32, 0);

    int lane_idx = threadIdx.x % 32;

    //
    // Main loop
    //

    // Construct thread-scoped matrix multiply
    Mma mma(shared_storage.main_loop, thread_idx, warp_idx, lane_idx);

    typename Mma::FragmentC accumulators;

    accumulators.clear();

    // Compute threadblock-scoped matrix multiply-add
    int gemm_k_iterations = (problem_size_k - offset_k + Mma::Shape::kK - 1) / Mma::Shape::kK;

    // Compute threadblock-scoped matrix multiply-add
    mma(
      gemm_k_iterations,
      accumulators,
      iterator_A,
      iterator_B,
      accumulators);
```
**EN**: The code derives warp and lane IDs, constructs the threadblock MMA object, clears the accumulator fragment, computes how many K-tiles remain for this partition, and runs the main matrix-multiply loop. Mma is the core policy type that already knows how to orchestrate warp-level Tensor Core operations.
**CN**: 代码在这里推导 warp 与 lane 编号，构造线程块级 MMA 对象，清零累加器片段，计算当前分区需要处理的 K tile 数量，并执行主矩阵乘循环。Mma 是核心策略类型，内部已经定义了如何组织 warp 级 Tensor Core 运算。

```cpp
    threadblock_tile_offset = threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);

    //assume identity swizzle
    MatrixCoord threadblock_offset(
      threadblock_tile_offset.m() * Mma::Shape::kM,
      threadblock_tile_offset.n() * Mma::Shape::kN
    );

    int block_idx = threadblock_tile_offset.m() + threadblock_tile_offset.n() * params.grid_tiled_shape.m();

    //
    // Construct the epilogue visitor
    //

    EpilogueVisitor epilogue_visitor(
      params.epilogue_visitor,
      shared_storage.epilogue.visitor,
      params.problem_size.mn(),
      thread_idx,
      warp_idx,
      lane_idx,
      params.params_C,
      params.params_D,
      params.ptr_C,
      params.ptr_D,
      params.ptr_Max,
      params.ptr_Sum,
      threadblock_offset,
      blockIdx.y *params.problem_size.m() );
```
**EN**: After the mainloop finishes, the kernel rebuilds the output tile coordinate and constructs an EpilogueVisitor. The visitor receives output iterator params, raw C/D pointers, partial max/sum buffers, problem extents, and tile offsets so it can fuse writeback with row-wise statistics collection needed for softmax.
**CN**: 主循环结束后，内核重新计算输出 tile 坐标，并构造一个 EpilogueVisitor。visitor 会拿到输出迭代器参数、原始 C/D 指针、局部最大值/求和缓冲区、问题尺寸以及 tile 偏移，从而把写回过程与 softmax 所需的逐行统计收集融合起来。

```cpp
    if (params.mode == GemmUniversalMode::kGemm) {
      // Indicate which position in a serial reduction the output operator is currently updating
      epilogue_visitor.set_k_partition(threadblock_tile_offset.k(), params.grid_tiled_shape.k());
    }
    else if (params.mode == GemmUniversalMode::kBatched || params.mode == GemmUniversalMode::kArray) {
      epilogue_visitor.set_batch_index(threadblock_tile_offset.k());
    }
```
**EN**: The visitor needs different state depending on execution mode. In single GEMM / Split-K mode it tracks which K partition is being reduced; in batched or array mode it instead records the batch index so loads and stores land in the correct slice.
**CN**: visitor 在不同执行模式下需要不同的状态。在普通 GEMM / Split-K 模式下，它要知道当前是第几个 K 分区；在 batched 或 array 模式下，它则记录 batch 索引，以便读写落到正确的批次切片。

```cpp
    Epilogue epilogue(
      shared_storage.epilogue.epilogue,
      thread_idx,
      warp_idx,
      lane_idx);

    // Execute the epilogue operator to update the destination tensor.
    epilogue(epilogue_visitor, accumulators);
  }
```
**EN**: Finally, the wrapper constructs the epilogue object and runs it with the visitor plus accumulator fragment. This is the key customization point: the main MMA path remains standard CUTLASS code, while output behavior is specialized through the visitor pattern.
**CN**: 最后，封装层构造 epilogue 对象，并以 visitor 和累加器片段为输入执行它。这正是核心定制点：MMA 主路径保持标准 CUTLASS 实现，而输出阶段则通过 visitor 模式实现专门化。

```cpp
} // namespace kernel
} // namespace gemm
} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The file closes the nested namespaces and leaves behind a reusable kernel building block for the higher-level softmax pipeline.
**CN**: 文件在这里结束命名空间，并把这个可复用的内核构件留给更高层的 softmax 组合流水线使用。

---
## Key Concepts / 关键概念
- Template-based kernel composition / 基于模板的内核组合
- Threadblock MMA mainloop reuse / 线程块级 MMA 主循环复用
- Epilogue visitor fusion for row-wise max and sum / 用于逐行最大值与求和的 epilogue visitor 融合
- Layout-aware alignment validation / 面向布局的对齐校验
- Universal GEMM modes: regular, Split-K, batched, array / 通用 GEMM 模式：普通、Split-K、batched、指针数组
- Shared-memory overlap between phases / 多阶段共享内存复用
## Dependencies / 依赖项
- `cutlass/cutlass.h` — core CUTLASS types, Status, TensorRef, and utility macros / CUTLASS 核心类型、Status、TensorRef 与基础宏
- `cutlass/fast_math.h` — helpers such as round_up and ceil_div used in tile sizing / 提供 round_up、ceil_div 等 tile 尺寸计算工具
- `cutlass/gemm/gemm.h` — GEMM coordinates, modes, and common GEMM abstractions / GEMM 坐标、模式与通用抽象
- `cutlass/matrix_coord.h` — row/column coordinate helpers for tile offsets / tile 偏移所需的矩阵坐标工具
- `cutlass/complex.h` — ComplexTransform tags inherited from the underlying MMA / 提供从底层 MMA 继承的 ComplexTransform 标记
- `cutlass/semaphore.h` — synchronization primitive included by the example kernel stack / 示例内核栈中使用的同步原语头
- `cutlass/trace.h` — host-side tracing macros used by can_implement() / can_implement() 使用的主机侧跟踪宏
