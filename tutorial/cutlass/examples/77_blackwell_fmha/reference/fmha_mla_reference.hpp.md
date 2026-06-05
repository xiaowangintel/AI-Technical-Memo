# fmha_mla_reference.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/reference/fmha_mla_reference.hpp`  
**Purpose / 用途**: Provides the MLA forward reference path, explicitly separating latent and RoPE score terms and optionally remapping K/V pages through a page table. / 提供 MLA 前向参考路径：显式分离 latent 与 RoPE 分数项，并可通过页表对 K/V 页做重映射。

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
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
  class ProblemShape,
  class TensorSeq,
  class TensorPageTable,
  class TensorQL,
  class TensorQR,
  class TensorCL,
  class TensorKR,
  class TensorO,
  class TensorLSE,
  class Scale
>
void __global__ fmha_mla_reference_kernel(
    ProblemShape problem_shape,
    TensorSeq mSeq, TensorPageTable mPT,
    TensorQL mQL, TensorQR mQR,
    TensorCL mCL, TensorKR mKR,
    TensorO mO, TensorLSE mLSE,
    Scale softmax_scale) {

  using namespace cute;

  auto [H, K, D, B] = problem_shape;
  auto [D_latent, D_rope] = D;

  using Element = typename TensorO::value_type;
  using ElementAcc = typename TensorLSE::value_type;

  extern __shared__ ElementAcc mS[];
  // ElementAcc* mS = reinterpret_cast<ElementAcc*>(mS_mem);

  for (int idx_B = blockIdx.y; idx_B < B; idx_B += gridDim.y) {
    if (mSeq.data() != nullptr) {
      K = mSeq(idx_B);
    }

    for (int idx_H = blockIdx.x; idx_H < H; idx_H += gridDim.x) {

      for (int idx_K = threadIdx.x; idx_K < K; idx_K += blockDim.x) {
        ElementAcc acc = 0;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
        for (int idx_D = 0; idx_D < D_latent; idx_D++) {
          int page_idx_K = idx_K;
          int page_idx_B = idx_B;
          if (mPT.data() != nullptr) {
            page_idx_B = mPT(idx_K / size<0>(mCL), idx_B); 
            page_idx_K = idx_K % size<0>(mCL);
          }
          ElementAcc eQ = mQL(idx_H, idx_D, idx_B);
          ElementAcc eK = mCL(page_idx_K, idx_D, page_idx_B);
          acc += eQ * eK;
        }

        for (int idx_D = 0; idx_D < D_rope; idx_D++) {
          int page_idx_K = idx_K;
          int page_idx_B = idx_B;
          if (mPT.data() != nullptr) {
            page_idx_B = mPT(idx_K / size<0>(mCL), idx_B); 
            page_idx_K = idx_K % size<0>(mCL);
          }
          ElementAcc eQ = mQR(idx_H, idx_D, idx_B);
          ElementAcc eK = mKR(page_idx_K, idx_D, page_idx_B);
          acc += eQ * eK;
        }
        mS[idx_K] = acc;
      }

      __syncthreads();

      ElementAcc maxS = -std::numeric_limits<ElementAcc>::infinity();
      for (int idx_K = 0; idx_K < K; idx_K++) {
        maxS = std::max<ElementAcc>(maxS, mS[idx_K]);
      }
      if (maxS == -std::numeric_limits<ElementAcc>::infinity()) maxS = 0;

      __syncthreads();

      for (int idx_K = threadIdx.x; idx_K < K; idx_K += blockDim.x) {
        mS[idx_K] = expf(softmax_scale * (mS[idx_K] - maxS));
      }

      __syncthreads();
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
      ElementAcc sum = 0;
      for (int idx_K = 0; idx_K < K; idx_K++) {
        sum += mS[idx_K];
      }

      ElementAcc o_scale = 1.0f / sum;

      for (int idx_D = threadIdx.x; idx_D < D_latent; idx_D += blockDim.x) {
        ElementAcc acc = 0;
        for (int idx_K = 0; idx_K < K; idx_K++) {
          int page_idx_K = idx_K;
          int page_idx_B = idx_B;
          if (mPT.data() != nullptr) {
            page_idx_B = mPT(idx_K / size<0>(mCL), idx_B); 
            page_idx_K = idx_K % size<0>(mCL);
          }
          ElementAcc eV = mCL(page_idx_K, idx_D, page_idx_B);
          ElementAcc eK = static_cast<Element>(mS[idx_K]);
          acc += eK * eV;
        }
        mO(idx_H, idx_D, idx_B) = static_cast<typename TensorO::value_type>(acc * o_scale);
      }

      if (threadIdx.x == 0) {
        mLSE(idx_H, idx_B) = log(sum) + softmax_scale * maxS;
      }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    }
  }
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
  class ProblemShape,
  class TensorSeq,
  class TensorPageTable,
  class TensorQL,
  class TensorQR,
  class TensorCL,
  class TensorKR,
  class TensorO,
  class TensorLSE,
  class Scale
>
void fmha_mla_reference(
    ProblemShape problem_shape,
    TensorSeq mSeq, TensorPageTable mPT,
    TensorQL mQL, TensorQR mQR,
    TensorCL mCL, TensorKR mKR,
    TensorO mO, TensorLSE mLSE,
    Scale scale) {

  using namespace cute;

  auto [H, K, D, B] = problem_shape;
  auto [D_latent, D_rope] = D;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  dim3 grid(H, B, 1);
  dim3 block(256);
  int shared_mem = K * int(sizeof(typename TensorLSE::value_type)) + 16;
  cudaError_t result;
  if (shared_mem >= (48 << 10)) {
    result = cudaFuncSetAttribute(
        &fmha_mla_reference_kernel<ProblemShape, TensorSeq, TensorPageTable, TensorQL, TensorQR, TensorCL, TensorKR, TensorO, TensorLSE, Scale>,
        cudaFuncAttributeMaxDynamicSharedMemorySize,
        shared_mem);
    if (cudaSuccess != result) {
      cudaGetLastError(); // Clear the error state
      throw std::runtime_error("Failed to allocate " +
                               std::to_string(shared_mem >> 10) + " KB dynamic smem for S/P tensor in ref. check - " +
                               "please try reducing seq_len or skipping ref. check");
    }    
  }
  fmha_mla_reference_kernel<<<grid, block, shared_mem>>>(
      problem_shape, mSeq, mPT, mQL, mQR, mCL, mKR, mO, mLSE, scale);
  cudaDeviceSynchronize();
  result = cudaGetLastError();
  if (cudaSuccess != result) {
    throw std::runtime_error("couldn't execute reference");
  }
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
