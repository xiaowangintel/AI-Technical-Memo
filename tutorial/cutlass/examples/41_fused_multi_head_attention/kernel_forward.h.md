# kernel_forward.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/kernel_forward.h`
**Purpose / 用途**: Core fused forward FMHA kernel that performs QK scoring, online softmax, masking/dropout, and the attention·V accumulation in one blockwise pipeline / 融合前向 FMHA 的核心内核，在同一分块流水中完成 QK 打分、在线 softmax、mask/dropout 和 attention·V 累加
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

#ifdef HAS_PYTORCH
#include <ATen/cuda/CUDAGeneratorImpl.h>
#include <ATen/cuda/CUDAGraphsUtils.cuh>
#endif

#include <curand_kernel.h>
#include <cmath>
#include <cinttypes>
#include <vector>

#include "cutlass/fast_math.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/layout/vector.h"
#include "cutlass/matrix.h"
#include "cutlass/numeric_types.h"
#include "cutlass/tensor_ref.h"

#include "cutlass/epilogue/threadblock/default_epilogue_simt.h"
#include "cutlass/epilogue/threadblock/default_epilogue_tensor_op.h"
#include "cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h"
#include "cutlass/gemm/device/default_gemm_configuration.h"
#include "cutlass/gemm/kernel/default_gemm.h"
#include "cutlass/gemm/threadblock/default_mma.h"
#include "cutlass/gemm/threadblock/default_mma_core_simt.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm70.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm75.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
#include "cutlass/gemm/threadblock/threadblock_swizzle.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/platform/platform.h"
#include "cutlass/transform/threadblock/predicated_tile_iterator.h"
#include "debug_utils.h"
#include "epilogue/epilogue_pipelined.h"
#include "epilogue/epilogue_rescale_output.h"
#include "gemm/custom_mma.h"
#include "gemm/find_default_mma.h"
#include "gemm/mma_from_smem.h"
#include "gemm_kernel_utils.h"
#include "transform/tile_smem_loader.h"

using namespace gemm_kernel_utils;

namespace {
template <typename scalar_t, typename Arch>
constexpr int getWarpsPerSmFw() {
  return (
      Arch::kMinComputeCapability >= 80 &&
              !cutlass::platform::is_same<scalar_t, float>::value
          ? 16
          : 12);
}
static CUTLASS_DEVICE float atomicMaxFloat(float* addr, float value) {
  // source: https://stackoverflow.com/a/51549250
  return (value >= 0)
      ? __int_as_float(atomicMax((int*)addr, __float_as_int(value)))
      : __uint_as_float(atomicMin((unsigned int*)addr, __float_as_uint(value)));
}
} // namespace
```
**EN**: Lines 1-92. This opening block pulls in CUDA, CUTLASS GEMM/epilogue/layout infrastructure, and local helper headers used by the fused attention implementation. The anonymous-namespace helpers are small but important: `getWarpsPerSmFw()` sets the occupancy target based on architecture and scalar type, while `atomicMaxFloat()` implements a float max reduction primitive used later when multiple lanes cooperate to update the online softmax row maximum.
**CN**: 第1-92行：开头部分引入了 CUDA、CUTLASS 的 GEMM/epilogue/layout 基础设施，以及融合注意力实现依赖的本地辅助头文件。匿名命名空间中的两个辅助函数虽小却关键：`getWarpsPerSmFw()` 根据架构和标量类型给出每个 SM 的 warp 预算；`atomicMaxFloat()` 提供浮点最大值原语，后续在线 softmax 在多 lane 协作更新行最大值时会用到它。
```cpp

// If ToBatchHookType_ is supplied other than this default (which is
// never the case in the xformers library) then the user is
// defining the logic which each block uses to find its data to work on,
// with the advance_to_batch function with the following signature.
// It should return false if there is no work to do for this block.
// In general this will not work with saving for backward due to fixed layout
// for logsumexp and incompatible rngs for dropout, so is likely only useful for
// custom inference.
struct DefaultToBatchHook {
  template <typename Params>
  CUTLASS_DEVICE static bool advance_to_batch(
      Params&,
      int64_t& /* q_start */,
      int64_t& /* k_start */) {
    return true;
  }
};
```
**EN**: Lines 93-111. `DefaultToBatchHook` is a lightweight extension point that lets a caller override how a thread block maps itself onto batch/query/key ranges. The default implementation does nothing and always returns `true`, but the surrounding comments explain why this is intentionally conservative: remapping blocks can invalidate the expected logsumexp layout and the deterministic dropout RNG sequence that backward relies on.
**CN**: 第93-111行：`DefaultToBatchHook` 是一个轻量级扩展点，允许调用方自定义线程块如何映射到 batch/query/key 范围。默认实现什么都不做并始终返回 `true`，但注释明确说明它之所以保守，是因为任意重映射可能破坏反向传播依赖的 logsumexp 布局以及 dropout RNG 序列一致性。
```cpp
template <
    // The datatype of Q/K/V
    typename scalar_t_,
    // Architecture we are targeting (eg `cutlass::arch::Sm80`)
    typename ArchTag,
    // If Q/K/V are correctly aligned in memory and we can run a fast kernel
    bool isAligned_,
    int kQueriesPerBlock_,
    int kKeysPerBlock_,
    // upperbound on `max(value.shape[-1], query.shape[-1])`
    int kMaxK_ = (int)cutlass::platform::numeric_limits<uint32_t>::max(),
    // This is quite slower on V100 for some reason
    // Set to false if you know at compile-time you will never need dropout
    bool kSupportsDropout_ = true,
    bool kSupportsBias_ = true,
    typename ToBatchHookType_ = DefaultToBatchHook>
struct AttentionKernel {
  enum CustomMaskType {
    NoCustomMask = 0,
    CausalFromTopLeft = 1,
    CausalFromBottomRight = 2,
    NumCustomMaskTypes,
  };

  using scalar_t = scalar_t_;
  using accum_t = float;
  using lse_scalar_t = float;
  using output_t = scalar_t;
  // Accumulator between 2 iterations
  // Using `accum_t` improves perf on f16 at the cost of
  // numerical errors
  using output_accum_t = accum_t;
  static constexpr bool kSupportsDropout = kSupportsDropout_;
  static constexpr bool kSupportsBias = kSupportsBias_;
  static constexpr int kKeysPerBlock = kKeysPerBlock_;
  static constexpr int kQueriesPerBlock = kQueriesPerBlock_;
  static constexpr int kMaxK = kMaxK_;
  static constexpr bool kIsAligned = isAligned_;
  static constexpr bool kSingleValueIteration = kMaxK <= kKeysPerBlock;
  static constexpr int32_t kAlignLSE = 32; // block size of backward
  static constexpr bool kIsHalf = cutlass::sizeof_bits<scalar_t>::value == 16;
  static constexpr bool kPreloadV =
      ArchTag::kMinComputeCapability >= 80 && kIsHalf;
  static constexpr bool kKeepOutputInRF = kSingleValueIteration;
  static constexpr bool kNeedsOutputAccumulatorBuffer = !kKeepOutputInRF &&
      !cutlass::platform::is_same<output_accum_t, output_t>::value;

  static_assert(kQueriesPerBlock % 32 == 0, "");
  static_assert(kKeysPerBlock % 32 == 0, "");
  static constexpr int kNumWarpsPerBlock =
      kQueriesPerBlock * kKeysPerBlock / (32 * 32);
  static constexpr int kWarpSize = 32;

  // Launch bounds
  static constexpr int kNumThreads = kWarpSize * kNumWarpsPerBlock;
  static constexpr int kMinBlocksPerSm =
      getWarpsPerSmFw<scalar_t, ArchTag>() / kNumWarpsPerBlock;
```
**EN**: Lines 112-169. This block declares the main `AttentionKernel` template and derives the compile-time policy for the whole forward path. Template parameters capture scalar type, architecture, alignment fast path, tile sizes in the query/key dimensions, maximum supported head width, optional dropout/bias support, and the batch hook type. Inside the class, the code fixes the accumulation type to `float`, defines custom causal mask modes, selects whether V should be preloaded on newer tensor-core architectures, decides if output can stay in registers for the full reduction, and computes thread/block launch geometry from the chosen CUTLASS warp shape.
**CN**: 第112-169行：这一部分声明核心模板 `AttentionKernel`，并推导整个前向路径的编译期策略。模板参数包含标量类型、目标架构、对齐快路径、query/key 方向的 tile 尺寸、支持的最大 head 宽度、是否启用 dropout/bias，以及 batch hook 类型。类体内部将累加类型固定为 `float`，定义自定义因果 mask 模式，决定在较新 Tensor Core 架构上是否预取 V，判断完整规约期间输出能否一直驻留在寄存器中，并依据 CUTLASS warp 形状计算线程块几何与 launch 配置。
```cpp
  struct Params {
    // Input tensors
    scalar_t* query_ptr = nullptr; // [num_queries, num_heads, head_dim]
    scalar_t* key_ptr = nullptr; // [num_keys, num_heads, head_dim]
    scalar_t* value_ptr = nullptr; // [num_keys, num_heads, head_dim_value]
    scalar_t* attn_bias_ptr = nullptr; // [num_heads, num_queries, num_keys]
    int32_t* seqstart_q_ptr = nullptr;
    int32_t* seqstart_k_ptr = nullptr;

