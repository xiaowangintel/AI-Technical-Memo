# epilogue_thread_apply_logsumexp.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/epilogue/epilogue_thread_apply_logsumexp.h`
**Purpose / 用途**: Thread-level log-sum-exp application for FMHA. It converts score fragments into exponentiated, max-shifted values by computing `exp(score - lse)` elementwise. / 该文件实现 FMHA 的线程级 log-sum-exp 应用逻辑，通过逐元素计算 `exp(score - lse)`，把分数 fragment 转成经过最大值平移后的指数形式。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-55 / 第 1-55 行
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
/*! \file
  \brief Functor performing linear combination operations used by epilogues.
*/

#pragma once

#include <cuda_fp16.h>

#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include "cutlass/epilogue/thread/activation.h"
#include "cutlass/functional.h"
#include "cutlass/numeric_conversion.h"
#include "cutlass/numeric_types.h"

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace epilogue {
namespace thread {

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace detail {
```
**EN**: License, includes, and namespace setup.
**CN**: 许可证、依赖头文件与命名空间准备。

### Lines 56-92 / 第 56-92 行
```cpp
template <typename Element, int ElementsPerAccess>
struct ArrayExponential {
  CUTLASS_HOST_DEVICE
  Array<Element, ElementsPerAccess> operator()(
      Array<Element, ElementsPerAccess> const& input) const {
    Array<Element, ElementsPerAccess> result;

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < ElementsPerAccess; ++i) {
      result[i] = expf(input[i]);
    }

    return result;
  }
};

template <int ElementsPerAccess>
struct ArrayExponential<half_t, ElementsPerAccess> {
  CUTLASS_DEVICE
  Array<half_t, ElementsPerAccess> operator()(
      Array<half_t, ElementsPerAccess> const& input) const {
    Array<half_t, ElementsPerAccess> result;

    int const kVectorCount = ElementsPerAccess / 2;

    __half2 const* input_ptr =
        reinterpret_cast<__half2 const*>(input.raw_data());
    __half2* res_ptr = reinterpret_cast<__half2*>(result.raw_data());

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kVectorCount; ++i) {
      res_ptr[i] = h2exp(input_ptr[i]);
    }

    return result;
  }
};
```
**EN**: `ArrayExponential` generic implementation and the `half_t` specialization using packed half operations.
**CN**: `ArrayExponential` 通用实现，以及基于打包 half 操作的 `half_t` 特化。

### Lines 93-166 / 第 93-166 行
```cpp
} // namespace detail

/////////////////////////////////////////////////////////////////////////////////////////////////

/// Applies:
/// output <- (input - lse).exp()
template <
    typename ElementOutput_, // output
    typename ElementLSE_, // accumulator from LSE
    typename ElementAccumulator_, // accumulator from matmul
    typename ElementCompute_, // intermediate compute (and exp calculation)
    int ElementsPerAccess>
class ApplyLogSumExp {
 public:
  using ElementOutput = ElementOutput_;
  using ElementAccumulator = ElementAccumulator_;
  using ElementCompute = ElementCompute_;
  using ElementLSE = ElementLSE_;

  static int const kElementsPerAccess = ElementsPerAccess;
  static int const kCount = kElementsPerAccess;
  static const ScaleType::Kind kScale =
      cutlass::epilogue::thread::ScaleType::NoBetaScaling;

  using FragmentOutput = Array<ElementOutput, kCount>;
  using FragmentAccumulator = Array<ElementAccumulator, kElementsPerAccess>;
  using FragmentCompute = Array<ElementCompute, kElementsPerAccess>;
  using FragmentLSE = Array<ElementLSE, kElementsPerAccess>;
  using FragmentScaleBias = FragmentLSE; // Used by epilogue_smem_accumulator.h

 public:
  //
  // Methods
  //

  CUTLASS_HOST_DEVICE
  ApplyLogSumExp() {}

  /// Returns true if source is needed
  CUTLASS_HOST_DEVICE
  bool is_source_needed() const {
    return true;
  }

  /// Functionally required for serial reduction in the epilogue
  CUTLASS_HOST_DEVICE
  void set_k_partition(int k_partition, int k_partition_count) {}

