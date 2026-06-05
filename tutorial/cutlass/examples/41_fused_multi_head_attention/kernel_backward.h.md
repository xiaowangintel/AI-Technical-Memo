# kernel_backward.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/kernel_backward.h`
**Purpose / 用途**: Core fused backward FMHA kernel that reconstructs attention, computes dV/dQ/dK and optional dB, and coordinates workspace plus split-key accumulation / 融合反向 FMHA 的核心内核，负责重建注意力、计算 dV/dQ/dK 及可选 dB，并协调工作区与 split-key 累加
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

#pragma once

#include <cmath>
#include <cinttypes>
#include <type_traits>
#include <vector>

#include <cuda_fp16.h>
#include <curand_kernel.h>

#ifdef HAS_PYTORCH
#include <ATen/cuda/CUDAContext.h>
#include <ATen/cuda/CUDAGeneratorImpl.h>
#include <c10/cuda/CUDAGuard.h>
#include <ATen/cuda/CUDAGraphsUtils.cuh>
#endif

#include "cutlass/cutlass.h"
#include "cutlass/epilogue/thread/linear_combination.h"
#include "cutlass/epilogue/thread/scale_type.h"
#include "cutlass/fast_math.h"
#include "cutlass/functional.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/layout/vector.h"
#include "cutlass/numeric_conversion.h"
#include "cutlass/numeric_types.h"
#include "cutlass/tensor_ref.h"

#include "debug_utils.h"
#include "gemm_kernel_utils.h"

#include "cutlass/epilogue/thread/linear_combination_relu.h"
#include "cutlass/epilogue/threadblock/epilogue_smem_accumulator.h"
#include "cutlass/epilogue/warp/fragment_iterator_tensor_op.h"
#include "cutlass/epilogue/warp/tile_iterator_tensor_op.h"
#include "cutlass/gemm/device/default_gemm_configuration.h"
#include "cutlass/gemm/kernel/default_gemm.h"
#include "cutlass/gemm/threadblock/default_mma.h"
#include "cutlass/gemm/threadblock/default_mma_core_simt.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm70.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm75.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
#include "cutlass/integer_subbyte.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/platform/platform.h"
#include "cutlass/transform/threadblock/predicated_tile_iterator.h"
#include "cutlass/transform/threadblock/vector_iterator.h"
#include "epilogue/epilogue_pipelined.h"
#include "iterators/epilogue_predicated_tile_iterator.h"

#include "gemm/custom_mma.h"
#include "gemm/find_default_mma.h"
#include "gemm/mma_accum_lambda_iterator.h"
#include "gemm/mma_from_smem.h"
#include "transform/tile_smem_loader.h"

using namespace gemm_kernel_utils;
```
**EN**: Lines 1-90. The opening section imports CUDA, CUTLASS, iterator/epilogue utilities, and local helpers needed by the backward kernel. Compared with the forward file, the dependency set is broader because backward needs extra fragment iterators, custom epilogues, and accumulator-storage helpers to move data between registers, shared memory, and global memory. The namespace setup also exposes common GEMM helper symbols that are used throughout the template definitions that follow.
**CN**: 第1-90行：开头部分引入了反向内核所需的 CUDA、CUTLASS、迭代器/epilogue 工具以及本地辅助头文件。与前向文件相比，这里的依赖更广，因为反向路径需要额外的片段迭代器、自定义 epilogue，以及在寄存器、共享内存和全局内存之间搬运累加结果的辅助工具。命名空间设置则把后续模板定义会反复使用的 GEMM 辅助符号统一暴露出来。
```cpp
namespace {

template <typename FragmentType, int32_t kNumThreads>
struct GmemTile {
  /*
    Helper functions to efficient store/load RF to gmem

    GEMM accumulators have a particular format on A100, and
    it takes some compute/shared-memory to rearrange them to
    a RowMajor or ColumnMajor format in global memory through
    an Epilogue. The same complexity goes for loading into RF.

    This class loads/stores RF as they are, and can be used for
    efficient accumulation across gemms for instance:

    ``​`
    GmemTile tile;
    for (int i = 0; i < N; ++i) {
      // ...

      Fragment accum;
      if (i == 0) {
        accum.clear();
      } else {
        tile.load(accum);
      }
      mma(accum, ...);
      if (i < N-1) {
        // Store for next GEMM
        tile.store(accum);
      } else {
        // Store in tensor (eg RowMajor)
        epilogue(accum);
      }

      // ...
    }
    ``​`
  */

  // 128bits per thread
  using AccessType = cutlass::Array<float, 4>;
  static constexpr int32_t kBytes = sizeof(AccessType);
  static constexpr int32_t kStride = kNumThreads * AccessType::kElements;
  static constexpr int32_t kNumIters =
      FragmentType::kElements / AccessType::kElements;
  static constexpr int32_t kElementsStored =
      kNumThreads * FragmentType::kElements;
  static_assert(
      FragmentType::kElements % AccessType::kElements == 0,
      "fragment not aligned on 128 bits");

  float* ptr;

  CUTLASS_DEVICE void load(FragmentType& fragment, int thread_id) {
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kNumIters; ++i) {
      AccessType* __restrict__ gmem_ptr = reinterpret_cast<AccessType*>(
          ptr + thread_id * AccessType::kElements + i * kStride);
      AccessType sub_fragment;
      cutlass::arch::global_load<AccessType, kBytes>(
          sub_fragment, gmem_ptr, true);
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < AccessType::kElements; ++j) {
        fragment[i * AccessType::kElements + j] = sub_fragment[j];
      }
    }
  }

  CUTLASS_DEVICE void store(FragmentType const& fragment, int thread_id) {
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kNumIters; ++i) {
      AccessType* __restrict__ gmem_ptr = reinterpret_cast<AccessType*>(
          ptr + thread_id * AccessType::kElements + i * kStride);
      AccessType sub_fragment;
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < AccessType::kElements; ++j) {
        sub_fragment[j] = fragment[i * AccessType::kElements + j];
      }
      cutlass::arch::global_store<AccessType, kBytes>(
          sub_fragment, gmem_ptr, true);
    }
  }

  CUTLASS_DEVICE void storeAtomicAdd(
      FragmentType const& fragment,
      int thread_id) {
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kNumIters; ++i) {
      float* gmem_ptr = ptr + thread_id * AccessType::kElements + i * kStride;
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < AccessType::kElements; ++j) {
        float val = fragment[i * AccessType::kElements + j];
        float* ptr = gmem_ptr + j;
        atomicAdd(ptr, val);
      }
    }
  }
};

struct AtomicLock {
  CUTLASS_DEVICE static void acquire(
      int32_t* lock,
      int set_val,
      int thread_id) {
    if (thread_id == 0) {
      while (atomicCAS(lock, 0 /*cmp*/, set_val /*setval*/) != set_val) {
#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 700
        __nanosleep(40);
#endif
      }
    }
    __syncthreads();
  }
  CUTLASS_DEVICE static void release(int32_t* lock, int thread_id) {
    if (thread_id == 0) {
      int status = 0;
#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 700
      asm volatile("st.global.release.gpu.b32 [%0], %1;\n"
                   :
                   : "l"(lock), "r"(status));
#else
      asm volatile("st.global.cg.b32 [%0], %1;\n" : : "l"(lock), "r"(status));
#endif
    }
  }
};

template <typename scalar_t, typename Arch>
constexpr int getWarpsPerSmBw() {
  bool is_half = !cutlass::platform::is_same<scalar_t, float>::value;
  if (Arch::kMinComputeCapability >= 80) {
    return is_half ? 12 : 8;
  }
  return 8;
}
} // namespace
```
**EN**: Lines 91-228. This utility block introduces several low-level building pieces. `GmemTile` provides raw register-fragment load/store helpers so partial accumulators can be checkpointed to global memory without first running a full epilogue layout transform. `AtomicLock` supplies a simple `atomicCAS`-based mutual exclusion primitive used when multiple split-key blocks must accumulate into the same `grad_query` tile. The architecture helper `getWarpsPerSmBw()` chooses the target resident-warp count, giving newer half-precision tensor-core paths more aggressive occupancy when the hardware can sustain it.
**CN**: 第91-228行：`GmemTile`、`AtomicLock` 与启发式函数构成了若干底层积木。`GmemTile` 直接对寄存器片段做全局内存 load/store，使部分累加结果可以在不经过完整 epilogue 布局变换的情况下被暂存到全局内存。`AtomicLock` 提供基于 `atomicCAS` 的简易互斥，用于多个 split-key 线程块需要同时累加同一 `grad_query` tile 的场景。架构辅助函数 `getWarpsPerSmBw()` 则给出目标驻留 warp 数，在较新的半精度 Tensor Core 路径上会采用更激进的占用率设置。
```cpp
template <
    // which arch we target (eg `cutlass::arch::Sm80`)
    typename ArchTag_,
    // input/output type
    typename scalar_t_,
    // run optimized kernel because memory accesses will be aligned
    bool kIsAligned_,
    // use dropout if enabled
    bool kApplyDropout_,
    // when doing a GEMM, preload the next one (uses more shmem)
    bool kPreload_,
    // block dimensions
    int kBlockSizeI_,
    int kBlockSizeJ_,
    // upperbound on `max(value.shape[-1], query.shape[-1])`
    int kMaxK_ = (int)cutlass::platform::numeric_limits<uint32_t>::max(),
    // assumes that `cu_seqlen` is None, and
    // (1) `num_queries % kBlockSizeI == 0`
    // (2) `num_keys % kBlockSizeJ == 0`
    bool kKeysQueriesAlignedToBlockSize_ = false,
    // Allows to parallelize across keys
    bool kEnableSplitKeys_ = true>
struct AttentionBackwardKernel {
  enum CustomMaskType {
    NoCustomMask = 0,
    CausalFromTopLeft = 1,
    CausalFromBottomRight = 2,
    NumCustomMaskTypes,
  };
  using scalar_t = scalar_t_;
  using output_t = scalar_t;
  using output_accum_t = float;
  using lse_scalar_t = float;
  using accum_t = float;
  using ArchTag = ArchTag_;
  static constexpr bool kIsAligned = kIsAligned_;
  static constexpr bool kApplyDropout = kApplyDropout_;
  static constexpr bool kPreload = kPreload_;
  static constexpr int kBlockSizeI = kBlockSizeI_;
  static constexpr int kBlockSizeJ = kBlockSizeJ_;
  static constexpr int kMaxK = kMaxK_;
  static constexpr bool kKeysQueriesAlignedToBlockSize =
      kKeysQueriesAlignedToBlockSize_;

  static constexpr int64_t kWarpSize = 32;

  // If this is true, we store and accumulate dK/dV in RF
  // rather than going back to gmem everytime
  static constexpr bool kIsHalf = cutlass::sizeof_bits<scalar_t>::value <= 16;
  static constexpr bool kOutputInRF = kIsHalf && kMaxK <= kBlockSizeI;
  static_assert(
      !kPreload ||
          (kIsHalf && ArchTag::kMinComputeCapability >= 80 && kOutputInRF),
      "preload MMA not supported");
  static constexpr bool kPrologueQK = kPreload;
  static constexpr bool kPrologueGV = kPreload;
  static constexpr bool kPrologueDOV = kPreload;
  static constexpr bool kPrologueGQ = kPreload;
  static constexpr bool kPrologueGK = kPreload;

  static constexpr int64_t kNumWarpsPerBlock =
      (kBlockSizeI * kBlockSizeJ) / (32 * 32);

  // Compute delta for the f16 kernels
  // TODO: Figure out why it's slower on the f32 kernels
  // (something due to RF pressure?)
  // TODO: Remove condition on `kOutputInRF` - this is needed to work
  // around a compiler bug on V100, not exactly sure why but I spent
  // too much time on this already. Reproducible with
  // (B, Mq, Mkv, K) = (1, 1, 1, 136) for instance
  static constexpr bool kKernelComputesDelta =
      kIsHalf && (kOutputInRF || ArchTag::kMinComputeCapability != 70);

  // Launch bounds
  static constexpr int64_t kNumThreads = kWarpSize * kNumWarpsPerBlock;
  static constexpr int64_t kMinBlocksPerSm =
      getWarpsPerSmBw<scalar_t, ArchTag>() / kNumWarpsPerBlock;

  using GemmType = DefaultGemmType<ArchTag, scalar_t>;
  using DefaultConfig =
      typename cutlass::gemm::device::DefaultGemmConfiguration<
          typename GemmType::OpClass,
          ArchTag,
          scalar_t,
          scalar_t,
          scalar_t, // ElementC
          accum_t // ElementAccumulator
          >;
  static constexpr auto kOptimalAlignement = cutlass::platform::max(
      DefaultConfig::kAlignmentA,
      DefaultConfig::kAlignmentB);
  static constexpr auto kMinimumAlignment = GemmType::kMinimumAlignment;

  struct MatmulQK {
    /*
    attn_T = k_j @ q_i.transpose(-2, -1) # matmul
    attn_T = (attn_T - logsumexp[i_start:i_end].unsqueeze(1).transpose(-2,
    -1)).exp() # epilogue

    with attn_T.shape = (kBlockSizeJ, kBlockSizeI)
    */
    using ThreadblockShape =
        cutlass::gemm::GemmShape<kBlockSizeJ, kBlockSizeI, GemmType::ThreadK>;
    using WarpShape = cutlass::gemm::GemmShape<32, 32, GemmType::WarpK>;
    using DefaultMma = typename cutlass::gemm::threadblock::DefaultMma<
        scalar_t, // ElementA
        cutlass::layout::RowMajor, // LayoutA
        kIsAligned ? DefaultConfig::kAlignmentA : GemmType::kMinimumAlignment,
        scalar_t, // ElementB
        cutlass::layout::ColumnMajor, // LayoutB
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment,
        accum_t, // ElementC
        cutlass::layout::RowMajor, // LayoutC
        typename GemmType::OpClass,
        ArchTag,
        ThreadblockShape,
        WarpShape,
        typename GemmType::InstructionShape,
        DefaultConfig::kStages,
        typename GemmType::Operator,
        false, // AccumulatorsInRowMajor = false,
        cutlass::gemm::SharedMemoryClearOption::kNone>;
    using MmaCore = typename DefaultMma::MmaCore;
    using Mma =
        typename MakeCustomMma<typename DefaultMma::ThreadblockMma, kMaxK>::Mma;

    // used for efficient load of bias tile (Bij) from global memory to shared
    // memory
    using BiasLoader = TileSmemLoader<
        scalar_t,
        // Bij is applied to transposed attn matrix tile (Pij.T). Bij is loaded
        // row-major but needs to have transposed shape so we get the same
        // elements.
        cutlass::MatrixShape<ThreadblockShape::kN, ThreadblockShape::kM>,
        MmaCore::kThreads,
        // input restriction: kv_len has to be a multiple of this value
        128 / cutlass::sizeof_bits<scalar_t>::value>;

