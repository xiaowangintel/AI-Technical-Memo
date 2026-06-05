# debug_utils.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/debug_utils.h`
**Purpose / 用途**: Device-side debugging helpers for the CUTLASS FMHA example. The file adds finite-value checks, thread-gated printing, fragment/tensor dump macros, and a warp-accumulator printer that understands CUTLASS iterator layout. / 这是 CUTLASS FMHA 示例中的设备端调试辅助头文件，提供有限值检查、线程受控打印、fragment/tensor 转储宏，以及理解 CUTLASS iterator 布局的 warp 累加器打印函数。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-31 / 第 1-31 行
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
```
**EN**: License, `#pragma once`, and standard C/C++ headers needed for assertions, math, and `printf`.
**CN**: 许可证、`#pragma once` 以及断言、数学函数和 `printf` 所需的标准头文件。

### Lines 32-47 / 第 32-47 行
```cpp
#pragma once
#include <cfloat>
#include <cstdio>
#include <cmath>

////////////////////////////////////////////////////////////////////////////////
// Debugging functions
////////////////////////////////////////////////////////////////////////////////
// Nans & inf detection
#define NANCHECK(frag)                         \
  {                                            \
    for (size_t _i = 0; _i < frag.size(); ++_i) { \
      assert(std::isfinite(float(frag[_i])));  \
      assert(!std::isnan(float(frag[_i])));    \
    }                                          \
  }
```
**EN**: `NANCHECK` macro for catching invalid numeric state in fragments.
**CN**: `NANCHECK` 宏，用于捕获 fragment 中的非法数值状态。

### Lines 48-95 / 第 48-95 行
```cpp

// Print on the first thread of the first block
#if 1
#define PRINT_WARP_ID 0
#define PRINT_LANE_ID 0
#define PRINT_B0_T0(msg, ...)                                         \
  if (blockIdx.x == 0 && blockIdx.y == 0 && blockIdx.z == 0 &&        \
      threadIdx.x == PRINT_LANE_ID && threadIdx.y == PRINT_WARP_ID && \
      threadIdx.z == 0) {                                             \
    printf(msg "\n", ##__VA_ARGS__);                                  \
  }
#define PRINT_T0(msg, ...)                                            \
  if (threadIdx.x == PRINT_LANE_ID && threadIdx.y == PRINT_WARP_ID && \
      threadIdx.z == 0) {                                             \
    printf(msg "\n", ##__VA_ARGS__);                                  \
  }
#define PRINT_TX_LX(msg, ...)                                                 \
  for (int bx = 0; bx < gridDim.x; ++bx) {                                    \
    for (int by = 0; by < gridDim.y; ++by) {                                  \
      for (int bz = 0; bz < gridDim.z; ++bz) {                                \
        for (int tx = 0; tx < blockDim.x; ++tx) {                             \
          for (int ty = 0; ty < blockDim.y; ++ty) {                           \
            for (int tz = 0; tz < blockDim.z; ++tz) {                         \
              __syncthreads();                                                \
              if (blockIdx.x == bx && blockIdx.y == by && blockIdx.z == bz && \
                  threadIdx.x == tx && threadIdx.y == ty &&                   \
                  threadIdx.z == tz) {                                        \
                printf(                                                       \
                    "[%d,%d,%d][%d,%d,%d]" msg "\n",                          \
                    bx,                                                       \
                    by,                                                       \
                    bz,                                                       \
                    tx,                                                       \
                    ty,                                                       \
                    tz,                                                       \
                    ##__VA_ARGS__);                                           \
              }                                                               \
            }                                                                 \
          }                                                                   \
        }                                                                     \
      }                                                                       \
    }                                                                         \
  }
#else
#define PRINT_B0_T0
#define PRINT_TX_LX
#endif
```
**EN**: Print gating macros that restrict output to one block/thread or serialize the whole launch.
**CN**: 打印控制宏：要么限制到单个 block/thread，要么对整个 launch 做串行化打印。

