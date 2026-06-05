# fmha_bwd_reference.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/reference/fmha_bwd_reference.hpp`  
**Purpose / 用途**: Provides the reference backward FMHA implementation for `dQ`, `dK`, and `dV`, including causal/varlen masking through the shared fusion policy objects. / 提供 `dQ/dK/dV` 的反向 FMHA 参考实现，并通过共享 fusion 策略对象支持因果/变长掩码。

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

#include "cute/tensor.hpp"
#include "collective/fmha_fusion.hpp"

using namespace cutlass::fmha::collective;
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
    class ProblemShape,
    class TensorQ, class TensorK, class TensorV,
    class TensorO, class TensorLSE, class TensorDO,
    class TensorDQ, /* class TensorDK, class TensorDV, */
    class Fusion
>
void __global__ fmha_bwd_reference_dQ_kernel(
    ProblemShape problem_shape_in,
    TensorQ mQ_in, TensorK mK_in, TensorV mV_in,
    TensorO mO_in, TensorLSE mLSE_in, TensorDO mDO_in,
    TensorDQ mDQ_in, /* TensorDK mDK, TensorDV mDV, */
    Fusion fusion) {

  using namespace cute;
  using namespace cutlass::fmha::collective;

  using Element = typename TensorO::value_type;
  using ElementAcc = typename TensorLSE::value_type;
  
  extern __shared__ char mS_mem[];
  Element* mS = reinterpret_cast<Element*>(mS_mem);

  ElementAcc softmax_scale = 1.0f / sqrtf(size<2>(problem_shape_in));

  for (int idx_L = blockIdx.y; idx_L < size<4>(problem_shape_in); idx_L += gridDim.y) {
    auto [problem_shape, offset] = apply_variable_length_offset(
        problem_shape_in,
        make_coord(_0{}, _0{}, _0{}, _0{},idx2crd(idx_L, get<4>(problem_shape_in)))
    );
    // problem_shape = problem_shape_in;
    // offset = repeat_like(problem_shape_in, _0{});
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    auto mQ = domain_offset(select<0,2,4>(offset), mQ_in);
    auto mK = domain_offset(select<1,2,4>(offset), mK_in);
    auto mV = domain_offset(select<1,3,4>(offset), mV_in);
    auto mO = domain_offset(select<0,3,4>(offset), mO_in);
    auto mLSE = domain_offset(select<0,4>(offset), mLSE_in);
    auto mDO = domain_offset(select<0,3,4>(offset), mDO_in);
    auto mDQ = domain_offset(select<0,2,4>(offset), mDQ_in);
    for (int idx_Q = blockIdx.x; idx_Q < size<0>(problem_shape); idx_Q += gridDim.x) {
      for (int idx_K = threadIdx.x; idx_K < size<1>(problem_shape); idx_K += blockDim.x) {
        ElementAcc acc_qk = 0;
        ElementAcc acc_dov = 0;
        ElementAcc acc_doo = 0;
        for (int idx_D0 = 0; idx_D0 < size<2>(problem_shape); idx_D0++) {
          acc_qk += mQ(idx_Q, idx_D0, idx_L) * mK(idx_K, idx_D0, idx_L);
          // acc_dov += mDO(idx_Q, idx_D0, idx_L) * mV(idx_K, idx_D0, idx_L);
          // acc_doo += mDO(idx_Q, idx_D0, idx_L) * mO(idx_Q, idx_D0, idx_L);
        }  // for idx_D0

        for (int idx_D1 = 0; idx_D1 < size<3>(problem_shape); idx_D1++) {
          acc_dov += mDO(idx_Q, idx_D1, idx_L) * mV(idx_K, idx_D1, idx_L);
          acc_doo += mDO(idx_Q, idx_D1, idx_L) * mO(idx_Q, idx_D1, idx_L);
        }

        auto id = make_identity_tensor(make_shape(1, 1));
        auto frag = make_tensor<ElementAcc>(Shape<_1, _1>{});
        frag(0) = acc_qk;
        fusion.apply_mask(frag, make_tensor(id.data() + make_arithmetic_tuple(idx_Q, idx_K), id.layout()), problem_shape);
        acc_qk = frag(0);

        mS[idx_K] = static_cast<Element>(expf(softmax_scale * acc_qk - mLSE(idx_Q, idx_L)) * softmax_scale * (acc_dov - acc_doo));
      }  // for idx_K

      __syncthreads();
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
      for (int idx_D = threadIdx.x; idx_D < size<2>(problem_shape); idx_D += blockDim.x) {
        ElementAcc acc = 0;
        for (int idx_K = 0; idx_K < size<1>(problem_shape); idx_K++) {
          ElementAcc rK = mK(idx_K, idx_D, idx_L);
          ElementAcc rDS = mS[idx_K];
          acc += rDS * rK;
        }
        mDQ(idx_Q, idx_D, idx_L) = static_cast<typename TensorDQ::value_type>(acc);
      }  // for idx_D
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
    class TensorQ, class TensorK, class TensorV,
    class TensorO, class TensorLSE, class TensorDO,
    /* class TensorDQ, */ class TensorDK, /* class TensorDV, */
    class Fusion
>
void __global__ fmha_bwd_reference_dK_kernel(
    ProblemShape problem_shape_in,
    TensorQ mQ_in, TensorK mK_in, TensorV mV_in,
    TensorO mO_in, TensorLSE mLSE_in, TensorDO mDO_in,
    /* TensorDQ mDQ_in, */ TensorDK mDK_in, /* TensorDV mDV_in, */
    Fusion fusion) {

  using namespace cute;
  using namespace cutlass::fmha::collective;

  using Element = typename TensorO::value_type;
  using ElementAcc = typename TensorLSE::value_type;
  
  extern __shared__ char mS_mem[];
  Element* mS = reinterpret_cast<Element*>(mS_mem);

  ElementAcc softmax_scale = 1.0f / sqrtf(size<2>(problem_shape_in));

  auto [H, B] = get<4>(problem_shape_in);
  auto [H_R, H_K] = H;

  for (int idx_HB = blockIdx.y; idx_HB < H_K * B; idx_HB += gridDim.y) {
    auto [idx_H_K, idx_B] = idx2crd(idx_HB, make_shape(H_K, B));
    auto [problem_shape, offset] = apply_variable_length_offset(
      problem_shape_in,
        make_coord(_0{}, _0{}, _0{}, _0{}, make_coord(make_coord(_0{}, idx_H_K), idx_B))
    );
    auto [Q, K, D, D_VO, HB] = problem_shape;
    auto [offset_Q, offset_K, offset_D, offset_D_VO, offset_HB] = offset;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    auto mQ = domain_offset(make_coord(offset_Q, offset_D, offset_HB), mQ_in);
    auto mK = domain_offset(make_coord(offset_K, offset_D, offset_HB), mK_in);
    auto mV = domain_offset(make_coord(offset_K, offset_D_VO, offset_HB), mV_in);
    auto mO = domain_offset(make_coord(offset_Q, offset_D_VO, offset_HB), mO_in);
    auto mLSE = domain_offset(make_coord(offset_Q, offset_HB), mLSE_in);
    auto mDO = domain_offset(make_coord(offset_Q, offset_D_VO, offset_HB), mDO_in);
    auto mDK = domain_offset(make_coord(offset_K, offset_D, offset_HB), mDK_in);

    for (int idx_K = blockIdx.x; idx_K < K; idx_K += gridDim.x) {
      ElementAcc acc_dk = 0;
      for (int idx_H_R = 0; idx_H_R < H_R; idx_H_R++) {
        auto coord_HB = make_coord(make_coord(idx_H_R, idx_H_K), idx_B);
        for (int idx_Q = threadIdx.x; idx_Q < Q; idx_Q += blockDim.x) {
          ElementAcc acc_qk = 0;
          ElementAcc acc_dov = 0;
          ElementAcc acc_doo = 0;
          for (int idx_D0 = 0; idx_D0 < D; idx_D0++) {
            ElementAcc rQ = mQ(idx_Q, idx_D0, coord_HB);
            ElementAcc rK = mK(idx_K, idx_D0, coord_HB);
            acc_qk += rQ * rK;
          }  // for idx_D0

          for (int idx_D1 = 0; idx_D1 < D_VO; idx_D1++) {
            ElementAcc rDO = mDO(idx_Q, idx_D1, coord_HB);
            ElementAcc rV = mV(idx_K, idx_D1, coord_HB);
            ElementAcc rO = mO(idx_Q, idx_D1, coord_HB);
            acc_dov += rDO * rV;
            acc_doo += rDO * rO ;
          }
          auto id = make_identity_tensor(make_shape(1, 1));
          auto frag = make_tensor<ElementAcc>(Shape<_1, _1>{});
          frag(0) = acc_qk;
          fusion.apply_mask(frag, make_tensor(id.data() + make_arithmetic_tuple(idx_Q, idx_K), id.layout()), problem_shape);
          acc_qk = frag(0);

          mS[idx_Q] = static_cast<Element>(expf(softmax_scale * acc_qk - mLSE(idx_Q, coord_HB)) * softmax_scale * (acc_dov - acc_doo));
        }  // for idx_Q

        __syncthreads();
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
        int idx_D = threadIdx.x;
        if (idx_D < D) {
          for (int idx_Q = 0; idx_Q < Q; idx_Q++) {
            ElementAcc rQ = mQ(idx_Q, idx_D, coord_HB);
            ElementAcc rDS = mS[idx_Q];
            acc_dk += rDS * rQ;
          }
        }
        __syncthreads();
      } // for idx_H_R

      int idx_D = threadIdx.x;
      if (idx_D < D) {
        auto coord_HB = make_coord(make_coord(0, idx_H_K), idx_B);
        mDK(idx_K, idx_D, coord_HB) = static_cast<typename TensorDK::value_type>(acc_dk);
      }
    }  // for idx_K
  }  // for idx_HB
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
    class ProblemShape,
    class TensorQ, class TensorK, class TensorV,
    class TensorO, class TensorLSE, class TensorDO,
    /* class TensorDQ, class TensorDK, */ class TensorDV,
    class Fusion
>
void __global__ fmha_bwd_reference_dV_kernel(
    ProblemShape problem_shape_in,
    TensorQ mQ_in, TensorK mK_in, TensorV mV_in,
    TensorO mO_in, TensorLSE mLSE_in, TensorDO mDO_in,
    /* TensorDQ mDQ_in, TensorDK mDK_in, */ TensorDV mDV_in,
    Fusion fusion) {

  using namespace cute;
  using namespace cutlass::fmha::collective;

  using Element = typename TensorO::value_type;
  using ElementAcc = typename TensorLSE::value_type;
  
  extern __shared__ char mS_mem[];
  Element* mS = reinterpret_cast<Element*>(mS_mem);

  ElementAcc softmax_scale = 1.0f / sqrtf(size<2>(problem_shape_in));

  auto [H, B] = get<4>(problem_shape_in);
  auto [H_R, H_K] = H;

  for (int idx_HB = blockIdx.y; idx_HB < H_K * B; idx_HB += gridDim.y) {
    auto [idx_H_K, idx_B] = idx2crd(idx_HB, make_shape(H_K, B));
    auto [problem_shape, offset] = apply_variable_length_offset(
      problem_shape_in,
        make_coord(_0{}, _0{}, _0{}, _0{}, make_coord(make_coord(_0{}, idx_H_K), idx_B))
    );
    auto [Q, K, D, D_VO, HB] = problem_shape;
    auto [offset_Q, offset_K, offset_D, offset_D_VO, offset_HB] = offset;
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    auto mQ = domain_offset(make_coord(offset_Q, offset_D, offset_HB), mQ_in);
    auto mK = domain_offset(make_coord(offset_K, offset_D, offset_HB), mK_in);
    auto mV = domain_offset(make_coord(offset_K, offset_D_VO, offset_HB), mV_in);
    auto mO = domain_offset(make_coord(offset_Q, offset_D_VO, offset_HB), mO_in);
    auto mLSE = domain_offset(make_coord(offset_Q, offset_HB), mLSE_in);
    auto mDO = domain_offset(make_coord(offset_Q, offset_D_VO, offset_HB), mDO_in);
    auto mDV = domain_offset(make_coord(offset_K, offset_D_VO, offset_HB), mDV_in);

    for (int idx_K = blockIdx.x; idx_K < K; idx_K += gridDim.x) {
      ElementAcc acc_dv = 0;
      for (int idx_H_R = 0; idx_H_R < H_R; idx_H_R++) {
        auto coord_HB = make_coord(make_coord(idx_H_R, idx_H_K), idx_B);
        for (int idx_Q = threadIdx.x; idx_Q < Q; idx_Q += blockDim.x) {
          ElementAcc acc_qk = 0;

          for (int idx_D0 = 0; idx_D0 < D; idx_D0++) {
            ElementAcc rQ = mQ(idx_Q, idx_D0, coord_HB);
            ElementAcc rK = mK(idx_K, idx_D0, coord_HB);
            acc_qk += rQ * rK;
          }  // for idx_D0

          auto id = make_identity_tensor(make_shape(1, 1));
          auto frag = make_tensor<ElementAcc>(Shape<_1, _1>{});
          frag(0) = acc_qk;
          fusion.apply_mask(frag, make_tensor(id.data() + make_arithmetic_tuple(idx_Q, idx_K), id.layout()), problem_shape);
          acc_qk = frag(0);

          mS[idx_Q] = static_cast<Element>(expf(softmax_scale * acc_qk - mLSE(idx_Q, coord_HB)));
        }  // for idx_Q

        __syncthreads();

        int idx_D_VO = threadIdx.x;
        if (idx_D_VO < D_VO) {
          for (int idx_Q = 0; idx_Q < Q; idx_Q++) {
            ElementAcc rDO = mDO(idx_Q, idx_D_VO, coord_HB);
            ElementAcc rP = mS[idx_Q];
            acc_dv +=  rP * rDO;
          }
        }  // for idx_D
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Relies on tma-style bulk movement or descriptor handling. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。依赖 TMA 风格的批量搬运或描述符处理。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
        __syncthreads();
      } // for idx_H_R

      int idx_D_VO = threadIdx.x;
      if (idx_D_VO < D_VO) {
        auto coord_HB = make_coord(make_coord(0, idx_H_K), idx_B);
        mDV(idx_K, idx_D_VO, coord_HB) = static_cast<typename TensorDV::value_type>(acc_dv);
      }
    }  // for idx_K
  }  // for idx_L
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
    class ProblemShape,
    class TensorQ, class TensorK, class TensorV,
    class TensorO, class TensorLSE, class TensorDO,
    /**/ class TensorDQ, /** / class TensorDK, / ** / class TensorDV, / **/
    class Fusion
>
void fmha_bwd_reference_dQ(
    ProblemShape problem_shape,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE, TensorDO mDO,
    /**/ TensorDQ mDQ, /** / TensorDK mDK, / ** / TensorDV mDV, / **/
    Fusion fusion) {

  using namespace cute;

  dim3 grid(size<0>(mDQ), size<2>(mDQ), 1);
  dim3 block(256);
  int shared_mem = size<0>(mK) * sizeof(typename TensorDQ::value_type);
  cudaError_t result;
  if (shared_mem >= (48 << 10)) {
    result = cudaFuncSetAttribute(
        &fmha_bwd_reference_dQ_kernel<ProblemShape, TensorQ, TensorK, TensorV,
        TensorO, TensorLSE, TensorDO, TensorDQ, Fusion>,
        cudaFuncAttributeMaxDynamicSharedMemorySize,
        shared_mem);
    if (cudaSuccess != result) {
      cudaGetLastError(); // Clear the error state
      throw std::runtime_error("Failed to allocate " +
                               std::to_string(shared_mem >> 10) + " KB dynamic smem for dQ tensor in ref. check - " +
			       "please try reducing seq_len or skipping ref. check");
    }
  }
  fmha_bwd_reference_dQ_kernel<<<grid, block, shared_mem>>>(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDQ, fusion);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
    class ProblemShape,
    class TensorQ, class TensorK, class TensorV,
    class TensorO, class TensorLSE, class TensorDO,
    /** / class TensorDQ, / **/ class TensorDK, /** / class TensorDV, / **/
    class Fusion
>
void fmha_bwd_reference_dK(
    ProblemShape problem_shape,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE, TensorDO mDO,
    /** / TensorDQ mDQ, / **/ TensorDK mDK, /** / TensorDV mDV, / **/
    Fusion fusion) {

  using namespace cute;

  auto [K, D, HB] = mDK.shape();
  auto [H, B] = HB;
  auto [H_R, H_K] = H;
  dim3 grid(K, H_K * B, 1);
  dim3 block(std::max(D, 256));
  int shared_mem = size<0>(mDO) * sizeof(typename TensorDK::value_type);
  cudaError_t result;
  if (shared_mem >= (48 << 10)) {
    result = cudaFuncSetAttribute(
        &fmha_bwd_reference_dK_kernel<ProblemShape, TensorQ, TensorK, TensorV, 
	TensorO, TensorLSE, TensorDO, TensorDK, Fusion>,
        cudaFuncAttributeMaxDynamicSharedMemorySize,
        shared_mem);
    if (cudaSuccess != result) {
      cudaGetLastError(); // Clear the error state
      throw std::runtime_error("Failed to allocate " +
                               std::to_string(shared_mem >> 10) + " KB dynamic smem for dO tensor in ref. check - " +
                               "please try reducing seq_len or skipping ref. check");
    }
  }  
  fmha_bwd_reference_dK_kernel<<<grid, block, shared_mem>>>(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDK, fusion);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
    class ProblemShape,
    class TensorQ, class TensorK, class TensorV,
    class TensorO, class TensorLSE, class TensorDO,
    /** / class TensorDQ, / ** / class TensorDK, / **/ class TensorDV, /**/
    class Fusion
>
void fmha_bwd_reference_dV(
    ProblemShape problem_shape,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE, TensorDO mDO,
    /** / TensorDQ mDQ, / ** / TensorDK mDK, / **/ TensorDV mDV, /**/
    Fusion fusion) {

  using namespace cute;

  auto [K, D_VO, HB] = mDV.shape();
  auto [H, B] = HB;
  auto [H_R, H_K] = H;
  dim3 grid(K, H_K * B, 1);
  dim3 block(std::max(D_VO, 256));
  int shared_mem = size<0>(mDO) * sizeof(typename TensorDV::value_type);
  cudaError_t result;
  if (shared_mem >= (48 << 10)) {
    result = cudaFuncSetAttribute(
        &fmha_bwd_reference_dV_kernel<ProblemShape, TensorQ, TensorK, TensorV,
        TensorO, TensorLSE, TensorDO, TensorDV, Fusion>,
        cudaFuncAttributeMaxDynamicSharedMemorySize,
        shared_mem);
    if (cudaSuccess != result) {
      cudaGetLastError(); // Clear the error state
      throw std::runtime_error("Failed to allocate " +
                               std::to_string(shared_mem >> 10) + " KB dynamic smem for dO tensor in ref. check - " +
                               "please try reducing seq_len or skipping ref. check");
    }
  }  
  fmha_bwd_reference_dV_kernel<<<grid, block, shared_mem>>>(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDV, fusion);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
    class ProblemShape,
    class TensorQ, class TensorK, class TensorV,
    class TensorO, class TensorLSE, class TensorDO,
    class TensorDQ, class TensorDK, class TensorDV,
    class Fusion
>
void fmha_bwd_reference(
    ProblemShape problem_shape,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE, TensorDO mDO,
    TensorDQ mDQ, TensorDK mDK, TensorDV mDV,
    Fusion fusion) {
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  fmha_bwd_reference_dQ(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDQ, fusion);
  fmha_bwd_reference_dK(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDK, fusion);
  fmha_bwd_reference_dV(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDV, fusion);
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