    // Epilogue to store to shared-memory in a format that we can use later for
    // the second matmul
    using B2bGemm = typename cutlass::gemm::threadblock::B2bGemm<
        typename Mma::Operator::IteratorC,
        typename Mma::Operator,
        scalar_t,
        WarpShape,
        ThreadblockShape>;
    using AccumLambdaIterator = typename DefaultMmaAccumLambdaIterator<
        typename Mma::Operator::IteratorC,
        accum_t,
        kWarpSize>::Iterator;
    using AccumulatorSharedStorage = typename B2bGemm::AccumulatorSharedStorage;
  };
```
**EN**: Lines 229-380. This block declares the main `AttentionBackwardKernel` template and immediately defines the first logical GEMM, `MatmulQK`. The template parameters capture architecture, scalar type, alignment, dropout support, whether to preload the next tile, block sizes, maximum supported K, block-alignment assumptions, and whether split-keys is enabled. Inside the class, the code derives policy flags such as whether outputs stay in registers, whether the kernel also computes `delta`, and how many warps/threads are launched. `MatmulQK` then reconstructs the transposed score tile `K @ Q^T`, defines the iterator used to walk accumulator fragments, and sets up shared-memory bias staging plus the back-to-back GEMM storage format consumed later by the gradient pipeline.
**CN**: 第229-380行：这一部分声明主模板 `AttentionBackwardKernel`，并紧接着定义第一个逻辑 GEMM：`MatmulQK`。模板参数覆盖目标架构、标量类型、对齐情况、是否支持 dropout、是否预取下一 tile、块大小、支持的最大 K、块对齐假设，以及是否启用 split-keys。类体内部会推导一系列策略标志，例如输出是否保存在寄存器、内核是否顺带计算 `delta`、以及应使用多少个 warp/线程。`MatmulQK` 本身负责重建转置后的得分块 `K @ Q^T`，定义遍历累加片段的迭代器，并设置 bias 的共享内存 staging 和后续梯度流水线要消费的 back-to-back GEMM 存储格式。
```cpp

  struct MatmulGradV {
    /*
    grad_v[j_start:j_end] += attn_T @ do_i # matmul

    Dimensions: (kBlockSizeJ * kNumWarpsPerBlock, kBlockSizeI, K)
    (we might need to iterate multiple times on K)
    */
    using ThreadblockShape =
        cutlass::gemm::GemmShape<kBlockSizeJ, kBlockSizeI, GemmType::ThreadK>;
    using WarpShape = cutlass::gemm::GemmShape<32, 32, GemmType::WarpK>;
    using InstructionShape = typename GemmType::InstructionShape;

    using DefaultGemm = cutlass::gemm::kernel::DefaultGemm<
        scalar_t, // ElementA,
        cutlass::layout::RowMajor, // LayoutA,
        DefaultConfig::kAlignmentA,
        scalar_t, // ElementB,
        cutlass::layout::RowMajor, // LayoutB,
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment,
        output_t,
        cutlass::layout::RowMajor, // LayoutC,
        accum_t,
        typename GemmType::OpClass,
        ArchTag,
        ThreadblockShape,
        WarpShape,
        typename GemmType::InstructionShape,
        typename DefaultConfig::EpilogueOutputOp,
        void, // ThreadblockSwizzle - not used
        DefaultConfig::kStages,
        false, // SplitKSerial
        typename GemmType::Operator>;

    // if dropout:
    //   for computing dVj += (Pij.T * Zij) @ dOi
    //   Pij_dropped.T = Pij.T * Zij is computed on the fly as fragments of
    //   Pij.T are loaded in. The reason we do it this way is because Pij.T and
    //   Zij are reused in later steps, while Pij_dropped.T is only needed in
    //   this step. computing Pij_dropped.T on the fly allows us to avoid
    //   keeping all 3 of Pij_dropped.T, Pij.T, and Zij in shared memory at the
    //   same time.
    // if no dropout:
    //   for computing dVj += Pij.T @ dOi
    using WarpIteratorA = typename cutlass::gemm::threadblock::
        DefaultWarpIteratorAFromSharedMemory<
            typename DefaultGemm::Mma::Operator::Shape, // WarpShape
            typename DefaultGemm::Mma::Operator::
                InstructionShape, // InstructionShape
            typename DefaultGemm::Mma::Operator::
                IteratorA, // RegularWarpIterator
            typename DefaultGemm::Mma::Policy // Policy
            >::WarpIterator;
    using DefaultMmaFromSmem =
        typename cutlass::gemm::threadblock::DefaultMmaFromSharedMemory<
            typename DefaultGemm::Mma,
            MatmulQK::AccumulatorSharedStorage::Shape::kN,
            WarpIteratorA,
            kApplyDropout>; // kScaleOperandA

    using Mma = typename DefaultMmaFromSmem::Mma;
    using IteratorB = typename Mma::IteratorB;
    using WarpCount = typename Mma::WarpCount;

    // Epilogue
    using DefaultOutputOp = typename DefaultConfig::EpilogueOutputOp;
    using DefaultEpilogue = typename DefaultGemm::Epilogue;
    using OutputTileIterator =
        typename cutlass::epilogue::threadblock::MakePrefetchableIterator<
            typename DefaultEpilogue::OutputTileIterator>::Iterator;
    using AccumTileGmem = GmemTile<typename Mma::FragmentC, (int)kNumThreads>;
  };
```
**EN**: Lines 381-452. `MatmulGradV` is the backward equivalent of the forward value application stage: it computes `dV += P^T @ dO`. The key design choice is that operand A (the attention tile or its dropout-masked form) is read from shared memory rather than global memory, so the definition swaps in a `DefaultMmaFromSmem` path and configures iterators accordingly. The block also defines the epilogue iterators and a `GmemTile` specialization used when partial `dV` fragments need to be parked in global memory between iterations or across split-key reductions.
**CN**: 第381-452行：`MatmulGradV` 是前向 value 应用阶段在反向中的对应物：它计算 `dV += P^T @ dO`。这里的关键设计点在于操作数 A（注意力块或其 dropout 后形式）来自共享内存而非全局内存，因此实现切换到了 `DefaultMmaFromSmem` 路径，并据此配置相关迭代器。本段还定义了写回 `dV` 时使用的 epilogue 迭代器，以及在跨迭代或 split-key 规约时把部分 `dV` 片段暂存在全局内存中的 `GmemTile` 特化。
```cpp

  struct MatmulDOIVJ {
    /*
    doi_t_vj = do_i @ v_j.transpose(-2, -1) # matmul
    tmp = (doi_t_vj - Di.unsqueeze(1)) * attn # inplace / epilogue?
    */
    using ThreadblockShape =
        cutlass::gemm::GemmShape<kBlockSizeI, kBlockSizeJ, GemmType::ThreadK>;
    using WarpShape = cutlass::gemm::GemmShape<32, 32, GemmType::WarpK>;

    using ElementC = output_t;
    using ElementAccum = accum_t;

    // no-op output op - epilogue just stores result to global memory
    using BiasGradEpilogueOutputOp =
        typename cutlass::epilogue::thread::LinearCombination<
            ElementC,
            DefaultConfig::EpilogueOutputOp::kCount,
            typename DefaultConfig::EpilogueOutputOp::ElementAccumulator,
            typename DefaultConfig::EpilogueOutputOp::ElementCompute,
            cutlass::epilogue::thread::ScaleType::Nothing>;

    using DefaultGemm = typename cutlass::gemm::kernel::DefaultGemm<
        scalar_t, // ElementA
        cutlass::layout::RowMajor, // LayoutA
        kIsAligned ? DefaultConfig::kAlignmentA : GemmType::kMinimumAlignment,
        scalar_t, // ElementB
        cutlass::layout::ColumnMajor, // LayoutB
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment,
        ElementC, // ElementC
        cutlass::layout::RowMajor, // LayoutC
        ElementAccum, // ElementAccumulator
        typename GemmType::OpClass,
        ArchTag,
        ThreadblockShape,
        WarpShape,
        typename GemmType::InstructionShape,
        BiasGradEpilogueOutputOp, // EpilogueOutputOp
        void, // ThreadblockSwizzle (not used)
        // multiple preloads, dropout Zij tile, and 3 stages push us over shared
        // memory capacity on A100. set a ceiling on number of stages to save
        // shared memory if dropout is in use.
        kPreload && kApplyDropout && (kBlockSizeI * kBlockSizeJ > 64 * 64)
            ? cutlass::const_min(2, DefaultConfig::kStages)
            : DefaultConfig::kStages, // Stages
        false, // SplitKSerial
        typename GemmType::Operator,
        cutlass::gemm::SharedMemoryClearOption::kNone>;
    using Mma = typename MakeCustomMma<typename DefaultGemm::Mma, kMaxK>::Mma;
    using AccumLambdaIterator = typename DefaultMmaAccumLambdaIterator<
        typename Mma::Operator::IteratorC,
        ElementAccum,
        kWarpSize>::Iterator;

    // epilogue used to write bias gradient, which is just the output of this
    // matmul with some operations applied to the fragment
    using BiasGradEpilogue = typename DefaultGemm::Epilogue;

    // Epilogue to store to shared-memory in a format that we can use later for
    // the second matmul
    using B2bGemm = typename cutlass::gemm::threadblock::B2bGemm<
        typename DefaultGemm::Mma::Operator::IteratorC,
        typename DefaultGemm::Mma::Operator,
        scalar_t,
        WarpShape,
        ThreadblockShape>;
    using AccumulatorSharedStorage = typename B2bGemm::AccumulatorSharedStorage;
  };

  struct MatmulGradQ {
    // grad_q <- tmp @ k_j
    using ThreadblockShape =
        cutlass::gemm::GemmShape<kBlockSizeI, kBlockSizeJ, GemmType::ThreadK>;
    using WarpShape = cutlass::gemm::GemmShape<32, 32, GemmType::WarpK>;
    using InstructionShape = typename GemmType::InstructionShape;

    using DefaultGemm = cutlass::gemm::kernel::DefaultGemm<
        scalar_t, // ElementA,
        cutlass::layout::RowMajor, // LayoutA,
        DefaultConfig::kAlignmentA,
        scalar_t, // ElementB,
        cutlass::layout::RowMajor, // LayoutB,
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment,
        output_t,
        cutlass::layout::RowMajor, // LayoutC,
        accum_t,
        typename GemmType::OpClass,
        ArchTag,
        ThreadblockShape,
        WarpShape,
        typename GemmType::InstructionShape,
        typename DefaultConfig::EpilogueOutputOp,
        void, // ThreadblockSwizzle - not used
        DefaultConfig::kStages,
        false, // SplitKSerial
        typename GemmType::Operator>;

    using WarpIteratorA = typename cutlass::gemm::threadblock::
        DefaultWarpIteratorAFromSharedMemory<
            typename DefaultGemm::Mma::Operator::Shape,
            typename DefaultGemm::Mma::Operator::InstructionShape,
            typename DefaultGemm::Mma::Operator::IteratorA,
            typename DefaultGemm::Mma::Policy>::WarpIterator;
    using DefaultMmaFromSmem =
        typename cutlass::gemm::threadblock::DefaultMmaFromSharedMemory<
            typename DefaultGemm::Mma,
            MatmulDOIVJ::AccumulatorSharedStorage::Shape::kN,
            WarpIteratorA,
            false>; // kScaleOperandA
    using Mma = typename DefaultMmaFromSmem::Mma;
    using IteratorB = typename Mma::IteratorB;
    using WarpCount = typename Mma::WarpCount;

    // Epilogue
    using DefaultOutputOp = typename DefaultConfig::EpilogueOutputOp;
    using DefaultEpilogue = typename DefaultGemm::Epilogue;
    using OutputTileIterator =
        typename cutlass::epilogue::threadblock::MakePrefetchableIterator<
            typename DefaultEpilogue::OutputTileIterator>::Iterator;
    using AccumTileGmem = GmemTile<typename Mma::FragmentC, (int)kNumThreads>;
  };
```
**EN**: Lines 453-573. This section defines two connected computations. `MatmulDOIVJ` performs `dO @ V^T`, yielding the pre-softmax gradient contribution that will later be corrected by subtracting `delta` and multiplying by the reconstructed probabilities to form `dS`; its epilogue can also write the bias gradient `dB`. `MatmulGradQ` then consumes the resulting `dS` tile and computes `dQ += dS @ K`. Because the shared-memory source layout can vary depending on whether the temporary tile is stored transposed, the code selects the appropriate shared-memory-backed MMA path through template aliases rather than hard-coding a single iterator shape.
**CN**: 第453-573行：这一节定义了两个紧密相连的计算。`MatmulDOIVJ` 执行 `dO @ V^T`，得到 softmax 之前的梯度贡献；后续再减去 `delta` 并乘上重建的概率即可形成 `dS`，其 epilogue 还可以顺带写出 bias 梯度 `dB`。随后 `MatmulGradQ` 消费得到的 `dS` tile 并计算 `dQ += dS @ K`。由于临时 tile 在共享内存中的布局可能是转置或非转置的，代码通过模板别名选择合适的共享内存 MMA 路径，而不是把迭代器形状写死。
```cpp
  struct MatmulGradK {
    // grad_k <- tmp.transpose(-2, -1) @ q_i
    using ThreadblockShape =
        cutlass::gemm::GemmShape<kBlockSizeJ, kBlockSizeI, GemmType::ThreadK>;
    using WarpShape = cutlass::gemm::GemmShape<32, 32, GemmType::WarpK>;
    using InstructionShape = typename GemmType::InstructionShape;

    using DefaultGemm = cutlass::gemm::kernel::DefaultGemm<
        scalar_t, // ElementA,
        cutlass::layout::RowMajor, // LayoutA,
        DefaultConfig::kAlignmentA,
        scalar_t, // ElementB,
        cutlass::layout::RowMajor, // LayoutB,
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment,
        output_t,
        cutlass::layout::RowMajor, // LayoutC,
        accum_t,
        typename GemmType::OpClass,
        ArchTag,
        ThreadblockShape,
        WarpShape,
        typename GemmType::InstructionShape,
        typename DefaultConfig::EpilogueOutputOp,
        void, // ThreadblockSwizzle - not used
        DefaultConfig::kStages,
        false, // SplitKSerial
        typename GemmType::Operator>;

    using WarpIteratorA = typename cutlass::gemm::threadblock::
        DefaultWarpIteratorAFromSharedMemory<
            typename DefaultGemm::Mma::Operator::Shape,
            typename DefaultGemm::Mma::Operator::InstructionShape,
            typename DefaultGemm::Mma::Operator::IteratorA,
            typename DefaultGemm::Mma::Policy>::WarpIterator;
    using DefaultMmaFromSmemN =
        typename cutlass::gemm::threadblock::DefaultMmaFromSharedMemory<
            typename DefaultGemm::Mma,
            MatmulQK::AccumulatorSharedStorage::Shape::kN, // kMaxK
            WarpIteratorA,
            false>; // kScaleOperandA
    using DefaultMmaFromSmemT =
        typename cutlass::gemm::threadblock::DefaultMmaFromSharedMemory<
            typename DefaultGemm::Mma,
            MatmulDOIVJ::AccumulatorSharedStorage::Shape::kM, // kMaxK
            WarpIteratorA,
            false, // kScaleOperandA
            kPreload>; // kTransposeA
    using DefaultMmaFromSmem = typename cutlass::platform::conditional<
        DefaultMmaFromSmemT::kIsTransposedA,
        DefaultMmaFromSmemT,
        DefaultMmaFromSmemN>::type;
    using Mma = typename DefaultMmaFromSmem::Mma;
    using IteratorB = typename Mma::IteratorB;
    using WarpCount = typename Mma::WarpCount;

    // Epilogue
    using DefaultOutputOp = typename DefaultConfig::EpilogueOutputOp;
    using DefaultEpilogue = typename DefaultGemm::Epilogue;
    using OutputTileIterator =
        typename cutlass::epilogue::threadblock::MakePrefetchableIterator<
            typename DefaultEpilogue::OutputTileIterator>::Iterator;
    using AccumTileGmem = GmemTile<typename Mma::FragmentC, (int)kNumThreads>;
  };

