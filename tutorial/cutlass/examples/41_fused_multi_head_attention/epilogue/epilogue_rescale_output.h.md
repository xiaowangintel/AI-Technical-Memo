# epilogue_rescale_output.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/epilogue/epilogue_rescale_output.h`
**Purpose / 用途**: Implements the FMHA output-rescaling functor used by the epilogue. It merges the current MM1 accumulator with previously accumulated output using row-wise online-softmax coefficients. / 该文件实现 FMHA epilogue 使用的输出重标定算子，利用按行维护的在线 softmax 系数，把当前 MM1 累加结果与之前累积的输出融合起来。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-81 / 第 1-81 行
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
  \brief Epilogue for threadblock scoped GEMMs using Tensor Ops.

  The epilogue rearranges the result of a matrix product through shared memory
  to match canonical tensor layouts in global memory. Epilogues support
  conversion and reduction operations.

  This is a copy of cutlass/epilogue/threadblock/epilogue.h that can
  handle "row_id" as a first argument, as uses it to get the corresponding
  `m_prime` / `s_prime` to rescale the output.
*/

#pragma once
#include "cutlass/cutlass.h"
#include CUDA_STD_HEADER(cassert)
#include "cutlass/aligned_buffer.h"
#include "cutlass/array.h"
#include "cutlass/functional.h"
#include "cutlass/layout/tensor.h"
#include "cutlass/layout/vector.h"
#include "cutlass/numeric_types.h"
#include "cutlass/tensor_coord.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/transform/pitch_linear_thread_map.h"
#include "cutlass/transform/threadblock/regular_tile_iterator.h"
#include "cutlass/epilogue/threadblock/epilogue_base.h"
#include "cutlass/epilogue/threadblock/predicated_tile_iterator.h"
#include "cutlass/numeric_types.h"
#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include "cutlass/epilogue/thread/scale_type.h"
#include "cutlass/functional.h"
#include "cutlass/numeric_conversion.h"
#include "cutlass/numeric_types.h"
#include "epilogue_pipelined.h"

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace cutlass {
namespace epilogue {
namespace thread {

/////////////////////////////////////////////////////////////////////////////////////////////////

/// Applies a linear combination operator to an array of elements.
// output <- alpha * accumulator + beta * source
//   with:
//     alpha = 1 / s_prime (to normalize when isLast=True, 1 otherwise)
//     beta = alpha / m_prime (renormalize the output when the max changes)
//     source is the current output
```
**EN**: License, includes, namespaces, and comments that explain the FMHA-specific modification relative to stock CUTLASS epilogue output ops.
**CN**: 许可证、头文件、命名空间以及说明注释，解释它相对标准 CUTLASS 输出算子的 FMHA 定制点。

### Lines 82-177 / 第 82-177 行
```cpp
template <
    typename ElementOutput_, ///< Data type used to store tensors
    typename ElementSource_, //< Data type for source (usually matches
                             //`ElementOutput`)
    int Count, ///< Number of elements computed per operation.
               ///< Usually it is 128/sizeof_bits<ElementOutput_>,
               ///< but we use 64 or 32 sometimes when there are not enough data
               ///< to store
    typename ElementAccumulator_, ///< Accumulator data type
    typename ElementCompute_, ///< Data type used to compute linear combination
    bool isFirst,
    bool isLast,
    typename FragmentAlphaBeta_,
    FloatRoundStyle Round = FloatRoundStyle::round_to_nearest>
class MemoryEfficientAttentionNormalize {
 public:
  using ElementOutput = ElementOutput_;
  using ElementSource = ElementSource_;
  using ElementAccumulator = ElementAccumulator_;
  using ElementCompute = ElementCompute_;

  static int const kCount = Count;

  using FragmentOutput = Array<ElementOutput, kCount>;
  using FragmentSource = Array<ElementSource, kCount>;
  using FragmentAccumulator = Array<ElementAccumulator, kCount>;
  using ComputeFragment = Array<ElementCompute, kCount>;
  using FragmentAlphaBeta = FragmentAlphaBeta_;

  static FloatRoundStyle const kRound = Round;

 private:
  //
  // Data members
  //

  FragmentAlphaBeta const& s_prime_;
  FragmentAlphaBeta const& m_prime_;

 public:
  /// Constructs the function object, possibly loading from pointers in host
  /// memory
  CUTLASS_HOST_DEVICE
  MemoryEfficientAttentionNormalize(
      FragmentAlphaBeta const& s_prime,
      FragmentAlphaBeta const& m_prime)
      : s_prime_(s_prime), m_prime_(m_prime) {}

  /// Returns true if source is needed
  CUTLASS_HOST_DEVICE
  bool is_source_needed() const {
    return !isFirst;
  }

  /// Functionally required for serial reduction in the epilogue
  CUTLASS_HOST_DEVICE
  void set_k_partition(int k_partition, int k_partition_count) {}

  /// Computes linear scaling: D = alpha * accumulator + beta * source
  CUTLASS_HOST_DEVICE
  FragmentOutput operator()(
      int row,
      FragmentAccumulator const& accumulator,
      FragmentSource const& source) const {
    assert(!isFirst);

    // Convert source to interal compute numeric type
    NumericArrayConverter<ElementCompute, ElementSource, kCount, Round>
        source_converter;
    NumericArrayConverter<ElementCompute, ElementAccumulator, kCount, Round>
        accumulator_converter;

    // Convert to destination numeric type
    NumericArrayConverter<ElementOutput, ElementCompute, kCount, Round>
        destination_converter;

    ComputeFragment converted_source = source_converter(source);
    ComputeFragment converted_accumulator = accumulator_converter(accumulator);

    // Perform binary operations
    ComputeFragment intermediate;

    multiplies<ComputeFragment> mul_add_source;
    multiply_add<ComputeFragment> mul_add_accumulator;

    ElementCompute alpha = isLast ? (1 / s_prime_[row]) : 1;
    ElementCompute beta = alpha * m_prime_[row];

    intermediate = mul_add_source(beta, converted_source); // X =  beta * C

    intermediate = mul_add_accumulator(
        alpha, converted_accumulator, intermediate); // D = alpha * Accum + X

    return destination_converter(intermediate);
  }
```
**EN**: `MemoryEfficientAttentionNormalize` declaration and the source-needed operator: `D = alpha * Accum + beta * Source`, with `alpha`/`beta` derived from `s_prime` and `m_prime`.
**CN**: `MemoryEfficientAttentionNormalize` 声明以及需要 source 的运算符：`D = alpha * Accum + beta * Source`，其中 `alpha`/`beta` 来自 `s_prime` 与 `m_prime`。

### Lines 178-203 / 第 178-203 行
```cpp
  /// Computes linear scaling: D = alpha * accumulator
  CUTLASS_HOST_DEVICE
  FragmentOutput operator()(int row, FragmentAccumulator const& accumulator)
      const {
    assert(isFirst);

    // Convert source to interal compute numeric type
    NumericArrayConverter<ElementCompute, ElementAccumulator, kCount, Round>
        accumulator_converter;

    // Convert to destination numeric type
    NumericArrayConverter<ElementOutput, ElementCompute, kCount, Round>
        destination_converter;

    ComputeFragment converted_accumulator = accumulator_converter(accumulator);

    ComputeFragment intermediate;
    multiplies<ComputeFragment> mul_accumulator;

    ElementCompute alpha = isLast ? (1 / s_prime_[row]) : 1;

    intermediate = mul_accumulator(
        alpha, converted_accumulator); // X =  alpha * C + uniform

    return destination_converter(intermediate);
  }
```
**EN**: Source-free operator for the first partition: `D = alpha * Accum`.
**CN**: 面向首个分块的无 source 运算符：`D = alpha * Accum`。

### Lines 204-244 / 第 204-244 行
```cpp
};

} // namespace thread

namespace threadblock {
template <
    typename EO,
    typename ES,
    int Count,
    typename EA,
    typename EC,
    bool F,
    bool L,
    typename FAB,
    FloatRoundStyle R>
struct ApplyEpilogueOp<thread::MemoryEfficientAttentionNormalize<
    EO,
    ES,
    Count,
    EA,
    EC,
    F,
    L,
    FAB,
    R>> {
  using Op = thread::
      MemoryEfficientAttentionNormalize<EO, ES, Count, EA, EC, F, L, FAB, R>;
  static CUTLASS_DEVICE typename Op::FragmentOutput apply(
      Op const& output_op,
      int row_id,
      typename Op::FragmentAccumulator const& accum,
      typename Op::FragmentSource const& source) {
    return output_op(row_id, accum, source);
  }
  static CUTLASS_DEVICE typename Op::FragmentOutput apply(
      Op const& output_op,
      int row_id,
      typename Op::FragmentAccumulator const& accum) {
    return output_op(row_id, accum);
  }
};
```
**EN**: Threadblock-level `ApplyEpilogueOp` specialization that forwards the row id into the row-aware functor.
**CN**: 线程块级 `ApplyEpilogueOp` 特化，把逻辑行号传给按行感知的 functor。

### Lines 245-252 / 第 245-252 行
```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace threadblock
} // namespace epilogue
} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Namespace closure.
**CN**: 命名空间收尾。

---
## Key Concepts / 关键概念
- **EN:** Online softmax keeps two row-wise states: a running max (`m'`) and a running denominator (`s'`). This file converts those states into output-scale factors.
  **CN:** 在线 softmax 会维护两组按行状态：运行中的最大值 `m'` 与分母 `s'`。该文件把这些状态转换成输出重标定系数。
- **EN:** For non-first partitions, the old output must be rescaled by `beta = alpha * m_prime[row]` before the new accumulator is added.
  **CN:** 对于非首个分块，旧输出需要先乘以 `beta = alpha * m_prime[row]`，然后再与新的累加结果相加。
- **EN:** For the last partition, `alpha = 1 / s_prime[row]` finishes normalization; earlier partitions keep `alpha = 1` and only maintain an unnormalized running result.
  **CN:** 对最后一个分块，`alpha = 1 / s_prime[row]` 完成最终归一化；更早的分块保持 `alpha = 1`，只维护未归一化的运行结果。
- **EN:** The compile-time `isFirst` / `isLast` flags let the kernel prune unused code paths and avoid unnecessary source loads.
  **CN:** 编译期的 `isFirst` / `isLast` 标志可以裁剪无用代码路径，避免不必要的 source 读取。
- **EN:** `fmha_grouped.h` builds this operator inside MM1 epilogues so every partial `Attn @ V` result is merged consistently across K tiles.
  **CN:** `fmha_grouped.h` 会在 MM1 的 epilogue 中构造该算子，使每个分块的 `Attn @ V` 结果都能在跨 K tile 的过程中被一致地合并。
- **EN:** Together with `epilogue_pipelined.h`, this file turns CUTLASS’s generic epilogue infrastructure into an FMHA-aware normalization/writeback stage.
  **CN:** 该文件与 `epilogue_pipelined.h` 一起，把 CUTLASS 通用 epilogue 设施改造成 FMHA 感知的归一化与写回阶段。
## Dependencies / 依赖项
- `"cutlass/cutlass.h"` — CUTLASS core definitions and portability macros / CUTLASS 核心定义与可移植性宏
- `CUDA_STD_HEADER(cassert)` — assert support routed through CUTLASS CUDA portability macros / 通过 CUTLASS CUDA 可移植宏引入的断言支持
- `"cutlass/aligned_buffer.h"` — aligned shared-memory/storage buffers / 对齐的共享内存 / 存储缓冲区
- `"cutlass/array.h"` — fixed-size array containers used for fragments / 用于 fragment 的定长数组容器
- `"cutlass/functional.h"` — numeric functors and elementwise operator helpers / 数值 functor 与逐元素算子辅助
- `"cutlass/layout/tensor.h"` — tensor layout primitives used by epilogue iterators / epilogue iterator 使用的张量布局原语
- `"cutlass/layout/vector.h"` — vector layout helpers for epilogue fragments / epilogue fragment 使用的向量布局辅助
- `"cutlass/numeric_types.h"` — CUTLASS scalar and numeric utility types / CUTLASS 标量与数值工具类型
- `"cutlass/tensor_coord.h"` — tensor coordinate types for tile addressing / 用于 tile 寻址的张量坐标类型
- `"cutlass/gemm/gemm.h"` — CUTLASS GEMM shapes, coordinates, and core GEMM types / CUTLASS GEMM 形状、坐标与核心 GEMM 类型
- `"cutlass/transform/pitch_linear_thread_map.h"` — thread maps that distribute vector accesses across threads / 在线程间分配向量访问的 thread map
- `"cutlass/transform/threadblock/regular_tile_iterator.h"` — threadblock iterators for regular shared-memory tile access / 用于常规共享内存 tile 访问的 threadblock iterator
- `"cutlass/epilogue/threadblock/epilogue_base.h"` — base epilogue scaffolding for threadblock output pipelines / 线程块输出流水的 epilogue 基类框架
- `"cutlass/epilogue/threadblock/predicated_tile_iterator.h"` — predicated output iterators used during epilogue writeback / epilogue 写回阶段使用的 predicated 输出 iterator
- `"cutlass/epilogue/thread/scale_type.h"` — scale-type tags for epilogue output operators / epilogue 输出算子的缩放类型标签
- `"cutlass/numeric_conversion.h"` — type-conversion helpers between accumulator and output fragments / 累加器 fragment 与输出 fragment 之间的类型转换辅助
- `"epilogue_pipelined.h"` — the pipelined epilogue implementation extended by this row-aware output operator / 被本行感知输出算子扩展的 pipelined epilogue 实现