    int32_t* seqlen_k_ptr = nullptr;
    uint32_t causal_diagonal_offset = 0;

    // Output tensors
    output_t* output_ptr = nullptr; // [num_queries, num_heads, head_dim_value]
    // [num_queries, num_heads, head_dim_value]
    output_accum_t* output_accum_ptr = nullptr;
    // [num_heads, num_queries] - can be null
    lse_scalar_t* logsumexp_ptr = nullptr;

    // Scale
    accum_t scale = 0.0;

    // Dimensions/strides
    int32_t head_dim = 0;
    int32_t head_dim_value = 0;
    int32_t num_queries = 0;
    int32_t num_keys = 0;
    int32_t num_keys_absolute = 0;

    uint8_t custom_mask_type = NoCustomMask;

    int32_t q_strideM = 0;
    int32_t k_strideM = 0;
    int32_t v_strideM = 0;
    int32_t bias_strideM = 0;

    int32_t o_strideM = 0;

    // Everything below is only used in `advance_to_block`
    // and shouldn't use registers
    int32_t q_strideH = 0;
    int32_t k_strideH = 0;
    int32_t v_strideH = 0;
    int64_t bias_strideH = 0;

    int64_t q_strideB = 0;
    int64_t k_strideB = 0;
    int64_t v_strideB = 0;
    int64_t bias_strideB = 0;

    int32_t num_batches = 0;
    int32_t num_heads = 0;

    // dropout
    bool use_dropout = false;
    unsigned long long dropout_batch_head_rng_offset = 0;
    float dropout_prob = 0.0f;
#ifdef HAS_PYTORCH
    at::PhiloxCudaState rng_engine_inputs = at::PhiloxCudaState(0, 0);
#endif

    // Moves pointers to what we should process
    // Returns "false" if there is no work to do
    CUTLASS_DEVICE bool advance_to_block() {
      auto batch_id = blockIdx.z;
      auto head_id = blockIdx.y;
      auto query_start = blockIdx.x * kQueriesPerBlock;

      auto lse_dim = ceil_div((int32_t)num_queries, kAlignLSE) * kAlignLSE;

      if (kSupportsDropout) {
        dropout_batch_head_rng_offset =
            batch_id * num_heads * num_queries * num_keys +
            head_id * num_queries * num_keys;
      }

      int64_t q_start = 0, k_start = 0;
      // Advance to current batch - in case of different sequence lengths
      constexpr bool kToBatchHook =
          !cutlass::platform::is_same<ToBatchHookType_, DefaultToBatchHook>::
              value;
      if (kToBatchHook) {
        // Call out to a custom implementation.
        if (!ToBatchHookType_::advance_to_batch(*this, q_start, k_start)) {
          return false;
        }
      } else if (seqstart_q_ptr != nullptr) {
        assert(seqstart_k_ptr != nullptr);
        seqstart_q_ptr += batch_id;

        q_start = seqstart_q_ptr[0];
        int64_t q_next_start = seqstart_q_ptr[1];
        int64_t k_end;
        seqstart_k_ptr += batch_id;

        if (seqlen_k_ptr) {
          k_start = seqstart_k_ptr[0];
          k_end = k_start + seqlen_k_ptr[batch_id];
        } else {
          k_start = seqstart_k_ptr[0];
          k_end = seqstart_k_ptr[1];
        }

        num_queries = q_next_start - q_start;
        num_keys = k_end - k_start;

        if (query_start >= num_queries) {
          return false;
        }
      } else {
        query_ptr += batch_id * q_strideB;
        key_ptr += batch_id * k_strideB;
        value_ptr += batch_id * v_strideB;
        output_ptr += int64_t(batch_id * num_queries) * o_strideM;
        if (output_accum_ptr != nullptr) {
          output_accum_ptr +=
              int64_t(batch_id * num_queries) * (head_dim_value * num_heads);
        }
        q_start = 0;
        k_start = 0;
      }

      // Advance to the current batch / head / query_start
      query_ptr += (q_start + query_start) * q_strideM + head_id * q_strideH;
      key_ptr += k_start * k_strideM + head_id * k_strideH;

      value_ptr += k_start * v_strideM + head_id * v_strideH;
      output_ptr +=
          int64_t(q_start + query_start) * o_strideM + head_id * head_dim_value;

      if (kSupportsBias && attn_bias_ptr != nullptr) {
        attn_bias_ptr += (batch_id * bias_strideB) + (head_id * bias_strideH);
      }
      if (output_accum_ptr != nullptr) {
        output_accum_ptr +=
            int64_t(q_start + query_start) * (head_dim_value * num_heads) +
            head_id * head_dim_value;
      } else {
        // Accumulate directly in the destination buffer (eg for f32)
        output_accum_ptr = (accum_t*)output_ptr;
      }

      if (logsumexp_ptr != nullptr) {
        // lse[batch_id, head_id, query_start]
        logsumexp_ptr +=
            batch_id * lse_dim * num_heads + head_id * lse_dim + query_start;
      }

      // Custom masking
      if (custom_mask_type == CausalFromBottomRight) {
        causal_diagonal_offset = num_keys - num_queries;
      }
      // We use num_keys_absolute to index into the rng_state
      // We need this index to match between forward and backwards
      num_keys_absolute = num_keys;
      if (custom_mask_type == CausalFromTopLeft ||
          custom_mask_type == CausalFromBottomRight) {
        // the bottom row of the current block is query_start + kQueriesPerBlock
        // the last active key is then query_start + causal_diagonal_offset +
        // kQueriesPerBlock so num_keys is the min between actual num_keys and
        // this to avoid extra computations
        num_keys = cutlass::fast_min(
            int32_t(query_start + causal_diagonal_offset + kQueriesPerBlock),
            num_keys);
      }

      num_queries -= query_start;
      num_batches = 0; // no longer used after

      // If num_queries == 1, and there is only one key head we're wasting
      // 15/16th of tensor core compute In that case :
      //  - we only launch kernels for head_id % kQueriesPerBlock == 0
      //  - we iterate over heads instead of queries (strideM = strideH)
      if (num_queries == 1 && k_strideH == 0 && v_strideH == 0) {
        if (head_id % kQueriesPerBlock != 0)
          return false;
        q_strideM = q_strideH;
        num_queries = num_heads;
        num_heads = 1; // unused but here for intent
        // remove causal since n_query = 1
        // otherwise, offset would change with head !
        custom_mask_type = NoCustomMask;
        o_strideM = head_dim_value;
      }

      // Make sure the compiler knows these variables are the same on all
      // the threads of the warp.
      // Only worth doing if they could have been modified above.
      query_ptr = warp_uniform(query_ptr);
      key_ptr = warp_uniform(key_ptr);
      value_ptr = warp_uniform(value_ptr);
      if (kSupportsBias) {
        attn_bias_ptr = warp_uniform(attn_bias_ptr);
      }
      output_ptr = warp_uniform(output_ptr);
      output_accum_ptr = warp_uniform(output_accum_ptr);
      logsumexp_ptr = warp_uniform(logsumexp_ptr);
      num_queries = warp_uniform(num_queries);
      num_keys = warp_uniform(num_keys);
      num_heads = warp_uniform(num_heads);
      o_strideM = warp_uniform(o_strideM);
      custom_mask_type = warp_uniform(custom_mask_type);
      return true;
    }

    __host__ dim3 getBlocksGrid() const {
      return dim3(
          ceil_div(num_queries, (int32_t)kQueriesPerBlock),
          num_heads,
          num_batches);
    }

