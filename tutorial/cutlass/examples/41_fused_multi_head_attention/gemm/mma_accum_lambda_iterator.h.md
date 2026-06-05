# mma_accum_lambda_iterator.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/gemm/mma_accum_lambda_iterator.h`
**Purpose / 用途**: This header abstracts how a register-resident accumulator fragment is traversed as logical `(m, n)` matrix elements. It gives later code a uniform way to find each row/column touched by one lane, independent of whether the accumulator layout comes from Sm80 Tensor Cores, Volta Tensor Cores, or SIMT. The practical use is row-wise post-processing before spilling accumulators to shared memory, especially LSE-based normalization in fused attention. / 这个头文件抽象了“寄存器中的累加器 fragment 如何被遍历为逻辑 `(m, n)` 矩阵元素”。无论累加器布局来自 Sm80 Tensor Core、Volta Tensor Core 还是 SIMT，后续代码都能用统一接口找到某个 lane 对应的行列位置。 实际用途是把累加器写回 shared memory 之前进行按行后处理，尤其是融合注意力里基于 LSE 的归一化。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-44
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

#pragma once

#include "cutlass/functional.h"
#include "cutlass/gemm/warp/mma_simt_tile_iterator.h"
#include "cutlass/gemm/warp/mma_tensor_op_tile_iterator_sm70.h"
#include "cutlass/gemm/warp/mma_tensor_op_tile_iterator_sm80.h"
#include "cutlass/matrix_shape.h"

/*
TensorCores have different accumulator layouts.
This file provides a class to easily map the accumulator
i-th element with the corresponding matrix row/col.
*/
```
**EN**: Lines 1-44 contain the file prologue, includes, and the high-level comment explaining that Tensor Core accumulator layouts differ across architectures.
**CN**: 第 1-44 行是文件前言、依赖包含以及总体说明，强调不同架构下 Tensor Core 累加器布局并不相同。

### Lines 45-117
```cpp

template <typename T, typename accum_t, int kWarpSize>
struct AccumLambdaIteratorSm80 {
  static_assert(
      cutlass::platform::
          is_same<typename T::Layout, cutlass::layout::RowMajor>::value,
      "only RowMajor is supported");

  using Policy = typename T::Policy;
  using InstructionShape = typename T::InstructionShape;
  using OpDelta = typename T::OpDelta;
  using Shape = typename T::Shape;
  static int const kElementsPerAccess = InstructionShape::kN / 4;
  static int const kRowsPerTile = 8;
  static int const kAccumulatorRows = InstructionShape::kM / kRowsPerTile;

  static cutlass::MatrixCoord CUTLASS_DEVICE get_lane_offset(
      int8_t lane_id,
      int8_t warp_id,
      typename T::TensorCoord const& tile_offset) {
    int quad = (lane_id >> 2);
    int lane_in_quad = (lane_id & 3);
    return cutlass::MatrixCoord(
        quad + tile_offset.row() * Shape::kRow,
        lane_in_quad * kElementsPerAccess +
            tile_offset.column() * Shape::kColumn);
  }

  template <typename FA, typename FB, typename FC>
  CUTLASS_DEVICE static void iterateRows(
      cutlass::MatrixCoord& lane_offset,
      FA beginRow,
      FB op,
      FC endRow) {
    // See cutlass/gemm/warp/mma_tensor_op_tile_iterator.h
    CUTLASS_PRAGMA_UNROLL
    for (int mma_m = 0; mma_m < Policy::MmaIterations::kRow; ++mma_m) {
      CUTLASS_PRAGMA_UNROLL
      for (int row = 0; row < kAccumulatorRows; ++row) {
        int accum_m = mma_m * InstructionShape::kM * OpDelta::kRow +
            row * kRowsPerTile + lane_offset.row();
        beginRow(accum_m);

        CUTLASS_PRAGMA_UNROLL
        for (int mma_n = 0; mma_n < Policy::MmaIterations::kColumn; ++mma_n) {
          int mma_accum_start = kAccumulatorRows * kElementsPerAccess *
              (mma_n * Policy::MmaIterations::kRow + mma_m);
          CUTLASS_PRAGMA_UNROLL
          for (int col = 0; col < kElementsPerAccess; ++col) {
            int accum_n = mma_n * InstructionShape::kN * OpDelta::kColumn +
                col + lane_offset.column();
            int idx = mma_accum_start + row * kElementsPerAccess + col;
            op(accum_m, accum_n, idx);
          }
        }

        endRow(accum_m);
      }
    }
  }

