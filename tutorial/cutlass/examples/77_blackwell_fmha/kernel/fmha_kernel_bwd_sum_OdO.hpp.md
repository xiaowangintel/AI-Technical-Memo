# fmha_kernel_bwd_sum_OdO.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/kernel/fmha_kernel_bwd_sum_OdO.hpp`  
**Purpose / 用途**: Computes the helper statistic `sum_OdO` and optional scaled LSE values used by the backward FMHA pipeline. / 计算反向 FMHA 流水线所需的辅助统计量 `sum_OdO`，并可选输出缩放后的 LSE。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
```

**EN**: This comment block frames the file: it usually states the target architecture, problem family, usage pattern, or design motivation before the code starts. That context matters because later template choices are easier to understand once the intended hardware path and workload are known. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一段注释用于给文件定调：通常先说明目标架构、问题类型、使用方式或设计动机，再进入具体代码。先理解这些背景，再看后面的模板选择与硬件路径会更清晰。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#pragma once

#include "cutlass/cutlass.h"
#include "cute/layout.hpp"

namespace cutlass::fmha::kernel {

using namespace cute;

template<class ProblemShape, class Element, class ElementAcc>
struct FmhaKernelBwdSumOdO {

  struct Arguments {
    ProblemShape problem_shape;

    const Element* ptr_O;
    cute::tuple<int, cute::_1, cute::tuple<cute::tuple<int, int>, int>> stride_O;
    const Element* ptr_dO;
    cute::tuple<int, cute::_1, cute::tuple<cute::tuple<int, int>, int>> stride_dO;

    ElementAcc* ptr_sum_OdO;
    cute::tuple<cute::_1, cute::tuple<cute::tuple<int, int>, int>> stride_sum_OdO;

    const ElementAcc* ptr_lse = nullptr;
    cute::tuple<cute::_1, cute::tuple<cute::tuple<int, int>, int>> stride_lse;

    ElementAcc* ptr_scaled_lse = nullptr;
    cute::tuple<cute::_1, cute::tuple<cute::tuple<int, int>, int>> stride_scaled_lse;

    ElementAcc sum_odo_scale = 1.0;
    ElementAcc lse_scale = 1.0;
  };

  using Params = Arguments;

  using ClusterShape = Shape<_1, _1, _1>;
  static constexpr int SharedStorageSize = 0;

  static const int MinBlocksPerMultiprocessor = 1;
  static const int MaxThreadsPerBlock = 128;
  using ArchTag = cutlass::arch::Sm100;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Targets blackwell sm100 execution paths. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。面向 Blackwell SM100 执行路径。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
  static size_t get_workspace_size(Arguments const& args) { return 0; }
  static cutlass::Status initialize_workspace(Arguments const&, void*, cudaStream_t) {
    return cutlass::Status::kSuccess;
  }

  static const int kBlockQ = 16;

  static const int kNumThreadsD = 8;
  static const int kNumThreadsQ = MaxThreadsPerBlock / kNumThreadsD;
  static const int kElementsPerLoad = 2;

  static const int kIterationsQ = kBlockQ / kNumThreadsQ;

  static bool can_implement(Arguments const& args) {
    return get<2>(args.problem_shape) % kElementsPerLoad == 0 && get<3>(args.problem_shape) % kElementsPerLoad == 0;
  }

  static dim3 get_grid_shape(Params const& params) {
    dim3 grid(ceil_div(size<0>(params.problem_shape), kBlockQ), size<4,0>(params.problem_shape), size<4,1>(params.problem_shape));
    return grid;
  }

  static dim3 get_block_shape() {
    dim3 block(kNumThreadsD, kNumThreadsQ, 1);
    return block;
  }

  static Params to_underlying_arguments(Arguments const& args, void* workspace) {
    return args;
  }

  CUTLASS_DEVICE void operator()(const Params &params, char* smem) {
    auto ptr_O_bh = params.ptr_O + blockIdx.y * get<2,0,0>(params.stride_O) + blockIdx.z * get<2,1>(params.stride_O);
    auto ptr_dO_bh = params.ptr_dO + blockIdx.y * get<2,0,0>(params.stride_dO) + blockIdx.z * get<2,1>(params.stride_dO);
    auto ptr_sum_OdO_bh = params.ptr_sum_OdO + blockIdx.y * get<1,0,0>(params.stride_sum_OdO) + blockIdx.z * get<1,1>(params.stride_sum_OdO);
    auto ptr_lse_bh = params.ptr_lse + blockIdx.y * get<1,0,0>(params.stride_lse) + blockIdx.z * get<1,1>(params.stride_lse);
    auto ptr_scaled_lse_bh = params.ptr_scaled_lse + blockIdx.y * get<1,0,0>(params.stride_scaled_lse) + blockIdx.z * get<1,1>(params.stride_scaled_lse);
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    auto problem_q = get<0>(params.problem_shape);
    int seqlen_q = problem_q;
    if constexpr (is_variable_length_v<decltype(problem_q)>) {
      int offset = problem_q.cumulative_length[blockIdx.z];
      ptr_O_bh += offset * get<0>(params.stride_O);
      ptr_dO_bh += offset * get<0>(params.stride_dO);
      ptr_lse_bh += offset * get<0>(params.stride_lse);
      seqlen_q = problem_q.cumulative_length[blockIdx.z + 1] - offset;
    }

    CUTLASS_PRAGMA_UNROLL
    for (int idx_q_t = threadIdx.y; idx_q_t < kBlockQ; idx_q_t += kNumThreadsQ) {
      int idx_q = idx_q_t + kBlockQ * blockIdx.x;
      if (idx_q >= seqlen_q) continue;
      ElementAcc acc = 0;
      auto ptr_O_bhq = ptr_O_bh + idx_q * get<0>(params.stride_O);
      auto ptr_dO_bhq = ptr_dO_bh + idx_q * get<0>(params.stride_dO);
      auto ptr_sum_OdO_bhq = ptr_sum_OdO_bh + idx_q * get<0>(params.stride_sum_OdO);
      auto ptr_lse_bhq = ptr_lse_bh + idx_q * get<0>(params.stride_lse);
      auto ptr_scaled_lse_bhq = ptr_scaled_lse_bh + idx_q * get<0>(params.stride_scaled_lse);

      for (int idx_d = threadIdx.x * kElementsPerLoad; idx_d < get<3>(params.problem_shape); idx_d += kElementsPerLoad * kNumThreadsD) {
        Element value_O[kElementsPerLoad];
        Element value_dO[kElementsPerLoad];
        
        using Vec = uint_bit_t<sizeof_bits_v<Element> * kElementsPerLoad>;
        *reinterpret_cast<Vec*>(value_O) = *reinterpret_cast<const Vec*>(&ptr_O_bhq[idx_d]);
        *reinterpret_cast<Vec*>(value_dO) = *reinterpret_cast<const Vec*>(&ptr_dO_bhq[idx_d]);

        for (int v = 0; v < kElementsPerLoad; v++) {
          acc += value_O[v] * value_dO[v];
        }
      }

      for (int i = 1; i < kNumThreadsD; i *= 2) {
        acc += __shfl_xor_sync((uint32_t)-1, acc, i, kNumThreadsD);
      }
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
      if (threadIdx.x == 0) {
        *ptr_sum_OdO_bhq = params.sum_odo_scale * acc;
        if (params.ptr_scaled_lse) {
          *ptr_scaled_lse_bhq = params.lse_scale * *ptr_lse_bhq;
        }
      }
    }
  }
};

}  // namespace cutlass::fmha::kernel
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- Cluster-shape tuning and possible multi-SM cooperation / Cluster 形状调优与可能的多 SM 协作
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层

## Dependencies / 依赖项

- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cute/layout.hpp` — CuTe tensor/layout support / CuTe 张量/布局支持