    __host__ dim3 getThreadsGrid() const {
      return dim3(kWarpSize, kNumWarpsPerBlock, 1);
    }
  };
```
**EN**: Lines 170-385. `Params` is the runtime contract between host code and the forward kernel. It stores all tensor pointers (Q/K/V, optional bias, output, accumulator output, logsumexp), batch/head/sequence metadata, stride descriptors, and dropout state including Philox inputs. The key method is `advance_to_block()`: it interprets `blockIdx`, applies packed-sequence offsets, computes per-block query/key spans, optionally invokes the custom batch hook, trims work for causal masks, derives the absolute key count used for reproducible RNG indexing, and applies a single-query optimization that reassigns work from the query axis to the head axis when tensor-core utilization would otherwise be poor. The final `warp_uniform()` calls make control data identical across the warp before execution proceeds.
**CN**: 第170-385行：`Params` 是主机端与前向内核之间的运行时契约，保存了所有张量指针（Q/K/V、可选 bias、输出、累加输出、logsumexp）、batch/head/sequence 元数据、步幅描述，以及包括 Philox 输入在内的 dropout 状态。最关键的方法是 `advance_to_block()`：它解释 `blockIdx`，应用 packed sequence 偏移，计算当前块的 query/key 范围，可选地调用自定义 batch hook，针对因果 mask 裁剪无效工作，推导用于可复现 RNG 索引的绝对 key 数，并在 `num_queries == 1` 且 Tensor Core 利用率偏低时把部分工作从 query 维重新映射到 head 维。末尾的 `warp_uniform()` 则把控制数据在 warp 内统一，确保后续执行一致。
```cpp

  struct MM0 {
    /*
      In this first matmul, we compute a block of `Q @ K.T`.
      While the calculation result is still hot in registers, we update
      `mi`, `m_prime`, `s_prime` in shared-memory, and then store this value
      into a shared-memory ("AccumulatorSharedStorage") that is used later as
      operand A for the second matmul (see MM1)
    */
    using GemmType = DefaultGemmType<ArchTag, scalar_t>;

    using OpClass = typename GemmType::OpClass;
    using DefaultConfig =
        typename cutlass::gemm::device::DefaultGemmConfiguration<
            OpClass,
            ArchTag,
            scalar_t,
            scalar_t,
            scalar_t, // ElementC
            accum_t // ElementAccumulator
            >;
    static constexpr int kAlignmentA =
        kIsAligned ? DefaultConfig::kAlignmentA : GemmType::kMinimumAlignment;
    static constexpr int kAlignmentB =
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment;
    using ThreadblockShape = cutlass::gemm::
        GemmShape<kQueriesPerBlock, kKeysPerBlock, GemmType::ThreadK>;
    using WarpShape = cutlass::gemm::GemmShape<32, 32, GemmType::WarpK>;
    using DefaultMma = typename cutlass::gemm::threadblock::FindDefaultMma<
        scalar_t, // ElementA,
        cutlass::layout::RowMajor, // LayoutA,
        kAlignmentA,
        scalar_t, // ElementB,
        cutlass::layout::ColumnMajor, // LayoutB,
        kAlignmentB,
        accum_t,
        cutlass::layout::RowMajor, // LayoutC,
        OpClass,
        ArchTag, // ArchTag
        ThreadblockShape, // ThreadblockShape
        WarpShape, // WarpShape
        typename GemmType::InstructionShape, // InstructionShape
        ArchTag::kMinComputeCapability >= 80 && kIsHalf
            ? 4
            : DefaultConfig::kStages,
        typename GemmType::Operator // Operator
        >::DefaultMma;
    using MmaCore = typename DefaultMma::MmaCore;
    using IteratorA = typename DefaultMma::IteratorA;
    using IteratorB = typename DefaultMma::IteratorB;
    using DefaultThreadblockMma = typename DefaultMma::ThreadblockMma;
    using Mma = typename cutlass::platform::conditional<
        kSingleValueIteration,
        typename MakeCustomMma<DefaultThreadblockMma, kMaxK>::Mma,
        DefaultThreadblockMma>::type;
    using AccumLambdaIterator = typename DefaultMmaAccumLambdaIterator<
        typename Mma::Operator::IteratorC,
        accum_t,
        kWarpSize>::Iterator;
    static_assert(
        MmaCore::WarpCount::kM * MmaCore::WarpCount::kN *
                MmaCore::WarpCount::kK ==
            kNumWarpsPerBlock,
        "");

    // used for efficient load of bias tile Bij from global to shared memory
    using BiasLoader = TileSmemLoader<
        scalar_t,
        cutlass::MatrixShape<kQueriesPerBlock, kKeysPerBlock>,
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
    using AccumulatorSharedStorage = typename B2bGemm::AccumulatorSharedStorage;
  };
```
**EN**: Lines 386-468. This section defines the first matrix multiplication, which computes the attention score tile `Q · K^T`. It chooses a CUTLASS GEMM configuration based on architecture and alignment, fixes the threadblock/warp/instruction shapes, and optionally swaps in a custom MMA implementation when the head dimension is small enough to complete in a single value iteration. The block also defines `AccumLambdaIterator`, which walks score fragments element-by-element for scaling, masking, and softmax reduction, plus `BiasLoader` and `B2bGemm` helpers that stage bias and move the accumulated score tile into the shared-memory layout expected by the next stage.
**CN**: 第386-468行：这一节定义第一阶段矩阵乘法，用于计算注意力得分块 `Q · K^T`。代码根据架构与对齐情况选择 CUTLASS GEMM 配置，固定 threadblock/warp/instruction 形状，并在 head 维足够小时切换到单次 value iteration 可完成的自定义 MMA 实现。同时，这里还定义了 `AccumLambdaIterator`，用于逐元素遍历得分片段以完成缩放、mask 和 softmax 规约；`BiasLoader` 与 `B2bGemm` 则负责加载 bias，并把累加后的得分块转换到下一阶段所需的共享内存布局。
```cpp

  struct MM1 {
    /**
      Second matmul: perform `attn @ V` where `attn` is the attention (not
      normalized) and stored in shared memory
    */
    using GemmType = DefaultGemmType<ArchTag, scalar_t>;

    using OpClass = typename GemmType::OpClass;
    using DefaultConfig =
        typename cutlass::gemm::device::DefaultGemmConfiguration<
            OpClass,
            ArchTag,
            scalar_t,
            scalar_t,
            output_accum_t, // ElementC
            accum_t // ElementAccumulator
            >;
    static constexpr int kAlignmentA = DefaultConfig::kAlignmentA; // from smem
    static constexpr int kAlignmentB =
        kIsAligned ? DefaultConfig::kAlignmentB : GemmType::kMinimumAlignment;
    using ThreadblockShape = cutlass::gemm::
        GemmShape<kQueriesPerBlock, kKeysPerBlock, GemmType::ThreadK>;
    using WarpShape = cutlass::gemm::GemmShape<32, 32, GemmType::WarpK>;
    using InstructionShape = typename GemmType::InstructionShape;

    using LayoutB = cutlass::layout::RowMajor;
    using DefaultGemm = cutlass::gemm::kernel::DefaultGemm<
        scalar_t, // ElementA,
        cutlass::layout::RowMajor, // LayoutA,
        kAlignmentA,
        scalar_t, // ElementB,
        LayoutB, // LayoutB,
        kAlignmentB,
        output_accum_t,
        cutlass::layout::RowMajor, // LayoutC,
        accum_t,
        OpClass,
        ArchTag,
        ThreadblockShape,
        WarpShape,
        typename GemmType::InstructionShape,
        typename DefaultConfig::EpilogueOutputOp,
        void, // ThreadblockSwizzle - not used
        ArchTag::kMinComputeCapability >= 80 && kIsHalf
            ? 4
            : DefaultConfig::kStages,
        false, // SplitKSerial
        typename GemmType::Operator>;

    using WarpIteratorA = typename cutlass::gemm::threadblock::
        DefaultWarpIteratorAFromSharedMemory<
            typename DefaultGemm::Mma::Policy::Operator::Shape, // WarpShape
            typename DefaultGemm::Mma::Policy::Operator::InstructionShape,
            typename DefaultGemm::Mma::Policy::Operator::IteratorA,
            typename DefaultGemm::Mma::Policy>::WarpIterator;
    using DefaultMmaFromSmem =
        typename cutlass::gemm::threadblock::DefaultMmaFromSharedMemory<
            typename DefaultGemm::Mma,
            MM0::AccumulatorSharedStorage::Shape::kN, // kMaxK
            WarpIteratorA,
            false>; // kScaleOperandA
    using Mma = typename DefaultMmaFromSmem::Mma;
    using IteratorB = typename Mma::IteratorB;
    using WarpCount = typename Mma::WarpCount;
    static_assert(
        WarpCount::kM * WarpCount::kN * WarpCount::kK == kNumWarpsPerBlock,
        "");