  template <typename DT, typename F>
  CUTLASS_DEVICE static bool reduceSameRow(int lane_id, DT& myValue, F fn) {
    // In each warp, 4 threads will work on the same row
    // - the ones with the same `quad`
    auto otherV = __shfl_xor_sync(0xffffffff, myValue, 1);
    myValue = fn(myValue, otherV);
    otherV = __shfl_xor_sync(0xffffffff, myValue, 2);
    myValue = fn(myValue, otherV);
    int lane_in_quad = (lane_id & 3);
    return lane_in_quad == 0;
  }
};
```
**EN**: Lines 46-117 define `AccumLambdaIteratorSm80`, including lane-to-matrix mapping (`get_lane_offset`), fragment traversal (`iterateRows`), and row reduction within a quad (`reduceSameRow`).
**CN**: 第 46-117 行定义 `AccumLambdaIteratorSm80`，包括 lane 到矩阵坐标的映射（`get_lane_offset`）、fragment 遍历（`iterateRows`）以及 quad 内同一行的归约（`reduceSameRow`）。

### Lines 118-230
```cpp

template <typename T, typename accum_t, int kWarpSize>
struct AccumLambdaIteratorSm70 {
  static_assert(
      cutlass::platform::
          is_same<typename T::Layout, cutlass::layout::RowMajor>::value,
      "only RowMajor is supported");

  using Policy = typename T::Policy;
  using InstructionShape = typename T::InstructionShape;
  using OpDelta = typename T::OpDelta;
  using Shape = typename T::Shape;
  using Element = accum_t;

  static int const kElementsPerPartial = 4;
  using EleShapePerPatial = typename cutlass::platform::conditional<
      cutlass::platform::is_same<Element, float>::value,
      cutlass::MatrixShape<2, 2>,
      cutlass::MatrixShape<1, 4>>::type;
  static int const kElementsPerMma = 8;
  static int const kAccumulatorPatials = 2;
  using QuadShapePerPatialMma = cutlass::MatrixShape<4, 4>;

  static cutlass::MatrixCoord CUTLASS_DEVICE get_lane_offset(
      int8_t lane_id,
      int8_t warp_id,
      typename T::TensorCoord const& tile_offset) {
    int quad = (lane_id >> 2);
    int lane_in_quad = (lane_id & 3);
    int accum_m, accum_n;

    if (cutlass::platform::is_same<Element, float>::value) {
      // (quad[2],quad[0])+lane_in_quad[0]
      accum_m = (((quad & 0x4) >> 1) + (quad & 0x1)) * 8 + (lane_in_quad & 1);
      // (quad[1])+lane_in_quad[1]
      accum_n =
          ((quad >> 1) & 0x1) * kElementsPerPartial * kAccumulatorPatials +
          (lane_in_quad & 2);
    } else {
      accum_m = (((quad & 0x4) >> 1) + (quad & 0x1)) * 8 +
          lane_in_quad; // (quad[2],quad[0])
      accum_n = ((quad >> 1) & 0x1) * kElementsPerPartial * kAccumulatorPatials;
    }
    return cutlass::MatrixCoord(
        accum_m + tile_offset.row() * Shape::kRow,
        accum_n + tile_offset.column() * Shape::kColumn);
  }

  template <typename DT, typename F>
  CUTLASS_DEVICE static bool reduceSameRow(int lane_id, DT& myValue, F fn) {
    static_assert(
        cutlass::platform::is_same<Element, float>::value,
        "update to support non-float accum");
    // https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#warp-level-matrix-fragment-mma-884-f16
    // T0 & T2 share same line within a quad
    auto otherV = __shfl_xor_sync(0xffffffff, myValue, 1 << 1);
    myValue = fn(myValue, otherV);
    // quad 0 and quad 2 are on the same lines
    otherV = __shfl_xor_sync(0xffffffff, myValue, 1 << 3);
    myValue = fn(myValue, otherV);
    return (lane_id & ((1 << 1) | (1 << 3))) == 0;
  }