  CUTLASS_HOST_DEVICE
  FragmentOutput operator()(
      FragmentAccumulator const& AB,
      FragmentLSE const& scale_unused,
      // bias used as LSE
      FragmentLSE const& bias) const {
    FragmentCompute frag_AB = NumericArrayConverter<
        ElementCompute,
        ElementAccumulator,
        kElementsPerAccess>()(AB);
    FragmentCompute frag_lse_compute =
        NumericArrayConverter<ElementCompute, ElementLSE, kElementsPerAccess>()(
            bias);
    FragmentCompute frag_compute;

    minus<FragmentCompute> minus_lse;
    detail::ArrayExponential<ElementCompute, kElementsPerAccess> apply_exp;
    frag_compute = minus_lse(frag_AB, frag_lse_compute);
    frag_compute = apply_exp(frag_compute);

    return NumericArrayConverter<
        ElementOutput,
        ElementCompute,
        kElementsPerAccess>()(frag_compute);
  }
};
```
**EN**: `ApplyLogSumExp` declaration and operator implementation that converts accumulator/LSE fragments, computes `exp(AB - LSE)`, and converts back to the output type.
**CN**: `ApplyLogSumExp` 的声明与运算符实现：把累加器/LSE fragment 转成计算类型，计算 `exp(AB - LSE)`，再转回输出类型。

### Lines 167-174 / 第 167-174 行
```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace thread
} // namespace epilogue
} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Namespace closure.
**CN**: 命名空间收尾。

---
## Key Concepts / 关键概念
- **EN:** Log-sum-exp is the numerically stable form of softmax normalization. Instead of directly exponentiating raw scores, FMHA subtracts a row-wise reference first.
  **CN:** log-sum-exp 是 softmax 归一化的数值稳定写法。FMHA 不会直接对原始分数求指数，而是先减去按行参考值。
- **EN:** The functor works on small vector fragments because CUTLASS epilogues are naturally vectorized by iterator access width.
  **CN:** 该 functor 面向小型向量 fragment 工作，因为 CUTLASS epilogue 天然按 iterator 的访问宽度进行向量化。
- **EN:** The `half_t` specialization matters for FMHA throughput: exponentiation can remain packed in `__half2` lanes rather than scalarizing every element.
  **CN:** `half_t` 特化对 FMHA 吞吐很重要：指数运算可以继续保持在 `__half2` 向量 lane 中，而不用把每个元素都拆成标量。
- **EN:** This file provides the thread-level math that turns raw or partially reduced score fragments into softmax-ready probabilities inside the FMHA pipeline.
  **CN:** 该文件提供线程级数学逻辑，把原始或部分归约后的分数 fragment 转成 FMHA 流水中可用于 softmax 的概率值。
- **EN:** Conceptually it sits between MM0 score production and MM1 value aggregation, even when the exact integration point is wrapped by higher-level epilogue machinery.
  **CN:** 从概念上看，它位于 MM0 分数生成与 MM1 value 聚合之间，即便具体接入点被更高层的 epilogue 机制封装起来。
## Dependencies / 依赖项
- `<cuda_fp16.h>` — CUDA half-precision vector intrinsics / CUDA half 精度向量内建
- `"cutlass/array.h"` — fixed-size array containers used for fragments / 用于 fragment 的定长数组容器
- `"cutlass/cutlass.h"` — CUTLASS core definitions and portability macros / CUTLASS 核心定义与可移植性宏
- `"cutlass/epilogue/thread/activation.h"` — thread-level activation / math helpers reused by exp logic / 线程级激活 / 数学辅助逻辑，可复用于指数计算
- `"cutlass/functional.h"` — numeric functors and elementwise operator helpers / 数值 functor 与逐元素算子辅助
- `"cutlass/numeric_conversion.h"` — type-conversion helpers between accumulator and output fragments / 累加器 fragment 与输出 fragment 之间的类型转换辅助
- `"cutlass/numeric_types.h"` — CUTLASS scalar and numeric utility types / CUTLASS 标量与数值工具类型