    using DefaultEpilogue = typename DefaultGemm::Epilogue;
    using OutputTileIterator =
        typename cutlass::epilogue::threadblock::PredicatedTileIterator<
            typename DefaultEpilogue::OutputTileIterator::ThreadMap,
            output_t>;
    using OutputTileIteratorAccum =
        typename cutlass::epilogue::threadblock::PredicatedTileIterator<
            typename DefaultEpilogue::OutputTileIterator::ThreadMap,
            output_accum_t>;
  };
```
**EN**: Lines 469-547. The second GEMM consumes the normalized attention tile from shared memory and multiplies it by V. Compared with MM0, this stage has to account for two output modes: direct writeback to the final output tensor or accumulation into an intermediate buffer when multiple key blocks contribute to the same result tile. The aliases defined here select the shared-memory-fed MMA variant, the iterator that streams V from global memory, and the predicated epilogue iterators that later write either the final tensor or the accumulation buffer with the correct bounds handling.
**CN**: 第469-547行：第二个 GEMM 从共享内存中读取归一化后的注意力块，并与 V 相乘。与 MM0 相比，这一阶段需要处理两种输出模式：若当前块已经完成该输出 tile 的全部累加，则直接写回最终输出；否则先写入中间累加缓冲区，等待后续 key 块继续累加。本段定义的类型别名选择了从共享内存取 A 操作数的 MMA 变体、从全局内存读取 V 的迭代器，以及在边界条件下正确写回最终张量或累加缓冲区的 predicated epilogue 迭代器。
```cpp

  static constexpr int64_t kAlignmentQ = MM0::kAlignmentA;
  static constexpr int64_t kAlignmentK = MM0::kAlignmentB;
  static constexpr int64_t kAlignmentV = 1;

  // Shared storage - depends on kernel params
  struct ScalingCoefs {
    cutlass::Array<accum_t, kQueriesPerBlock> m_prime;
    cutlass::Array<accum_t, kQueriesPerBlock> s_prime;
    cutlass::Array<accum_t, kQueriesPerBlock> mi;
    cutlass::Array<accum_t, kQueriesPerBlock> out_rescale;
    cutlass::Array<accum_t, kQueriesPerBlock * MM0::MmaCore::WarpCount::kN>
        addition_storage;
  };

  struct SharedStorageEpilogueAtEnd : ScalingCoefs {
    struct SharedStorageAfterMM0 {
      // Everything here might be overwritten during MM0
      union {
        typename MM0::BiasLoader::SmemTile bias;
        typename MM0::AccumulatorSharedStorage si;
      };
      typename MM1::Mma::SharedStorage mm1;
    };

    union {
      typename MM0::Mma::SharedStorage mm0;
      SharedStorageAfterMM0 after_mm0;
      typename MM1::DefaultEpilogue::SharedStorage epilogue;
    };

    CUTLASS_DEVICE typename MM1::DefaultEpilogue::SharedStorage&
    epilogue_shared_storage() {
      return epilogue;
    }
  };

  struct SharedStorageEpilogueInLoop : ScalingCoefs {
    struct SharedStorageAfterMM0 {
      // Everything here might be overwritten during MM0
      union {
        typename MM0::BiasLoader::SmemTile bias;
        typename MM0::AccumulatorSharedStorage si;
      };
      typename MM1::Mma::SharedStorage mm1;
      typename MM1::DefaultEpilogue::SharedStorage epilogue;
    };

    union {
      typename MM0::Mma::SharedStorage mm0;
      SharedStorageAfterMM0 after_mm0;
    };

    CUTLASS_DEVICE typename MM1::DefaultEpilogue::SharedStorage&
    epilogue_shared_storage() {
      return after_mm0.epilogue;
    }
  };

  using SharedStorage = typename cutlass::platform::conditional<
      kSingleValueIteration || kKeepOutputInRF,
      SharedStorageEpilogueAtEnd,
      SharedStorageEpilogueInLoop>::type;

  static bool __host__ check_supported(Params const& p) {
    CHECK_ALIGNED_PTR(p.query_ptr, kAlignmentQ);
    CHECK_ALIGNED_PTR(p.key_ptr, kAlignmentK);
    CHECK_ALIGNED_PTR(p.value_ptr, kAlignmentV);
    if (kSupportsBias) {
      CHECK_ALIGNED_PTR(p.attn_bias_ptr, kAlignmentQ);
      XFORMERS_CHECK(
          p.num_batches <= 1 || p.bias_strideB % kAlignmentQ == 0,
          "attn_bias is not correctly aligned (strideB)");
      XFORMERS_CHECK(
          p.num_heads <= 1 || p.bias_strideH % kAlignmentQ == 0,
          "attn_bias is not correctly aligned (strideH)");
      XFORMERS_CHECK(
          p.bias_strideM % kAlignmentQ == 0,
          "attn_bias is not correctly aligned");
    }
    XFORMERS_CHECK(
        p.q_strideM % kAlignmentQ == 0,
        "query is not correctly aligned (strideM)");
    XFORMERS_CHECK(
        p.k_strideM % kAlignmentK == 0,
        "key is not correctly aligned (strideM)");
    XFORMERS_CHECK(
        p.v_strideM % kAlignmentV == 0,
        "value is not correctly aligned (strideM)");
    XFORMERS_CHECK(
        p.num_heads <= 1 || p.q_strideH % kAlignmentQ == 0,
        "query is not correctly aligned (strideH)");
    XFORMERS_CHECK(
        p.num_heads <= 1 || p.k_strideH % kAlignmentK == 0,
        "key is not correctly aligned (strideH)");
    XFORMERS_CHECK(
        p.num_heads <= 1 || p.v_strideH % kAlignmentV == 0,
        "value is not correctly aligned (strideH)");
    XFORMERS_CHECK(
        p.custom_mask_type < NumCustomMaskTypes,
        "invalid value for `custom_mask_type`");
    return true;
  }
```
**EN**: Lines 548-650. This block describes how the kernel reuses shared memory across different phases. `ScalingCoefs` stores the row-wise online softmax state (`m_prime`, `s_prime`, `mi`, and `out_rescale`) plus scratch reduction buffers, while the subsequent unions overlay MM0 accumulators, optional bias tiles, MM1 staging buffers, and epilogue storage so the same SRAM budget can serve multiple pipeline stages. `check_supported()` complements that layout by validating alignment assumptions on all major pointers and strides and rejecting unsupported custom-mask modes before launch.
**CN**: 第548-650行：这一部分描述内核如何在不同阶段之间复用共享内存。`ScalingCoefs` 保存按行在线 softmax 状态（`m_prime`、`s_prime`、`mi`、`out_rescale`）以及规约暂存区；后续的联合体则把 MM0 累加器、可选 bias tile、MM1 staging 区和 epilogue 存储覆盖到同一片 SRAM 上，以便在有限共享内存预算内服务多个流水阶段。`check_supported()` 则从运行时角度补充这一布局：它在 launch 前验证关键指针和步幅的对齐条件，并拒绝不支持的自定义 mask 模式。
```cpp

  static void CUTLASS_DEVICE attention_kernel(Params& p) {
    // In this block, we will only ever:
    // - read query[query_start:query_end, :]
    // - write to output[query_start:query_end, :]

    extern __shared__ char smem_buffer[];
    SharedStorage& shared_storage = *((SharedStorage*)smem_buffer);
    auto& m_prime = shared_storage.m_prime;
    auto& s_prime = shared_storage.s_prime;
    auto& mi = shared_storage.mi;
    auto& out_rescale = shared_storage.out_rescale;
    const uint32_t query_start = blockIdx.x * kQueriesPerBlock;

    static_assert(kQueriesPerBlock < kNumWarpsPerBlock * kWarpSize, "");
    if (thread_id() < kQueriesPerBlock) {
      s_prime[thread_id()] = accum_t(0);
      out_rescale[thread_id()] = accum_t(1.0);
      m_prime[thread_id()] =
          -cutlass::platform::numeric_limits<accum_t>::infinity();
      mi[thread_id()] = -cutlass::platform::numeric_limits<accum_t>::infinity();
    }
    typename MM1::Mma::FragmentC accum_o;
    accum_o.clear();

    auto createOutputIter = [&](int col) -> typename MM1::OutputTileIterator {
      using OutputTileIterator = typename MM1::OutputTileIterator;
      return OutputTileIterator(
          typename OutputTileIterator::Params{(int32_t)p.o_strideM},
          p.output_ptr,
          typename OutputTileIterator::TensorCoord{
              p.num_queries, p.head_dim_value},
          thread_id(),
          {0, col});
    };

    auto createOutputAccumIter = [&](int col) ->
        typename MM1::OutputTileIteratorAccum {
          using OutputTileIteratorAccum = typename MM1::OutputTileIteratorAccum;
          return OutputTileIteratorAccum(
              typename OutputTileIteratorAccum::Params{
                  (int32_t)(p.head_dim_value * p.num_heads)},
              p.output_accum_ptr,
              typename OutputTileIteratorAccum::TensorCoord{
                  p.num_queries, p.head_dim_value},
              thread_id(),
              {0, col});
        };

#ifdef HAS_PYTORCH
    curandStatePhilox4_32_10_t curand_state_init;
    if (kSupportsDropout && p.use_dropout) {
      const auto seeds = at::cuda::philox::unpack(p.rng_engine_inputs);

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
          &curand_state_init);
    }
#endif

