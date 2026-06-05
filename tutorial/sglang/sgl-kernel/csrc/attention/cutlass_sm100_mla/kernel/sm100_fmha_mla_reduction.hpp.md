# sm100_fmha_mla_reduction.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/attention/cutlass_sm100_mla/kernel/sm100_fmha_mla_reduction.hpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements attention-related kernels, layouts, or dispatch helpers for high-throughput inference. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 实现面向高吞吐推理的 attention 内核、数据布局或分发辅助逻辑。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Local implementation details
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 23-40: Local implementation details
```cpp
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

// clang-format off
#pragma once

#include "cutlass/cutlass.h"
#include "cutlass/arch/arch.h"
#include "cute/tensor.hpp"

namespace cutlass::fmha::kernel {
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 41-57: Types and data layout
```cpp
using namespace cute;
template<
    class ElementOut,
    class ElementAcc,
    class ElementScale,
    size_t kNumHeads,
    size_t kHeadDimLatent,
    int kMaxSplits
>
struct Sm100FmhaMlaReductionKernel {

  static const int SharedStorageSize = 0;
  static const int MaxThreadsPerBlock = 128;
  static const int MinBlocksPerMultiprocessor = 1;

  using ArchTag = cutlass::arch::Sm100;
```
**EN:** This section defines `ElementOut`, `ElementAcc`, `ElementScale`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`ElementOut`、`ElementAcc`、`ElementScale`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 58-79: Types and data layout
```cpp
  static_assert(kHeadDimLatent % MaxThreadsPerBlock == 0);
  struct Arguments {
    ElementAcc* ptr_oaccum = nullptr;
    ElementOut* ptr_o = nullptr;
    ElementAcc* ptr_lseaccum = nullptr;
    ElementAcc* ptr_lse = nullptr;
    ElementScale scale = 1.f;
    int num_batches = 0;
    int split_kv = -1;
    int dim_k = -1;
    int* ptr_seq = nullptr;
    int* ptr_split_kv = nullptr;
    int tile_shape_s = 128;
  };
  using Params = Arguments;

  static Params to_underlying_arguments(Arguments const& args, void* workspace) {
    return {args.ptr_oaccum, args.ptr_o, args.ptr_lseaccum, args.ptr_lse,
	    args.scale, args.num_batches, args.split_kv, args.dim_k, args.ptr_seq,
	    args.ptr_split_kv, args.tile_shape_s};
  }
```
**EN:** This section defines `Arguments`, `to_underlying_arguments`, `static_assert`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Arguments`、`to_underlying_arguments`、`static_assert`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 80-101: Runtime integration and dispatch
```cpp
  static size_t get_workspace_size(Arguments const& /*args*/) {
    return 0;
  }

  static Status initialize_workspace(
      Arguments const& /*args*/, void* /*ws*/, cudaStream_t /*stream*/) {
    return Status::kSuccess;
  }

  static dim3 get_grid_shape(Params const& params) {
    return dim3(kNumHeads, 1, params.num_batches);
  }

  static dim3 get_block_shape() {
    return dim3(MaxThreadsPerBlock, 1, 1);
  }

  static bool can_implement(Arguments const& args) {
    if (args.num_batches <= 0) return false;
    if (args.split_kv <= 0) return false;
    return true;
  }
```
**EN:** This section uses `get_workspace_size`, `initialize_workspace`, `get_grid_shape` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`get_workspace_size`、`initialize_workspace`、`get_grid_shape`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 102-122: Device helpers and synchronization
```cpp

  CUTLASS_DEVICE void operator() (Params const& params, char* smem_raw) {
    if (params.split_kv <= 1) return;
    auto blk_coord = make_coord(blockIdx.x, _0{}, blockIdx.z);

    __shared__ ElementAcc sLseScale[kMaxSplits];
    const size_t offset_lseaccum = get<0>(blk_coord) + kNumHeads * params.split_kv * get<2>(blk_coord);
    const size_t offset_lse = get<0>(blk_coord) + kNumHeads * get<2>(blk_coord);

    Tensor gLSEaccum = make_tensor(make_gmem_ptr(params.ptr_lseaccum + offset_lseaccum),
                                   make_shape(params.split_kv), Stride<Int<kNumHeads>>{});

    Tensor gLSE = make_tensor(make_gmem_ptr(params.ptr_lse + offset_lse),
                              Shape<_1>{}, Stride<_1>{});

    auto dim_k = params.ptr_seq == nullptr ?  params.dim_k : params.ptr_seq[get<2>(blk_coord)];
    auto local_split_kv = params.ptr_split_kv == nullptr ? params.split_kv : params.ptr_split_kv[get<2>(blk_coord)];
    auto k_tile_total = ceil_div(dim_k, params.tile_shape_s);
    auto k_tile_per_cta = ceil_div(k_tile_total, local_split_kv);
    local_split_kv = ceil_div(k_tile_total, k_tile_per_cta);
```
**EN:** This section implements `ceil_div`, `offset_lseaccum`, `offset_lse`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ceil_div`、`offset_lseaccum`、`offset_lse`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 123-143: Device helpers and synchronization
```cpp
    int warp_idx = cutlass::canonical_warp_idx_sync();
    if (warp_idx == 0) {
      constexpr int kNLsePerThread = cute::ceil_div(kMaxSplits, 32);

      ElementAcc local_lse[kNLsePerThread];

      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kNLsePerThread; ++i) {
        const int split = i * 32 + threadIdx.x;
        local_lse[i] = split < local_split_kv ? gLSEaccum(split) : -std::numeric_limits<ElementAcc>::infinity();
      }

