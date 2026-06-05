# custom_mma_pipelined.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/gemm/custom_mma_pipelined.h`
**Purpose / 用途**: Two-stage pipelined threadblock MMA used when the kernel follows the classic double-buffer schedule instead of the cp.async multistage path / 在不走 cp.async 多级路径时使用的两级流水 threadblock MMA，实现经典双缓冲调度
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
    \brief Template for a double-buffered threadblock-scoped GEMM kernel.
*/

#pragma once

#include "cutlass/aligned_buffer.h"
#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include "cutlass/numeric_conversion.h"

#include "cutlass/matrix_shape.h"
#include "cutlass/numeric_types.h"

#include "custom_mma_base.h"
#include "cutlass/gemm/gemm.h"
```
**EN**: Lines 1-47. The prologue of the file contains licensing text, basic numeric/shape includes, and the dependency on CustomMmaBase. The include list is noticeably lighter than the multistage version because this class does not need cp.async-specific machinery.
**CN**: 第1-47行：文件前部是许可证文本、基础数值/形状头文件，以及对 CustomMmaBase 的依赖。相比 multistage 版本，这里的 include 更精简，因为该类不需要 cp.async 专用机制。
```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace gemm {
namespace threadblock {

/////////////////////////////////////////////////////////////////////////////////////////////////

/// Structure to compute the matrix product targeting CUDA cores and SIMT math
/// instructions.
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
    /// Iterates over tiles of B operand in global memory
    //  (concept: ReadableTileIterator | ForwardTileIterator |
    //  MaskedTileIterator)
    typename IteratorB_,
    /// Iterates over tiles of B operand in shared memory
    /// (concept: WriteableTileIterator | RandomAccessTileIterator)
    typename SmemIteratorB_,
    /// Data type of accumulator matrix
    typename ElementC_,
    /// Data type of accumulator matrix
    typename LayoutC_,
    /// Policy describing tuning details (concept: MmaPolicy)
    typename Policy_,
    /// Transformation applied to A operand
    typename TransformA_ = NumericArrayConverter<
        typename SmemIteratorA_::Element,
        typename IteratorA_::Element,
        IteratorA_::Fragment::kElements>,
    ///
    /// Transformation applied to B operand
    typename TransformB_ = NumericArrayConverter<
        typename SmemIteratorB_::Element,
        typename IteratorB_::Element,
        IteratorB_::Fragment::kElements>,
    /// Used for partial specialization
    typename Enable = bool>
class CustomMmaPipelined : public CustomMmaBase<Shape_, Policy_, 2> {
```
**EN**: Lines 48-94. The template parameters describe a classic two-stage threadblock pipeline: global-memory iterators for A/B, shared-memory iterators for A/B, accumulator/layout types, an MMA policy, and optional elementwise transforms applied before storing into shared memory. The inheritance from CustomMmaBase<..., 2> bakes the double-buffer depth into the type.
**CN**: 第48-94行：模板参数描述了一个经典的两级 threadblock 流水：A/B 的全局内存迭代器、A/B 的共享内存迭代器、累加器/布局类型、MMA policy，以及在写入共享内存前施加的可选逐元素变换。继承自 CustomMmaBase<..., 2> 则把双缓冲深度直接编码进类型中。
```cpp
 public:
  ///< Base class
  using Base = CustomMmaBase<Shape_, Policy_, 2>;

  using Shape =
      Shape_; ///< Size of the Gemm problem - concept: gemm::GemmShape<>
  using IteratorA =
      IteratorA_; ///< Iterates over tiles of A operand in global memory
  using IteratorB =
      IteratorB_; ///< Iterates over tiles of B operand in global memory
  using ElementC = ElementC_; ///< Data type of accumulator matrix
  using LayoutC = LayoutC_; ///< Layout of accumulator matrix
  using Policy = Policy_; ///< Policy describing tuning details

  using SmemIteratorA = SmemIteratorA_;
  using SmemIteratorB = SmemIteratorB_;

  using TransformA = TransformA_;
  using TransformB = TransformB_;

  //
  // Dependent types
  //

  /// Fragment of operand A loaded from global memory
  using FragmentA = typename IteratorA::Fragment;

  /// Fragment of operand B loaded from global memory
  using FragmentB = typename IteratorB::Fragment;

  /// Fragment of accumulator tile
  using FragmentC = typename Policy::Operator::FragmentC;

  /// Warp-level Mma
  using Operator = typename Policy::Operator;

  /// Obtain the arch tag from the warp-level operator
  using ArchTag = typename Policy::Operator::ArchTag;

  /// Complex transform on A operand
  static ComplexTransform const kTransformA = Operator::kTransformA;

  /// Complex transform on B operand
  static ComplexTransform const kTransformB = Operator::kTransformB;

  // staticaly assert kStages for MmaPipelined is two (Double-buffered pipeline)
  static_assert(
      (Base::kStages == 2),
      "MmaPipelined requires kStages set to value 2");

  static bool const kSmemContainsEntireMat = false;

 private:
  using WarpFragmentA = typename Operator::FragmentA;
  using WarpFragmentB = typename Operator::FragmentB;
```
**EN**: Lines 95-150. This section declares the important aliases used by the execution loop: threadblock fragments from global memory, warp fragments from shared memory, the warp-level operator, architecture tag, and complex-transform metadata. The static_assert enforces the central design assumption that MmaPipelined is always exactly two stages.
**CN**: 第95-150行：这一段声明了执行循环会用到的关键别名：来自全局内存的 threadblock fragment、来自共享内存的 warp fragment、warp 级算子、架构标签，以及复数变换元数据。static_assert 强制了核心设计前提：MmaPipelined 永远只能是两级流水。
```cpp
 protected:
  /// Iterator to write threadblock-scoped tile of A operand to shared memory
  SmemIteratorA smem_iterator_A_;

  /// Iterator to write threadblock-scoped tile of B operand to shared memory
  SmemIteratorB smem_iterator_B_;

 public:
  /// Construct from tensor references
  CUTLASS_DEVICE
  CustomMmaPipelined(
      typename Base::SharedStorageA& shared_storageA,
      typename Base::SharedStorageB& shared_storageB,
      int thread_idx, ///< ID within the threadblock
      int warp_idx, ///< ID of warp
      int lane_idx ///< ID of each thread within a warp
      )
      : Base(shared_storageA, shared_storageB, thread_idx, warp_idx, lane_idx),
        smem_iterator_A_(shared_storageA.ref(), thread_idx),
        smem_iterator_B_(shared_storageB.ref(), thread_idx) {
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
    this->warp_tile_iterator_B_.add_tile_offset(
        {Base::kWarpGemmIterations * warp_idx_k, warp_idx_n});
  }
  CUTLASS_DEVICE
  CustomMmaPipelined(
      ///< Shared storage needed for internal use by threadblock-scoped GEMM
      typename Base::SharedStorage& st,
      ///< ID within the threadblock
      int thread_idx,
      ///< ID of warp
      int warp_idx,
      ///< ID of each thread within a warp
      int lane_idx)
      : CustomMmaPipelined(
            st.operand_A,
            st.operand_B,
            thread_idx,
            warp_idx,
            lane_idx) {}
```
**EN**: Lines 151-205. The class adds shared-memory write iterators and constructors that bind them to the per-thread shared-memory view. The constructor also maps warp_idx into CTA-space (m, n, k) coordinates and offsets the inherited warp tile iterators so each warp reads the correct tile inside the threadblock.
**CN**: 第151-205行：该类加入了共享内存写迭代器，并在构造函数中把它们绑定到每个线程对应的共享内存视图。构造函数还会把 warp_idx 映射到 CTA 空间中的 (m, n, k) 坐标，并调整继承而来的 warp tile 迭代器，使每个 warp 都能读取 threadblock 内正确的 tile。
```cpp
  CUTLASS_DEVICE
  bool set_prologue_done(bool value) {
    // NOT IMPLEMENTED FOR PIPELINED
  }

  CUTLASS_DEVICE
  bool set_zero_outside_bounds(bool value) {
    // NOT NEEDED FOR PIPELINED
    // shared memory will always be zero-filled
  }

  template <bool kLoadA = true, bool kLoadB = true>
  CUTLASS_DEVICE static void prologue(
      typename Base::SharedStorage& shared_storage,
      ///< iterator over A operand in global memory
      IteratorA iterator_A,
      ///< iterator over B operand in global memory
      IteratorB iterator_B,
      int thread_idx,
      int problem_size_k) {
    prologue<kLoadA, kLoadB>(
        shared_storage.operand_A,
        shared_storage.operand_B,
        iterator_A,
        iterator_B,
        thread_idx,
        problem_size_k);
  }

  template <bool kLoadA = true, bool kLoadB = true>
  CUTLASS_DEVICE static void prologue(
      typename Base::SharedStorageA& shared_storageA,
      typename Base::SharedStorageB& shared_storageB,
      ///< iterator over A operand in global memory
      IteratorA iterator_A,
      ///< iterator over B operand in global memory
      IteratorB iterator_B,
      int thread_idx,
      int problem_size_k) {
    // NOT IMPLEMENTED FOR PIPELINED
  }
```
**EN**: Lines 206-247. These methods show the limits of this implementation. set_prologue_done() and the public prologue() overload are intentionally left unimplemented, and set_zero_outside_bounds() is documented as unnecessary because this path assumes shared memory will be zero-filled through its normal flow. In other words, the pipelined kernel does not expose the same external prefetch/zeroing controls as the multistage kernel.
**CN**: 第206-247行：这些方法体现了该实现的边界：set_prologue_done() 和公开的 prologue() 重载都被刻意留空，而 set_zero_outside_bounds() 也注明“不需要”，因为该路径假设共享内存会在正常流程中被零填充。换句话说，pipelined 内核并不像 multistage 那样暴露外部预取/清零控制能力。
```cpp
  /// Perform a threadblock-scoped matrix multiply-accumulate
  CUTLASS_DEVICE
  void operator()(
      int gemm_k_iterations, ///< number of iterations of the mainloop
      FragmentC& accum, ///< destination accumulator tile
      IteratorA iterator_A, ///< iterator over A operand in global memory
      IteratorB iterator_B, ///< iterator over B operand in global memory
      FragmentC const& src_accum, ///< source accumulator tile
      TransformA transform_A =
          TransformA(), ///< transformation applied to A fragment
      TransformB transform_B =
          TransformB()) { ///< transformation applied to B fragment

    //
    // Prologue
    //

    // Perform accumulation in the 'd' output operand
    accum = src_accum;

    FragmentA tb_frag_A;
    FragmentB tb_frag_B;

    tb_frag_A.clear();
    tb_frag_B.clear();

    // The last kblock is loaded in the prolog
    iterator_A.load(tb_frag_A);
    iterator_B.load(tb_frag_B);

    ++iterator_A;
    ++iterator_B;

    this->smem_iterator_A_.store(transform_A(tb_frag_A));
    this->smem_iterator_B_.store(transform_B(tb_frag_B));

    ++this->smem_iterator_A_;
    ++this->smem_iterator_B_;

    __syncthreads();

    // Pair of fragments used to overlap shared memory loads and math
    // instructions
    WarpFragmentA warp_frag_A[2];
    WarpFragmentB warp_frag_B[2];

    this->warp_tile_iterator_A_.set_kgroup_index(0);
    this->warp_tile_iterator_B_.set_kgroup_index(0);

    this->warp_tile_iterator_A_.load(warp_frag_A[0]);
    this->warp_tile_iterator_B_.load(warp_frag_B[0]);

    ++this->warp_tile_iterator_A_;
    ++this->warp_tile_iterator_B_;

    Operator warp_mma;

    int smem_write_stage_idx = 1;

    // Avoid reading out of bounds
    iterator_A.clear_mask(gemm_k_iterations <= 1);
    iterator_B.clear_mask(gemm_k_iterations <= 1);
```
**EN**: Lines 248-310. The first half of operator() performs the standard pipeline setup. It seeds the accumulator from src_accum, loads one threadblock fragment of A and B from global memory, stores those fragments into shared memory, synchronizes the CTA, primes the first warp fragments from shared memory, and initializes bookkeeping such as smem_write_stage_idx and the out-of-bounds masks.
**CN**: 第248-310行：operator() 的前半部分完成标准流水准备：先用 src_accum 初始化累加器，再从全局内存加载一组 threadblock 级 A/B fragment，写入共享内存并同步 CTA，随后从共享内存预装载第一组 warp fragment，并初始化 smem_write_stage_idx 和越界掩码等状态。
```cpp
    // Issue loads during the first warp-level matrix multiply-add *AFTER*
    // issuing shared memory loads (which have the tightest latency
    // requirement).

    //
    // Mainloop
    //

    // Note: The main loop does not support Base::kWarpGemmIterations == 2.
    CUTLASS_GEMM_LOOP
    for (; gemm_k_iterations > 0; --gemm_k_iterations) {
      //
      // Loop over GEMM K dimension
      //

      CUTLASS_PRAGMA_UNROLL
      for (int warp_mma_k = 0; warp_mma_k < Base::kWarpGemmIterations;
           ++warp_mma_k) {
        // Load warp-level tiles from shared memory, wrapping to k offset if
        // this is the last group as the case may be.

        if (warp_mma_k == Base::kWarpGemmIterations - 1) {
          // Write fragments to shared memory
          this->smem_iterator_A_.store(transform_A(tb_frag_A));

          this->smem_iterator_B_.store(transform_B(tb_frag_B));

          __syncthreads();

          ++this->smem_iterator_A_;
          ++this->smem_iterator_B_;

          // Add negative offsets to return iterators to the 'start' of the
          // circular buffer in shared memory
          if (smem_write_stage_idx == 1) {
            this->smem_iterator_A_.add_tile_offset({0, -Base::kStages});
            this->smem_iterator_B_.add_tile_offset({-Base::kStages, 0});
          } else {
            this->warp_tile_iterator_A_.add_tile_offset(
                {0,
                 -Base::kStages * Policy::kPartitionsK *
                     Base::kWarpGemmIterations});
            this->warp_tile_iterator_B_.add_tile_offset(
                {-Base::kStages * Policy::kPartitionsK *
                     Base::kWarpGemmIterations,
                 0});
          }

          smem_write_stage_idx ^= 1;
        }

        this->warp_tile_iterator_A_.set_kgroup_index(
            (warp_mma_k + 1) % Base::kWarpGemmIterations);
        this->warp_tile_iterator_B_.set_kgroup_index(
            (warp_mma_k + 1) % Base::kWarpGemmIterations);

        this->warp_tile_iterator_A_.load(warp_frag_A[(warp_mma_k + 1) % 2]);
        this->warp_tile_iterator_B_.load(warp_frag_B[(warp_mma_k + 1) % 2]);

        ++this->warp_tile_iterator_A_;
        ++this->warp_tile_iterator_B_;

        if (warp_mma_k == 0) {
          iterator_A.load(tb_frag_A);
          iterator_B.load(tb_frag_B);

          ++iterator_A;
          ++iterator_B;

          // Avoid reading out of bounds if this was the last loop iteration
          iterator_A.clear_mask(gemm_k_iterations <= 2);
          iterator_B.clear_mask(gemm_k_iterations <= 2);
        }

        warp_mma(
            accum,
            warp_frag_A[warp_mma_k % 2],
            warp_frag_B[warp_mma_k % 2],
            accum);
      }
    }
  }
};

/////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace gemm
} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Lines 311-401. The mainloop overlaps double-buffered warp reads with regular shared-memory writes. Near the end of each warp-level K-group it commits the next threadblock fragments to shared memory, synchronizes, wraps either the write iterators or the warp read iterators when the two-stage circular buffer flips, fetches the next global-memory fragments at warp_mma_k == 0, and executes the warp MMA on alternating fragment buffers. The comment about kWarpGemmIterations == 2 highlights that this scheduling pattern assumes more than two warp-level K groups.
**CN**: 第311-401行：主循环通过双缓冲 warp 读取与常规共享内存写入实现重叠。在每个 warp 级 K group 接近结束时，它会把下一批 threadblock fragment 提交到共享内存、执行同步，并在两级循环缓冲翻转时回卷写迭代器或 warp 读迭代器；同时在 warp_mma_k == 0 时抓取下一批全局内存 fragment，并在交替的 fragment 缓冲上执行 warp MMA。关于 kWarpGemmIterations == 2 的注释说明，这种调度模式默认假设 warp 级 K group 数量大于 2。
---
## Key Concepts / 关键概念
- CustomMmaPipelined is the fixed two-stage alternative to the cp.async multistage kernel, favoring simpler control flow over maximum flexibility. / CustomMmaPipelined 是 cp.async 多级内核的固定两级替代方案，它更偏向简单控制流，而不是最大灵活性。
- Its pipeline is driven by synchronous threadblock loads/stores and CTA barriers rather than explicit cp.async fences and waits. / 它的流水依赖同步的 threadblock 读写与 CTA barrier，而不是显式的 cp.async fence/wait。
- The implementation intentionally omits external prologue management features, so the object expects to own its complete data movement schedule. / 该实现有意省略了外部 prologue 管理能力，因此对象默认自己掌控完整的数据搬运调度。
## Dependencies / 依赖项
- `custom_mma_base.h` — Shares the common staged storage and warp iterator scaffold / 共享通用的分级存储与 warp 迭代器骨架
- `cutlass/gemm/gemm.h` — Provides the GEMM shape metadata that sizes the double buffer / 提供用于确定双缓冲尺寸的 GEMM 形状元数据
- `cutlass/numeric_conversion.h` — Supports predicate-controlled conversion and zero-fill paths / 支持带谓词控制的类型转换与零填充路径
- `cutlass/array.h` — Carries fragment containers moved between iterators and warp MMA operators / 承载在迭代器与 warp MMA 算子之间传递的片段容器