  template <typename FA, typename FB, typename FC>
  CUTLASS_DEVICE static void iterateRows(
      cutlass::MatrixCoord& lane_offset,
      FA beginRow,
      FB op,
      FC endRow) {
    CUTLASS_PRAGMA_UNROLL
    for (int tile_m = 0; tile_m < Policy::TileIterations::kRow; ++tile_m) {
      CUTLASS_PRAGMA_UNROLL
      for (int mma_m = 0; mma_m < Policy::MmaIterations::kRow; ++mma_m) {
        CUTLASS_PRAGMA_UNROLL
        for (int m = 0; m < EleShapePerPatial::kRow; ++m) {
          int accum_m = tile_m * Policy::InterleavedTile::kRow +
              mma_m * QuadShapePerPatialMma::kRow + m * 2 + lane_offset.row();
          beginRow(accum_m);

          CUTLASS_PRAGMA_UNROLL
          for (int tile_n = 0; tile_n < Policy::TileIterations::kColumn;
               ++tile_n) {
            CUTLASS_PRAGMA_UNROLL
            for (int mma_n = 0; mma_n < Policy::MmaIterations::kColumn;
                 ++mma_n) {
              CUTLASS_PRAGMA_UNROLL
              for (int p = 0; p < kAccumulatorPatials; ++p) {
                CUTLASS_PRAGMA_UNROLL
                for (int n = 0; n < EleShapePerPatial::kColumn; ++n) {
                  int mma_accum_start =
                      (((tile_n * Policy::TileIterations::kRow + tile_m) *
                            Policy::MmaIterations::kColumn +
                        mma_n) *
                           Policy::MmaIterations::kRow +
                       mma_m) *
                      kElementsPerMma;
                  int accum_n = tile_n * Policy::InterleavedTile::kColumn +
                      mma_n * QuadShapePerPatialMma::kColumn +
                      p * Policy::InterleavedTile::kColumn / 2 + n +
                      lane_offset.column();
                  int idx = mma_accum_start + p * kElementsPerPartial +
                      m * EleShapePerPatial::kColumn + n;
                  op(accum_m, accum_n, idx);
                }
              }
            }
          }
          endRow(accum_m);
        }
      }
    }
  }
};
```
**EN**: Lines 119-230 define `AccumLambdaIteratorSm70`, which reconstructs Volta’s accumulator layout and provides a different traversal order and reduction rule.
**CN**: 第 119-230 行定义 `AccumLambdaIteratorSm70`，重建 Volta 的累加器布局，并提供不同的遍历顺序与归约规则。

### Lines 231-304
```cpp

template <typename T, typename accum_t, int kWarpSize>
struct AccumLambdaIteratorSimt {
  using Policy = typename T::Policy;
  using Iterations = typename T::Iterations;
  using Element = typename T::Element;
  using Delta = typename T::Delta;
  using Shape = typename T::Shape;
  static_assert(
      cutlass::platform::
          is_same<typename T::Layout, cutlass::layout::RowMajor>::value,
      "only RowMajor is supported");

  template <typename DT, typename F>
  CUTLASS_DEVICE static bool reduceSameRow(int lane_id, DT& myValue, F fn) {
    CUTLASS_PRAGMA_UNROLL
    for (int bit = 1; bit < Policy::WarpShape::kColumn; bit *= 2) {
      auto otherV = __shfl_xor_sync(0xffffffff, myValue, bit);
      myValue = fn(myValue, otherV);
    }
    return (lane_id & (Policy::WarpShape::kColumn - 1)) == 0;
  }

  template <typename FA, typename FB, typename FC>
  CUTLASS_DEVICE static void iterateRows(
      cutlass::MatrixCoord& lane_offset,
      FA beginRow,
      FB op,
      FC endRow) {
    CUTLASS_PRAGMA_UNROLL
    for (int mma_m = 0; mma_m < Iterations::kRow; ++mma_m) {
      CUTLASS_PRAGMA_UNROLL
      for (int m = 0; m < Policy::LaneMmaShape::kM; ++m) {
        int accum_m = mma_m * Delta::kRow + m + lane_offset.row();
        beginRow(accum_m);

        CUTLASS_PRAGMA_UNROLL
        for (int mma_n = 0; mma_n < Iterations::kColumn; ++mma_n) {
          int accum_n =
              mma_n * Policy::WarpShape::kColumn * Policy::LaneMmaShape::kN +
              lane_offset.column();
          CUTLASS_PRAGMA_UNROLL
          for (int n = 0; n < Policy::LaneMmaShape::kN; ++n) {
            int idx = n +
                Policy::LaneMmaShape::kN *
                    (mma_n +
                     Iterations::kColumn *
                         (m + mma_m * Policy::LaneMmaShape::kM));
            op(accum_m, accum_n + n, idx);
          }
        }
        endRow(accum_m);
      }
    }
  }