### Lines 96-129 / 第 96-129 行
```cpp
struct __string_view {
  char const* data;
  std::size_t size;
};
#if __cplusplus >= 201402L
template <class T>
constexpr __string_view __get_type_name() {
  char const* p = __PRETTY_FUNCTION__;
  while (*p++ != '=')
    ;
  for (; *p == ' '; ++p)
    ;
  char const* p2 = p;
  int count = 1;
  for (;; ++p2) {
    switch (*p2) {
      case '[':
        ++count;
        break;
      case ']':
        --count;
        if (!count)
          return {p, std::size_t(p2 - p)};
    }
  }
  return {};
}
#else
template <class T>
constexpr __string_view __get_type_name() {
  return {"unsupported", 11};
}
#endif
```
**EN**: Tiny `__string_view` plus `__get_type_name<T>()` helper for labeling debug output with actual C++ types.
**CN**: `__string_view` 与 `__get_type_name<T>()` 辅助函数，用于在调试输出中标注真实 C++ 类型。

### Lines 130-193 / 第 130-193 行
```cpp
// Print a given array
#define PRINT_ACCUM8_T0_L0_START(name, accum, start)  \
  PRINT_B0_T0(                                        \
      "%s[%d:%d] - {%f, %f, %f, %f, %f, %f, %f, %f}", \
      name,                                           \
      int(start),                                     \
      int(start + 8),                                 \
      float(accum[start + 0]),                        \
      float(accum[start + 1]),                        \
      float(accum[start + 2]),                        \
      float(accum[start + 3]),                        \
      float(accum[start + 4]),                        \
      float(accum[start + 5]),                        \
      float(accum[start + 6]),                        \
      float(accum[start + 7]));
#define PRINT_ACCUM8_T0_L0(name, accum) PRINT_ACCUM8_T0_L0_START(name, accum, 0)
#define PRINT_FRAG_T0_L0(name, frag)                          \
  {                                                           \
    auto typeStr = __get_type_name<decltype(frag)>();         \
    PRINT_B0_T0("printing %s (%s)", name, typeStr.data);      \
    for (size_t _start = 0; _start < frag.size(); _start += 8) { \
      PRINT_ACCUM8_T0_L0_START("  ", frag, _start);           \
    }                                                         \
    /*__syncthreads();                                        \
    NANCHECK(frag); */                                        \
  }
#define PRINT_ARRAY_T0_L0_INCR(name, array, length, incr)   \
  {                                                         \
    PRINT_B0_T0("printing %s (len=%d)", name, int(length)); \
    for (int _start = 0; _start < length; _start += incr) { \
      PRINT_ACCUM8_T0_L0_START("  ", array, _start);        \
    }                                                       \
  }
#define PRINT_ARRAY_T0_L0(name, array, length) \
  PRINT_ARRAY_T0_L0_INCR(name, array, length, 8)

// Print a 4x4 matrix
#define PRINT_TENSOR4x4_T0_L0_START(name, ref, start_x, start_y)                                           \
  PRINT_B0_T0(                                                                                             \
      "%s[%d:%d, %d:%d]:\n    %f, %f, %f, %f\n    %f, %f, %f, %f\n    %f, %f, %f, %f\n    %f, %f, %f, %f", \
      name,                                                                                                \
      int(start_x),                                                                                        \
      int(start_x + 4),                                                                                    \
      int(start_y),                                                                                        \
      int(start_y + 4),                                                                                    \
      float(ref.at({start_x + 0, start_y + 0})),                                                           \
      float(ref.at({start_x + 0, start_y + 1})),                                                           \
      float(ref.at({start_x + 0, start_y + 2})),                                                           \
      float(ref.at({start_x + 0, start_y + 3})),                                                           \
      float(ref.at({start_x + 1, start_y + 0})),                                                           \
      float(ref.at({start_x + 1, start_y + 1})),                                                           \
      float(ref.at({start_x + 1, start_y + 2})),                                                           \
      float(ref.at({start_x + 1, start_y + 3})),                                                           \
      float(ref.at({start_x + 2, start_y + 0})),                                                           \
      float(ref.at({start_x + 2, start_y + 1})),                                                           \
      float(ref.at({start_x + 2, start_y + 2})),                                                           \
      float(ref.at({start_x + 2, start_y + 3})),                                                           \
      float(ref.at({start_x + 3, start_y + 0})),                                                           \
      float(ref.at({start_x + 3, start_y + 1})),                                                           \
      float(ref.at({start_x + 3, start_y + 2})),                                                           \
      float(ref.at({start_x + 3, start_y + 3})));
#define PRINT_TENSOR4x4_T0_L0(name, ref) \
  PRINT_TENSOR4x4_T0_L0_START(name, ref, 0, 0)
```
**EN**: Fragment/array/tensor pretty-printers for 1D slices and 4x4 views.
**CN**: fragment / 数组 / tensor 的格式化打印宏，覆盖 1D 切片与 4x4 视图。