    // Iterate through keys
    for (int32_t iter_key_start = 0; iter_key_start < p.num_keys;
         iter_key_start += kKeysPerBlock) {
      int32_t problem_size_0_m =
          cutlass::fast_min((int32_t)kQueriesPerBlock, p.num_queries);
      int32_t problem_size_0_n = cutlass::fast_min(
          int32_t(kKeysPerBlock), p.num_keys - iter_key_start);
      int32_t const& problem_size_0_k = p.head_dim;
      int32_t const& problem_size_1_n = p.head_dim_value;
      int32_t const& problem_size_1_k = problem_size_0_n;

      auto prologueV = [&](int blockN) {
        typename MM1::Mma::IteratorB iterator_V(
            typename MM1::IteratorB::Params{typename MM1::LayoutB(p.v_strideM)},
            p.value_ptr + iter_key_start * p.v_strideM,
            {problem_size_1_k, problem_size_1_n},
            thread_id(),
            cutlass::MatrixCoord{0, blockN * MM1::Mma::Shape::kN});
        MM1::Mma::prologue(
            shared_storage.after_mm0.mm1,
            iterator_V,
            thread_id(),
            problem_size_1_k);
      };

      __syncthreads(); // Need to have shared memory initialized, and `m_prime`
                       // updated from end of prev iter
      //
      // MATMUL: Q.K_t
      //
      // Computes the block-matrix product of:
      // (a) query[query_start:query_end, :]
      // with
      // (b) key[iter_key_start:iter_key_start + kKeysPerBlock]
      // and stores that into `shared_storage.si`
      //

      // Compute threadblock location
      cutlass::gemm::GemmCoord tb_tile_offset = {0, 0, 0};

      cutlass::MatrixCoord tb_offset_A{
          tb_tile_offset.m() * MM0::Mma::Shape::kM, tb_tile_offset.k()};

      cutlass::MatrixCoord tb_offset_B{
          tb_tile_offset.k(), tb_tile_offset.n() * MM0::Mma::Shape::kN};

      // Construct iterators to A and B operands
      typename MM0::IteratorA iterator_A(
          typename MM0::IteratorA::Params(
              typename MM0::MmaCore::LayoutA(p.q_strideM)),
          p.query_ptr,
          {problem_size_0_m, problem_size_0_k},
          thread_id(),
          tb_offset_A);

      typename MM0::IteratorB iterator_B(
          typename MM0::IteratorB::Params(
              typename MM0::MmaCore::LayoutB(p.k_strideM)),
          p.key_ptr + iter_key_start * p.k_strideM,
          {problem_size_0_k, problem_size_0_n},
          thread_id(),
          tb_offset_B);

      auto my_warp_id = warp_uniform(warp_id());
      auto my_lane_id = lane_id();

      // Construct thread-scoped matrix multiply
      typename MM0::Mma mma(
          shared_storage.mm0, thread_id(), my_warp_id, my_lane_id);

      typename MM0::Mma::FragmentC accum;

      accum.clear();

      auto gemm_k_iterations =
          (problem_size_0_k + MM0::Mma::Shape::kK - 1) / MM0::Mma::Shape::kK;

      // Compute threadblock-scoped matrix multiply-add
      mma(gemm_k_iterations, accum, iterator_A, iterator_B, accum);
      __syncthreads();

      if (kPreloadV) {
        prologueV(0);
      } else {
        MM1::Mma::drain_cp_asyncs();
      }

      typename MM0::Mma::Operator::IteratorC::TensorCoord
          iteratorC_tile_offset = {
              (tb_tile_offset.m() * MM0::Mma::WarpCount::kM) +
                  (my_warp_id % MM0::Mma::WarpCount::kM),
              (tb_tile_offset.n() * MM0::Mma::WarpCount::kN) +
                  (my_warp_id / MM0::Mma::WarpCount::kM)};

      // multiply by scaling factor
      if (kSupportsBias) {
        accum =
            cutlass::multiplies<typename MM0::Mma::FragmentC>()(p.scale, accum);
      }

      // apply attention bias if applicable
      if (kSupportsBias && p.attn_bias_ptr != nullptr) {
        // load bias tile Bij into shared memory
        typename MM0::BiasLoader::GmemTileIterator bias_iter(
            {cutlass::layout::RowMajor(p.bias_strideM)},
            // attn_bias_pointer points to matrix of size (n_queries, n_keys)
            // for the relevant batch_id and head_id
            p.attn_bias_ptr + query_start * p.bias_strideM + iter_key_start,
            {problem_size_0_m, problem_size_0_n},
            thread_id());
        cutlass::TensorRef<scalar_t, cutlass::layout::RowMajor> bias_tensor_ref(
            shared_storage.after_mm0.bias.data(),
            cutlass::layout::RowMajor(MM0::ThreadblockShape::kN));
        typename MM0::BiasLoader::SmemTileIterator smem_tile_iter(
            bias_tensor_ref, thread_id());
        MM0::BiasLoader::load(bias_iter, smem_tile_iter);

        // Pij += Bij, Pij is in register fragment and Bij is in shared memory
        auto lane_offset = MM0::AccumLambdaIterator::get_lane_offset(
            my_lane_id, my_warp_id, iteratorC_tile_offset);
        MM0::AccumLambdaIterator::iterateRows(
            lane_offset,
            [&](int accum_m) {},
            [&](int accum_m, int accum_n, int idx) {
              if (accum_m < problem_size_0_m && accum_n < problem_size_0_n) {
                accum[idx] += bias_tensor_ref.at({accum_m, accum_n});
              }
            },
            [&](int accum_m) {});
      }

      // Mask out last if causal
      // This is only needed if upper-right corner of current query / key block
      // intersects the mask Coordinates of upper-right corner of current block
      // is y=query_start x=min(iter_key_start + kKeysPerBlock, num_keys)) The
      // first masked element is x = y + offset -> query_start + offset There is
      // intersection (and we need to mask) if min(iter_key_start +
      // kKeysPerBlock, num_keys)) >= query_start + offset
      if (p.custom_mask_type &&
          cutlass::fast_min(iter_key_start + kKeysPerBlock, p.num_keys) >=
              (query_start + p.causal_diagonal_offset)) {
        auto query_start = blockIdx.x * kQueriesPerBlock;
        auto lane_offset = MM0::AccumLambdaIterator::get_lane_offset(
            my_lane_id, my_warp_id, iteratorC_tile_offset);
        int32_t last_col;
        MM0::AccumLambdaIterator::iterateRows(
            lane_offset,
            [&](int accum_m) {
              // last absolute col is (last absolute query + offset)
              // last local col is (last absolute query + offset -
              // iter_key_start)
              last_col = query_start + accum_m + p.causal_diagonal_offset -
                  iter_key_start;
            },
            [&](int accum_m, int accum_n, int idx) {
              if (accum_n > last_col) {
                accum[idx] =
                    -cutlass::platform::numeric_limits<accum_t>::infinity();
              }
            },
            [&](int accum_m) {});
      }
      // Update `mi` from accum stored in registers
      // Also does accum[i] <- exp(accum[i] - mi)
      iterative_softmax<typename MM0::Mma::Operator::IteratorC>(
          accum_o,
          accum,
          mi,
          m_prime,
          s_prime,
          out_rescale,
          shared_storage.addition_storage,
          my_lane_id,
          thread_id(),
          my_warp_id,
          p.num_keys - iter_key_start,
          iter_key_start == 0,
          iteratorC_tile_offset,
          kSupportsBias ? 1.0f : p.scale);

      // Output results to shared-memory
      int warp_idx_mn_0 = my_warp_id %
          (MM0::Mma::Base::WarpCount::kM * MM0::Mma::Base::WarpCount::kN);
      auto output_tile_coords = cutlass::MatrixCoord{
          warp_idx_mn_0 % MM0::Mma::Base::WarpCount::kM,
          warp_idx_mn_0 / MM0::Mma::Base::WarpCount::kM};

      MM0::B2bGemm::accumToSmem(
          shared_storage.after_mm0.si, accum, my_lane_id, output_tile_coords);

      __syncthreads();

#ifdef HAS_PYTORCH
      // apply dropout (if applicable) after we've written Pij to smem.
      // dropout is applied by multiplying each element of Pij by:
      // - 0 with probability dropout_p
      // - 1 / (1 - dropout_p) with probability 1 - dropout_p
      //
      // for backward purposes we want to be able to map each element of the
      // attention matrix to the same random uniform number as the one we used
      // in forward, without needing to use the same iteration order or having
      // to store the dropout matrix. its possible to do this in registers but
      // it ends up being very slow because each thread having noncontiguous
      // strips of the Pij tile means we have to skip around a lot, and also
      // have to generate a single random number at a time
      if (kSupportsDropout && p.use_dropout) {
        auto si = shared_storage.after_mm0.si.accum_ref();
        // each thread handles a contiguous sequence of elements from Sij, all
        // coming from the same row. the reason they have to come from the same
        // row is that the sampling random numbers from a contiguous random
        // number sequence is much more efficient than jumping around, and the
        // linear offset of each element of S (the global matrix) maps to an
        // offset in a random number sequence. for S, the end of a row and the
        // beginning of the next have adjacent offsets, but for Sij, this is not
        // necessarily the case.
        const int num_threads = blockDim.x * blockDim.y * blockDim.z;
        const int threads_per_row =
            cutlass::fast_min(num_threads / problem_size_0_m, problem_size_0_n);
        const int elts_per_thread = cutlass::round_nearest(
            cutlass::ceil_div(problem_size_0_n, threads_per_row), 4);

        const int thread_i = thread_id() / threads_per_row;
        const int thread_start_j =
            (thread_id() % threads_per_row) * elts_per_thread;

        if (thread_i < problem_size_0_m && thread_start_j < problem_size_0_n) {
          curandStatePhilox4_32_10_t curand_state = curand_state_init;
          skipahead(
              static_cast<unsigned long long>(
                  (query_start + thread_i) * p.num_keys_absolute +
                  (iter_key_start + thread_start_j)),
              &curand_state);
          const float dropout_scale = 1.0 / (1.0 - p.dropout_prob);

          // apply dropout scaling to elements this thread is responsible for,
          // in chunks of 4
          for (int sij_start_col_idx = thread_start_j; sij_start_col_idx <
               cutlass::fast_min(thread_start_j + elts_per_thread,
                                 problem_size_0_n);
               sij_start_col_idx += 4) {
            const float4 rand_uniform_quad = curand_uniform4(&curand_state);

            CUTLASS_PRAGMA_UNROLL
            for (int quad_idx = 0; quad_idx < 4; ++quad_idx) {
              si.at({thread_i, sij_start_col_idx + quad_idx}) *=
                  static_cast<scalar_t>(
                      dropout_scale *
                      ((&rand_uniform_quad.x)[quad_idx] > p.dropout_prob));
            }
          }
        }
        __syncthreads(); // p.use_dropout should have same value kernel-wide
      }
#endif
```
**EN**: Lines 651-975. This is the first half of `attention_kernel()`. After computing thread/warp identities and binding the shared-storage object, the kernel initializes per-row softmax state and prepares output iterators. The outer loop streams over key blocks: each iteration constructs MM0 iterators for Q and K, optionally preloads V for the following phase, runs the tensor-core QK matmul, scales the score fragment, optionally adds bias from shared memory, applies the selected causal/custom mask by forcing invalid logits to `-inf`, and calls `iterative_softmax()` to merge the current score block into the running row maximum and normalization sum. The normalized attention tile is then written to shared memory in the layout MM1 expects. Under `HAS_PYTORCH` and `use_dropout`, the same block immediately applies dropout in shared memory using Philox-generated random numbers indexed by the absolute matrix position, which preserves deterministic correspondence with backward without materializing a separate mask tensor.
**CN**: 第651-975行：这是 `attention_kernel()` 的前半部分。内核首先计算线程/warp 身份并绑定共享存储对象，随后初始化每一行的 softmax 状态并准备输出迭代器。外层循环按 key 块推进：每次迭代都会构造 MM0 的 Q/K 迭代器，可选地为下一阶段预取 V，执行 Tensor Core QK 乘法，对得分片段按 `p.scale` 缩放，可选地从共享内存加载并叠加 bias，再通过把无效位置设为 `-inf` 应用选定的因果/自定义 mask，最后调用 `iterative_softmax()` 把当前块并入运行中的行最大值与归一化和。归一化后的注意力块随后以 MM1 需要的布局写入共享内存。在 `HAS_PYTORCH` 且启用 `use_dropout` 时，同一代码块还会立刻在共享内存中用 Philox 随机数施加 dropout，随机数索引按全局矩阵绝对位置计算，因此无需显式保存 mask 也能与反向传播保持严格对应。
```cpp
      //
      // MATMUL: Attn . V
      // Run the matmul `attn @ V` for a block of attn and V.
      // `attn` is read from shared memory (in `shared_storage_si`)
      // `V` is read from global memory (with iterator_B)
      //

      const int64_t nBlockN = kSingleValueIteration
          ? 1
          : ceil_div(
                (int64_t)problem_size_1_n, int64_t(MM1::ThreadblockShape::kN));
      for (int blockN = 0; blockN < nBlockN; ++blockN) {
        int gemm_k_iterations =
            (problem_size_1_k + MM1::Mma::Shape::kK - 1) / MM1::Mma::Shape::kK;

        // Compute threadblock-scoped matrix multiply-add and store it in accum
        // (in registers)
        if (!kPreloadV) {
          __syncthreads(); // we share shmem between mma and epilogue
        }

        typename MM1::Mma::IteratorB iterator_V(
            typename MM1::IteratorB::Params{typename MM1::LayoutB(p.v_strideM)},
            p.value_ptr + iter_key_start * p.v_strideM,
            {problem_size_1_k, problem_size_1_n},
            thread_id(),
            cutlass::MatrixCoord{0, blockN * MM1::Mma::Shape::kN});
        typename MM1::Mma mma_pv(
            // operand A: Pij_dropped in shared memory
            shared_storage.after_mm0.si.accum_ref(),
            // operand B: shared memory staging area for Vj, which is loaded
            // from global memory
            shared_storage.after_mm0.mm1.operand_B_ref(),
            (int)thread_id(),
            (int)my_warp_id,
            (int)my_lane_id);
        mma_pv.set_prologue_done(kPreloadV);
        if (!kKeepOutputInRF) {
          accum_o.clear();
        }
        mma_pv(gemm_k_iterations, accum_o, iterator_V, accum_o);
        __syncthreads();

        if (kPreloadV && !kSingleValueIteration && blockN + 1 < nBlockN) {
          prologueV(blockN + 1);
        }

        if (!kKeepOutputInRF) {
          MM1::Mma::drain_cp_asyncs();
          DISPATCH_BOOL(
              iter_key_start == 0, kIsFirst, ([&] {
                DISPATCH_BOOL(
                    (iter_key_start + kKeysPerBlock) >= p.num_keys,
                    kIsLast,
                    ([&] {
                      using DefaultEpilogue = typename MM1::DefaultEpilogue;
                      using DefaultOp =
                          typename MM1::DefaultConfig::EpilogueOutputOp;
                      using ElementCompute = typename DefaultOp::ElementCompute;
                      using EpilogueOutputOp = typename cutlass::epilogue::
                          thread::MemoryEfficientAttentionNormalize<
                              typename cutlass::platform::conditional<
                                  kIsLast::value,
                                  output_t,
                                  output_accum_t>::type,
                              output_accum_t,
                              DefaultOp::kCount,
                              typename DefaultOp::ElementAccumulator,
                              ElementCompute,
                              kIsFirst::value,
                              kIsLast::value,
                              cutlass::Array<ElementCompute, kQueriesPerBlock>>;
                      using Epilogue = typename cutlass::epilogue::threadblock::
                          EpiloguePipelined<
                              typename DefaultEpilogue::Shape,
                              typename MM1::Mma::Operator,
                              DefaultEpilogue::kPartitionsK,
                              typename cutlass::platform::conditional<
                                  kIsLast::value,
                                  typename MM1::OutputTileIterator,
                                  typename MM1::OutputTileIteratorAccum>::type,
                              typename DefaultEpilogue::
                                  AccumulatorFragmentIterator,
                              typename DefaultEpilogue::WarpTileIterator,
                              typename DefaultEpilogue::SharedLoadIterator,
                              EpilogueOutputOp,
                              typename DefaultEpilogue::Padding,
                              DefaultEpilogue::kFragmentsPerIteration,
                              true, // IterationsUnroll
                              typename MM1::OutputTileIteratorAccum // Read
                                                                    // iterator
                              >;

                      int col = blockN * MM1::Mma::Shape::kN;
                      auto source_iter = createOutputAccumIter(col);
                      auto dest_iter = call_conditional<
                          kIsLast::value,
                          decltype(createOutputIter),
                          decltype(createOutputAccumIter)>::
                          apply(createOutputIter, createOutputAccumIter, col);
                      EpilogueOutputOp rescale(s_prime, out_rescale);
                      Epilogue epilogue(
                          shared_storage.epilogue_shared_storage(),
                          thread_id(),
                          my_warp_id,
                          my_lane_id);
                      epilogue(rescale, dest_iter, accum_o, source_iter);
                    }));
              }));
          if (!kSingleValueIteration) {
            __syncthreads();
          }
        }
      }
      __syncthreads(); // we modify `m_prime` after
    }