      ElementAcc lse_max = -std::numeric_limits<ElementAcc>::infinity();
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kNLsePerThread; ++i) {
        lse_max = max(lse_max, local_lse[i]);
      }
      CUTLASS_PRAGMA_UNROLL
      for (int offset = 16; offset >= 1; offset /= 2) {
        lse_max = max(lse_max, __shfl_xor_sync(0xffffffff, lse_max, offset));
      }
```
**EN:** This section implements `canonical_warp_idx_sync`, `ceil_div`, `gLSEaccum`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`canonical_warp_idx_sync`、`ceil_div`、`gLSEaccum`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 144-164: Device helpers and synchronization
```cpp
      lse_max = lse_max == -std::numeric_limits<ElementAcc>::infinity() ? 0.0f : lse_max;  // In case all local LSEs are -inf
      lse_max = __shfl_sync(0xffffffff, lse_max, 0);

      ElementAcc sum_lse = 0;
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kNLsePerThread; ++i) {
        sum_lse = sum_lse + expf(local_lse[i] - lse_max);
      }

      CUTLASS_PRAGMA_UNROLL
      for (int offset = 16; offset >= 1; offset /= 2) {
        sum_lse = sum_lse + __shfl_xor_sync(0xffffffff, sum_lse, offset);
      }

      sum_lse = __shfl_sync(0xffffffff, sum_lse, 0);

      ElementAcc global_lse = (sum_lse == 0.f || sum_lse != sum_lse) ? std::numeric_limits<ElementAcc>::infinity() : logf(sum_lse) + lse_max;
      if (threadIdx.x == 0 and params.ptr_lse != nullptr) {
        gLSE(0) = global_lse;
      }
```
**EN:** This section implements `__shfl_sync`, `expf`, `__shfl_xor_sync`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__shfl_sync`、`expf`、`__shfl_xor_sync`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 165-187: Device helpers and synchronization
```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kNLsePerThread; ++i) {
        const int split = i * 32 + threadIdx.x;
        if (split < local_split_kv) {
          sLseScale[split] = expf(local_lse[i] - global_lse);
        }
      }
    }
    __syncthreads();

    constexpr int Elements = kHeadDimLatent / MaxThreadsPerBlock;
    const size_t offset_oaccum = kHeadDimLatent * params.split_kv * (get<0>(blk_coord) + kNumHeads * get<2>(blk_coord));
    Tensor gOaccum = make_tensor(make_gmem_ptr(params.ptr_oaccum + offset_oaccum),
                               Shape<Int<kHeadDimLatent>>{}, Stride<_1>{});
    ElementAcc local_val[Elements] = {0};
    for (int split = 0; split < local_split_kv; ++split) {
      ElementAcc lse_scale = sLseScale[split];
      CUTLASS_PRAGMA_UNROLL
      for(int i = 0; i < Elements; ++i) {
        local_val[i] += lse_scale * gOaccum(threadIdx.x + MaxThreadsPerBlock * i);
      }
      gOaccum.data() = gOaccum.data() + kHeadDimLatent;
    }
```
**EN:** This section implements `expf`, `__syncthreads`, `gOaccum`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`expf`、`__syncthreads`、`gOaccum`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 188-197: Device helpers and synchronization
```cpp
    auto ptr_o_local = params.ptr_o + (get<0>(blk_coord) + get<2>(blk_coord) * kNumHeads) * kHeadDimLatent;
    Tensor gO = make_tensor(make_gmem_ptr(ptr_o_local), Shape<Int<kHeadDimLatent>>{}, Stride<_1>{});

    CUTLASS_PRAGMA_UNROLL
    for(int i = 0; i < Elements; ++i) {
      gO(threadIdx.x + MaxThreadsPerBlock * i) = static_cast<ElementOut>(local_val[i]);
    }
  }
};
```
**EN:** This section implements `gO`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`gO`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 198-198: Local implementation details
```cpp
}  // namespace cutlass::fmha::kernel
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass/cutlass.h`, `cutlass/arch/arch.h`, `cute/tensor.hpp`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: attention / cutlass_sm100_mla / kernel / sm100_fmha_mla_reduction.hpp
