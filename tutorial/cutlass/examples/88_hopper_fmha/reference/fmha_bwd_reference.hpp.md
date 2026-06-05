# fmha_bwd_reference.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/88_hopper_fmha/reference/fmha_bwd_reference.hpp`  
**Purpose / 用途**: Implements simple reference kernels for backward FMHA (`dQ`, `dK`, `dV`) so the optimized Hopper kernels can be numerically checked. / 为反向 FMHA（`dQ`、`dK`、`dV`）实现简单参考 kernel，用于对优化后的 Hopper kernel 做数值校验。

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

**EN**: This comment block frames the file: it usually states the target architecture, problem family, usage pattern, or design motivation before the code starts. That context matters because later template choices are easier to understand once the intended hardware path and workload are known. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一段注释用于给文件定调：通常先说明目标架构、问题类型、使用方式或设计动机，再进入具体代码。先理解这些背景，再看后面的模板选择与硬件路径会更清晰。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#pragma once

#include "cute/tensor.hpp"
```

**EN**: This preprocessor block gates architecture-specific code or closes a specialized compilation path. It keeps the file buildable across toolchains and GPU targets even when the most optimized path only exists on a subset of architectures. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一预处理块负责为特定架构的代码设置编译门槛，或者结束某条专门化路径。这样即便最优路径只存在于部分 GPU 架构上，文件依旧可以在不同工具链与目标上保持可编译。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

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
    ProblemShape problem_shape,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE, TensorDO mDO,
    TensorDQ mDQ, /* TensorDK mDK, TensorDV mDV, */
    Fusion fusion
) {
  using namespace cute;

  using Element = typename TensorO::value_type;
  using ElementAccumulator = typename TensorLSE::value_type;
  
  extern __shared__ char mS_mem[];
  Element* mS = reinterpret_cast<Element*>(mS_mem);

  Element softmax_scale = static_cast<Element>(1.0 / sqrt(1.0 * size<1>(mO)));

  for (int idx_L = blockIdx.y; idx_L < size<2>(mDQ); idx_L += gridDim.y) {
    for (int idx_Q = blockIdx.x; idx_Q < size<0>(mDQ); idx_Q += gridDim.x) {

      for (int idx_K = threadIdx.x; idx_K < size<0>(mK); idx_K += blockDim.x) {
        ElementAccumulator acc_qk = 0;
        ElementAccumulator acc_dov = 0;
        ElementAccumulator acc_doo = 0;
        for (int idx_D0 = 0; idx_D0 < size<1>(mK); idx_D0++) {
          acc_qk += mQ(idx_Q, idx_D0, idx_L) * mK(idx_K, idx_D0, idx_L);
          acc_dov += mDO(idx_Q, idx_D0, idx_L) * mV(idx_K, idx_D0, idx_L);
          acc_doo += mDO(idx_Q, idx_D0, idx_L) * mO(idx_Q, idx_D0, idx_L);
        }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
        auto id = make_identity_tensor(make_shape(1, 1));
        auto frag = make_tensor<ElementAccumulator>(Shape<_1, _1>{});
        frag(0) = acc_qk;
        fusion.before_softmax(frag, make_tensor(id.data() + make_arithmetic_tuple(idx_Q, idx_K), id.layout()), problem_shape);
        acc_qk = frag(0);

        mS[idx_K] = static_cast<Element>(exp(softmax_scale * acc_qk - mLSE(idx_Q, idx_L)) * softmax_scale * (acc_dov - acc_doo));
      }

      __syncthreads();

      for (int idx_D = threadIdx.x; idx_D < size<1>(mDQ); idx_D += blockDim.x) {
        ElementAccumulator acc = 0;
        for (int idx_K = 0; idx_K < size<0>(mK); idx_K++) {
          acc += mS[idx_K] * mK(idx_K, idx_D, idx_L);
        }
        mDQ(idx_Q, idx_D, idx_L) = acc;
      }
    }
  }
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

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
    ProblemShape problem_shape,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE, TensorDO mDO,
    /* TensorDQ mDQ, */ TensorDK mDK, /* TensorDV mDV, */
    Fusion fusion
) {
  using namespace cute;

  using Element = typename TensorO::value_type;
  using ElementAccumulator = typename TensorLSE::value_type;
  
  extern __shared__ char mS_mem[];
  Element* mS = reinterpret_cast<Element*>(mS_mem);

  Element softmax_scale = static_cast<Element>(1.0 / sqrt(1.0 * size<1>(mO)));

  for (int idx_L = blockIdx.y; idx_L < size<2>(mDK); idx_L += gridDim.y) {
    for (int idx_K = blockIdx.x; idx_K < size<0>(mDK); idx_K += gridDim.x) {

      for (int idx_Q = threadIdx.x; idx_Q < size<0>(mDO); idx_Q += blockDim.x) {
        ElementAccumulator acc_qk = 0;
        ElementAccumulator acc_dov = 0;
        ElementAccumulator acc_doo = 0;
        for (int idx_D0 = 0; idx_D0 < size<1>(mK); idx_D0++) {
          acc_qk += mQ(idx_Q, idx_D0, idx_L) * mK(idx_K, idx_D0, idx_L);
          acc_dov += mDO(idx_Q, idx_D0, idx_L) * mV(idx_K, idx_D0, idx_L);
          acc_doo += mDO(idx_Q, idx_D0, idx_L) * mO(idx_Q, idx_D0, idx_L);
        }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
        auto id = make_identity_tensor(make_shape(1, 1));
        auto frag = make_tensor<ElementAccumulator>(Shape<_1, _1>{});
        frag(0) = acc_qk;
        fusion.before_softmax(frag, make_tensor(id.data() + make_arithmetic_tuple(idx_Q, idx_K), id.layout()), problem_shape);
        acc_qk = frag(0);

        mS[idx_Q] = static_cast<Element>(exp(softmax_scale * acc_qk - mLSE(idx_Q, idx_L)) * softmax_scale * (acc_dov - acc_doo));
      }

      __syncthreads();

      for (int idx_D = threadIdx.x; idx_D < size<1>(mDK); idx_D += blockDim.x) {
        ElementAccumulator acc = 0;
        for (int idx_Q = 0; idx_Q < size<0>(mDO); idx_Q++) {
          acc += mS[idx_Q] * mQ(idx_Q, idx_D, idx_L);
        }
        mDK(idx_K, idx_D, idx_L) = acc;
      }
    }
  }
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

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
    ProblemShape problem_shape,
    TensorQ mQ, TensorK mK, TensorV mV,
    TensorO mO, TensorLSE mLSE, TensorDO mDO,
    /* TensorDQ mDQ, TensorDK mDK, */ TensorDV mDV,
    Fusion fusion
) {
  using namespace cute;

  using Element = typename TensorO::value_type;
  using ElementAccumulator = typename TensorLSE::value_type;
  
  extern __shared__ char mS_mem[];
  Element* mS = reinterpret_cast<Element*>(mS_mem);

  Element softmax_scale = static_cast<Element>(1.0 / sqrt(1.0 * size<1>(mO)));

  for (int idx_L = blockIdx.y; idx_L < size<2>(mDV); idx_L += gridDim.y) {
    for (int idx_K = blockIdx.x; idx_K < size<0>(mDV); idx_K += gridDim.x) {

      for (int idx_Q = threadIdx.x; idx_Q < size<0>(mDO); idx_Q += blockDim.x) {
        ElementAccumulator acc_qk = 0;
        for (int idx_D0 = 0; idx_D0 < size<1>(mK); idx_D0++) {
          acc_qk += mQ(idx_Q, idx_D0, idx_L) * mK(idx_K, idx_D0, idx_L);
        }

        auto id = make_identity_tensor(make_shape(1, 1));
        auto frag = make_tensor<ElementAccumulator>(Shape<_1, _1>{});
        frag(0) = acc_qk;
        fusion.before_softmax(frag, make_tensor(id.data() + make_arithmetic_tuple(idx_Q, idx_K), id.layout()), problem_shape);
        acc_qk = frag(0);

        mS[idx_Q] = static_cast<Element>(exp(softmax_scale * acc_qk - mLSE(idx_Q, idx_L)));
      }
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
      __syncthreads();

      for (int idx_D = threadIdx.x; idx_D < size<1>(mDV); idx_D += blockDim.x) {
        ElementAccumulator acc = 0;
        for (int idx_Q = 0; idx_Q < size<0>(mDO); idx_Q++) {
          acc += mS[idx_Q] * mDO(idx_Q, idx_D, idx_L);
        }
        mDV(idx_K, idx_D, idx_L) = acc;
      }
    }
  }
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

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
    Fusion fusion
) {
  using namespace cute;

  dim3 grid(size<0>(mDQ), size<2>(mDQ), 1);
  dim3 block(256);
  int shared_mem = size<0>(mK) * sizeof(typename TensorO::value_type);

  if (shared_mem >= (48 << 10)) {
    CUTLASS_TRACE_HOST("  Setting smem size to " << shared_mem);
    auto result = cudaFuncSetAttribute(
        fmha_bwd_reference_dQ_kernel<ProblemShape, TensorQ, TensorK, TensorV, TensorO, TensorLSE, TensorDO, TensorDQ, Fusion>,
        cudaFuncAttributeMaxDynamicSharedMemorySize,
        shared_mem);
    if (cudaSuccess != result) {
      result = cudaGetLastError(); // to clear the error bit
      CUTLASS_TRACE_HOST(
        "  cudaFuncSetAttribute() returned error: "
        << cudaGetErrorString(result));
      return;
    }
  }

  fmha_bwd_reference_dQ_kernel<<<grid, block, shared_mem>>>(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDQ, fusion);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

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
    Fusion fusion
) {
  using namespace cute;

  dim3 grid(size<0>(mDK), size<2>(mDK), 1);
  dim3 block(256);
  int shared_mem = size<0>(mDO) * sizeof(typename TensorO::value_type);

  if (shared_mem >= (48 << 10)) {
    CUTLASS_TRACE_HOST("  Setting smem size to " << shared_mem);
    auto result = cudaFuncSetAttribute(
        fmha_bwd_reference_dK_kernel<ProblemShape, TensorQ, TensorK, TensorV, TensorO, TensorLSE, TensorDO, TensorDK, Fusion>,
        cudaFuncAttributeMaxDynamicSharedMemorySize,
        shared_mem);
    if (cudaSuccess != result) {
      result = cudaGetLastError(); // to clear the error bit
      CUTLASS_TRACE_HOST(
        "  cudaFuncSetAttribute() returned error: "
        << cudaGetErrorString(result));
      return;
    }
  }

  fmha_bwd_reference_dK_kernel<<<grid, block, shared_mem>>>(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDK, fusion);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

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
    Fusion fusion
) {
  using namespace cute;

  dim3 grid(size<0>(mDV), size<2>(mDV), 1);
  dim3 block(256);
  int shared_mem = size<0>(mDO) * sizeof(typename TensorO::value_type);

  if (shared_mem >= (48 << 10)) {
    CUTLASS_TRACE_HOST("  Setting smem size to " << shared_mem);
    auto result = cudaFuncSetAttribute(
        fmha_bwd_reference_dV_kernel<ProblemShape, TensorQ, TensorK, TensorV, TensorO, TensorLSE, TensorDO, TensorDV, Fusion>,
        cudaFuncAttributeMaxDynamicSharedMemorySize,
        shared_mem);
    if (cudaSuccess != result) {
      result = cudaGetLastError(); // to clear the error bit
      CUTLASS_TRACE_HOST(
        "  cudaFuncSetAttribute() returned error: "
        << cudaGetErrorString(result));
      return;
    }
  }

  fmha_bwd_reference_dV_kernel<<<grid, block, shared_mem>>>(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDV, fusion);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

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
    Fusion fusion
) {
  fmha_bwd_reference_dQ(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDQ, fusion);
  fmha_bwd_reference_dK(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDK, fusion);
  fmha_bwd_reference_dV(problem_shape, mQ, mK, mV, mO, mLSE, mDO, mDV, fusion);
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- TMA-driven data movement / 基于 TMA 的数据搬运
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Reference-based numerical validation / 基于参考实现的数值验证

## Dependencies / 依赖项

- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