    if (kKeepOutputInRF) {
      constexpr bool kIsFirst = true;
      constexpr bool kIsLast = true;
      using DefaultEpilogue = typename MM1::DefaultEpilogue;
      using DefaultOp = typename MM1::DefaultConfig::EpilogueOutputOp;
      using ElementCompute = typename DefaultOp::ElementCompute;
      using EpilogueOutputOp =
          typename cutlass::epilogue::thread::MemoryEfficientAttentionNormalize<
              output_t, // output
              output_accum_t, // source
              DefaultOp::kCount,
              typename DefaultOp::ElementAccumulator, // accum
              output_accum_t, // compute
              kIsFirst,
              kIsLast,
              cutlass::Array<ElementCompute, kQueriesPerBlock>>;
      using Epilogue =
          typename cutlass::epilogue::threadblock::EpiloguePipelined<
              typename DefaultEpilogue::Shape,
              typename MM1::Mma::Operator,
              DefaultEpilogue::kPartitionsK,
              typename MM1::OutputTileIterator, // destination
              typename DefaultEpilogue::AccumulatorFragmentIterator,
              typename DefaultEpilogue::WarpTileIterator,
              typename DefaultEpilogue::SharedLoadIterator,
              EpilogueOutputOp,
              typename DefaultEpilogue::Padding,
              DefaultEpilogue::kFragmentsPerIteration,
              true, // IterationsUnroll
              typename MM1::OutputTileIteratorAccum // source tile
              >;
      auto dest_iter = createOutputIter(0);
      EpilogueOutputOp rescale(s_prime, out_rescale);
      Epilogue epilogue(
          shared_storage.epilogue_shared_storage(),
          thread_id(),
          warp_id(),
          lane_id());
      MM1::Mma::drain_cp_asyncs();
      epilogue(rescale, dest_iter, accum_o);
    }
```
**EN**: Lines 976-1134. The second half of the main loop performs `Attn · V`. It iterates over N tiles of the value/output dimension, constructs the MM1 MMA object, manages the preload state for architectures that keep V ahead of consumption, and executes the shared-memory-to-global-memory GEMM. The writeback path is heavily specialized: if the full reduction cannot stay in registers, the code dispatches an `EpiloguePipelined` instantiation that knows whether this is the first or final key block and whether the destination is the final output tensor or an accumulation buffer. If the output can remain in registers (`kKeepOutputInRF`), the kernel defers normalization and the final epilogue until all key blocks have been processed, reducing shared-memory traffic at the cost of more register pressure.
**CN**: 第976-1134行：主循环的后半部分执行 `Attn · V`。代码沿着 value/output 维度遍历 N 方向的 tile，构造 MM1 的 MMA 对象，管理在支持预取的架构上对 V 的前置装载状态，并执行从共享内存到全局内存的 GEMM。写回路径高度特化：如果完整规约无法一直保存在寄存器中，代码会分派 `EpiloguePipelined` 实例，感知当前是否是首个或最后一个 key 块，以及目标是最终输出张量还是中间累加缓冲区。若输出可以一直留在寄存器中（`kKeepOutputInRF`），则内核会把归一化和最终 epilogue 延迟到全部 key 块处理完之后，以更高寄存器压力换取更低的共享内存流量。
```cpp
    // 7. Calculate logsumexp
    // To make the backward easier, we pad logsumexp with `inf`
    // this avoids a few bound checks, and is not more expensive during fwd
    static_assert(kQueriesPerBlock < kNumWarpsPerBlock * kWarpSize, "");
    if (p.logsumexp_ptr && thread_id() < kQueriesPerBlock) {
      auto lse_dim = ceil_div((int32_t)p.num_queries, kAlignLSE) * kAlignLSE;
      constexpr float kLog2e = 1.4426950408889634074; // log_2(e) = M_LOG2E
      if (thread_id() < p.num_queries) {
        p.logsumexp_ptr[thread_id()] = accum_t(mi[thread_id()] / kLog2e) +
            cutlass::fast_log(accum_t(s_prime[thread_id()]));
      } else if (thread_id() < lse_dim) {
        p.logsumexp_ptr[thread_id()] =
            cutlass::platform::numeric_limits<accum_t>::infinity();
      }
    }
  }