  static cutlass::MatrixCoord CUTLASS_DEVICE get_lane_offset(
      int8_t lane_id,
      int8_t warp_id,
      typename T::TensorCoord const& tile_offset) {
    static_assert(
        cutlass::platform::is_same<
            typename Policy::LaneLayout,
            cutlass::layout::RowMajorInterleaved<1>>::value,
        "");
    typename Policy::LaneLayout lane_layout = Policy::get_lane_layout();

    cutlass::MatrixCoord lane_offset = lane_layout.inverse(lane_id) *
        cutlass::MatrixCoord(Policy::LaneMmaShape::kM,
                             Policy::LaneMmaShape::kN);
    return lane_offset +
        tile_offset * cutlass::MatrixCoord(Shape::kRow, Shape::kColumn);
  }
};
```
**EN**: Lines 232-304 define `AccumLambdaIteratorSimt`, using SIMT lane layout metadata instead of Tensor Core-specific swizzles.
**CN**: 第 232-304 行定义 `AccumLambdaIteratorSimt`，它不依赖 Tensor Core 特有 swizzle，而是使用 SIMT 的 lane layout 元数据。

### Lines 305-378
```cpp

template <typename T, typename accum_t, int kWarpSize>
struct DefaultMmaAccumLambdaIterator;

// Simt
template <typename S, typename P, typename accum_t, int kWarpSize>
struct DefaultMmaAccumLambdaIterator<
    cutlass::gemm::warp::MmaSimtTileIterator<
        S,
        cutlass::gemm::Operand::kC,
        accum_t,
        cutlass::layout::RowMajor,
        P,
        1,
        1>,
    accum_t,
    kWarpSize> {
  using WarpIterator = typename cutlass::gemm::warp::MmaSimtTileIterator<
      S,
      cutlass::gemm::Operand::kC,
      accum_t,
      cutlass::layout::RowMajor,
      P,
      1,
      1>;
  using Iterator = AccumLambdaIteratorSimt<WarpIterator, accum_t, kWarpSize>;
};

// TensorOp - Volta
template <typename S1, typename S2, typename accum_t, int kWarpSize>
struct DefaultMmaAccumLambdaIterator<
    cutlass::gemm::warp::MmaVoltaTensorOpAccumulatorTileIterator<
        S1,
        accum_t,
        cutlass::layout::RowMajor,
        S2,
        cutlass::MatrixShape<1, 1>>,
    accum_t,
    kWarpSize> {
  using WarpIterator =
      typename cutlass::gemm::warp::MmaVoltaTensorOpAccumulatorTileIterator<
          S1,
          accum_t,
          cutlass::layout::RowMajor,
          S2,
          cutlass::MatrixShape<1, 1>>;
  using Iterator = AccumLambdaIteratorSm70<WarpIterator, accum_t, kWarpSize>;
};

// TensorOp - Sm75+
template <
    typename S1,
    typename S2,
    typename S3,
    typename accum_t,
    int kWarpSize>