### Lines 194-201 / 第 194-201 行
```cpp
#define PRINT_PROBLEM_SIZE(name, ps)            \
  PRINT_B0_T0(                                  \
      "%s.problem_size: {.m=%d, .n=%d, .k=%d}", \
      name,                                     \
      int(ps.m()),                              \
      int(ps.n()),                              \
      int(ps.k()))
```
**EN**: `PRINT_PROBLEM_SIZE` macro for reporting GEMM/FMHA problem coordinates.
**CN**: `PRINT_PROBLEM_SIZE` 宏，用于输出 GEMM / FMHA 问题尺寸。

### Lines 202-234 / 第 202-234 行
```cpp
template <typename LambdaIterator, typename LaneOffsetT, typename AccumT>
CUTLASS_DEVICE void print_warp_accum(
    AccumT accum,
    LaneOffsetT lane_offset,
    int32_t num_rows,
    int32_t num_cols) {
  bool is_main = blockIdx.x == 0 && blockIdx.y == 0 && blockIdx.z == 0 &&
      threadIdx.x == 0 && threadIdx.y == 0 && threadIdx.z == 0;
  for (int row = 0; row < num_rows; ++row) {
    for (int col = 0; col < num_cols; ++col) {
      if (col % 32 == 0) {
        if (is_main) {
          printf("\nmat[%3d, %3d:%3d]", row, col, col + 32);
        }
        __syncthreads();
      }
      LambdaIterator::iterateRows(
          lane_offset,
          [&](int accum_m) {},
          [&](int accum_m, int accum_n, int idx) {
            if (row == accum_m && col == accum_n &&
                (blockIdx.x == 0 && blockIdx.y == 0 && blockIdx.z == 0)) {
              printf(" %6.1f", float(accum[idx]));
            }
          },
          [&](int accum_m) {});
      __syncthreads();
    }
    if (is_main) {
      printf("\n");
    }
  }
}
```
**EN**: `print_warp_accum()` prints a logical matrix view of the warp accumulator by iterating rows and columns through a `LambdaIterator`.
**CN**: `print_warp_accum()` 通过 `LambdaIterator` 的行列遍历规则打印 warp 累加器的逻辑矩阵视图。

---
## Key Concepts / 关键概念
- **EN:** FMHA kernels keep intermediate score tiles and output fragments in registers/shared memory. These macros make it possible to inspect those states without rewriting the compute path.
  **CN:** FMHA 内核会把中间分数 tile 和输出 fragment 保存在寄存器或共享内存中，这些宏可以在不改写计算路径的前提下检查这些状态。
- **EN:** `PRINT_TX_LX` is intentionally expensive, but it is valuable when debugging grouped kernels where different blocks may be visiting different problems or query tiles.
  **CN:** `PRINT_TX_LX` 开销很大，但在调试 grouped kernel 时很有价值，因为不同 block 可能正在访问不同的问题实例或 query tile。
- **EN:** `print_warp_accum()` matters for CUTLASS because the accumulator layout is iterator-defined; logical `(m, n)` coordinates are not the same as raw fragment indices.
  **CN:** `print_warp_accum()` 对 CUTLASS 很重要，因为累加器布局由 iterator 定义；逻辑上的 `(m, n)` 坐标并不等同于原始 fragment 下标。
- **EN:** This file is not part of the performance-critical FMHA math path. It exists to validate score tiles from MM0, masked values, softmax-related intermediates, and MM1 output fragments during kernel bring-up.
  **CN:** 该文件不属于性能关键的 FMHA 数学路径，而是在内核联调时用于验证 MM0 产生的分数 tile、mask 后的值、softmax 相关中间量以及 MM1 输出 fragment。
- **EN:** It complements CUTLASS iterator/kernel composition by exposing what each iterator-driven stage is actually reading or writing.
  **CN:** 它通过可视化每个 iterator 驱动阶段实际读写的内容，帮助理解 CUTLASS 的 iterator / kernel 组合方式。
## Dependencies / 依赖项
- `<cfloat>` — standard floating-point limits support / 标准浮点极值支持
- `<cstdio>` — C stdio printing utilities used by debug macros / 调试宏使用的 C 标准输出工具
- `<cmath>` — math predicates such as finite / NaN checks / finite / NaN 检查等数学谓词
