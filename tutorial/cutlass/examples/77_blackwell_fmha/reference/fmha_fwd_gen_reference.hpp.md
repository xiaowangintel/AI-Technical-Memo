# fmha_fwd_gen_reference.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/reference/fmha_fwd_gen_reference.hpp`  
**Purpose / 用途**: Provides the generation-style forward reference path that first appends new K/V tokens into cache and then runs attention against the updated cache. / 提供生成式前向参考路径：先把新 K/V token 追加到缓存，再对更新后的缓存执行注意力。

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

#include <vector>
#include "cute/tensor.hpp"
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
  class ElementAcc,
  class ProblemShape,
  class TensorQ,
  class TensorNewK,
  class TensorNewV,
  class TensorCacheK,
  class TensorCacheV,
  class TensorO
>
void __global__ fmha_fwd_gen_reference_kernel(
    ProblemShape problem_shape,
    const int* seqlen_kv, const int* cache_batch_idx, 
    TensorQ mQ, TensorNewK mNewK, TensorNewV mNewV,
    TensorCacheK mCacheK, TensorCacheV mCacheV, TensorO mO) {

  using namespace cute;
  extern __shared__ char mS_mem[];
  ElementAcc* mS = reinterpret_cast<ElementAcc*>(mS_mem);

  float scale = 1.0f / std::sqrt(float(get<2>(problem_shape)));

  if (mNewK.data() != nullptr) {
    // 1. copy in new_k to cache
    for (int idx_h = blockIdx.x; idx_h < size<3,0,1>(problem_shape); idx_h += gridDim.x) {
      for (int idx_b = blockIdx.z; idx_b < size<3,1>(problem_shape); idx_b += gridDim.z) {
        int idx_b_kv = cache_batch_idx != nullptr ? cache_batch_idx[idx_b] : idx_b;
        for (int idx_d = threadIdx.x; idx_d < size<2>(problem_shape); idx_d += blockDim.x) {
          mCacheK(seqlen_kv[idx_b], idx_d, make_coord(make_coord(_0{}, idx_h), idx_b_kv)) =
              mNewK(_0{}, idx_d, make_coord(make_coord(_0{}, idx_h), idx_b));
          mCacheV(seqlen_kv[idx_b], idx_d, make_coord(make_coord(_0{}, idx_h), idx_b_kv)) =
              mNewV(_0{}, idx_d, make_coord(make_coord(_0{}, idx_h), idx_b));
        }
      }
    }
  }

  // 2. compute attention
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  for (int idx_h_kv = blockIdx.x; idx_h_kv < size<3,0,1>(problem_shape); idx_h_kv += gridDim.x) {
    for (int idx_h_qo = blockIdx.y; idx_h_qo < size<3,0,0>(problem_shape); idx_h_qo += gridDim.y) {
      int idx_h = idx_h_qo + size<3,0,0>(problem_shape) * idx_h_kv;
      for (int idx_b = blockIdx.z; idx_b < size<3,1>(problem_shape); idx_b += gridDim.z) {
        int idx_b_kv = cache_batch_idx != nullptr ? cache_batch_idx[idx_b] : idx_b;
        const int kDim = 128;
        ElementAcc reg_o[kDim] = {0};
        ElementAcc row_max = -INFINITY;
        ElementAcc row_sum = 0;
        auto iteration = [&](auto const& tK, auto const& tV) {
          ElementAcc reg_s = 0;
          for (int idx_d = 0; idx_d < kDim; idx_d++) {
            ElementAcc eQ = mQ(_0{}, idx_d, make_coord(idx_h, idx_b));
            ElementAcc eK = tK(idx_d);
            reg_s += eQ * eK;
          }

          ElementAcc old_row_max = row_max;
          row_max = std::max(row_max, reg_s);

          ElementAcc adjustment = std::exp(scale * (old_row_max - row_max));
          row_sum *= adjustment;
          for (int idx_d = 0; idx_d < kDim; idx_d++) {
            reg_o[idx_d] *= adjustment;
          }

          ElementAcc reg_p = std::exp(scale * (reg_s - row_max));
          row_sum += reg_p;

          for (int idx_d = 0; idx_d < kDim; idx_d++) {
            ElementAcc eV = tV(idx_d);
            reg_o[idx_d] += reg_p * eV;
          }
        };

        for (int idx_s = threadIdx.x; idx_s < seqlen_kv[idx_b]; idx_s += blockDim.x) {
          iteration(mCacheK(idx_s, _, make_coord(idx_h, idx_b_kv)), mCacheV(idx_s, _, make_coord(idx_h, idx_b_kv)));
        }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
        if (mNewK.data() != nullptr && threadIdx.x == 0) {
          iteration(mNewK(_0{}, _, make_coord(idx_h, idx_b)), mNewV(_0{}, _, make_coord(idx_h, idx_b)));
        }

        mS[threadIdx.x] = row_max;
        __syncthreads();
        float old_row_max = row_max;
        for (int i = 0; i < blockDim.x; i++) {
          row_max = std::max(row_max, mS[i]);
        }
        __syncthreads();

        ElementAcc adjustment = std::exp(scale * (old_row_max - row_max));
        row_sum *= adjustment;
        for (int idx_d = 0; idx_d < kDim; idx_d++) {
          reg_o[idx_d] *= adjustment;
        }
        mS[threadIdx.x] = row_sum;
        __syncthreads();

        row_sum = 0;
        for (int i = 0; i < blockDim.x; i++) {
          row_sum += mS[i];
        }
        __syncthreads();
        for (int idx_d = 0; idx_d < kDim; idx_d++) {
          mS[idx_d] = 0;
        }
        __syncthreads();

        for (int idx_d = 0; idx_d < kDim; idx_d++) {
          reg_o[idx_d] /= row_sum;
          atomicAdd(&mS[idx_d], reg_o[idx_d]);
        }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
        __syncthreads();
        for (int idx_d = threadIdx.x; idx_d < kDim; idx_d += blockDim.x) {
          mO(_0{}, idx_d, make_coord(idx_h, idx_b)) = static_cast<typename TensorO::value_type>(mS[idx_d]);
        }
      }
    }
  }
}

template<
  class ElementAcc,
  class ProblemShape,
  class TensorQ,
  class TensorNewK,
  class TensorNewV,
  class TensorCacheK,
  class TensorCacheV,
  class TensorO
>
void fmha_fwd_gen_reference(
    ProblemShape problem_shape,
    const int* seqlen_kv, const int* cache_batch_idx, 
    TensorQ mQ, TensorNewK mNewK, TensorNewV mNewV,
    TensorCacheK mCacheK, TensorCacheV mCacheV, TensorO mO) {

  using namespace cute;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  dim3 grid(get<3,0,1>(problem_shape), get<3,0,0>(problem_shape), get<3,1>(problem_shape));
  dim3 block(128);
  int shared_mem = int(sizeof(ElementAcc)) * std::max<int>(128, block.x);
  assert(get<2>(problem_shape) == 128);
  fmha_fwd_gen_reference_kernel<ElementAcc><<<grid, block, shared_mem>>>(
      problem_shape, seqlen_kv, cache_batch_idx,
      mQ, mNewK, mNewV, mCacheK, mCacheV, mO
  );
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Reference-based numerical validation / 基于参考实现的数值验证

## Dependencies / 依赖项

- `vector` — included dependency used by this example / 此示例使用的包含依赖
- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