```
**EN**: Lines 1135-1151. Once the key loop finishes, this short block materializes the row-wise log-sum-exp values needed by backward. The kernel writes padded rows in chunks aligned to 32 elements, storing `mi / log2(e) + log(s_prime)` for valid queries and `+inf` for padding lanes so downstream kernels can safely assume a fixed alignment and still recognize out-of-range rows.
**CN**: 第1135-1151行：key 循环结束后，这一小段代码把反向传播需要的按行 log-sum-exp 写回全局内存。内核按 32 元素对齐的粒度写入：对有效 query 行存储 `mi / log2(e) + log(s_prime)`，对填充行写入 `+inf`，这样后续内核既可以假设固定对齐，又能可靠识别越界行。
```cpp
  template <typename WarpIteratorC>
  CUTLASS_DEVICE static void iterative_softmax(
      typename WarpIteratorC::Fragment& frag_o, // output so far
      typename WarpIteratorC::Fragment& frag,
      cutlass::Array<accum_t, kQueriesPerBlock>& mi,
      cutlass::Array<accum_t, kQueriesPerBlock>& m_prime,
      cutlass::Array<accum_t, kQueriesPerBlock>& s_prime,
      cutlass::Array<accum_t, kQueriesPerBlock>& out_rescale,
      cutlass::Array<accum_t, kQueriesPerBlock * MM0::MmaCore::WarpCount::kN>&
          addition_storage,
      int8_t lane_id,
      int8_t thread_id,
      int8_t warp_id,
      int max_col,
      bool is_first,
      typename WarpIteratorC::TensorCoord const& tile_offset,
      float scaling) {
    /* Iterates on the accumulator and corresponding position on result matrix

    (1) Update `mi[r]` to the max value of the row `r`
    (2) In a second iteration do the following:
        (a) accum   <- exp(accum - mi)
        (b) m_prime <- exp(m_prime - mi)
        (c) s_prime <- s_prime * m_prime + sum(accum)

    All of this is done on registers, before we store all of this
    on shared memory for the next matmul with Value.
    */
    using Fragment = typename WarpIteratorC::Fragment;
    using LambdaIterator = typename DefaultMmaAccumLambdaIterator<
        WarpIteratorC,
        accum_t,
        kWarpSize>::Iterator;
    // Convert to `accum_t` (rather than double)
    constexpr float kLog2e = 1.4426950408889634074; // log_2(e) = M_LOG2E

    static_assert(kQueriesPerBlock % kNumWarpsPerBlock == 0, "");
    static constexpr int kLinesPerWarp = kQueriesPerBlock / kNumWarpsPerBlock;

    frag = cutlass::multiplies<Fragment>()(scaling * kLog2e, frag);

    auto lane_offset =
        LambdaIterator::get_lane_offset(lane_id, warp_id, tile_offset);

    // First update `mi` to the max per-row
    {
      accum_t max;
      LambdaIterator::iterateRows(
          lane_offset,
          [&](int accum_m) {
            max = -cutlass::platform::numeric_limits<accum_t>::infinity();
          },
          [&](int accum_m, int accum_n, int idx) {
            if (accum_n < max_col) {
              max = cutlass::fast_max(max, frag[idx]);
            }
          },
          [&](int accum_m) {
            // Having 4x atomicMax seems faster than reduce within warp
            // first...
            atomicMaxFloat(&mi[accum_m], max);
          });
    }

    // Make sure we all share the update values for `mi`
    __syncthreads();

    // Doing this `exp` is quite expensive. Let's
    // split it across the warps
    bool restore_mi_to_minus_inf = false;
    if (lane_id < kLinesPerWarp) {
      int id = warp_id * kLinesPerWarp + lane_id;
      auto m_prime_id = m_prime[id];
      auto mi_id = mi[id];
      bool changed = m_prime_id < mi_id; // `false` if both are -inf
      if (changed) {
        auto m_prime_exp = exp2f(m_prime_id - mi_id);
        out_rescale[id] = m_prime_exp;
        s_prime[id] *= m_prime_exp;
      } else {
        // Only when bias is enabled, it's possible that all the first values
        // of attention are masked to `-inf`. In that case we want to avoid
        // `nan = exp2f(-inf - (-inf))` so we temporarily set `mi` to 0
        if (kSupportsBias &&
            mi_id == -cutlass::platform::numeric_limits<accum_t>::infinity()) {
          restore_mi_to_minus_inf = true;
          mi[id] = 0.0f;
        }
        out_rescale[id] = 1.0f;
      }
    }
    __syncthreads(); // Update output fragments
    if (kKeepOutputInRF && !is_first) {
      accum_t line_rescale;
      LambdaIterator::iterateRows(
          lane_offset,
          [&](int accum_m) { line_rescale = out_rescale[accum_m]; },
          [&](int accum_m, int accum_n, int idx) {
            frag_o[idx] = frag_o[idx] * line_rescale;
          },
          [&](int accum_m) {});
    }
    // Update accum_m, accum_n, ...
    {
      accum_t mi_row, total_row;
      LambdaIterator::iterateRows(
          lane_offset,
          [&](int accum_m) { mi_row = mi[accum_m]; },
          [&](int accum_m, int accum_n, int idx) {
            frag[idx] =
                (accum_n < max_col) ? exp2f(frag[idx] - mi_row) : accum_t(0.0);
          },
          [&](int accum_m) {});
      LambdaIterator::iterateRows(
          lane_offset,
          [&](int accum_m) { total_row = 0.0; },
          [&](int accum_m, int accum_n, int idx) { total_row += frag[idx]; },
          [&](int accum_m) {
            if (LambdaIterator::reduceSameRow(
                    lane_id, total_row, [](accum_t a, accum_t b) {
                      return a + b;
                    })) {
              // NOTE: we could atomically add `total_row` to `s_prime`, but
              // it's faster (and deterministic) to avoid atomics here
              addition_storage
                  [accum_m + kQueriesPerBlock * tile_offset.column()] =
                      total_row;
            }
          });
    }
    __syncthreads();
    if (lane_id < kLinesPerWarp) {
      int id = warp_id * kLinesPerWarp + lane_id;
      accum_t total_row = s_prime[id];
      if (restore_mi_to_minus_inf) {
        // Restore `mi`, see above when we set `restore_mi_to_minus_inf=true`
        mi[id] = -cutlass::platform::numeric_limits<accum_t>::infinity();
      } else {
        m_prime[id] = mi[id];
      }
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < MM0::MmaCore::WarpCount::kN; ++i) {
        total_row += addition_storage[id + kQueriesPerBlock * i];
      }
      s_prime[id] = total_row;
    }
  }
