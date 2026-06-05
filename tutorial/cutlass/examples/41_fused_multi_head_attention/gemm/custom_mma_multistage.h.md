# custom_mma_multistage.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/gemm/custom_mma_multistage.h`
**Purpose / 用途**: cp.async multistage threadblock MMA used by FMHA, handling staged global-to-shared prefetch, circular buffering, and FastF32-friendly policies / FMHA 使用的 cp.async 多级 threadblock MMA，负责分级 global-to-shared 预取、循环缓冲和适配 FastF32 的策略
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
#include "cutlass/arch/cache_operation.h"
#include "cutlass/arch/memory.h"
#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/numeric_types.h"

#include "custom_mma_base.h"
```
**EN**: Lines 1-47. The file begins with the usual legal text and then pulls in the pieces needed for an asynchronous multistage pipeline: cache-operation tags, low-level memory primitives, GEMM shapes, numeric utilities, and the shared scaffolding from CustomMmaBase.
**CN**: 第1-47行：文件开头仍是常规许可证文本，然后引入构建异步多级流水所需的组件：cache-operation 标记、底层内存原语、GEMM 形状、数值工具，以及来自 CustomMmaBase 的共享骨架。
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
    /// Cache operation for operand A
    cutlass::arch::CacheOperation::Kind CacheOpA,
    /// Iterates over tiles of B operand in global memory
    //  (concept: ReadableTileIterator | ForwardTileIterator |
    //  MaskedTileIterator)
    typename IteratorB_,
    /// Iterates over tiles of B operand in shared memory
    /// (concept: WriteableTileIterator | RandomAccessTileIterator)
    typename SmemIteratorB_,
    /// Cache operation for operand B
    cutlass::arch::CacheOperation::Kind CacheOpB,
    /// Data type of accumulator matrix
    typename ElementC_,
    /// Data type of accumulator matrix
    typename LayoutC_,
    /// Policy describing tuning details (concept: MmaPolicy)
    typename Policy_,
    /// Number of stages,
    int Stages,
    /// Use zfill or predicate for out-of-bound cp.async
    SharedMemoryClearOption SharedMemoryClear = SharedMemoryClearOption::kNone,
    /// Upper boundon the K dimension
    int kMaxK = cutlass::platform::numeric_limits<int>::max(),
    /// Used for partial specialization
    typename Enable = bool>
class CustomMmaMultistage : public CustomMmaBase<Shape_, Policy_, Stages> {
```
**EN**: Lines 48-93. The class template exposes all policy knobs required by a cp.async mainloop: iterator types for A/B, cache operations, accumulator types, the number of pipeline stages, shared-memory clear behavior, and an optional kMaxK upper bound. It inherits stage-aware storage and warp iterators from CustomMmaBase.
**CN**: 第48-93行：类模板暴露了 cp.async 主循环所需的所有策略参数：A/B 的迭代器类型、cache 操作、累加器类型、流水 stage 数、共享内存清零策略，以及可选的 kMaxK 上界。它从 CustomMmaBase 继承了 stage 感知的存储和 warp 迭代器。
```cpp
 public:
  ///< Base class
  using Base = CustomMmaBase<Shape_, Policy_, Stages>;
  ///< Size of the Gemm problem - concept: gemm::GemmShape<>
  using Shape = Shape_;
  ///< Iterates over tiles of A operand in global memory
  using IteratorA = IteratorA_;
  ///< Iterates over tiles of B operand in global memory
  using IteratorB = IteratorB_;
  ///< Data type of accumulator matrix
  using ElementC = ElementC_;
  ///< Layout of accumulator matrix
  using LayoutC = LayoutC_;
  ///< Policy describing tuning details
  using Policy = Policy_;

  using SmemIteratorA = SmemIteratorA_;
  using SmemIteratorB = SmemIteratorB_;

  static cutlass::arch::CacheOperation::Kind const kCacheOpA = CacheOpA;
  static cutlass::arch::CacheOperation::Kind const kCacheOpB = CacheOpB;

  //
  // Dependent types
  //

  /// Fragment of accumulator tile
  using FragmentC = typename Policy::Operator::FragmentC;

  /// Warp-level Mma
  using Operator = typename Policy::Operator;

  /// Minimum architecture is Sm80 to support cp.async
  using ArchTag = arch::Sm80;

  /// Complex transform on A operand
  static ComplexTransform const kTransformA = Operator::kTransformA;

  /// Complex transform on B operand
  static ComplexTransform const kTransformB = Operator::kTransformB;

  /// Internal structure exposed for introspection.
  struct Detail {
    static_assert(
        Base::kWarpGemmIterations > 1,
        "The pipelined structure requires at least two warp-level "
        "GEMM operations.");

    /// Number of cp.async instructions to load one stage of operand A
    static int const AsyncCopyIterationsPerStageA =
        IteratorA::ThreadMap::Iterations::kCount;

    /// Number of cp.async instructions to load one stage of operand B
    static int const AsyncCopyIterationsPerStageB =
        IteratorB::ThreadMap::Iterations::kCount;

    /// Number of stages
    static int const kStages = Stages;

    /// Number of cp.async instructions to load on group of operand A
    static int const kAccessesPerGroupA =
        (AsyncCopyIterationsPerStageA + Base::kWarpGemmIterations - 1) /
        Base::kWarpGemmIterations;

    /// Number of cp.async instructions to load on group of operand B
    static int const kAccessesPerGroupB =
        (AsyncCopyIterationsPerStageB + Base::kWarpGemmIterations - 1) /
        Base::kWarpGemmIterations;
  };

  static bool const kSmemContainsEntireMat = kMaxK <= Shape::kK * Stages;
  static constexpr int kNumStagesConcurrentLoad =
      kSmemContainsEntireMat ? Stages : Stages - 1;
```
**EN**: Lines 94-167. This section defines the main type aliases and the internal Detail helper. Detail converts iterator thread-map metadata into concrete copy counts: how many cp.async operations are needed per stage and how those accesses are partitioned across warp-level GEMM groups. The kSmemContainsEntireMat / kNumStagesConcurrentLoad constants additionally detect when the whole K range fits in shared memory so the circular-buffer logic can be simplified.
**CN**: 第94-167行：这一段定义了主要类型别名和内部辅助结构 Detail。Detail 会把迭代器 thread-map 元数据转成具体的拷贝数量：每个 stage 需要多少次 cp.async，以及这些访问如何分配到各个 warp 级 GEMM 分组中。kSmemContainsEntireMat / kNumStagesConcurrentLoad 还会判断完整 K 范围是否可装入共享内存，以便简化循环缓冲逻辑。
```cpp
 private:
  using WarpLoadedFragmentA = typename Operator::FragmentA;
  using WarpLoadedFragmentB = typename Operator::FragmentB;
  using WarpTransformedFragmentA = typename Operator::TransformedFragmentA;
  using WarpTransformedFragmentB = typename Operator::TransformedFragmentB;

 private:
  //
  // Data members
  //

  /// Iterator to write threadblock-scoped tile of A operand to shared memory
  SmemIteratorA smem_iterator_A_;

  /// Iterator to write threadblock-scoped tile of B operand to shared memory
  SmemIteratorB smem_iterator_B_;

  bool prologue_done_;

  // Set to `True` to ensure the accumulator will be zero outside the GEMM
  // footprint
  bool zero_outside_bounds_;

 public:
  /// Construct from tensor references
  CUTLASS_DEVICE
  CustomMmaMultistage(
      ///< Shared storage needed for internal use by threadblock-scoped GEMM
      typename Base::SharedStorageA& shared_storageA,
      typename Base::SharedStorageB& shared_storageB,
      ///< ID within the threadblock
      int thread_idx,
      ///< ID of warp
      int warp_idx,
      ///< ID of each thread within a warp
      int lane_idx)
      : Base(shared_storageA, shared_storageB, thread_idx, warp_idx, lane_idx),
        smem_iterator_A_(shared_storageA.ref(), thread_idx),
        smem_iterator_B_(shared_storageB.ref(), thread_idx),
        prologue_done_(false),
        zero_outside_bounds_(false) {
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
  CustomMmaMultistage(
      ///< Shared storage needed for internal use by threadblock-scoped GEMM
      typename Base::SharedStorage& st,
      ///< ID within the threadblock
      int thread_idx,
      ///< ID of warp
      int warp_idx,
      ///< ID of each thread within a warp
      int lane_idx)
      : CustomMmaMultistage(
            st.operand_A,
            st.operand_B,
            thread_idx,
            warp_idx,
            lane_idx) {}
```
**EN**: Lines 168-242. The private state adds shared-memory write iterators plus two runtime flags: prologue_done_ allows an external caller to skip the initial fill if it was already performed, and zero_outside_bounds_ forces zero-fill for out-of-bounds copies. The constructors also compute each warp’s (m, n, k) coordinates inside the CTA and apply the matching shared-memory tile offsets.
**CN**: 第168-242行：私有状态中加入了共享内存写迭代器以及两个运行时标志：prologue_done_ 允许外部调用者在预填充已完成时跳过初始装载；zero_outside_bounds_ 则强制对越界拷贝执行零填充。构造函数还会计算每个 warp 在 CTA 内部的 (m, n, k) 坐标，并施加对应的共享内存 tile 偏移。
```cpp

  CUTLASS_DEVICE
  bool set_prologue_done(bool value) {
    prologue_done_ = value;
    return true;
  }

  CUTLASS_DEVICE
  bool set_zero_outside_bounds(bool value) {
    zero_outside_bounds_ = value;
    return true;
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
    SmemIteratorA smem_iterator_A(shared_storageA.ref(), thread_idx);
    SmemIteratorB smem_iterator_B(shared_storageB.ref(), thread_idx);
    int32_t iter = (problem_size_k + Base::Shape::kK - 1) / Base::Shape::kK;
    _prologue<kLoadA, kLoadB>(
        iterator_A, iterator_B, iter, smem_iterator_A, smem_iterator_B);
  }
```
**EN**: Lines 243-289. These helper entry points expose limited runtime configurability and public prologue overloads. The overload taking SharedStorage simply forwards to the operand-specific version, while the operand-specific version constructs smem iterators, converts problem_size_k into a threadblock K-iteration count, and delegates the real work to _prologue.
**CN**: 第243-289行：这些辅助入口提供了有限的运行时可配置性以及公共 prologue 重载。接收 SharedStorage 的版本只是转发到按操作数区分的版本；后者会构造 smem 迭代器、把 problem_size_k 转成 threadblock 级 K 迭代次数，并把真正的工作交给 _prologue。
```cpp

  CUTLASS_DEVICE
  void copy_tiles_and_advance(
      IteratorA& iterator_A,
      IteratorB& iterator_B,
      int group_start_A = 0,
      int group_start_B = 0) {
    iterator_A.set_iteration_index(
        group_start_A * IteratorA::kAccessesPerVector);
    this->smem_iterator_A_.set_iteration_index(group_start_A);

    // Async Copy for operand A
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < Detail::kAccessesPerGroupA; ++j) {
      if (group_start_A + j < Detail::AsyncCopyIterationsPerStageA) {
        typename IteratorA::AccessType* dst_ptr =
            reinterpret_cast<typename IteratorA::AccessType*>(
                this->smem_iterator_A_.get());

        int const kSrcBytes = sizeof_bits<typename IteratorA::Element>::value *
            IteratorA::ThreadMap::kElementsPerAccess /
            IteratorA::kAccessesPerVector / 8;

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorA::kAccessesPerVector; ++v) {
          auto gmem_ptr = iterator_A.get();

          if (zero_outside_bounds_ ||
              SharedMemoryClear == SharedMemoryClearOption::kZfill) {
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

    iterator_B.set_iteration_index(
        group_start_B * IteratorB::kAccessesPerVector);
    this->smem_iterator_B_.set_iteration_index(group_start_B);

    // Async Copy for operand B
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < Detail::kAccessesPerGroupB; ++j) {
      if (group_start_B + j < Detail::AsyncCopyIterationsPerStageB) {
        typename IteratorB::AccessType* dst_ptr =
            reinterpret_cast<typename IteratorB::AccessType*>(
                this->smem_iterator_B_.get());

        int const kSrcBytes = sizeof_bits<typename IteratorB::Element>::value *
            IteratorB::ThreadMap::kElementsPerAccess /
            IteratorB::kAccessesPerVector / 8;

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorB::kAccessesPerVector; ++v) {
          auto gmem_ptr = iterator_B.get();

          if (zero_outside_bounds_ ||
              SharedMemoryClear == SharedMemoryClearOption::kZfill) {
            cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB>(
                dst_ptr + v, gmem_ptr, iterator_B.valid());
          } else {
            cutlass::arch::cp_async<kSrcBytes, kCacheOpB>(
                dst_ptr + v, gmem_ptr, iterator_B.valid());
          }

          ++iterator_B;
        }
        ++this->smem_iterator_B_;
      }
    }
  }
```
**EN**: Lines 290-367. copy_tiles_and_advance is the asynchronous copy workhorse used during steady-state execution. It programs the iterator positions for one copy group, emits cp.async or cp_async_zfill instructions for operand A and B, honors per-element validity masks, and advances both the global-memory iterators and the shared-memory write iterators in lockstep.
**CN**: 第290-367行：copy_tiles_and_advance 是稳态执行期间使用的异步拷贝核心。它为一个 copy group 设置迭代器位置，为 A/B 操作数发出 cp.async 或 cp_async_zfill 指令，遵守逐元素有效性掩码，并让全局内存迭代器和共享内存写迭代器保持锁步前进。
```cpp

  template <bool kLoadA = true, bool kLoadB = true>
  CUTLASS_DEVICE static void _prologue(
      IteratorA& iterator_A,
      IteratorB& iterator_B,
      int32_t& gemm_k_iterations,
      SmemIteratorA& smem_iterator_A_,
      SmemIteratorB& smem_iterator_B_) {
    // Issue several complete stages
    CUTLASS_PRAGMA_UNROLL
    for (int stage = 0; stage < kNumStagesConcurrentLoad;
         ++stage, --gemm_k_iterations) {
      iterator_A.clear_mask(gemm_k_iterations == 0);
      iterator_B.clear_mask(gemm_k_iterations == 0);

      iterator_A.set_iteration_index(0);
      smem_iterator_A_.set_iteration_index(0);

      // Async Copy for operand A
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageA; ++j) {
        typename IteratorA::AccessType* dst_ptr =
            reinterpret_cast<typename IteratorA::AccessType*>(
                smem_iterator_A_.get());

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorA::kAccessesPerVector; ++v) {
          int const kSrcBytes =
              sizeof_bits<typename IteratorA::Element>::value *
              IteratorA::ThreadMap::kElementsPerAccess /
              IteratorA::kAccessesPerVector / 8;

          int src_bytes = (iterator_A.valid() ? kSrcBytes : 0);

          if (kLoadA) {
            cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpA>(
                dst_ptr + v, iterator_A.get(), iterator_A.valid());
          }

          ++iterator_A;
        }

        ++smem_iterator_A_;
      }

      iterator_B.set_iteration_index(0);
      smem_iterator_B_.set_iteration_index(0);

      // Async Copy for operand B
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageB; ++j) {
        typename IteratorB::AccessType* dst_ptr =
            reinterpret_cast<typename IteratorB::AccessType*>(
                smem_iterator_B_.get());

        CUTLASS_PRAGMA_UNROLL
        for (int v = 0; v < IteratorB::kAccessesPerVector; ++v) {
          int const kSrcBytes =
              sizeof_bits<typename IteratorB::Element>::value *
              IteratorB::ThreadMap::kElementsPerAccess /
              IteratorB::kAccessesPerVector / 8;

          if (kLoadB) {
            cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB>(
                dst_ptr + v, iterator_B.get(), iterator_B.valid());
          }

          ++iterator_B;
        }

        ++smem_iterator_B_;
      }

      // Move to the next stage
      iterator_A.add_tile_offset({0, 1});
      iterator_B.add_tile_offset({1, 0});

      smem_iterator_A_.add_tile_offset({0, 1});
      smem_iterator_B_.add_tile_offset({1, 0});

      // Defines the boundary of a stage of cp.async.
      cutlass::arch::cp_async_fence();
    }
  }
```
**EN**: Lines 368-451. The static _prologue bulk-loads the initial pipeline stages before computation starts. For each stage it resets iteration indices, issues the full set of cp_async_zfill copies for A and B, advances both global and shared-memory tile offsets to the next stage, and inserts a cp_async_fence to delimit the commit group.
**CN**: 第368-451行：静态函数 _prologue 会在计算开始前批量装载初始流水 stage。对每个 stage，它都会重置迭代索引，为 A/B 发出完整的 cp_async_zfill 拷贝，推进全局和共享内存的 tile 偏移到下一个 stage，并插入 cp_async_fence 作为该提交组的边界。
```cpp

  /// Perform a threadblock-scoped matrix multiply-accumulate
  CUTLASS_DEVICE
  void operator()(
      ///< problem size of GEMM
      int gemm_k_iterations,
      ///< destination accumulator tile
      FragmentC& accum,
      ///< iterator over A operand in global memory
      IteratorA iterator_A,
      ///< iterator over B operand in global memory
      IteratorB iterator_B,
      ///< initial value of accumulator
      FragmentC const& src_accum) {
    //
    // Prologue
    //

    if (!prologue_done_) {
      _prologue<true, true>(
          iterator_A,
          iterator_B,
          gemm_k_iterations,
          smem_iterator_A_,
          smem_iterator_B_);
    } else if (!kSmemContainsEntireMat) {
      _prologue<false, false>(
          iterator_A,
          iterator_B,
          gemm_k_iterations,
          smem_iterator_A_,
          smem_iterator_B_);
    } else {
      gemm_k_iterations -= kNumStagesConcurrentLoad;
    }

    // Perform accumulation in the 'd' output operand
    accum = src_accum;

    //
    // Clear the remaining tiles of SMEM. This is a functional requirement for
    // some kernels so that all accumulator elements outside the GEMM footprint
    // are zero.
    //

    if (SharedMemoryClear == SharedMemoryClearOption::kClearLastStage) {
      /// Iterator to write threadblock-scoped tile of A operand to shared
      /// memory
      SmemIteratorA last_smem_iterator_A(this->smem_iterator_A_);

      typename IteratorA::AccessType zero_A;
      zero_A.clear();

      last_smem_iterator_A.set_iteration_index(0);

      // Async Copy for operand A
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageA; ++j) {
        typename IteratorA::AccessType* dst_ptr =
            reinterpret_cast<typename IteratorA::AccessType*>(
                last_smem_iterator_A.get());

        *dst_ptr = zero_A;

        ++last_smem_iterator_A;
      }

      /// Iterator to write threadblock-scoped tile of B operand to shared
      /// memory
      SmemIteratorB last_smem_iterator_B(this->smem_iterator_B_);
      typename IteratorB::AccessType zero_B;

      zero_B.clear();
      last_smem_iterator_B.set_iteration_index(0);

      // Async Copy for operand B
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageB; ++j) {
        typename IteratorB::AccessType* dst_ptr =
            reinterpret_cast<typename IteratorB::AccessType*>(
                last_smem_iterator_B.get());

        *dst_ptr = zero_B;

        ++last_smem_iterator_B;
      }
    }

    // Waits until kStages-2 stages have committed.
    cutlass::arch::cp_async_wait<kNumStagesConcurrentLoad - 1>();
    __syncthreads();
```
**EN**: Lines 452-543. The beginning of operator() decides how much setup is still needed: run the prologue, skip only the copies, or merely decrement the remaining K-iteration count when shared memory already contains the full matrix. It then initializes the accumulator, optionally clears the last shared-memory stage for correctness outside the GEMM footprint, and waits until enough async stages are committed before letting warps read shared memory.
**CN**: 第452-543行：operator() 的开头先判断还需要做多少准备工作：执行完整 prologue、只跳过拷贝，或者在共享内存已经包含完整矩阵时仅减少剩余 K 迭代计数。随后它初始化累加器，并在需要时清零最后一个共享内存 stage，以保证 GEMM 有效区域外的结果正确；最后等待足够多的异步 stage 提交完成，再允许 warp 从共享内存读取。
```cpp
    // Pair of fragments used to overlap shared memory loads and math
    // instructions
    WarpLoadedFragmentA warp_loaded_frag_A[2];
    WarpLoadedFragmentB warp_loaded_frag_B[2];
    WarpTransformedFragmentA warp_transformed_frag_A[2];
    WarpTransformedFragmentB warp_transformed_frag_B[2];

    Operator warp_mma;

    this->warp_tile_iterator_A_.set_kgroup_index(0);
    this->warp_tile_iterator_B_.set_kgroup_index(0);

    this->warp_tile_iterator_A_.load(warp_loaded_frag_A[0]);
    this->warp_tile_iterator_B_.load(warp_loaded_frag_B[0]);

    ++this->warp_tile_iterator_A_;
    ++this->warp_tile_iterator_B_;

    iterator_A.clear_mask(gemm_k_iterations == 0);
    iterator_B.clear_mask(gemm_k_iterations == 0);

    int smem_write_stage_idx = Base::kStages - 1;
    int smem_read_stage_idx = 0;

    warp_mma.transform(
        warp_transformed_frag_A[0],
        warp_transformed_frag_B[0],
        warp_loaded_frag_A[0],
        warp_loaded_frag_B[0]);

    // tf32x3 kernels use staging accumulation. warp_mma uses a temporary
    // accumulator and this temporary accumulator is added to the final
    // accumulator once in every mainloop iteration.
    plus<FragmentC> plus_accum;

    FragmentC tmp_accum;

    if (platform::is_same<
            typename Operator::MathOperator,
            arch::OpMultiplyAddFastF32>::value ||
        platform::is_same<
            typename Operator::MathOperator,
            arch::OpMultiplyAddComplexFastF32>::value) {
      tmp_accum.clear();
    }
```
**EN**: Lines 544-589. This block sets up the double-buffered warp fragments used for shared-memory reads and data transformation. It eagerly loads and transforms the first warp tile, creates the warp-level MMA operator, and enables a special temporary-accumulator path for FastF32 / ComplexFastF32 kernels whose mma primitive internally stages accumulation.
**CN**: 第544-589行：这一段初始化用于共享内存读取和数据变换的双缓冲 warp fragment。代码会预先装载并变换第一个 warp tile，构造 warp 级 MMA 算子，并为 FastF32 / ComplexFastF32 内核启用临时累加器路径，因为这些 mma 原语内部采用分阶段累加。
```cpp
    //
    // Mainloop
    //

    CUTLASS_GEMM_LOOP
    for (; gemm_k_iterations > (-kNumStagesConcurrentLoad);) {
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

        this->warp_tile_iterator_A_.set_kgroup_index(
            (warp_mma_k + 1) % Base::kWarpGemmIterations);
        this->warp_tile_iterator_B_.set_kgroup_index(
            (warp_mma_k + 1) % Base::kWarpGemmIterations);

        // In case of a non-circular buffer ("kSmemContainsEntireMat")
        // make sure we don't load out of bounds data.
        if (!kSmemContainsEntireMat ||
            gemm_k_iterations > (-kNumStagesConcurrentLoad) ||
            warp_mma_k < Base::kWarpGemmIterations - 1) {
          this->warp_tile_iterator_A_.load(
              warp_loaded_frag_A[(warp_mma_k + 1) % 2]);
          this->warp_tile_iterator_B_.load(
              warp_loaded_frag_B[(warp_mma_k + 1) % 2]);
        }

        ++this->warp_tile_iterator_A_;
        ++this->warp_tile_iterator_B_;

        if (warp_mma_k > 0)
          warp_mma.transform(
              warp_transformed_frag_A[warp_mma_k % 2],
              warp_transformed_frag_B[warp_mma_k % 2],
              warp_loaded_frag_A[warp_mma_k % 2],
              warp_loaded_frag_B[warp_mma_k % 2]);

        if (platform::is_same<
                typename Operator::MathOperator,
                arch::OpMultiplyAddFastF32>::value ||
            platform::is_same<
                typename Operator::MathOperator,
                arch::OpMultiplyAddComplexFastF32>::value) {
          warp_mma(
              tmp_accum,
              warp_transformed_frag_A[warp_mma_k % 2],
              warp_transformed_frag_B[warp_mma_k % 2],
              tmp_accum);

          if (warp_mma_k == 0) {
            accum = plus_accum(accum, tmp_accum);
            tmp_accum.clear();
          }
        } else {
          warp_mma(
              accum,
              warp_transformed_frag_A[warp_mma_k % 2],
              warp_transformed_frag_B[warp_mma_k % 2],
              accum);
        }

        // Issue global->shared copies for the this stage
        if (!kSmemContainsEntireMat &&
            warp_mma_k < Base::kWarpGemmIterations - 1) {
```
**EN**: Lines 590-660. The first half of the steady-state mainloop overlaps four activities: reading the next warp tile from shared memory, transforming the previously loaded fragment, executing the current warp-level MMA, and launching part of the next global-to-shared async copy group. The copy work is intentionally distributed across warp_mma_k iterations so memory traffic is hidden under math.
**CN**: 第590-660行：主循环稳态部分的前半段重叠了四类工作：从共享内存读取下一个 warp tile、变换前一个已装载 fragment、执行当前 warp 级 MMA，以及发起下一批 global-to-shared 异步拷贝中的一部分。拷贝任务被刻意分散到各个 warp_mma_k 迭代中，以便把内存流量隐藏在计算之下。
```cpp
          int group_start_iteration_A, group_start_iteration_B;

          group_start_iteration_A = warp_mma_k * Detail::kAccessesPerGroupA;
          group_start_iteration_B = warp_mma_k * Detail::kAccessesPerGroupB;

          copy_tiles_and_advance(
              iterator_A,
              iterator_B,
              group_start_iteration_A,
              group_start_iteration_B);
        }

        if (warp_mma_k + 2 == Base::kWarpGemmIterations) {
          if (!kSmemContainsEntireMat) {
            int group_start_iteration_A, group_start_iteration_B;
            group_start_iteration_A =
                (warp_mma_k + 1) * Detail::kAccessesPerGroupA;
            group_start_iteration_B =
                (warp_mma_k + 1) * Detail::kAccessesPerGroupB;

            copy_tiles_and_advance(
                iterator_A,
                iterator_B,
                group_start_iteration_A,
                group_start_iteration_B);
          }

          // Inserts a memory fence between stages of cp.async instructions.
          cutlass::arch::cp_async_fence();

          // Waits until kStages-2 stages have committed.
          cutlass::arch::cp_async_wait<kNumStagesConcurrentLoad - 1>();
          __syncthreads();

          // Move to the next stage
          iterator_A.add_tile_offset({0, 1});
          iterator_B.add_tile_offset({1, 0});

          this->smem_iterator_A_.add_tile_offset({0, 1});
          this->smem_iterator_B_.add_tile_offset({1, 0});

          // Add negative offsets to return iterators to the 'start' of the
          // circular buffer in shared memory
          if (smem_write_stage_idx == (Base::kStages - 1)) {
            this->smem_iterator_A_.add_tile_offset({0, -Base::kStages});
            this->smem_iterator_B_.add_tile_offset({-Base::kStages, 0});
            smem_write_stage_idx = 0;
          } else {
            ++smem_write_stage_idx;
          }

          if (!kSmemContainsEntireMat &&
              smem_read_stage_idx == (Base::kStages - 1)) {
            this->warp_tile_iterator_A_.add_tile_offset(
                {0,
                 -Base::kStages * Policy::kPartitionsK *
                     Base::kWarpGemmIterations});
            this->warp_tile_iterator_B_.add_tile_offset(
                {-Base::kStages * Policy::kPartitionsK *
                     Base::kWarpGemmIterations,
                 0});
            smem_read_stage_idx = 0;
          } else {
            ++smem_read_stage_idx;
          }

          --gemm_k_iterations;
          iterator_A.clear_mask(gemm_k_iterations == 0);
          iterator_B.clear_mask(gemm_k_iterations == 0);
        }

        // Do any conversions feeding the first stage at the end of the loop so
        // we can start right away on mma instructions
        if (warp_mma_k + 1 == Base::kWarpGemmIterations)
          warp_mma.transform(
              warp_transformed_frag_A[(warp_mma_k + 1) % 2],
              warp_transformed_frag_B[(warp_mma_k + 1) % 2],
              warp_loaded_frag_A[(warp_mma_k + 1) % 2],
              warp_loaded_frag_B[(warp_mma_k + 1) % 2]);
      }
    }
```
**EN**: Lines 661-742. The second half of the mainloop completes the final copy group for the stage, fences and waits on cp.async progress, synchronizes the CTA, advances both global and shared-memory stage pointers, and wraps the read/write iterators when the circular buffer reaches the last stage. It also decrements the K-iteration counter and pre-transforms the fragment that will feed the next loop trip.
**CN**: 第661-742行：主循环的后半段会完成该 stage 的最后一个 copy group，对 cp.async 进度执行 fence 和 wait，并同步整个 CTA；随后推进全局与共享内存的 stage 指针，并在循环缓冲到达最后一个 stage 时回卷读写迭代器。同时它还会递减 K 迭代计数，并提前变换下一轮循环将要使用的 fragment。
```cpp
    if (platform::is_same<
            typename Operator::MathOperator,
            arch::OpMultiplyAddFastF32>::value ||
        platform::is_same<
            typename Operator::MathOperator,
            arch::OpMultiplyAddComplexFastF32>::value) {
      accum = plus_accum(accum, tmp_accum);
    }
  }
};

/////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace gemm
} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Lines 743-760. After the loop, kernels using staged FastF32 accumulation flush the remaining temporary fragment into the final accumulator. The file then closes the CUTLASS threadblock namespaces.
**CN**: 第743-760行：循环结束后，使用分阶段 FastF32 累加的内核会把剩余的临时 fragment 合并回最终累加器。最后文件关闭 CUTLASS 的 threadblock 命名空间。
---
## Key Concepts / 关键概念
- CustomMmaMultistage is the performance-oriented variant: it relies on cp.async, explicit fences/waits, and a circular shared-memory pipeline to keep tensor cores fed. / CustomMmaMultistage 是面向性能的变体：它依赖 cp.async、显式 fence/wait，以及循环共享内存流水来持续喂饱 tensor core。
- The implementation distinguishes between “whole K fits in shared memory” and true circular-buffer cases, which avoids unnecessary wraparound logic for bounded problems. / 实现中区分了“整个 K 都能放入共享内存”和“必须使用真正循环缓冲”这两种情况，从而在有界问题上避免不必要的回卷逻辑。
- FastF32 handling is integrated directly into the mainloop because its warp-level MMA semantics require staging accumulation before folding into the final FragmentC. / FastF32 的处理被直接集成进主循环，因为它的 warp 级 MMA 语义要求先做临时累加，再折叠进最终的 FragmentC。
## Dependencies / 依赖项
- `custom_mma_base.h` — Supplies shared-storage layout and warp-iterator setup reused by the multistage kernel / 提供 multistage 内核复用的共享内存布局与 warp 迭代器设置
- `cutlass/arch/cache_operation.h` — Carries cache policy tags for asynchronous global-memory loads / 携带异步全局内存加载所需的 cache 策略标签
- `cutlass/arch/memory.h` — Provides cp.async, fence, and wait primitives that drive the pipeline / 提供驱动流水线的 cp.async、fence 与 wait 原语
- `cutlass/gemm/gemm.h` — Defines the GEMM shapes and loop geometry used throughout the mainloop / 定义主循环全程使用的 GEMM 形状与循环几何信息