  static constexpr bool kEnableSplitKeys = kEnableSplitKeys_;

  static constexpr bool kNeedsAccumGradQ = kEnableSplitKeys ||
      !cutlass::platform::is_same<output_accum_t, output_t>::value;
  static constexpr bool kNeedsAccumGradK = !kOutputInRF &&
      !cutlass::platform::is_same<output_accum_t, output_t>::value;
  static constexpr bool kNeedsAccumGradV = !kOutputInRF &&
      !cutlass::platform::is_same<output_accum_t, output_t>::value;
```
**EN**: Lines 574-646. The final GEMM policy block defines `MatmulGradK`, which computes `dK += dS^T @ Q`, and derives several accumulation-policy booleans. These flags decide whether `dQ`, `dK`, or `dV` can be written directly to their final destinations or must first accumulate through workspace buffers because of split-key parallelism or output-type mismatch. The nested `GradQTempStorage` structure packages the lock array, completion counters, and padding needed to coordinate split-key accumulation for `grad_query` safely and efficiently.
**CN**: 第574-646行：最后一个 GEMM 策略块定义了 `MatmulGradK`，用于计算 `dK += dS^T @ Q`，并推导出若干累加策略布尔量。这些标志决定 `dQ`、`dK`、`dV` 是否可以直接写到最终输出，还是因为 split-key 并行或输出类型不匹配而必须先经过工作区缓冲。本段中的 `GradQTempStorage` 结构则打包了锁数组、完成计数器以及必要的 padding，用于安全高效地协调 `grad_query` 的 split-key 累加。
```cpp
  struct GradQTempStorage {
    int32_t lock;
    int32_t counter;
    int32_t pad[2]; // pad to 128bits
    output_accum_t buffer[MatmulGradQ::AccumTileGmem::kElementsStored];
  };

  struct Params {
    // Input tensors
    scalar_t* query_ptr = nullptr; // [Mq, nH, K]
    scalar_t* key_ptr = nullptr; // [Mk, nH, K]
    scalar_t* value_ptr = nullptr; // [Mk, nH, Kv]
    scalar_t* bias_ptr = nullptr;
    lse_scalar_t* logsumexp_ptr = nullptr; // [nH, Mq]
    scalar_t* output_ptr = nullptr; // [Mq, nH, Kv]
    scalar_t* grad_output_ptr = nullptr; // [Mq, nH, Kv]
    accum_t* delta_ptr = nullptr; // [nH, Mq]
    int32_t* cu_seqlens_q_ptr = nullptr;
    int32_t* cu_seqlens_k_ptr = nullptr;

    // Output tensors
    output_t* grad_query_ptr = nullptr; //  [Mq, nH, K]
    output_t* grad_key_ptr = nullptr; //    [Mk, nH, K]
    output_t* grad_value_ptr = nullptr; //  [Mk, nH, Kv]
    output_t* grad_bias_ptr = nullptr;

    // Accumulators
    output_accum_t* workspace = nullptr; // [Mq, Kq] + [Mkv, Kq] + [Mkv, Kv]
    output_accum_t* workspace_gv =
        nullptr; // (will be calculated by the kernel)
    GradQTempStorage* workspace_gq =
        nullptr; // (will be calculated by the kernel)

    // Scale
    accum_t scale = 1.0f;

    // Dimensions/strides
    int32_t head_dim = -1;
    int32_t head_dim_value = -1;
    int32_t num_queries = -1;
    int32_t num_keys = -1;
    int32_t num_heads = -1;
    uint8_t custom_mask_type = NoCustomMask;

    int32_t q_strideM = -1;
    int32_t k_strideM = -1;
    int32_t v_strideM = -1;
    int32_t bias_strideM = 0;
    int32_t gO_strideM = -1;
    int32_t gB_strideM = -1;
    int8_t gQKV_strideM_multiplier = 1; // 3 for packed, 1 otherwise

#ifdef HAS_PYTORCH
    // dropout
    at::PhiloxCudaState rng_engine_inputs = {0, 0};
#endif
    // RNG sequence offset based on batch_id and head_id
    unsigned long long dropout_batch_head_rng_offset = 0;
    float dropout_prob = 0.0f;

    CUTLASS_HOST_DEVICE int32_t o_strideM() const {
      return head_dim_value * num_heads;
    }
    CUTLASS_HOST_DEVICE int32_t gQ_strideM() const {
      return gQKV_strideM_multiplier * num_heads * head_dim;
    }
    CUTLASS_HOST_DEVICE int32_t gK_strideM() const {
      return gQKV_strideM_multiplier * num_heads * head_dim;
    }
    CUTLASS_HOST_DEVICE int32_t gV_strideM() const {
      return gQKV_strideM_multiplier * num_heads * head_dim_value;
    }

    // Everything below is only used in `advance_to_block`
    // and shouldn't use registers
    int64_t o_strideH = -1;
    int32_t q_strideH = -1;
    int32_t k_strideH = -1;
    int32_t v_strideH = -1;
    int64_t bias_strideH = 0;
    int64_t o_strideB = -1;
    int64_t q_strideB = -1;
    int64_t k_strideB = -1;
    int64_t v_strideB = -1;
    int64_t bias_strideB = 0;
    int64_t lse_strideB = -1;
    int64_t lse_strideH = -1;
    int64_t delta_strideB = -1;
    int64_t delta_strideH = -1;
    int32_t num_batches = -1;
    int16_t num_splits_key = 1; // We use `gridDim.x` inside kernel

    int64_t gO_strideB = 0;
    int64_t gQ_strideB = 0;
    int64_t gK_strideB = 0;
    int64_t gV_strideB = 0;
    int64_t gB_strideB = 0;
    int64_t gO_strideH = 0;
    int64_t gQ_strideH = 0;
    int64_t gK_strideH = 0;
    int64_t gV_strideH = 0;
    int64_t gB_strideH = 0;

    CUTLASS_DEVICE int16_t num_splits_key_device() const {
      return kEnableSplitKeys ? gridDim.x : 1;
    }
    CUTLASS_DEVICE int16_t split_key_device() const {
      return kEnableSplitKeys ? blockIdx.x : 0;
    }

    CUTLASS_DEVICE bool advance_to_block() {
      int64_t batch_id = blockIdx.z;
      int32_t head_id = blockIdx.y;

      if (kNeedsAccumGradQ || kNeedsAccumGradK || kNeedsAccumGradV) {
        assert(workspace_size() == 0 || workspace != nullptr);

        workspace += (batch_id * num_heads + head_id) * workspace_strideBH();
        workspace = warp_uniform(workspace);
        workspace_gv = workspace + workspace_elements_gk();
        workspace_gq =
            (GradQTempStorage*)(workspace_gv + workspace_elements_gv());
        if (kEnableSplitKeys) {
          workspace_gv += workspace_elements_gv() * split_key_device() /
              num_splits_key_device();
          workspace += workspace_elements_gk() * split_key_device() /
              num_splits_key_device();
        }
      } else {
        workspace = nullptr;
      }

      // Advance pointers that depend on the total concatenated
      // number of queries, as `num_queries` is modified in the block
      // below
      dropout_batch_head_rng_offset =
          batch_id * (num_heads * num_queries * num_keys) +
          head_id * (num_queries * num_keys);
      logsumexp_ptr += batch_id * lse_strideB + head_id * lse_strideH;

      if (cu_seqlens_q_ptr != nullptr) {
        assert(cu_seqlens_k_ptr != nullptr);
        cu_seqlens_q_ptr += batch_id;
        cu_seqlens_k_ptr += batch_id;
        int32_t q_start = cu_seqlens_q_ptr[0];
        int32_t k_start = cu_seqlens_k_ptr[0];
        int64_t q_next_start = cu_seqlens_q_ptr[1];
        int64_t k_next_start = cu_seqlens_k_ptr[1];
        assert(q_next_start - q_start <= num_queries);
        assert(k_next_start - k_start <= num_keys);
        num_queries = q_next_start - q_start;
        num_keys = k_next_start - k_start;

        // Jump manually
        batch_id = 0;

        query_ptr += q_start * q_strideM;
        key_ptr += k_start * k_strideM;
        value_ptr += k_start * v_strideM;
        assert(bias_ptr == nullptr);
        assert(grad_bias_ptr == nullptr);
        output_ptr += q_start * o_strideM();
        grad_output_ptr += q_start * gO_strideM;
        delta_ptr += q_start;

        grad_query_ptr += q_start * gQ_strideM();
        grad_key_ptr += k_start * gK_strideM();
        grad_value_ptr += k_start * gV_strideM();
      }

      query_ptr += batch_id * q_strideB + head_id * q_strideH;
      key_ptr += batch_id * k_strideB + head_id * k_strideH;
      value_ptr += batch_id * v_strideB + head_id * v_strideH;
      if (bias_ptr != nullptr) {
        bias_ptr += batch_id * bias_strideB + head_id * bias_strideH;
      }
      output_ptr += batch_id * o_strideB + head_id * o_strideH;
      grad_output_ptr += batch_id * gO_strideB + head_id * gO_strideH;
      delta_ptr += batch_id * delta_strideB + head_id * delta_strideH;

      grad_query_ptr += batch_id * gQ_strideB + head_id * gQ_strideH;
      grad_key_ptr += batch_id * gK_strideB + head_id * gK_strideH;
      grad_value_ptr += batch_id * gV_strideB + head_id * gV_strideH;
      if (grad_bias_ptr != nullptr) {
        grad_bias_ptr += batch_id * gB_strideB + head_id * gB_strideH;
      }

      // Some values are modified above
      // Signal to the compiler that they are the same in all threads
      // and can be stored in warp-uniform registers (Sm75+)
      num_queries = warp_uniform(num_queries);
      num_keys = warp_uniform(num_keys);
      custom_mask_type = warp_uniform(custom_mask_type);

      query_ptr = warp_uniform(query_ptr);
      key_ptr = warp_uniform(key_ptr);
      value_ptr = warp_uniform(value_ptr);
      bias_ptr = warp_uniform(bias_ptr);
      logsumexp_ptr = warp_uniform(logsumexp_ptr);
      output_ptr = warp_uniform(output_ptr);
      grad_output_ptr = warp_uniform(grad_output_ptr);
      delta_ptr = warp_uniform(delta_ptr);

      grad_query_ptr = warp_uniform(grad_query_ptr);
      grad_key_ptr = warp_uniform(grad_key_ptr);
      grad_value_ptr = warp_uniform(grad_value_ptr);
      grad_bias_ptr = warp_uniform(grad_bias_ptr);

#if 0
      PRINT_T0("[b:%d h:%d] dp[0]:%f Q:%f K:%f V:%f LSE:%f",
        int(blockIdx.z), int(blockIdx.y),
        float(delta_ptr[0]),
        float(query_ptr[0]), float(key_ptr[0]), float(value_ptr[0]),
        float(logsumexp_ptr[0])
      )
#endif
      return true;
    }

    __host__ dim3 getBlocksGrid() const {
      return dim3(num_splits_key, num_heads, num_batches);
    }
    __host__ dim3 getThreadsGrid() const {
      return dim3(kWarpSize * kNumWarpsPerBlock, 1, 1);
    }
    CUTLASS_HOST_DEVICE int64_t workspace_elements_gk() const {
      if (!kNeedsAccumGradK) {
        return 0;
      }
      return num_splits_key * align_up(num_keys, (int32_t)kBlockSizeJ) *
          align_up(head_dim, (int32_t)kBlockSizeI);
    }
    CUTLASS_HOST_DEVICE int64_t workspace_elements_gv() const {
      if (!kNeedsAccumGradV) {
        return 0;
      }
      return num_splits_key * align_up(num_keys, (int32_t)kBlockSizeJ) *
          align_up(head_dim_value, (int32_t)kBlockSizeI);
    }
    CUTLASS_HOST_DEVICE int64_t workspace_elements_gq() const {
      if (!kNeedsAccumGradQ) {
        return 0;
      }
      int num_blocks = ceil_div(num_queries, kBlockSizeI);
      int num_cols = ceil_div(head_dim, MatmulGradQ::ThreadblockShape::kN);
      return num_blocks * num_cols * sizeof(GradQTempStorage) /
          sizeof(output_accum_t);
    }
    CUTLASS_HOST_DEVICE int64_t workspace_strideBH() const {
      // Aligned on 128bits
      return align_up(
          workspace_elements_gk() + workspace_elements_gv() +
              workspace_elements_gq(),
          int64_t(4));
    }
    CUTLASS_HOST_DEVICE int64_t workspace_size() const {
      // Returns size of buffer we need to run this kernel
      return num_batches * num_heads * workspace_strideBH() * sizeof(float);
    }
    CUTLASS_HOST_DEVICE bool should_zero_workspace() const {
      return num_splits_key > 1;
    }
  };
```
**EN**: Lines 647-909. The backward `Params` struct is substantially richer than the forward one because it must describe every input, output, workspace, and traversal rule used by the gradient kernel. Besides Q/K/V, output, grad-output, and logsumexp, it owns pointers for `delta`, `grad_query`, `grad_key`, `grad_value`, optional `grad_bias`, temporary workspaces, and split-key metadata. `advance_to_block()` computes the per-block pointer offsets for batch/head/query/key slices, handles variable-length sequences, partitions workspace for `dK`, `dV`, and `dQ`, derives the dropout RNG offset that matches forward ordering, and exposes helpers such as `num_splits_key_device()` and `split_key_device()` that define how the grid maps onto key partitions.
**CN**: 第647-909行：反向路径的 `Params` 结构比前向更复杂，因为它必须描述梯度内核使用到的全部输入、输出、工作区与遍历规则。除了 Q/K/V、输出、输出梯度和 logsumexp 外，它还维护 `delta`、`grad_query`、`grad_key`、`grad_value`、可选 `grad_bias`、临时工作区以及 split-key 元数据等指针。`advance_to_block()` 会为 batch/head/query/key 子块计算指针偏移，处理变长序列，为 `dK`、`dV`、`dQ` 划分工作区，推导与前向顺序一致的 dropout RNG 偏移，并提供 `num_splits_key_device()`、`split_key_device()` 等辅助函数来定义网格如何映射到 key 分区。
```cpp

  // shared storage for keeping Zij matrix. not needed if we aren't using
  // dropout, in which case we use an empty array to save shared memory
  using ZijSharedStorage = typename cutlass::platform::conditional<
      kApplyDropout,
      typename MatmulQK::AccumulatorSharedStorage,
      // dummy shared storage object that takes up no space.
      typename cutlass::gemm::threadblock::AccumulatorSharedStorage<
#ifdef _WIN32
          // windows builds throw the error:
          // "type containing an unknown-size array is not allowed"
          // if we try to make Zij shared storage zero-sized.
          // To get around this just make it sized 1 on windows.
          typename cutlass::gemm::GemmShape<1, 1, 0>,
#else
          typename cutlass::gemm::GemmShape<0, 0, 0>,
#endif
          typename MatmulQK::AccumulatorSharedStorage::Element,
          typename MatmulQK::AccumulatorSharedStorage::Layout,
          typename cutlass::MatrixShape<0, 0>>>::type;

