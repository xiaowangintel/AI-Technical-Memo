# fmha_fwd_reference.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/reference/fmha_fwd_reference.hpp`  
**Purpose / 用途**: Provides the standard forward FMHA reference path: explicit QK score computation, masking, softmax normalization, and PV contraction. / 提供标准前向 FMHA 参考路径：显式计算 QK 分数、施加掩码、执行 softmax 归一化，再与 V 做收缩。

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

#include "cute/tensor.hpp"
#include "collective/fmha_fusion.hpp"
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
  class ProblemShapeIn,
  class TensorQ,
  class TensorK,
  class TensorV,
  class TensorO,
  class TensorLSE,
  class Mask
>
void __global__ fmha_reference_kernel(
    ProblemShapeIn problem_shape_in,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE,
    Mask mask) {

  using namespace cute;
  using namespace cutlass::fmha::collective;

  using Element = typename TensorO::value_type;
  using ElementAccumulator = typename TensorLSE::value_type;

  extern __shared__ char mS_mem[];
  ElementAccumulator* mS = reinterpret_cast<ElementAccumulator*>(mS_mem);

  ElementAccumulator softmax_scale = static_cast<ElementAccumulator>(1.0 / sqrt(1.0 * size<1>(mQ)));

  auto id = make_identity_tensor(make_shape(1, 1));

  for (int idx_L = blockIdx.y; idx_L < size<4>(problem_shape_in); idx_L += gridDim.y) {
    for (int idx_Q = blockIdx.x; idx_Q < size<0>(problem_shape_in); idx_Q += gridDim.x) {
      
      auto coord_L = idx2crd(idx_L, shape<4>(problem_shape_in));
      auto get_coord_in = [&]() {
        if constexpr (rank_v<decltype(get<2>(ProblemShapeIn{}))> == 2) {
          return cute::make_tuple(idx_Q, _0{}, cute::make_tuple(_0{}, _0{}), cute::make_tuple(_0{}, _0{}), coord_L);
        } else {
          return cute::make_tuple(idx_Q, _0{}, _0{}, _0{}, coord_L);
        }
      };
      auto coord_in = get_coord_in();
      auto [problem_shape, coord] = apply_variable_length(problem_shape_in, coord_in, get<4,1>(coord_in));
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
      int head_qk = 0;
      int head_v = 0;
      if constexpr (rank_v<decltype(get<2>(problem_shape))> == 2) {
        // MLA case: head_qk 192, head_v = 128
        head_qk = size<2, 0>(problem_shape) + size<2, 1>(problem_shape);
        head_v = size<2, 0>(problem_shape);
      } else {
        head_qk = size<3>(problem_shape);
        head_v = head_qk;
      }

      if (get<0,0>(coord) >= get<0>(problem_shape)) continue;

      int offset_Q = 0;
      if constexpr (rank<0>(decltype(coord){}) == 2) {
        offset_Q = get<0,1>(coord);
      }
  
      int offset_K = 0;
      if constexpr (rank<1>(decltype(coord){}) == 2) {
        offset_K = get<1,1>(coord);
      }

      if (get<1>(problem_shape) == 0) {
        for (int idx_D = threadIdx.x; idx_D < head_qk; idx_D += blockDim.x) {
          mO(idx_Q + offset_Q, idx_D, idx_L) = Element(0);
        }

        if (threadIdx.x == 0 && mLSE.data() != nullptr) {
          mLSE(idx_Q + offset_Q, idx_L) = -INFINITY;
        }
        continue;
      }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
      for (int idx_K = threadIdx.x; idx_K < size<1>(problem_shape); idx_K += blockDim.x) {
        ElementAccumulator acc = 0;
        for (int idx_D = 0; idx_D < head_qk; idx_D++) {
          ElementAccumulator eQ = mQ(idx_Q + offset_Q, idx_D, idx_L);
          ElementAccumulator eK = mK(idx_K + offset_K, idx_D, idx_L);
          acc += eQ * eK;
        }
        auto frag = make_tensor<ElementAccumulator>(Shape<_1, _1>{});
        frag(0) = acc;
        mask.apply_mask(frag, make_tensor(id.data() + make_arithmetic_tuple(idx_Q, idx_K), id.layout()), problem_shape);
        mS[idx_K] = frag(0);
      }

      __syncthreads();

      ElementAccumulator maxS = -std::numeric_limits<ElementAccumulator>::infinity();
      for (int idx_K = 0; idx_K < size<1>(problem_shape); idx_K++) {
        maxS = std::max<ElementAccumulator>(maxS, mS[idx_K]);
      }
      if (maxS == -std::numeric_limits<ElementAccumulator>::infinity()) maxS = 0;

      __syncthreads();

      for (int idx_K = threadIdx.x; idx_K < size<1>(problem_shape); idx_K += blockDim.x) {
        mS[idx_K] = expf(softmax_scale * (mS[idx_K] - maxS));
      }

      __syncthreads();

      ElementAccumulator sum = 0;
      for (int idx_K = 0; idx_K < size<1>(problem_shape); idx_K++) {
        sum += mS[idx_K];
      }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
      ElementAccumulator scale = 1.0f / sum;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
      for (int idx_D = threadIdx.x; idx_D < head_v; idx_D += blockDim.x) {
        ElementAccumulator acc = 0;
        for (int idx_K = 0; idx_K < size<1>(problem_shape); idx_K++) {
          ElementAccumulator eV = mV(idx_K + offset_K, idx_D, idx_L);
          ElementAccumulator eK = static_cast<Element>(mS[idx_K]);
          acc += eK * eV;
        }
        mO(idx_Q + offset_Q, idx_D, idx_L) = static_cast<typename TensorO::value_type>(acc * scale);
      }

      if (threadIdx.x == 0 && mLSE.data() != nullptr) {
        mLSE(idx_Q + offset_Q, idx_L) = log(sum) + softmax_scale * maxS;
      }

    }
  }
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
  class ProblemShapeIn,
  class TensorQ,
  class TensorK,
  class TensorV,
  class TensorO,
  class TensorLSE,
  class Mask
>
void fmha_reference(
    ProblemShapeIn problem_shape_in,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE,
    Mask mask) {

  using namespace cute;

  dim3 grid(size<0>(mO), size<2>(mO), 1);
  dim3 block(256);
  int shared_mem = size<0>(mK) * int(sizeof(typename TensorLSE::value_type));
  cudaError_t result;
  if (shared_mem >= (48 << 10)) {
    result = cudaFuncSetAttribute(
        &fmha_reference_kernel<ProblemShapeIn, TensorQ, TensorK, TensorV, TensorO, TensorLSE, Mask>,
        cudaFuncAttributeMaxDynamicSharedMemorySize,
        shared_mem);
    if (cudaSuccess != result) {
      cudaGetLastError(); // Clear the error state
      throw std::runtime_error("Failed to allocate " +
                               std::to_string(shared_mem >> 10) + " KB dynamic smem for S/P tensor in ref. check - " +
                               "please try reducing seq_len or skipping ref. check");
    }
  }
  fmha_reference_kernel<<<grid, block, shared_mem>>>(problem_shape_in, mQ, mK, mV, mO, mLSE, mask);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- TMA-driven data movement / 基于 TMA 的数据搬运
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Reference-based numerical validation / 基于参考实现的数值验证

## Dependencies / 依赖项

- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
- `collective/fmha_fusion.hpp` — collective-stage helper for the attention/GEMM pipeline / 注意力或 GEMM 流水线的 collective 阶段辅助