```
**EN**: Lines 1152-1299. `iterative_softmax()` is the numerical core of the forward kernel. For every score fragment, it rescales logits into base-2 exponent space, finds the row maximum cooperatively across lanes/warps, updates the running maximum in shared memory, rescales any previously accumulated output when the reference maximum changes, exponentiates the current tile relative to the updated maximum, and reduces the row sums back into `s_prime`. The implementation contains careful handling for rows that are fully masked (especially when bias is present) so that `-inf - (-inf)` does not introduce NaNs. This routine is the reason the kernel can stream over key blocks while maintaining exact softmax normalization semantics.
**CN**: 第1152-1299行：`iterative_softmax()` 是前向内核的数值核心。对于每个得分片段，它会先把 logits 映射到以 2 为底的指数空间，在线程/warp 间协同求出每一行最大值，再更新共享内存中的运行最大值；若参考最大值发生变化，还要相应缩放此前已经累计的输出；随后针对更新后的最大值对当前 tile 做指数化，并把行和规约回 `s_prime`。实现中特别处理了整行都被 mask 掉的情况（尤其是带 bias 时），避免 `-inf - (-inf)` 产生 NaN。正是这个例程让内核能够在按 key 分块流式处理的同时，仍保持与标准 softmax 一致的归一化语义。
```cpp
  static CUTLASS_DEVICE int8_t lane_id() {
    return threadIdx.x;
  }
  static CUTLASS_DEVICE int8_t warp_id() {
    return threadIdx.y;
  }
  static CUTLASS_DEVICE int16_t thread_id() {
    return threadIdx.x + threadIdx.y * blockDim.x;
  }
};

template <typename AK>
__global__ void __launch_bounds__(AK::kNumThreads, AK::kMinBlocksPerSm)
    attention_kernel_batched_impl(typename AK::Params p) {
  if (!p.advance_to_block()) {
    return;
  }
  AK::attention_kernel(p);
}

template <typename AK>
__global__ void __launch_bounds__(AK::kNumThreads, AK::kMinBlocksPerSm)
    attention_kernel_batched(typename AK::Params params);
```
**EN**: Lines 1300-1322. The file ends with thin helpers that derive lane/warp/thread identifiers from CUDA thread indices and two global entry points. `attention_kernel_batched_impl` performs the per-block `advance_to_block()` check and then dispatches into `AK::attention_kernel`, while `attention_kernel_batched` exists as the launch-bounded wrapper that external code instantiates with a concrete `AttentionKernel` policy.
**CN**: 第1300-1322行：文件结尾是一些轻量辅助函数与两个全局入口。辅助函数把 CUDA 的 `threadIdx` 映射为 lane/warp/thread 标识；`attention_kernel_batched_impl` 先执行每个线程块的 `advance_to_block()` 检查，再调用 `AK::attention_kernel`；`attention_kernel_batched` 则作为带 `launch_bounds` 的外部包装入口，由具体的 `AttentionKernel` 策略实例化后供调用方使用。
---
## Key Concepts / 关键概念
- The forward kernel is organized as a fused two-GEMM pipeline: MM0 computes blockwise scores, `iterative_softmax()` normalizes them online, and MM1 applies the normalized attention to V. / 前向内核本质上是一个融合的双 GEMM 流水线：MM0 计算分块得分，`iterative_softmax()` 在线归一化，MM1 再把归一化后的注意力应用到 V。
- Template policy drives architecture specialization. Alignment, head dimension, and SM capability influence whether custom MMA paths, V preloading, and register-resident accumulation are enabled. / 模板策略直接决定架构特化。对齐情况、head 维度以及目标 SM 能力会影响是否启用自定义 MMA 路径、V 预取，以及寄存器驻留式累加。
- The online softmax state (`m_prime`, `s_prime`, `mi`, `out_rescale`) is the central invariant that allows numerically stable streaming over key blocks without storing the full attention matrix. / 在线 softmax 状态（`m_prime`、`s_prime`、`mi`、`out_rescale`）是整个实现的核心不变量，使内核能够在不保存完整注意力矩阵的前提下，对 key 块进行数值稳定的流式处理。
- Synchronization is used at phase boundaries where shared memory changes ownership: after MM0 score storage, after optional dropout, before MM1 consumption, and around shared reductions for softmax statistics. / 同步主要出现在共享内存“所有权”切换的阶段边界：MM0 得分写入之后、可选 dropout 之后、MM1 消费之前，以及 softmax 统计量的共享规约前后。
## Dependencies / 依赖项
- `gemm/custom_mma.h` — Substitutes FMHA-specific threadblock MMA pipelines for the CUTLASS defaults / 用 FMHA 专用 threadblock MMA 流水替换 CUTLASS 默认实现
- `gemm/find_default_mma.h` — Controls MMA selection details such as FastF32 stage count and policy compatibility / 控制 FastF32 stage 数和策略兼容性等 MMA 选择细节
- `gemm/mma_from_smem.h` — Implements the second GEMM that consumes attention tiles from shared memory / 实现从共享内存消费注意力 tile 的第二个 GEMM
- `epilogue/epilogue_pipelined.h, epilogue/epilogue_rescale_output.h` — Write results while rescaling partial accumulations produced by online softmax / 在在线 softmax 产生部分累加时负责重缩放并写出结果
- `transform/tile_smem_loader.h` — Repackages staged tiles for the transitions between QK, softmax, and AV phases / 为 QK、softmax 与 AV 阶段切换重新组织分级 tile
- `debug_utils.h, gemm_kernel_utils.h` — Supply utility code for launch bounds, layouts, and kernel diagnostics / 提供 launch bounds、布局与内核诊断所需的工具代码