  struct SharedStoragePrologue {
    struct {
      cutlass::Array<accum_t, kBlockSizeI> di; // (do_i * o_i).sum(-1)
      typename MatmulQK::Mma::SharedStorageA mm_qk_k;
    } persistent;
    union {
      struct {
        // part1 - after Q.K / dV / dO.V
        union {
          // 1. efficient load of bias tile Bij, which is then applied to Pij
          typename MatmulQK::BiasLoader::SmemTile bias;
          // 4. store Pij. it is needed:
          // - in dVj += (Pij.T * Zij) @ dOi
          // - in dSij = Pij * (dPij - Di)
          // 6. dVj += (Pij.T * Zij) @ dOi
          // 10. write to fragment
          typename MatmulQK::AccumulatorSharedStorage attn_shared_storage;
        };
        // 5. store Zij. it is needed in dVj += (Pij.T * Zij) @ dOi
        ZijSharedStorage zij;

        union {
          // 2. prologue for dVj
          // 6. workspace for dVj += (Pij.T * Zij) @ dOi
          typename MatmulGradV::Mma::SharedStorage mm_gradV;
          // 7. dVj epilogue
          typename MatmulGradV::DefaultEpilogue::SharedStorage gradV_epilogue;
        };

        // 3. prologue for dPij_dropped
        // 8. used in dPij_dropped = dOi @ Vj.T
        typename MatmulDOIVJ::Mma::SharedStorage mm_doivj;
      } part1;

      struct {
        // part2 - dQ
        union {
          typename MatmulQK::AccumulatorSharedStorage
              tmpT_shared_storage; // (from part1)
          typename MatmulDOIVJ::AccumulatorSharedStorage tmp_shared_storage;
        };
        typename MatmulGradK::Mma::SharedStorage mm_gradK; // (preload)
        typename MatmulGradQ::Mma::SharedStorage mm_gradQ; // (preload)
        union {
          // store dB = dSij to global memory
          typename MatmulDOIVJ::BiasGradEpilogue::SharedStorage gradB_epilogue;
          typename MatmulGradQ::DefaultEpilogue::SharedStorage gradQ_epilogue;
        };

      } part2;

      struct {
        // part3 - after last iteration on dQ's epilogue / dK
        union {
          typename MatmulQK::AccumulatorSharedStorage
              tmpT_shared_storage; // (from part1)
          typename MatmulDOIVJ::AccumulatorSharedStorage tmp_shared_storage;
        };
        typename MatmulGradK::Mma::SharedStorage mm_gradK; // (preload)
        typename MatmulGradQ::DefaultEpilogue::SharedStorage
            gradQ_epilogue_lastIter;

        typename MatmulGradK::DefaultEpilogue::SharedStorage gradK_epilogue;
      } part3;

      struct {
        // part4 - after last iteration on dK's epilogue / preload next K.Q_t
        typename MatmulQK::Mma::SharedStorageB mm_qk_q;

        // If we reach end of current key, dump RF->gmem with "final" epilogues
        typename MatmulGradK::DefaultEpilogue::SharedStorage
            gradK_epilogue_final;
        typename MatmulGradV::DefaultEpilogue::SharedStorage
            gradV_epilogue_final;
      } part4;
    };
    static void print_size() {
      // Field size
#define FSZ(f) int((sizeof(((SharedStoragePrologue*)0)->f)))

      printf("Total smem: %d bytes\n", int(sizeof(SharedStoragePrologue)));
      printf("  persistent: %db\n", FSZ(persistent));
      printf("    mm_qk_k: %db\n", FSZ(persistent.mm_qk_k));
      printf("  part1: %db\n", FSZ(part1));
      printf("    bias: %db\n", FSZ(part1.bias));
      printf("    attn_shared_storage: %db\n", FSZ(part1.attn_shared_storage));
      printf("    zij: %db\n", FSZ(part1.zij));
      printf("    mm_gradV: %db\n", FSZ(part1.mm_gradV));
      printf("    gradV_epilogue: %db\n", FSZ(part1.gradV_epilogue));
      printf("    mm_doivj: %db\n", FSZ(part1.mm_doivj));
      printf("  part2: %db\n", FSZ(part2));
      printf("    tmpT_shared_storage: %db\n", FSZ(part2.tmpT_shared_storage));
      printf("    tmp_shared_storage: %db\n", FSZ(part2.tmp_shared_storage));
      printf("    mm_gradK: %db\n", FSZ(part2.mm_gradK));
      printf("    mm_gradQ: %db\n", FSZ(part2.mm_gradQ));
      printf("    gradB_epilogue: %db\n", FSZ(part2.gradB_epilogue));
      printf("    gradQ_epilogue: %db\n", FSZ(part2.gradQ_epilogue));
      printf("  part3: %db\n", FSZ(part3));
      printf("    tmpT_shared_storage: %db\n", FSZ(part3.tmpT_shared_storage));
      printf("  part4: %db\n", FSZ(part4));
      printf("    mm_qk_q: %db\n", FSZ(part4.mm_qk_q));
      printf(
          "    gradK_epilogue_final: %db\n", FSZ(part4.gradK_epilogue_final));
      printf(
          "    gradV_epilogue_final: %db\n", FSZ(part4.gradV_epilogue_final));
    }
// ===========================================
#define FIELD(INSIDE_STRUCT, FIELDNAME) \
  CUTLASS_DEVICE auto& FIELDNAME() {    \
    return INSIDE_STRUCT.FIELDNAME;     \
  }

    FIELD(persistent, di)
    FIELD(persistent, mm_qk_k)
    FIELD(part1, bias)
    FIELD(part1, attn_shared_storage)
    FIELD(part1, zij)
    FIELD(part1, mm_gradV)
    FIELD(part1, gradV_epilogue)
    FIELD(part1, mm_doivj)
    FIELD(part2, mm_gradK)
    FIELD(part2, mm_gradQ)
    FIELD(part2, gradB_epilogue)
    FIELD(part2, gradQ_epilogue)
    FIELD(part2, tmp_shared_storage)
    FIELD(part3, tmpT_shared_storage)
    FIELD(part3, gradQ_epilogue_lastIter)
    FIELD(part3, gradK_epilogue)
    FIELD(part4, mm_qk_q)
    FIELD(part4, gradK_epilogue_final)
    FIELD(part4, gradV_epilogue_final)
  };

  struct SharedStorageNoPrologue {
    struct {
      cutlass::Array<accum_t, kBlockSizeI> di; // (do_i * o_i).sum(-1)
    } persistent;
    union {
      struct {
        // part1 - Q.K matmul
        typename MatmulQK::Mma::SharedStorageA mm_qk_k;
        typename MatmulQK::Mma::SharedStorageB mm_qk_q;
      } part1;

      struct {
        // part2 - compute gradV
        union {
          // 1. efficient load of bias tile Bij, which is then applied to Pij
          typename MatmulQK::BiasLoader::SmemTile bias;
          // 2. store Pij to shared memory. it is needed:
          // - in this step, where it is used in dVj += (Pij.T * Zij) @ dOi
          // - in next step where it is used in dSij = Pij * (dPij - Di)
          typename MatmulQK::AccumulatorSharedStorage attn_shared_storage;
        };
        // 3. store Zij. it is needed in this step, where it is used
        // to compute Pij_dropped = Pij * Zij on the fly as fragments of Pij are
        // loaded for the computation of dVj.
        ZijSharedStorage zij;

        union {
          typename MatmulGradV::Mma::SharedStorage mm_gradV;
          typename MatmulGradV::DefaultEpilogue::SharedStorage gradV_epilogue;
        };
      } part2;

      struct {
        // part3 - DO.V matmul
        union {
          // first compute dPij = (dOi @ Vj.T) * Zij
          // and dSij = Pij * (dPij - Di)
          struct {
            // (from part2) - Pij for computing dSij = Pij * (dPij - Di)
            typename MatmulQK::AccumulatorSharedStorage attn_shared_storage;
            // matmul to compute dOiVj
            typename MatmulDOIVJ::Mma::SharedStorage mm_doivj;
          };
          // then store dB = dSij to global memory
          typename MatmulDOIVJ::BiasGradEpilogue::SharedStorage gradB_epilogue;
        };
      } part3;

      struct {
        // part4 - compute gradQ
        typename MatmulQK::AccumulatorSharedStorage
            tmpT_shared_storage; // (from part2)
        typename MatmulDOIVJ::AccumulatorSharedStorage tmp_shared_storage;
        union {
          typename MatmulGradQ::Mma::SharedStorage mm_gradQ;
          typename MatmulGradQ::DefaultEpilogue::SharedStorage gradQ_epilogue;
          typename MatmulGradQ::DefaultEpilogue::SharedStorage
              gradQ_epilogue_lastIter;
        };
      } part4;

      struct {
        // part5 - compute gradK
        typename MatmulQK::AccumulatorSharedStorage
            tmpT_shared_storage; // (from part2)
        typename MatmulDOIVJ::AccumulatorSharedStorage tmp_shared_storage;
        union {
          typename MatmulGradK::Mma::SharedStorage mm_gradK;
          typename MatmulGradK::DefaultEpilogue::SharedStorage gradK_epilogue;
        };
      } part5;

      struct {
        // part6 - store RF accumulated into gmem
        typename MatmulGradK::DefaultEpilogue::SharedStorage
            gradK_epilogue_final;
        typename MatmulGradV::DefaultEpilogue::SharedStorage
            gradV_epilogue_final;
      } part6;
    };
    static void print_size() {
#define FIELD_SIZEOF(f) int((sizeof(((SharedStorageNoPrologue*)0)->f)))
      printf("Total smem: %d bytes\n", int(sizeof(SharedStorageNoPrologue)));
      printf("  persistent: %db\n", FIELD_SIZEOF(persistent));
      printf("  part1: %db\n", FIELD_SIZEOF(part1));
      printf("  part2: %db\n", FIELD_SIZEOF(part2));
      printf("  part3: %db\n", FIELD_SIZEOF(part3));
      printf("  part4: %db\n", FIELD_SIZEOF(part4));
      printf("  part5: %db\n", FIELD_SIZEOF(part5));
      printf("  part6: %db\n", FIELD_SIZEOF(part6));
    }
// ===========================================
#define FIELD(INSIDE_STRUCT, FIELDNAME) \
  CUTLASS_DEVICE auto& FIELDNAME() {    \
    return INSIDE_STRUCT.FIELDNAME;     \
  }

    FIELD(persistent, di)
    FIELD(part1, mm_qk_k)
    FIELD(part1, mm_qk_q)
    FIELD(part2, bias)
    FIELD(part2, attn_shared_storage)
    FIELD(part2, zij)
    FIELD(part2, mm_gradV)
    FIELD(part2, gradV_epilogue)
    FIELD(part3, mm_doivj)
    FIELD(part3, gradB_epilogue)
    FIELD(part4, tmpT_shared_storage)
    FIELD(part4, tmp_shared_storage)
    FIELD(part4, mm_gradQ)
    FIELD(part4, gradQ_epilogue)
    FIELD(part4, gradQ_epilogue_lastIter)
    FIELD(part5, mm_gradK)
    FIELD(part5, gradK_epilogue)
    FIELD(part6, gradK_epilogue_final)
    FIELD(part6, gradV_epilogue_final)
  };

  using SharedStorage = typename cutlass::platform::conditional<
      kPreload,
      SharedStoragePrologue,
      SharedStorageNoPrologue>::type;
```
**EN**: Lines 910-1185. This long block defines the on-chip memory layout used by the backward pipeline. There are two variants: `SharedStoragePrologue` for the preload-heavy path and `SharedStorageNoPrologue` for the leaner path. Both layouts carve the same physical shared memory into named regions for QK operands, reconstructed attention tiles, dropout masks, `dV` epilogues, `dO·V^T` temporaries, transposed staging buffers, and final `dQ`/`dK` epilogues; the preload-enabled version simply reserves more structure so one iteration can prepare data for the next. The accessor macros and `print_size()` helpers make the otherwise dense memory partitioning easier to reason about and debug.
**CN**: 第910-1185行：这一大段代码定义了反向流水线所使用的片上共享内存布局。实现提供两种变体：预取更积极的 `SharedStoragePrologue`，以及更精简的 `SharedStorageNoPrologue`。两者都把同一片物理共享内存切分为多个命名区域，用于存放 QK 操作数、重建的注意力块、dropout mask、`dV` epilogue、`dO·V^T` 临时结果、转置 staging 区，以及最终 `dQ`/`dK` epilogue；其中启用预取的版本只是额外预留了更多结构，让当前迭代可以为下一迭代准备数据。访问器宏和 `print_size()` 辅助函数则使这种高密度内存分区更易理解和调试。
```cpp

  struct OutputFragments {
    typename MatmulGradV::Mma::FragmentC gradV;
    typename MatmulGradK::Mma::FragmentC gradK;

    CUTLASS_DEVICE void clear() {
      gradV.clear();
      gradK.clear();
    }
  };