struct DefaultMmaAccumLambdaIterator<
    cutlass::gemm::warp::MmaTensorOpAccumulatorTileIterator<
        S1,
        accum_t,
        cutlass::layout::RowMajor,
        S2,
        S3>,
    accum_t,
    kWarpSize> {
  using WarpIterator =
      typename cutlass::gemm::warp::MmaTensorOpAccumulatorTileIterator<
          S1,
          accum_t,
          cutlass::layout::RowMajor,
          S2,
          S3>;
  using Iterator = AccumLambdaIteratorSm80<WarpIterator, accum_t, kWarpSize>;
};
```
**EN**: Lines 306-378 define `DefaultMmaAccumLambdaIterator` and its SIMT, Volta, and Sm75+ specializations, selecting the correct adapter for a given warp accumulator iterator type.
**CN**: 第 306-378 行定义 `DefaultMmaAccumLambdaIterator` 以及对应的 SIMT、Volta、Sm75+ 特化，为给定的 warp 累加器迭代器类型选择正确适配器。

---
## Key Concepts / 关键概念
- **EN**: `AccumLambdaIteratorSm80<T, accum_t, kWarpSize>` targets `MmaTensorOpAccumulatorTileIterator`-style layouts on Sm75+ and uses `T::Policy`, `T::InstructionShape`, `T::OpDelta`, and `T::Shape` to decode lane-local accumulator order.
  **CN**: `AccumLambdaIteratorSm80<T, accum_t, kWarpSize>` 面向 Sm75+ 上 `MmaTensorOpAccumulatorTileIterator` 风格的布局，使用 `T::Policy`、`T::InstructionShape`、`T::OpDelta`、`T::Shape` 解码 lane 本地的累加器顺序。
- **EN**: `AccumLambdaIteratorSm70<T, accum_t, kWarpSize>` specializes the mapping for Volta’s different accumulator swizzle and partial-fragment organization.
  **CN**: `AccumLambdaIteratorSm70<T, accum_t, kWarpSize>` 针对 Volta 不同的累加器 swizzle 与 partial-fragment 组织方式做专门映射。
- **EN**: `AccumLambdaIteratorSimt<T, accum_t, kWarpSize>` handles `MmaSimtTileIterator` layouts by using lane layout metadata from the iterator policy.
  **CN**: `AccumLambdaIteratorSimt<T, accum_t, kWarpSize>` 处理 `MmaSimtTileIterator` 布局，依赖迭代器策略中的 lane layout 元数据。
- **EN**: `DefaultMmaAccumLambdaIterator<WarpIterator, accum_t, kWarpSize>` is the compile-time selector that maps a concrete CUTLASS warp accumulator iterator type to one of the three iterator adapters above.
  **CN**: `DefaultMmaAccumLambdaIterator<WarpIterator, accum_t, kWarpSize>` 是编译期选择器，把具体的 CUTLASS warp 累加器迭代器类型映射到上面三种适配器之一。
- **EN**: The important aliases are `Policy`, `InstructionShape`, `OpDelta`, `Shape`, `Iterations`, and `Delta`. They encode tile shape, MMA step size, and lane-level arrangement.
  **CN**: 关键别名包括 `Policy`、`InstructionShape`、`OpDelta`、`Shape`、`Iterations`、`Delta`。它们共同描述 tile 形状、MMA 步长以及 lane 级布局。
- **EN**: Sm80 keeps constants such as `kElementsPerAccess`, `kRowsPerTile`, and `kAccumulatorRows`; Sm70 keeps `kElementsPerPartial`, `EleShapePerPatial`, `kElementsPerMma`, and `kAccumulatorPatials`; SIMT derives shape from `Policy::LaneMmaShape` and `Policy::WarpShape`.
  **CN**: Sm80 维护 `kElementsPerAccess`、`kRowsPerTile`、`kAccumulatorRows` 等常量；Sm70 维护 `kElementsPerPartial`、`EleShapePerPatial`、`kElementsPerMma`、`kAccumulatorPatials`；SIMT 则从 `Policy::LaneMmaShape` 与 `Policy::WarpShape` 推导形状。
- **EN**: There is almost no persistent runtime state. The API is entirely static; effective state comes from the template policy plus runtime inputs `lane_id`, `warp_id`, and `tile_offset`.
  **CN**: 这里几乎没有持久化的运行时状态。接口全部是静态函数；真正的“状态”来自模板策略以及运行时输入 `lane_id`、`warp_id` 和 `tile_offset`。
## Dependencies / 依赖项
- `cutlass/functional.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/gemm/warp/mma_simt_tile_iterator.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/gemm/warp/mma_tensor_op_tile_iterator_sm70.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/gemm/warp/mma_tensor_op_tile_iterator_sm80.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
- `cutlass/matrix_shape.h` — Direct include used by this file / 本文件直接包含并依赖的头文件
