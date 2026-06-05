# fmha_kernel_bwd_convert.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/88_hopper_fmha/kernel/fmha_kernel_bwd_convert.hpp`  
**Purpose / 用途**: Converts backward intermediate accumulators into the final output element type for `dQ`, `dK`, and `dV`. / 将反向传播中的中间累加缓冲区转换成最终输出元素类型，用于 `dQ`、`dK` 和 `dV`。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#include "cutlass/cutlass.h"
#include "cute/layout.hpp"

namespace cutlass::fmha::kernel {

using namespace cute;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<class Element, class ElementAccumulator>
struct FmhaKernelBwdConvert {

  struct Arguments {
    tuple<int, int, int, int, int> problem_size;

    const ElementAccumulator* ptr_src_dQ;
    tuple<int, int, int, _1> stride_src_dQ;
    const ElementAccumulator* ptr_src_dK;
    tuple<int, int, int, _1> stride_src_dK;
    const ElementAccumulator* ptr_src_dV;
    tuple<int, int, int, _1> stride_src_dV;
    
    Element* ptr_dest_dQ;
    tuple<int, int, int, _1> stride_dest_dQ;
    Element* ptr_dest_dK;
    tuple<int, int, int, _1> stride_dest_dK;
    Element* ptr_dest_dV;
    tuple<int, int, int, _1> stride_dest_dV;
  };

  using Params = Arguments;

  using ClusterShape = Shape<_1, _1, _1>;
  static constexpr int SharedStorageSize = 0;

  static const int MinBlocksPerMultiprocessor = 1;
  static const int MaxThreadsPerBlock = 128;
  using ArchTag = cutlass::arch::Sm90;

  static const int kBlockSeq = 8;

  static size_t get_workspace_size(Arguments const& args) { return 0; }
  static cutlass::Status initialize_workspace(Arguments const&, void*, cudaStream_t) {
    return cutlass::Status::kSuccess;
  }

  static const int kNumThreadsD = 16;
  static const int kNumThreadsSeq = MaxThreadsPerBlock / kNumThreadsD;
  static const int kElementsPerLoad = 4;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Makes cluster geometry an explicit compile-time tuning knob.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 把 cluster 几何作为显式的编译期调优参数。

---

```cpp
  static const int kIterationsSeq = kBlockSeq / kNumThreadsSeq;

  static bool can_implement(Arguments const& args) {
    return get<4>(args.problem_size) % kElementsPerLoad == 0;
  }

  static dim3 get_grid_shape(Params const& params) {
    dim3 grid(size<0>(params.problem_size), size<1>(params.problem_size), ceil_div(std::max(size<2>(params.problem_size), size<3>(params.problem_size)), kBlockSeq));
    return grid;
  }

  static dim3 get_block_shape() {
    dim3 block(kNumThreadsD, kNumThreadsSeq, 1);
    return block;
  }

  static Params to_underlying_arguments(Arguments const& args, void* workspace) {
    return args;
  }

  template<class StrideSrc, class StrideDest>
  CUTLASS_DEVICE void copy(Params const& params, const ElementAccumulator* ptr_src, StrideSrc const& stride_src, Element* ptr_dest, StrideDest const& stride_dest, int count) {
    auto ptr_src_bh = ptr_src + get<0>(stride_src) * blockIdx.x + get<1>(stride_src) * blockIdx.y;
    auto ptr_dest_bh = ptr_dest + get<0>(stride_dest) * blockIdx.x + get<1>(stride_dest) * blockIdx.y;

    for (int idx_s_t = threadIdx.y; idx_s_t < kBlockSeq; idx_s_t += kNumThreadsSeq) {
      int idx_s = idx_s_t + kBlockSeq * blockIdx.z;
      if (idx_s >= count) continue;
      auto ptr_src_bhs = ptr_src_bh + idx_s * get<2>(stride_src);
      auto ptr_dest_bhs = ptr_dest_bh + idx_s * get<2>(stride_dest);

      for (int idx_d = threadIdx.x * kElementsPerLoad; idx_d < get<4>(params.problem_size); idx_d += kElementsPerLoad * kNumThreadsD) {
        ElementAccumulator value_src[kElementsPerLoad];
        Element value_dest[kElementsPerLoad];
        
        using VecSrc = uint_bit_t<sizeof_bits_v<ElementAccumulator> * kElementsPerLoad>;
        using VecDest = uint_bit_t<sizeof_bits_v<Element> * kElementsPerLoad>;
        *reinterpret_cast<VecSrc*>(value_src) = *reinterpret_cast<const VecSrc*>(&ptr_src_bhs[idx_d]);
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
        for (int v = 0; v < kElementsPerLoad; v++) {
          value_dest[v] = value_src[v];
        }

        *reinterpret_cast<VecDest*>(&ptr_dest_bhs[idx_d]) = *reinterpret_cast<const VecDest*>(value_dest);
      }
    }
  }

  CUTLASS_DEVICE void operator()(const Params &params, char* smem) {
    if (params.ptr_src_dQ != nullptr) {
      copy(params, params.ptr_src_dQ, params.stride_src_dQ, params.ptr_dest_dQ, params.stride_dest_dQ, get<2>(params.problem_size));
    }
    if (params.ptr_src_dK != nullptr) {
      copy(params, params.ptr_src_dK, params.stride_src_dK, params.ptr_dest_dK, params.stride_dest_dK, get<3>(params.problem_size));
    }
    if (params.ptr_src_dV != nullptr) {
      copy(params, params.ptr_src_dV, params.stride_src_dV, params.ptr_dest_dV, params.stride_dest_dV, get<3>(params.problem_size));
    }
  }
};
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
}  // namespace cutlass::fmha::kernel
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Cluster-shape tuning and possible multi-SM cooperation / Cluster 形状调优与可能的多 SM 协作
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Convolution expressed through CUTLASS-style template composition / 通过 CUTLASS 风格模板组合表达卷积

## Dependencies / 依赖项

- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cute/layout.hpp` — CuTe tensor/layout support / CuTe 张量/布局支持