  static bool __host__ check_supported(Params const& p) {
    CHECK_ALIGNED_PTR(p.query_ptr, kMinimumAlignment);
    CHECK_ALIGNED_PTR(p.key_ptr, kMinimumAlignment);
    CHECK_ALIGNED_PTR(p.value_ptr, kMinimumAlignment);
    CHECK_ALIGNED_PTR(p.output_ptr, kMinimumAlignment);
    CHECK_ALIGNED_PTR(p.grad_output_ptr, kMinimumAlignment);
    CHECK_ALIGNED_PTR(p.bias_ptr, kMinimumAlignment);
    XFORMERS_CHECK(p.lse_strideH % 8 == 0, "LSE is not correctly aligned");
    XFORMERS_CHECK(p.lse_strideB % 8 == 0, "LSE is not correctly aligned");
    XFORMERS_CHECK(
        p.num_heads <= 1 || p.q_strideH % kMinimumAlignment == 0,
        "query is not correctly aligned (strideH)");
    XFORMERS_CHECK(
        p.num_heads <= 1 || p.k_strideH % kMinimumAlignment == 0,
        "key is not correctly aligned (strideH)");
    XFORMERS_CHECK(
        p.num_heads <= 1 || p.v_strideH % kMinimumAlignment == 0,
        "value is not correctly aligned (strideH)");
    XFORMERS_CHECK(
        p.num_batches <= 1 || p.q_strideB % kMinimumAlignment == 0,
        "query is not correctly aligned (strideB)");
    XFORMERS_CHECK(
        p.num_batches <= 1 || p.k_strideB % kMinimumAlignment == 0,
        "key is not correctly aligned (strideB)");
    XFORMERS_CHECK(
        p.num_batches <= 1 || p.v_strideB % kMinimumAlignment == 0,
        "value is not correctly aligned (strideB)");
    XFORMERS_CHECK(
        p.q_strideM % kMinimumAlignment == 0,
        "query is not correctly aligned (strideM)");
    XFORMERS_CHECK(
        p.k_strideM % kMinimumAlignment == 0,
        "key is not correctly aligned (strideM)");
    XFORMERS_CHECK(
        p.v_strideM % kMinimumAlignment == 0,
        "value is not correctly aligned (strideM)");
    if (p.bias_ptr) {
      XFORMERS_CHECK(
          p.num_batches <= 1 || p.bias_strideB % kMinimumAlignment == 0,
          "attn_bias is not correctly aligned (strideB)");
      XFORMERS_CHECK(
          p.num_heads <= 1 || p.bias_strideH % kMinimumAlignment == 0,
          "attn_bias is not correctly aligned (strideH)");
      XFORMERS_CHECK(
          p.bias_strideM % kMinimumAlignment == 0,
          "attn_bias is not correctly aligned (strideM)");
    }
    if (p.grad_bias_ptr) {
      XFORMERS_CHECK(
          p.num_batches <= 1 || p.gB_strideB % kMinimumAlignment == 0,
          "attn_bias.grad is not correctly aligned (strideB)");
      XFORMERS_CHECK(
          p.num_heads <= 1 || p.gB_strideH % kMinimumAlignment == 0,
          "attn_bias.grad is not correctly aligned (strideH)");
      XFORMERS_CHECK(
          p.gB_strideM % kMinimumAlignment == 0,
          "attn_bias.grad is not correctly aligned (strideM)");
    }
    XFORMERS_CHECK(
        !(p.cu_seqlens_q_ptr && p.bias_ptr),
        "CuSeqlen + bias not implemented yet");
    XFORMERS_CHECK(
        p.custom_mask_type < NumCustomMaskTypes,
        "Invalid value for `custom_mask_type`");
    XFORMERS_CHECK(
        p.dropout_prob <= 1.0f && p.dropout_prob >= 0.0f,
        "Invalid value for `dropout_prob`");
    XFORMERS_CHECK(
        kApplyDropout || p.dropout_prob == 0.0f,
        "Set `kApplyDropout`=True to support `dropout_prob > 0`");
    XFORMERS_CHECK(p.head_dim > 0, "Invalid value for `head_dim`");
    XFORMERS_CHECK(p.head_dim_value > 0, "Invalid value for `head_dim_value`");
    XFORMERS_CHECK(p.num_queries > 0, "Invalid value for `num_queries`");
    XFORMERS_CHECK(p.num_keys > 0, "Invalid value for `num_keys`");
    XFORMERS_CHECK(p.num_heads > 0, "Invalid value for `num_heads`");
    XFORMERS_CHECK(p.num_batches > 0, "Invalid value for `num_batches`");
    XFORMERS_CHECK(p.head_dim <= kMaxK, "kMaxK: Expected `head_dim < kMaxK`");
    XFORMERS_CHECK(
        p.head_dim_value <= kMaxK, "kMaxK: Expected `head_dim_value < kMaxK`");
    if (kKeysQueriesAlignedToBlockSize) {
      XFORMERS_CHECK(
          p.cu_seqlens_k_ptr == nullptr,
          "This kernel does not support cu_seqlen");
      XFORMERS_CHECK(
          p.cu_seqlens_q_ptr == nullptr,
          "This kernel does not support cu_seqlen");
      XFORMERS_CHECK(
          p.num_queries % kBlockSizeI == 0,
          "kKeysQueriesAlignedToBlockSize condition not respected");
      XFORMERS_CHECK(
          p.num_keys % kBlockSizeJ == 0,
          "kKeysQueriesAlignedToBlockSize condition not respected");
    }
    XFORMERS_CHECK(
        kEnableSplitKeys || p.num_splits_key == 1, "SplitKeys is disabled");
    XFORMERS_CHECK(
        p.num_splits_key > 0, "Invalid `num_splits_key` (expected >0)");
    XFORMERS_CHECK(
        p.num_splits_key <= cutlass::ceil_div(p.num_keys, kBlockSizeJ),
        "Invalid `num_splits_key` (too large)");
    return true;
  }
```
**EN**: Lines 1186-1298. `OutputFragments` packages the register-resident `gradV` and `gradK` accumulators that may survive across multiple key blocks when the kernel chooses to delay global writeback. The adjacent `check_supported()` routine is the runtime gatekeeper: it verifies pointer alignment, stride alignment, head-dimension bounds, mask/dropout constraints, and split-key legality before launch. This combination is important because many later fast paths assume those invariants without rechecking them inside the kernel body.
**CN**: 第1186-1298行：`OutputFragments` 封装了可驻留在寄存器中的 `gradV` 与 `gradK` 累加片段；当内核选择推迟全局写回时，这些片段可能跨多个 key 块持续存在。紧邻的 `check_supported()` 则是运行时守门人：它在 launch 前验证指针对齐、步幅对齐、head 维度上界、mask/dropout 约束，以及 split-key 合法性。二者组合很关键，因为后续很多快路径都默认这些不变量成立，而不会在内核体内重复检查。
```cpp

  static CUTLASS_DEVICE void attention_kernel(Params p) {
    extern __shared__ char smem_buffer[];
    SharedStorage& shared_storage = *((SharedStorage*)smem_buffer);

    uint16_t thread_id = threadIdx.x;
    uint8_t warp_id = warp_uniform(thread_id / 32);
    uint8_t lane_id = thread_id % 32;

    int32_t key_start = p.split_key_device() * kBlockSizeJ;
    if (key_start >= p.num_keys) {
      return;
    }
    if (kPrologueQK) {
      int32_t query_start = getQueryStart(p, key_start);
      prologueQkNextIteration<true>(
          shared_storage, p, query_start, key_start, warp_id, lane_id);
    }

    // Computes (dO*out).sum(-1) and writes it to `p.delta_ptr`
    if (kKernelComputesDelta) {
      constexpr int kOptimalElements =
          128 / cutlass::sizeof_bits<scalar_t>::value;
      if (p.head_dim_value % kOptimalElements == 0) {
        for (int query_start = 0; query_start < p.num_queries;
             query_start += kBlockSizeI) {
          computeDelta<kOptimalElements>(p, query_start, warp_id, lane_id);
        }
      } else {
        for (int query_start = 0; query_start < p.num_queries;
             query_start += kBlockSizeI) {
          computeDelta<1>(p, query_start, warp_id, lane_id);
        }
      }
      __syncthreads();
    }

    OutputFragments output_frags;

    curandStatePhilox4_32_10_t rng_state_init;
#ifdef HAS_PYTORCH
    if (kApplyDropout) {
      auto seeds = at::cuda::philox::unpack(p.rng_engine_inputs);
      // each element of the attention matrix P with shape
      // (batch_sz, n_heads, n_queries, n_keys) is associated with a single
      // offset in RNG sequence. we initialize the RNG state with offset that
      // starts at the beginning of a (n_queries, n_keys) matrix for this
      // block's batch_id and head_id
      // initializing rng state is very expensive, so we run once per kernel,
      // rather than once per iteration. each iteration takes a copy of the
      // initialized RNG state and offsets it as needed.
      curand_init(
          std::get<0>(seeds),
          0,
          std::get<1>(seeds) + p.dropout_batch_head_rng_offset,
          &rng_state_init);
    }
#endif
    CUTLASS_PRAGMA_UNROLL
    for (; key_start < p.num_keys;
         key_start += p.num_splits_key_device() * kBlockSizeJ) {
      output_frags.clear();

      CUTLASS_PRAGMA_UNROLL
      for (int32_t query_start_shifted = getQueryStart(p, key_start);
           query_start_shifted < getQueryStartShift(p) + getQueryEnd(p);
           query_start_shifted += kBlockSizeI) {
        // This line here
        // vvvvvvvvvvvvvv
        warp_id = warp_uniform(warp_id);
        // ^^^^^^^^^^^^^^
        // ... makes everything use less RF and be 10% faster. Why?
        // I don't know. My theory is that it forces `nvcc` to
        // re-compute indices, offsets etc... and not keep them
        // from the previous iteration, which prevents MASSIVE
        // register spilling.

        int32_t query_start = query_start_shifted;
        if (query_start >= p.num_queries) {
          query_start = query_start % getQueryEnd(p);
        }

        processBlockIJ<kKeysQueriesAlignedToBlockSize>(
            shared_storage,
            output_frags,
            p,
            query_start,
            key_start,
            rng_state_init,
            warp_id,
            lane_id);
      }
      if (kOutputInRF) {
        writeFragsToGmem<kKeysQueriesAlignedToBlockSize>(
            shared_storage, output_frags, p, key_start, warp_id, lane_id);
      } else if (getQueryStart(p, key_start) >= p.num_queries) {
        zfillGradKV<kKeysQueriesAlignedToBlockSize>(
            p, key_start, warp_id, lane_id);
      }
      __syncthreads();
    }
  }
```
**EN**: Lines 1299-1400. This block begins the actual backward kernel. After binding shared storage and deriving thread/warp identities, `attention_kernel()` checks whether the current block has work, optionally preloads the first Q/K tile, and may call `computeDelta()` to generate the per-query `delta = sum(dO * O)` term that later corrects the softmax gradient. It then initializes Philox state for dropout and enters the outer traversal over key/query tiles, with additional bookkeeping to flush register-held outputs or zero-fill dead regions when the traversal finishes.
**CN**: 第1299-1400行：这一部分真正进入反向内核主体。`attention_kernel()` 在绑定共享存储并计算线程/warp 身份后，会检查当前块是否有有效工作，可选地预取首个 Q/K tile，并可能调用 `computeDelta()` 生成每个 query 的 `delta = sum(dO * O)` 项，后续 softmax 梯度修正要用到它。随后代码为 dropout 初始化 Philox 状态，并进入按 key/query tile 的外层遍历；在遍历结束时还要负责刷新寄存器中暂存的输出，或对无效区域做零填充。
```cpp

  template <bool skipBoundsChecks>
  static CUTLASS_DEVICE void zfillGradKV(
      Params const& p,
      int32_t key_start,
      uint8_t warp_id,
      uint8_t lane_id) {
    constexpr int kThreadsPerKey = 8;
    constexpr int kParallelKeys = kNumThreads / kThreadsPerKey;
    static_assert(kBlockSizeJ % kParallelKeys == 0, "");
    // This function is not really optimized, but should rarely be used
    // It's only used when some keys are "useless" and don't attend to
    // any query, due to causal masking

    int thread_id = 32 * warp_id + lane_id;
    int k_shift = lane_id % kThreadsPerKey;

    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < kBlockSizeJ; j += kParallelKeys) {
      int key = key_start + j + (thread_id / kThreadsPerKey);
      if (!skipBoundsChecks && key >= p.num_keys) {
        continue;
      }
      auto gv_ptr = p.grad_value_ptr + key * p.gV_strideM();
      auto gk_ptr = p.grad_key_ptr + key * p.gK_strideM();

      for (int k = k_shift; k < p.head_dim_value; k += kThreadsPerKey) {
        gv_ptr[k] = scalar_t(0);
      }
      for (int k = k_shift; k < p.head_dim; k += kThreadsPerKey) {
        gk_ptr[k] = scalar_t(0);
      }
    }
  }

  template <bool skipBoundsChecks>
  static CUTLASS_DEVICE void processBlockIJ(
      SharedStorage& shared_storage,
      OutputFragments& output_frags,
      Params& p,
      int32_t query_start,
      int32_t key_start,
      const curandStatePhilox4_32_10_t& curand_state_init,
      uint8_t warp_id,
      uint8_t lane_id) {
    cutlass::Array<cutlass::uint1b_t, MatmulDOIVJ::Mma::FragmentC::kElements>
        dropout_keep_mask_doivj;
    dropout_keep_mask_doivj.fill(cutlass::uint1b_t{1});
    const float dropout_scale =
        kApplyDropout ? 1.0 / (1.0 - p.dropout_prob) : 1.0f;

    cutlass::MatrixCoord no_offset{0, 0};
    accum_t scale = p.scale;
    int16_t thread_id = 32 * warp_id + lane_id;

    auto rematerializeThreadIds = [&]() {
      // Prevents `nvcc` from keeping values deduced from
      // `thread_id`, `warp_id`, ... in RF - to reduce register pressure
      warp_id = warp_uniform(thread_id / 32);
      lane_id = thread_id % 32;
      thread_id = 32 * warp_id + lane_id;
    };

    bool isFirstQuery = (query_start == getQueryStart(p, key_start));
    int32_t next_query, next_key;
    incrIteration(p, query_start, key_start, next_query, next_key);
    bool isLastQuery = next_key != key_start;

    accum_t di_rf = accum_t(0);
    if (thread_id < kBlockSizeI) {
      if (query_start + thread_id < p.num_queries) {
        di_rf = p.delta_ptr[query_start + thread_id];
      }
      shared_storage.di()[thread_id] = di_rf;
    }

    int32_t num_queries_in_block = skipBoundsChecks
        ? MatmulQK::Mma::Shape::kN
        : warp_uniform(cutlass::fast_min(
              (int32_t)MatmulQK::Mma::Shape::kN, p.num_queries - query_start));
    int32_t num_keys_in_block = skipBoundsChecks
        ? MatmulQK::Mma::Shape::kM
        : warp_uniform(cutlass::fast_min(
              (int32_t)MatmulQK::Mma::Shape::kM, p.num_keys - key_start));

    auto prologueGradV = [&](int col) {
      typename MatmulGradV::Mma::IteratorB iterator_dO(
          {int32_t(p.gO_strideM)},
          p.grad_output_ptr + query_start * p.gO_strideM + col,
          {num_queries_in_block, p.head_dim_value - col},
          thread_id,
          no_offset);
      MatmulGradV::Mma::prologue(
          shared_storage.mm_gradV(),
          iterator_dO,
          thread_id,
          num_queries_in_block);
    };
    auto prologueGradQ = [&](int col) {
      typename MatmulGradQ::Mma::IteratorB iterator_K(
          {int32_t(p.k_strideM)},
          p.key_ptr + key_start * p.k_strideM + col,
          {num_keys_in_block, p.head_dim - col},
          thread_id,
          no_offset);
      MatmulGradQ::Mma::prologue(
          shared_storage.mm_gradQ(), iterator_K, thread_id, num_keys_in_block);
    };
    auto prologueGradK = [&](int col) {
      typename MatmulGradK::Mma::IteratorB iterator_Q(
          {int32_t(p.q_strideM)},
          p.query_ptr + query_start * p.q_strideM + col,
          {num_queries_in_block, p.head_dim - col},
          thread_id,
          no_offset);
      MatmulGradK::Mma::prologue(
          shared_storage.mm_gradK(),
          iterator_Q,
          thread_id,
          num_queries_in_block);
    };
    auto prologueDOV = [&]() {
      typename MatmulDOIVJ::Mma::IteratorA iterator_A(
          {int32_t(p.gO_strideM)},
          p.grad_output_ptr + query_start * p.gO_strideM,
          {num_queries_in_block, p.head_dim_value},
          thread_id,
          no_offset);
      typename MatmulDOIVJ::Mma::IteratorB iterator_B(
          {int32_t(p.v_strideM)},
          p.value_ptr + key_start * p.v_strideM,
          {p.head_dim_value, num_keys_in_block},
          thread_id,
          no_offset);
      MatmulDOIVJ::Mma::prologue(
          shared_storage.mm_doivj(),
          iterator_A,
          iterator_B,
          thread_id,
          p.head_dim_value);
    };

    /////////////////////////////////////////////////////////////////////////////////////////////////
    // MatmulQK
    /////////////////////////////////////////////////////////////////////////////////////////////////
    {
      using Mma = typename MatmulQK::Mma;

      cutlass::gemm::GemmCoord problem_size(
          num_keys_in_block,
          num_queries_in_block,
          p.head_dim // k
      );

      // k_j
      typename Mma::IteratorA iterator_A(
          {int32_t(p.k_strideM)},
          p.key_ptr + key_start * p.k_strideM,
          {problem_size.m(), problem_size.k()},
          thread_id,
          no_offset);

      // q_i.transpose(-2, -1)
      typename Mma::IteratorB iterator_B(
          {int32_t(p.q_strideM)},
          p.query_ptr + query_start * p.q_strideM,
          {problem_size.k(), problem_size.n()},
          thread_id,
          no_offset);

      Mma mma(
          shared_storage.mm_qk_k(),
          shared_storage.mm_qk_q(),
          thread_id,
          warp_id,
          lane_id);

      typename Mma::FragmentC accum;

      accum.clear();

      auto gemm_k_iterations =
          (problem_size.k() + Mma::Shape::kK - 1) / Mma::Shape::kK;

      // Compute threadblock-scoped matrix multiply-add
      mma.set_prologue_done(kPrologueQK);
      mma.set_zero_outside_bounds(!skipBoundsChecks);
      mma(gemm_k_iterations, accum, iterator_A, iterator_B, accum);
      accum = cutlass::multiplies<typename Mma::FragmentC>()(scale, accum);

      // Epilogue: add LSE + exp and store that to our shared memory buffer
      // shmem <- (matmul_result -
      // logsumexp[i_start:i_end].unsqueeze(1)).exp()
      int warp_idx_mn_0 =
          warp_id % (Mma::Base::WarpCount::kM * Mma::Base::WarpCount::kN);
      auto output_tile_coords = cutlass::MatrixCoord{
          warp_idx_mn_0 % Mma::Base::WarpCount::kM,
          warp_idx_mn_0 / Mma::Base::WarpCount::kM};

      // apply bias if applicable
      if (p.bias_ptr != nullptr) {
        // load bias tile Bij into shared memory
        typename MatmulQK::BiasLoader::GmemTileIterator bias_iter(
            {cutlass::layout::RowMajor(p.bias_strideM)},
            p.bias_ptr + query_start * p.bias_strideM + key_start,
            {num_queries_in_block, num_keys_in_block},
            thread_id);
        cutlass::TensorRef<scalar_t, cutlass::layout::RowMajor> bias_tensor_ref(
            shared_storage.bias().data(),
            cutlass::layout::RowMajor(MatmulQK::ThreadblockShape::kM));
        typename MatmulQK::BiasLoader::SmemTileIterator smem_tile_iter(
            bias_tensor_ref, thread_id);
        MatmulQK::BiasLoader::load(bias_iter, smem_tile_iter);

        // Pij += Bij, where Pij is in register fragment and Bij is in shmem
        auto lane_offset = MatmulQK::AccumLambdaIterator::get_lane_offset(
            lane_id, warp_id, output_tile_coords);
        MatmulQK::AccumLambdaIterator::iterateRows(
            lane_offset,
            [&](int accum_n) {},
            [&](int accum_m, int accum_n, int idx) {
              // remember we are transposed
              accum[idx] += bias_tensor_ref.at({accum_n, accum_m});
            },
            [&](int accum_n) {});
      }

      // Apply mask
      if (p.custom_mask_type == CausalFromTopLeft ||
          p.custom_mask_type == CausalFromBottomRight) {
        auto lane_offset = MatmulQK::AccumLambdaIterator::get_lane_offset(
            lane_id, warp_id, output_tile_coords);
        int shift = query_start - key_start;
        if (p.custom_mask_type == CausalFromBottomRight) {
          shift += p.num_keys - p.num_queries;
        }
        // current_key = key_start + accum_m
        // current_query = query_start + accum_n
        // mask if: `current_key > current_query`
        MatmulQK::AccumLambdaIterator::iterateRows(
            lane_offset,
            [&](int accum_m) {},
            [&](int accum_m, int accum_n, int idx) {
              if (accum_m > accum_n + shift) {
                accum[idx] =
                    -cutlass::platform::numeric_limits<accum_t>::infinity();
              }
            },
            [&](int accum_m) {});
      }

      __syncthreads();
      if (kPrologueGV) {
        prologueGradV(0);
      }
      if (kPrologueDOV) {
        prologueDOV();
      }

      MatmulQK::B2bGemm::accumApplyLSEToSmem(
          shared_storage.attn_shared_storage(),
          accum,
          p.logsumexp_ptr + query_start,
          problem_size.n(),
          thread_id,
          warp_id,
          lane_id,
          output_tile_coords);
#if 0
      auto accum_ref_attnT = shared_storage.attn_shared_storage().accum_ref();
      PRINT_TENSOR4x4_T0_L0("attn_T", accum_ref_attnT);
#endif

      // if we are using dropout, compute Zij, writing it to shared memory.
      // each element of Zij is:
      // - 0 with probability dropout_p
      // - 1 / (1 - dropout_p) with probability 1 - dropout_p
      if (kApplyDropout) {
        auto zij = shared_storage.zij().accum_ref();
        // each thread generates a contiguous sequence of elements in Zij, all
        // in the same row. the reason they have to come from the same row is
        // that sampling random numbers from a contiguous random number sequence
        // is much more efficient than jumping around, and the linear offset of
        // each element of Z (the global matrix) maps to an offset in a random
        // number sequence. for Z, the end of a row and the beginning of the
        // next have adjacent offsets, but for Zij (tile of global matrix), this
        // is not necessarily the case.
        // We must fill the entire `zij` shmem with values (even out of bounds
        // on the K-dimension) otherwise we can get NaNs during the GEMM
        const int kQueriesPerBlock = kBlockSizeI;
        const int threads_per_row = cutlass::fast_min(
            int32_t(kNumThreads / kQueriesPerBlock), num_keys_in_block);
        const int elts_per_thread = cutlass::round_nearest(
            cutlass::ceil_div(num_keys_in_block, threads_per_row), 4);

        const int thread_i = thread_id / threads_per_row;
        const int thread_start_j =
            (thread_id % threads_per_row) * elts_per_thread;

        if (thread_i < kQueriesPerBlock && thread_start_j < num_keys_in_block) {
          curandStatePhilox4_32_10_t curand_state = curand_state_init;
          skipahead(
              (query_start + thread_i) * p.num_keys +
                  (key_start + thread_start_j),
              &curand_state);

          // generate elements of Zij, 4 elements at a time
          for (int zij_start_col_idx = thread_start_j; zij_start_col_idx <
               cutlass::fast_min<int32_t>(thread_start_j + elts_per_thread,
                                          num_keys_in_block);
               zij_start_col_idx += 4) {
            const float4 rand_uniform_quad = curand_uniform4(&curand_state);

            CUTLASS_PRAGMA_UNROLL
            for (int quad_idx = 0; quad_idx < 4; ++quad_idx) {
              // we'll write Zij transposed since attention is also transposed
              // during the matmul to compute dV.
              zij.at({zij_start_col_idx + quad_idx /*k*/, thread_i /*q*/}) =
                  (&rand_uniform_quad.x)[quad_idx] > p.dropout_prob
                  ? scalar_t(dropout_scale)
                  : scalar_t(0);
            }
          }
        }
        __syncthreads();
#if 0
        PRINT_TENSOR4x4_T0_L0("zij", zij);
        PRINT_TENSOR4x4_T0_L0_START("zij", zij, kBlockSizeJ - 4, kBlockSizeI - 4);
#endif

        // Save mask for later DOIVJ matmul

        int warp_idx_mn_0 = warp_id %
            (MatmulDOIVJ::Mma::Base::WarpCount::kM *
             MatmulDOIVJ::Mma::Base::WarpCount::kN);
        auto output_tile_coords_doivj = cutlass::MatrixCoord{
            warp_idx_mn_0 % MatmulDOIVJ::Mma::Base::WarpCount::kM,
            warp_idx_mn_0 / MatmulDOIVJ::Mma::Base::WarpCount::kM};
        auto lane_offset = MatmulDOIVJ::AccumLambdaIterator::get_lane_offset(
            lane_id, warp_id, output_tile_coords_doivj);
        MatmulDOIVJ::AccumLambdaIterator::iterateRows(
            lane_offset,
            [&](int accum_m) {},
            [&](int accum_m /*q*/, int accum_n /*k*/, int idx) {
              if (zij.at({accum_n, accum_m}) == scalar_t(0)) {
                dropout_keep_mask_doivj[idx] = cutlass::uint1b_t{0};
              }
            },
            [&](int accum_m) {});
      }
      __syncthreads();
    }
    rematerializeThreadIds();
```
**EN**: Lines 1401-1753. This is the setup-heavy first half of `processBlockIJ()`. For each `(query_start, key_start)` pair, the kernel determines tile bounds, loads or computes the `delta`-related row term, reconstructs the score tile with `MatmulQK`, rescales it by the original attention scale, adds bias if present, and reapplies the same causal mask logic used in forward. It then uses the stored logsumexp to recover the normalized probabilities in shared memory, optionally builds a dropout mask tile that exactly matches forward RNG ordering, and prepares the shared-memory views and prologue lambdas needed by the later `dV`, `dQ`, and `dK` GEMMs. This block is where the backward kernel conceptually reconstructs the forward softmax state instead of loading a full probability matrix from memory.
**CN**: 第1401-1753行：这是 `processBlockIJ()` 中以准备工作为主的前半段。对于每个 `(query_start, key_start)` 组合，内核会先确定 tile 边界，加载或计算与 `delta` 相关的行项，再通过 `MatmulQK` 重建得分块，按原始 attention scale 缩放，必要时叠加 bias，并重新应用与前向一致的因果 mask 逻辑。接着它利用保存下来的 logsumexp 在共享内存中恢复归一化概率，可选地构造与前向 RNG 顺序完全一致的 dropout mask tile，并为后续 `dV`、`dQ`、`dK` GEMM 准备所需的共享内存视图和 prologue lambda。可以说，反向内核正是在这里完成了“重建前向 softmax 状态”这一关键步骤，而不是从显存中读取完整概率矩阵。
```cpp

    /////////////////////////////////////////////////////////////////////////////////////////////////
    // GradV matmul
    //
    // grad_v[j_start:j_end] += attn_T @ do_i
    /////////////////////////////////////////////////////////////////////////////////////////////////
    constexpr bool kSingleIterationGradV =
        kMaxK <= MatmulGradV::ThreadblockShape::kN;
    for (int col = 0; col < (kSingleIterationGradV ? 1 : p.head_dim_value);
         col += MatmulGradV::ThreadblockShape::kN) {
      using Mma = typename MatmulGradV::Mma;
      using AccumTileGmem = typename MatmulGradQ::AccumTileGmem;

      cutlass::gemm::GemmCoord problem_size(
          num_keys_in_block, p.head_dim_value - col, num_queries_in_block);
      auto createEpilogueIter = [&]() {
        return typename MatmulGradV::OutputTileIterator(
            typename MatmulGradV::OutputTileIterator::Params{p.gV_strideM()},
            p.grad_value_ptr + key_start * p.gV_strideM() + col,
            {num_keys_in_block, p.head_dim_value - col},
            thread_id);
      };
      typename Mma::IteratorB iterator_B(
          {int32_t(p.gO_strideM)},
          p.grad_output_ptr + query_start * p.gO_strideM + col,
          {num_queries_in_block, p.head_dim_value - col},
          thread_id,
          no_offset);

      // if dropout: dVj += (Pij.T * Zij) @ dOi
      // otherwise:  dVj += Pij.T @ dOi
      Mma mma(
          // operand A: Pij.T
          shared_storage.attn_shared_storage().accum_ref(),
          // operand A_scale Zij.T:
          // if we're using dropout, operand A is Pij_dropped.T = Pij.T * Zij.T
          // which is computed on the fly as fragments of Pij.T are loaded in
          shared_storage.zij().accum_ref(),
          // operand B: dOi - which was loaded into shared memory previously
          // when we computed dVj
          shared_storage.mm_gradV().operand_B_ref(),
          thread_id,
          warp_id,
          lane_id);

      int storage_id = col / MatmulGradV::ThreadblockShape::kN;
      AccumTileGmem gmem_tile{
          p.workspace_gv + storage_id * AccumTileGmem::kElementsStored};
      if (!kOutputInRF) {
        if (isFirstQuery || !kNeedsAccumGradV) {
          output_frags.gradV.clear();
        } else {
          gmem_tile.load(output_frags.gradV, thread_id);
        }
      }
      mma.set_prologue_done(kPrologueGV);

      auto gemm_k_iterations =
          (problem_size.k() + Mma::Shape::kK - 1) / Mma::Shape::kK;

      // Compute threadblock-scoped matrix multiply-add
      __syncthreads();

      mma(gemm_k_iterations,
          output_frags.gradV,
          iterator_B,
          output_frags.gradV);
      __syncthreads();
      if (kPrologueGV && !kSingleIterationGradV &&
          col + MatmulGradV::ThreadblockShape::kN < p.head_dim_value) {
        prologueGradV(col + MatmulGradV::ThreadblockShape::kN);
      }

      if (!kOutputInRF) {
        if (kNeedsAccumGradV && !isLastQuery) {
          gmem_tile.store(output_frags.gradV, thread_id);
        } else {
          accumulateInGmem<MatmulGradV>(
              shared_storage.gradV_epilogue(),
              output_frags.gradV,
              createEpilogueIter(),
              isFirstQuery || kNeedsAccumGradV,
              warp_id,
              lane_id);
        }
      }
    }
    __syncthreads();

    /////////////////////////////////////////////////////////////////////////////////////////////////
    // MatmulDOIVJ
    /////////////////////////////////////////////////////////////////////////////////////////////////
    {
      using Mma = typename MatmulDOIVJ::Mma;
      // do_i
      typename Mma::IteratorA iterator_A(
          {int32_t(p.gO_strideM)},
          p.grad_output_ptr + query_start * p.gO_strideM,
          {num_queries_in_block, p.head_dim_value},
          thread_id,
          no_offset);

      // v_j.transpose(-2, -1)
      typename Mma::IteratorB iterator_B(
          {int32_t(p.v_strideM)},
          p.value_ptr + key_start * p.v_strideM,
          {p.head_dim_value, num_keys_in_block},
          thread_id,
          no_offset);

      Mma mma(shared_storage.mm_doivj(), thread_id, warp_id, lane_id);
      mma.set_prologue_done(kPrologueDOV);
      mma.set_zero_outside_bounds(!skipBoundsChecks);

      typename Mma::FragmentC accum;

      accum.clear();

      auto gemm_k_iterations =
          (p.head_dim_value + Mma::Shape::kK - 1) / Mma::Shape::kK;

      // Compute threadblock-scoped matrix multiply-add
      mma(gemm_k_iterations, accum, iterator_A, iterator_B, accum);
      __syncthreads();
      if (kPrologueGQ) {
        prologueGradQ(0);
      }
      if (kPrologueGK) {
        prologueGradK(0);
      }

      int warp_idx_mn_0 =
          warp_id % (Mma::Base::WarpCount::kM * Mma::Base::WarpCount::kN);
      auto output_tile_coords = cutlass::MatrixCoord{
          warp_idx_mn_0 % Mma::Base::WarpCount::kM,
          warp_idx_mn_0 / Mma::Base::WarpCount::kM};
      // TODO: This must be terribly inefficient. There must be a better way
      // tmp [RF] <- (accum [RF] - Di [smem] ) * attn_T.T [smem]
      // attn_shared_storage  [smem] <- tmp.T
      // tmp_shared_storage [smem] <- tmp
      {
        using LambdaIterator = typename MatmulDOIVJ::AccumLambdaIterator;
        auto lane_offset = LambdaIterator::get_lane_offset(
            lane_id, warp_id, output_tile_coords);
        // if dropout was used, compute dPij = dPij_dropped * Zij
        if (kApplyDropout) {
          LambdaIterator::iterateRows(
              lane_offset,
              [&](int accum_m) {},
              [&](int accum_m, int accum_n, int idx) {
                if (dropout_keep_mask_doivj[idx].get()) {
                  accum[idx] *= dropout_scale;
                } else {
                  accum[idx] = 0;
                }
              },
              [&](int accum_m) {});
        }

        auto attn_T = shared_storage.attn_shared_storage().accum_ref();
#if 0
        PRINT_B0_T0("doivj_dropped");
        print_warp_accum<LambdaIterator>(accum, lane_offset, 4, 4);
        PRINT_TENSOR4x4_T0_L0("attn_T", attn_T)
#endif
        accum_t current_di;
        // dSij = (dPij - Di) * Pij
        LambdaIterator::iterateRows(
            lane_offset,
            [&](int accum_m) { current_di = shared_storage.di()[accum_m]; },
            [&](int accum_m, int accum_n, int idx) {
              // TODO: Otherwise we can get nans as we
              // might have infs here (only seen on f16 tho)
              if (skipBoundsChecks ||
                  (accum_m < num_queries_in_block &&
                   accum_n < num_keys_in_block)) {
                accum_t attn = attn_T.at({accum_n, accum_m});
                accum[idx] = (accum[idx] - current_di) * attn;
              } else {
                accum[idx] = 0;
              }
            },
            [&](int accum_m) {

            });

        // store bias gradient tile dBij to global memory,
        // where dBij = dSij = Pij * (dPij - Di)
        if (p.grad_bias_ptr != nullptr) {
          typename MatmulDOIVJ::BiasGradEpilogue::OutputTileIterator
              output_iter(
                  typename MatmulDOIVJ::BiasGradEpilogue::OutputTileIterator::
                      Params{p.gB_strideM},
                  // grad_bias_ptr is offset to point at beginning of
                  // matrix of shape (queries, keys) for a given
                  // (batch_id, head_id) the pointer arithmetic here produces
                  // a pointer to the start of the current tile within that
                  // matrix
                  p.grad_bias_ptr + query_start * p.gB_strideM + key_start,
                  {num_queries_in_block, num_keys_in_block},
                  thread_id);

          // no-op epilogue operator - just casting and storing contents of
          // accum to global memory
          typename MatmulDOIVJ::BiasGradEpilogue::OutputOp output_op(
              typename MatmulDOIVJ::BiasGradEpilogue::OutputOp::Params{1, 1});
          typename MatmulDOIVJ::BiasGradEpilogue epilogue(
              shared_storage.gradB_epilogue(), thread_id, warp_id, lane_id);
          epilogue(output_op, output_iter, accum, output_iter);
        }

        accum = accum * scale;

#if 0
        PRINT_B0_T0("(doivj - di) * attn * scale");
        print_warp_accum<LambdaIterator>(accum, lane_offset, 4, 4);
#endif

        __syncthreads();
        if (!MatmulGradK::DefaultMmaFromSmem::kIsTransposedA) {
          auto tmpT = shared_storage.tmpT_shared_storage().accum_ref();
          // attn <- attn_T.T
          LambdaIterator::iterateRows(
              lane_offset,
              [&](int accum_m) {},
              [&](int accum_m, int accum_n, int idx) {
                tmpT.at({accum_n, accum_m}) = scalar_t(accum[idx]);
              },
              [&](int accum_m) {});
        }
      }

      MatmulDOIVJ::B2bGemm::accumToSmem(
          shared_storage.tmp_shared_storage(),
          accum,
          lane_id,
          output_tile_coords);
      __syncthreads();
    }
    // Force `nvcc` to recompute values that depend on the variables just below
    // to use less RF and prevent some spilling
    p.head_dim = warp_uniform(p.head_dim);
    p.k_strideM = warp_uniform(p.k_strideM);
    rematerializeThreadIds();

    /////////////////////////////////////////////////////////////////////////////////////////////////
    // GradQ matmul
    //
    // grad_q[i_start:i_end] += tmp @ k_j
    /////////////////////////////////////////////////////////////////////////////////////////////////
    // Skip the loop & associated branches if we know at compile time the number
    // of iterations
    constexpr bool kSingleIterationGradQ =
        kMaxK <= MatmulGradQ::ThreadblockShape::kN;
    for (int col = 0; col < (kSingleIterationGradQ ? 1 : p.head_dim);
         col += MatmulGradQ::ThreadblockShape::kN) {
      using Mma = typename MatmulGradQ::Mma;
      using AccumTileGmem = typename MatmulGradQ::AccumTileGmem;

      cutlass::gemm::GemmCoord problem_size(
          num_queries_in_block,
          false ? MatmulGradQ::ThreadblockShape::kN : p.head_dim - col,
          num_keys_in_block);

      // k_j
      typename Mma::IteratorB iterator_B(
          {int32_t(p.k_strideM)},
          p.key_ptr + key_start * p.k_strideM + col,
          {problem_size.k(), problem_size.n()},
          thread_id,
          no_offset);

      auto a = shared_storage.tmp_shared_storage().accum_ref();
      Mma mma(
          // operand A: dSij
          shared_storage.tmp_shared_storage().accum_ref(),
          // operand B: Kj
          shared_storage.mm_gradQ().operand_B_ref(),
          thread_id,
          warp_id,
          lane_id);

      typename Mma::FragmentC accum;

      int col_id = col / MatmulGradQ::ThreadblockShape::kN;
      int num_cols = kSingleIterationGradQ
          ? 1
          : ceil_div(p.head_dim, MatmulGradQ::ThreadblockShape::kN);
      int storage_id = (col_id + query_start / kBlockSizeI * num_cols);

      if (p.num_splits_key_device() > 1) {
        AtomicLock::acquire(
            &p.workspace_gq[storage_id].lock,
            p.split_key_device() + 1,
            thread_id);
        // Make sure we can see other block's output
        __threadfence();
      }

      AccumTileGmem gmem_tile{&p.workspace_gq[storage_id].buffer[0]};
      if (!kNeedsAccumGradQ ||
          (p.num_splits_key_device() == 1 && key_start == 0)) {
        // if we know we are the first to access it, we know it's only zeros.
        // Avoids a load from gmem (and gmem init as well)
        accum.clear();
      } else {
        gmem_tile.load(accum, thread_id);
      }

      auto gemm_k_iterations =
          (problem_size.k() + Mma::Shape::kK - 1) / Mma::Shape::kK;

      // Compute threadblock-scoped matrix multiply-add
      __syncthreads();
      mma.set_prologue_done(kPrologueGQ);
      mma(gemm_k_iterations, accum, iterator_B, accum);
      __syncthreads();
      bool isLastColumn = kSingleIterationGradQ ||
          (col + MatmulGradQ::ThreadblockShape::kN >= p.head_dim);
      if (kPrologueGQ && !isLastColumn) {
        prologueGradQ(col + MatmulGradQ::ThreadblockShape::kN);
      }

      bool isLast = [&]() {
        int32_t next_key = key_start + p.num_splits_key_device() * kBlockSizeJ;
        if (p.num_keys <= next_key) {
          return true;
        }
        if (query_start < getSmallestQueryForKey(p, next_key)) {
          return true;
        }
        return false;
      }();
      // Output results
      if (p.num_splits_key_device() > 1) {
        int32_t numAddsSoFar = -1;
        if (isLast && thread_id == 0) {
          numAddsSoFar = atomicAdd(&p.workspace_gq[storage_id].counter, 1) +
              1; // `atomicAdd` returns the old value
        }
        isLast = __syncthreads_or(
            numAddsSoFar == getNumParallelBlocksForQuery(p, query_start));
        assert(numAddsSoFar <= getNumParallelBlocksForQuery(p, query_start));
      }
      if (kNeedsAccumGradQ && !isLast) {
        gmem_tile.store(accum, thread_id);
        if (p.num_splits_key_device() > 1) {
          // Make sure everyone wrote before we release the lock
          __threadfence();
          __syncthreads();
          AtomicLock::release(&p.workspace_gq[storage_id].lock, thread_id);
        }
      } else {
        // NOTE: We're not releasing the lock because no one is expected
        // to come after us (we're the last one to write)
        typename MatmulGradQ::OutputTileIterator output_it(
            typename MatmulGradQ::OutputTileIterator::Params{p.gQ_strideM()},
            p.grad_query_ptr + query_start * p.gQ_strideM() + col,
            {problem_size.m(), problem_size.n()},
            thread_id);
        bool storage_contains_zeros = kNeedsAccumGradQ || key_start == 0 ||
            (p.num_splits_key_device() > 1);
        accumulateInGmem<MatmulGradQ>(
            isLastColumn ? shared_storage.gradQ_epilogue_lastIter()
                         : shared_storage.gradQ_epilogue(),
            accum,
            output_it,
            storage_contains_zeros,
            warp_id,
            lane_id);
      }
    }
    /////////////////////////////////////////////////////////////////////////////////////////////////
    // GradK matmul
    //
    // grad_k[i_start:i_end] += tmp.transpose(-2, -1) @ q_i
    /////////////////////////////////////////////////////////////////////////////////////////////////
    rematerializeThreadIds();

    constexpr bool kSingleIterationGradK =
        kMaxK <= MatmulGradK::ThreadblockShape::kN;
    for (int col = 0; col < (kSingleIterationGradK ? 1 : p.head_dim);
         col += MatmulGradK::ThreadblockShape::kN) {
      using Mma = typename MatmulGradK::Mma;
      using AccumTileGmem = typename MatmulGradQ::AccumTileGmem;

      cutlass::gemm::GemmCoord problem_size(
          num_keys_in_block,
          false ? MatmulGradK::ThreadblockShape::kN : p.head_dim - col,
          num_queries_in_block);
      auto createEpilogueIter = [&]() {
        return typename MatmulGradK::OutputTileIterator(
            typename MatmulGradK::OutputTileIterator::Params{p.gK_strideM()},
            p.grad_key_ptr + key_start * p.gK_strideM() + col,
            {num_keys_in_block,
             false ? MatmulGradK::ThreadblockShape::kN : p.head_dim - col},
            thread_id);
      };

      // q_i
      typename Mma::IteratorB iterator_B(
          {int32_t(p.q_strideM)},
          p.query_ptr + query_start * p.q_strideM + col,
          {problem_size.k(), problem_size.n()},
          thread_id,
          no_offset);

      auto getTmp = [&](int) { return &shared_storage.tmp_shared_storage(); };
      auto getTmpT = [&](int) { return &shared_storage.tmpT_shared_storage(); };
      // this is basically:
      // opA = kIsTransposedA ? getTmp() : getTmpT();
      bool constexpr kIsTransposedA =
          MatmulGradK::DefaultMmaFromSmem::kIsTransposedA;
      auto& opA = *call_conditional<
          kIsTransposedA,
          decltype(getTmp),
          decltype(getTmpT)>::apply(getTmp, getTmpT, 0);
      Mma mma(
          // operand A: dSij.T
          opA.accum_ref(),
          // operand B: Qi
          shared_storage.mm_gradK().operand_B_ref(),
          thread_id,
          warp_id,
          lane_id);

      int storage_id = col / MatmulGradK::ThreadblockShape::kN;
      AccumTileGmem gmem_tile{
          p.workspace + storage_id * AccumTileGmem::kElementsStored};
      if (!kOutputInRF) {
        if (isFirstQuery || !kNeedsAccumGradK) {
          output_frags.gradK.clear();
        } else {
          gmem_tile.load(output_frags.gradK, thread_id);
        }
      }
      mma.set_prologue_done(kPrologueGK);

      auto gemm_k_iterations =
          (problem_size.k() + Mma::Shape::kK - 1) / Mma::Shape::kK;

      // Compute threadblock-scoped matrix multiply-add
      __syncthreads();

      mma(gemm_k_iterations,
          output_frags.gradK,
          iterator_B,
          output_frags.gradK);
      __syncthreads();
      bool isLastColumn = kSingleIterationGradK ||
          col + MatmulGradK::ThreadblockShape::kN >= p.head_dim;
      if (kPrologueGK && !isLastColumn) {
        prologueGradK(col + MatmulGradK::ThreadblockShape::kN);
      }

      if (kPrologueQK && isLastColumn) {
        int32_t next_query, next_key;
        incrIteration(p, query_start, key_start, next_query, next_key);
        DISPATCH_BOOL(
            next_key != key_start, kForceReloadK, ([&]() {
              prologueQkNextIteration<kForceReloadK::value>(
                  shared_storage, p, next_query, next_key, warp_id, lane_id);
            }));
      }

      // Output results
      if (!kOutputInRF) {
        if (kNeedsAccumGradK && !isLastQuery) {
          gmem_tile.store(output_frags.gradK, thread_id);
        } else {
          accumulateInGmem<MatmulGradK>(
              isLastColumn ? shared_storage.gradK_epilogue_final()
                           : shared_storage.gradK_epilogue(),
              output_frags.gradK,
              createEpilogueIter(),
              isFirstQuery || kNeedsAccumGradK,
              warp_id,
              lane_id);
          __syncthreads();
        }
      }
    }
  }
```
**EN**: Lines 1754-2237. This large block executes the actual gradient-producing matmuls. First, `MatmulGradV` consumes the reconstructed probability tile (or dropout-masked tile) and `dO` to accumulate `dV`. Next, `MatmulDOIVJ` computes `dO @ V^T`; its output is corrected by subtracting `delta`, multiplying by the probability tile, and optionally writing `dB`, thereby producing the score gradient `dS`. That `dS` tile is then fed into `MatmulGradQ` and `MatmulGradK` to produce `dQ` and `dK`. The code carefully manages synchronization between phases, shared-memory transposition/reinterpretation, and the distinction between direct final writes and workspace accumulation. In split-key mode, `dQ` uses locks and completion counters so multiple blocks can safely merge partial results into the same destination tile.
**CN**: 第1754-2237行：这一大段代码执行真正产生梯度的 GEMM 流水线。首先，`MatmulGradV` 消费重建出的概率块（或经过 dropout mask 的概率块）以及 `dO`，累加得到 `dV`。随后 `MatmulDOIVJ` 计算 `dO @ V^T`；其输出再减去 `delta`、乘上概率块，并可选地写出 `dB`，从而形成得分梯度 `dS`。接着，这个 `dS` tile 会被送入 `MatmulGradQ` 与 `MatmulGradK` 以生成 `dQ` 和 `dK`。整个过程非常强调阶段间同步、共享内存中的转置/重解释布局，以及“直接最终写回”和“先写工作区再规约”两种路径的切换。在 split-key 模式下，`dQ` 还通过锁和完成计数器保证多个线程块能够安全地把部分结果合并到同一目标 tile。
```cpp
  static CUTLASS_DEVICE int32_t getQueryStartShift(Params const& p) {
    if (p.custom_mask_type == NoCustomMask && p.num_splits_key_device() > 1) {
      return (p.split_key_device() * kBlockSizeI) % getQueryEnd(p);
    }
    return 0;
  }

  // Iteration order logic
  static CUTLASS_DEVICE int32_t
  getQueryStart(Params const& p, int32_t key_start) {
    return getSmallestQueryForKey(p, key_start) + getQueryStartShift(p);
  };
  static CUTLASS_DEVICE int32_t getQueryEnd(Params const& p) {
    return align_up(p.num_queries, kBlockSizeI);
  };

  static CUTLASS_DEVICE int32_t
  getSmallestQueryForKey(Params const& p, int32_t key_start) {
    if (p.custom_mask_type == CausalFromTopLeft) {
      return (key_start / kBlockSizeI) * kBlockSizeI;
    } else if (p.custom_mask_type == CausalFromBottomRight) {
      int first_query =
          cutlass::fast_max(0, key_start - p.num_keys + p.num_queries);
      return (first_query / kBlockSizeI) * kBlockSizeI;
    }
    return 0;
  };

  // Returns how many kernel blocks will write to a given block in `grad_query`
  // This is usually equal to the number of key splits, but can be different
  // for instance in the causal case, or varying seqlen
  static CUTLASS_DEVICE int32_t
  getNumParallelBlocksForQuery(Params const& p, int32_t query_start) {
    int16_t num_key_blocks = ceil_div(p.num_keys, kBlockSizeJ);
    if (p.custom_mask_type == CausalFromTopLeft) {
      int32_t last_key_for_block = query_start + kBlockSizeI - 1;
      last_key_for_block = cutlass::fast_min(last_key_for_block, p.num_keys);
      num_key_blocks = ceil_div(last_key_for_block, kBlockSizeJ);
    } else if (p.custom_mask_type == CausalFromBottomRight) {
      int32_t last_key_for_block =
          query_start + (kBlockSizeI - 1) + (1 + p.num_keys - p.num_queries);
      last_key_for_block = cutlass::fast_min(last_key_for_block, p.num_keys);
      num_key_blocks = ceil_div(last_key_for_block, kBlockSizeJ);
    }
    return cutlass::fast_min(p.num_splits_key_device(), num_key_blocks);
  };

  // Returns the next block to process
  static CUTLASS_DEVICE void incrIteration(
      Params const& p,
      int32_t query_start,
      int32_t key_start,
      int32_t& next_query,
      int32_t& next_key) {
    next_query = query_start + kBlockSizeI;
    next_key = key_start;
    auto query_shift = getQueryStartShift(p);
    // Wrap around
    if (query_shift) {
      if (next_query >= p.num_queries) {
        next_query = getSmallestQueryForKey(p, key_start);
        return;
      } else if (query_start < query_shift && query_shift <= next_query) {
        // jump to next key
      } else {
        return;
      }
    } else {
      if (next_query < p.num_queries) {
        return;
      }
      // jump to next key
    }
    // Next key
    next_key = key_start + p.num_splits_key_device() * kBlockSizeJ;
    next_query = getQueryStart(p, next_key);
  }

  template <bool kForceReloadK>
  static CUTLASS_DEVICE void prologueQkNextIteration(
      SharedStorage& shared_storage,
      Params const& p,
      int32_t query_start,
      int32_t key_start,
      uint8_t warp_id,
      uint8_t lane_id) {
    if (query_start >= p.num_queries || key_start >= p.num_keys) {
      return;
    }

    static constexpr bool kReloadK =
        kForceReloadK || !MatmulQK::Mma::kSmemContainsEntireMat;
    int thread_id = 32 * warp_id + lane_id;
    typename MatmulQK::Mma::IteratorA iterator_A(
        {int32_t(p.k_strideM)},
        p.key_ptr + key_start * p.k_strideM,
        {p.num_keys - key_start, p.head_dim},
        thread_id,
        cutlass::MatrixCoord{0, 0});

    typename MatmulQK::Mma::IteratorB iterator_B(
        {int32_t(p.q_strideM)},
        p.query_ptr + query_start * p.q_strideM,
        {p.head_dim, p.num_queries - query_start},
        thread_id,
        cutlass::MatrixCoord{0, 0});

    MatmulQK::Mma::template prologue<kReloadK, true>(
        shared_storage.mm_qk_k(),
        shared_storage.mm_qk_q(),
        iterator_A,
        iterator_B,
        thread_id,
        p.head_dim);
  }

  template <bool skipBoundsChecks>
  static CUTLASS_DEVICE void writeFragsToGmem(
      SharedStorage& shared_storage,
      OutputFragments& output_frags,
      Params const& p,
      int32_t key_start,
      uint8_t warp_id,
      uint8_t lane_id) {
    uint16_t thread_id = 32 * warp_id + lane_id;
    int32_t num_keys_in_block = skipBoundsChecks
        ? MatmulQK::Mma::Shape::kM
        : cutlass::fast_min(
              (int32_t)MatmulQK::Mma::Shape::kM, p.num_keys - key_start);
    typename MatmulGradV::OutputTileIterator outputV_it(
        typename MatmulGradV::OutputTileIterator::Params{p.gV_strideM()},
        p.grad_value_ptr + key_start * p.gV_strideM(),
        {num_keys_in_block, p.head_dim_value},
        thread_id);

    accumulateInGmem<MatmulGradV>(
        shared_storage.gradV_epilogue_final(),
        output_frags.gradV,
        outputV_it,
        true,
        warp_id,
        lane_id);

    typename MatmulGradK::OutputTileIterator outputK_it(
        typename MatmulGradK::OutputTileIterator::Params{p.gK_strideM()},
        p.grad_key_ptr + key_start * p.gK_strideM(),
        {num_keys_in_block,
         false ? MatmulGradK::ThreadblockShape::kN : p.head_dim},
        thread_id);
    accumulateInGmem<MatmulGradK>(
        shared_storage.gradK_epilogue_final(),
        output_frags.gradK,
        outputK_it,
        true,
        warp_id,
        lane_id);
  }

  template <typename MatmulT>
  static CUTLASS_DEVICE void accumulateInGmem(
      typename MatmulT::DefaultEpilogue::SharedStorage& epilogue_smem,
      typename MatmulT::Mma::FragmentC const& accum,
      typename MatmulT::OutputTileIterator output_it,
      bool first,
      uint8_t warp_id,
      uint8_t lane_id) {
    using DefaultEpilogue = typename MatmulT::DefaultEpilogue;
    using DefaultOutputOp = typename MatmulT::DefaultOutputOp;
    using Mma = typename MatmulT::Mma;
    int thread_id = 32 * warp_id + lane_id;
    DISPATCH_BOOL(
        first, kIsFirst, ([&]() {
          static constexpr auto ScaleType = kIsFirst::value
              ? cutlass::epilogue::thread::ScaleType::Nothing
              : cutlass::epilogue::thread::ScaleType::NoBetaScaling;
          using EpilogueOutputOp =
              typename cutlass::epilogue::thread::LinearCombination<
                  typename DefaultOutputOp::ElementOutput,
                  DefaultOutputOp::kCount,
                  typename DefaultOutputOp::ElementAccumulator,
                  typename DefaultOutputOp::ElementCompute,
                  ScaleType>;
          using Epilogue =
              typename cutlass::epilogue::threadblock::EpiloguePipelined<
                  typename DefaultEpilogue::Shape,
                  typename Mma::Operator,
                  DefaultEpilogue::kPartitionsK,
                  typename MatmulT::OutputTileIterator,
                  typename DefaultEpilogue::AccumulatorFragmentIterator,
                  typename DefaultEpilogue::WarpTileIterator,
                  typename DefaultEpilogue::SharedLoadIterator,
                  EpilogueOutputOp,
                  typename DefaultEpilogue::Padding,
                  DefaultEpilogue::kFragmentsPerIteration,
                  true // IterationsUnroll
                  >;
          EpilogueOutputOp rescale({1, 1});
          Epilogue epilogue(epilogue_smem, thread_id, warp_id, lane_id);
          epilogue(rescale, output_it, accum, output_it);
        }));
  }

  template <int kElementsPerAccess>
  static CUTLASS_DEVICE void computeDelta(
      Params const& p,
      int32_t query_start,
      uint8_t warp_id,
      uint8_t lane_id) {
    // Each thread computes one value for Delta
    // Depending on warp configuration, we might have multiple
    // threads of the same warp working on the same row
    using AccessType = cutlass::Array<scalar_t, kElementsPerAccess>;
    static_assert(kNumThreads >= kBlockSizeI, "");
    static constexpr int kNumThreadsPerLine = kNumThreads / kBlockSizeI;
    int16_t thread_id = 32 * warp_id + lane_id;

    int16_t laneFirstCol = kElementsPerAccess * (lane_id % kNumThreadsPerLine);
    int16_t laneRow = thread_id / kNumThreadsPerLine;
    bool rowPred = (query_start + laneRow) < p.num_queries;
    bool pred = rowPred;

    // on windows, previous syntax __restrict__ AccessType*
    // resulted in error: "restrict" is not allowed
    const AccessType* __restrict__ grad_output_ptr =
        reinterpret_cast<const AccessType*>(
            p.grad_output_ptr + (query_start + laneRow) * p.gO_strideM +
            laneFirstCol);
    const AccessType* __restrict__ output_ptr =
        reinterpret_cast<const AccessType*>(
            p.output_ptr + (query_start + laneRow) * p.o_strideM() +
            laneFirstCol);

    static constexpr int64_t kMaxIters =
        kMaxK / (kElementsPerAccess * kNumThreadsPerLine);
    constexpr int kPipelineStages = 2;
    accum_t delta_value = accum_t(0);
    using GlobalLoad =
        cutlass::arch::global_load<AccessType, sizeof(AccessType)>;
    AccessType frag_grad_output[kPipelineStages];
    AccessType frag_output[kPipelineStages];

    auto loadAndIncrement = [&](int ld_pos, bool is_valid) {
      frag_grad_output[ld_pos].clear();
      frag_output[ld_pos].clear();
      GlobalLoad(frag_grad_output[ld_pos], grad_output_ptr, is_valid);
      GlobalLoad(frag_output[ld_pos], output_ptr, is_valid);
      grad_output_ptr += kNumThreadsPerLine;
      output_ptr += kNumThreadsPerLine;
    };

    CUTLASS_PRAGMA_UNROLL
    for (int iter = 0; iter < kPipelineStages - 1; ++iter) {
      int ld_pos = iter % kPipelineStages;
      pred = pred &&
          (laneFirstCol + iter * kElementsPerAccess * kNumThreadsPerLine) <
              p.head_dim_value;
      loadAndIncrement(ld_pos, pred);
    }
    auto columnIteration = [&](int iter) {
      // Load for next iter
      int ld_pos = (iter + kPipelineStages - 1) % kPipelineStages;
      pred = pred &&
          (laneFirstCol +
           (iter + kPipelineStages - 1) * kElementsPerAccess *
               kNumThreadsPerLine) < p.head_dim_value;
      loadAndIncrement(ld_pos, pred);
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < AccessType::kElements; ++i) {
        delta_value += accum_t(frag_output[iter % kPipelineStages][i]) *
            accum_t(frag_grad_output[iter % kPipelineStages][i]);
      }
    };

    // If we have a small lower-bound for K, we can unroll the loop
    if (kMaxK <= 256) {
      CUTLASS_PRAGMA_UNROLL
      for (int iter = 0; iter < kMaxIters; ++iter) {
        columnIteration(iter);
      }
    } else {
      int num_iters =
          ceil_div(p.head_dim_value, kElementsPerAccess * kNumThreadsPerLine) *
          (kElementsPerAccess * kNumThreadsPerLine);
      for (int iter = 0; iter < num_iters; ++iter) {
        columnIteration(iter);
      }
    }

    // Reduce between workers
    static_assert(
        kNumThreadsPerLine == 1 || kNumThreadsPerLine == 2 ||
            kNumThreadsPerLine == 4,
        "");
    CUTLASS_PRAGMA_UNROLL
    for (int i = 1; i < kNumThreadsPerLine; i *= 2) {
      delta_value = delta_value + __shfl_xor_sync(0xffffffff, delta_value, i);
    }

    // Store in gmem
    if (rowPred) {
      p.delta_ptr[query_start + laneRow] = delta_value;
    }
  }
};

template <typename AK>
__global__ void __launch_bounds__(AK::kNumThreads, AK::kMinBlocksPerSm)
    attention_kernel_backward_batched_impl(typename AK::Params p) {
  if (!p.advance_to_block()) {
    return;
  }
  AK::attention_kernel(p);
}

template <typename AK>
__global__ void __launch_bounds__(AK::kNumThreads, AK::kMinBlocksPerSm)
    attention_kernel_backward_batched(typename AK::Params params);
```
**EN**: Lines 2238-2554. The closing section contains the traversal logic and global-write helpers that tie the kernel together. Functions such as `getQueryStartShift()`, `getSmallestQueryForKey()`, and `incrIteration()` determine the order in which query/key tiles are visited, including the wraparound behavior required by split-key and causal layouts. `prologueQkNextIteration()` preloads the next Q/K tile, `writeFragsToGmem()` and `accumulateInGmem()` flush register fragments through CUTLASS epilogues, and `computeDelta()` implements the pipelined dot product reduction that forms the per-query delta term. The file ends with the two CUDA global entry points that perform `advance_to_block()` and instantiate the launch-bounded kernel wrapper.
**CN**: 第2238-2554行：收尾部分包含把整个内核串起来的遍历逻辑与全局写回辅助函数。`getQueryStartShift()`、`getSmallestQueryForKey()`、`incrIteration()` 等函数决定 query/key tile 的访问顺序，包括 split-key 与因果布局所需的回绕行为。`prologueQkNextIteration()` 负责预取下一块 Q/K，`writeFragsToGmem()` 与 `accumulateInGmem()` 则通过 CUTLASS epilogue 把寄存器片段刷新到全局内存；`computeDelta()` 实现了形成每个 query 的 delta 项所需的流水化点积规约。文件最后以两个 CUDA 全局入口结束：它们负责执行 `advance_to_block()` 并实例化带 launch bounds 的内核包装器。
---
## Key Concepts / 关键概念
- The backward kernel is a fused multi-stage pipeline: reconstruct forward scores/probabilities, compute `dV`, form `dS`, then accumulate `dQ` and `dK`. / 反向内核是一个融合的多阶段流水线：先重建前向得分/概率，再计算 `dV`，形成 `dS`，最后累加 `dQ` 与 `dK`。
- Template policy is even more important here than in forward because it controls prologue preloading, shared-memory layout variants, split-key behavior, and whether intermediate gradients live in registers, shared memory, or global workspace. / 这里的模板策略比前向更关键，因为它同时控制 prologue 预取、共享内存布局变体、split-key 行为，以及中间梯度究竟驻留在寄存器、共享内存还是全局工作区。
- Synchronization and layout conversion are central: each stage hands off fragments through carefully chosen shared-memory layouts so the next GEMM can consume them without materializing large intermediate tensors. / 同步与布局转换是本文件的核心：每个阶段都通过精心设计的共享内存布局交接片段，使后续 GEMM 无需物化大型中间张量就能直接消费这些结果。
- Split-key support adds extra machinery—locks, counters, workspace slices, and final epilogues—but allows more parallelism when the key dimension is large or sequence shapes are irregular. / split-key 支持引入了额外机制——锁、计数器、工作区切片和最终 epilogue——但在 key 维较大或序列形状不规则时，能显著提高并行度。
## Dependencies / 依赖项
- `gemm/custom_mma.h` — Redirects CUTLASS-selected threadblock MMAs to FMHA-aware custom implementations / 将 CUTLASS 选出的 threadblock MMA 重定向到 FMHA 感知的自定义实现
- `gemm/find_default_mma.h` — Overrides default MMA selection so backward kernels can tune stage counts and FastF32 behavior / 覆写默认 MMA 选择，使反向内核能调节 stage 数和 FastF32 行为
- `gemm/mma_from_smem.h` — Builds the shared-memory-fed MMA path used after attention tiles are materialized on chip / 构建注意力 tile 片上物化后使用的共享内存输入 MMA 路径
- `epilogue/epilogue_pipelined.h, iterators/epilogue_predicated_tile_iterator.h` — Handle accumulator layout conversion and bounded stores for dQ/dK/dV style outputs / 处理 dQ/dK/dV 等输出所需的累加器布局转换与边界写回
- `transform/tile_smem_loader.h` — Moves staged tiles between shared memory layouts required by different backward subproblems / 在不同反向子问题所需的共享内存布局之间搬运 tile
- `debug_utils.h, gemm_kernel_utils.h` — Provide launch, layout, and debugging helpers shared across FMHA kernels / 提供 FMHA 内核共享的启动、布局与调试辅助工具
