# sm90_visitor_topk_softmax.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/fusion/sm90_visitor_topk_softmax.hpp`

- **Purpose (EN):** Visitor tree Top-K + Softmax fusion operation for sm90 TMA warp-specialized epilogue.

- **作用 (CN):** 实现 `SM90 visitor topk Softmax` 访问者逻辑，用于遍历融合后的 epilogue 操作树。


## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Lines 32-34

```cpp
/*! \file
  \brief Visitor tree Top-K + Softmax fusion operation for sm90 TMA warp-specialized epilogue
*/
```

**EN:** This documentation block explains the intent of the next declaration: ! Visitor tree Top-K + Softmax fusion operation for sm90 TMA warp-specialized epilogue.

**CN:** 这一段文档注释说明了紧随其后的声明意图，帮助理解后续模板或实现要解决的问题。


### Line 36

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 38-39

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/workspace.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`, `cutlass/workspace.h`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`，`cutlass/workspace.h`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 41-42

```cpp
#include "cute/tensor.hpp"
#include "sm90_visitor_tma_warpspecialized.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cute/tensor.hpp`, `sm90_visitor_tma_warpspecialized.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cute/tensor.hpp`，`sm90_visitor_tma_warpspecialized.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 47

```cpp
namespace cutlass::epilogue::fusion {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 49-55

```cpp
namespace {
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
constexpr bool IS_NVIDIA_GPU = true;
#else
constexpr bool IS_NVIDIA_GPU = false;
#endif
}
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Line 69

```cpp
namespace detail {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 77-92

```cpp
CUTLASS_DEVICE
Array<float, 2> top_2_reduce_scalar(Array<float, 2> a, float scalar) {
  Array<float, 2> out;
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .f32 mx;\n"
      "  .reg .pred p;\n"
      "  max.f32 mx, %3, %4;\n"
      "  setp.gtu.f32 p, %2, %4;\n"
      "  selp.f32 %1, mx, %2, p;\n"
      "  selp.f32 %0, %2, %4, p;\n"
      "}\n" : "=f"(out[0]), "=f"(out[1]) : "f"(a[0]), "f"(a[1]), "f"(scalar));
#endif
  return out;
}
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 94-111

```cpp
CUTLASS_DEVICE
Array<float, 2> top_2_reduce(Array<float, 2> a, Array<float, 2> b) {
  Array<float, 2> out;
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .v2 .f32 mx;\n"
      "  .reg .pred p;\n"
      "  max.f32 mx.x, %3, %4;\n"           // max(a1, b0)
      "  max.f32 mx.y, %2, %5;\n"           // max(a0, b1)
      "  setp.gtu.f32 p, %2, %4;\n"         // a0 > b0
      "  selp.f32 %1, mx.x, mx.y, p;\n"     // a0 > b0 ? max(a1, b0) : max(a0, b1)
      "  selp.f32 %0, %2, %4, p;\n"         // a0 > b0 ? a0 : b0
      "}\n" : "=f"(out[0]), "=f"(out[1]) :
      "f"(a[0]), "f"(a[1]), "f"(b[0]), "f"(b[1]));
#endif
  return out;
}
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 113-138

```cpp
CUTLASS_DEVICE
Array<float, 4> top_4_reduce_scalar(Array<float, 4> a, float scalar) {
  Array<float, 4> out;
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .f32 mx;\n"                   // max(a3, b)
      "  .reg .pred p0;\n"                  // a0 > b
      "  .reg .pred p1;\n"                  // a1 > b
      "  .reg .pred p2;\n"                  // a2 > b
      "  max.f32 mx, %7, %8;\n"             // max(a3, b)
      "  setp.gtu.f32 p0, %4, %8;\n"        // a0 > b
      "  setp.gtu.f32 p1, %5, %8;\n"        // a1 > b
      "  setp.gtu.f32 p2, %6, %8;\n"        // a2 > b
      "  selp.f32 %3, mx, %6, p2;\n"        // a2 > b ? max(a3, b) : a2
      "  selp.f32 %2, %6, %8, p2;\n"        // a1 = a2 > b ? a2 : b
      "  selp.f32 %2, %2, %5, p1;\n"        // a1 > b ? max(a2, b) : a1 == a1 > b ? a1 : old_a1
      "  selp.f32 %1, %5, %8, p1;\n"        // a0 = a1 > b ? a1 : b
      "  selp.f32 %1, %1, %4, p0;\n"        // a0 > b ? max(a1, b) : a0 == a0 > b ? a0 : old_a0
      "  selp.f32 %0, %4, %8, p0;\n"        // a0 = a0 > b ? a0 : b
      "}\n" :
      "=f"(out[0]), "=f"(out[1]), "=f"(out[2]), "=f"(out[3]) :
      "f"(a[0]), "f"(a[1]), "f"(a[2]), "f"(a[3]), "f"(scalar));
#endif
  return out;
}
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 140-147

```cpp
CUTLASS_DEVICE
Array<float, 4> top_4_reduce(Array<float, 4> a, Array<float, 4> b) {
  Array<float, 4> out;
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .f32 mxa0b1;\n"                          // max(a0, b1)
      "  .reg .f32 mxa1b0;\n"                          // max(a1, b0)
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 149-151

```cpp
      "  .reg .f32 mxa2b0;\n"                          // max(a2, b0)
      "  .reg .f32 mxa1b1;\n"                          // max(a1, b1)
      "  .reg .f32 mxa0b2;\n"                          // max(a1, b1)
```

**EN:** This method block implements `max`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `max`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 153-156

```cpp
      "  .reg .f32 mxa1b2;\n"                          // max(a1, b2)
      "  .reg .f32 mxa2b1;\n"                          // max(a2, b1)
      "  max.f32 mxa1b2, %5, %10;\n"
      "  max.f32 mxa2b1, %6, %9;\n"
```

**EN:** This method block implements `max`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `max`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 158-161

```cpp
      "  .reg .f32 mxa3b0;\n"                          // max(a1, b2)
      "  .reg .f32 mxa0b3;\n"                          // max(a2, b1)
      "  max.f32 mxa3b0, %7, %8;\n"
      "  max.f32 mxa0b3, %4, %11;\n"
```

**EN:** This method block implements `max`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `max`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 163-170

```cpp
      "  .reg .pred pa0b0;\n"                          // a0 > b0
      "  .reg .pred pa1b0;\n"                          // a1 > b0
      "  .reg .pred pa2b0;\n"                          // a2 > b0
      "  .reg .pred pa0b1;\n"                          // a0 > b1
      "  .reg .pred pa1b1;\n"                          // a1 > b1
      "  .reg .pred pa0b2;\n"                          // a0 > b2
      "  .reg .pred pb2a0;\n"                          // b1 > a0
      "  .reg .pred pb1a0;\n"                          // b1 > a0
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 172-177

```cpp
      "  setp.gtu.f32 pa0b0, %4, %8;\n"                // a0 > b0
      "  setp.gtu.f32 pa1b0, %5, %8;\n"                // a1 > b0
      "  setp.gtu.f32 pa2b0, %6, %8;\n"                // a2 > b0
      "  setp.gtu.f32 pa0b1, %4, %9;\n"                // a0 > b1
      "  setp.gtu.f32 pa1b1, %5, %9;\n"                // a1 > b1
      "  setp.gtu.f32 pa0b2, %4, %10;\n"               // a0 > b2
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 179-180

```cpp
      "  not.pred pb2a0, pa0b2;\n"
      "  not.pred pb1a0, pa0b1;\n"
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 182-183

```cpp
      "  selp.f32 mxa1b0, %5, %8, pa1b0;\n"            // max(a1, b0)
      "  selp.f32 mxa0b1, %4, %9, pa0b1;\n"            // max(a0, b1)
```

**EN:** This method block implements `max`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `max`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 185-187

```cpp
      "  selp.f32 mxa1b1, %5, %9, pa1b1;\n"            // max(a1, b1)
      "  selp.f32 mxa2b0, %6, %8, pa2b0;\n"            // max(a2, b0)
      "  selp.f32 mxa0b2, %4, %10, pa0b2;\n"           // max(a0, b2)
```

**EN:** This method block implements `max`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `max`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 189-190

```cpp
      // a0
      "  selp.f32 %0, %4, %8, pa0b0;\n"                // a0 = a0 > b0 ? a0 : b0
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 192-193

```cpp
      // a1
      "  selp.f32 %1, mxa1b0, mxa0b1, pa0b0;\n"        // a1 = a0 > b0 ? max(a1, b0) : max(a0, b1)
```

**EN:** This method block implements `max`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `max`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 195-198

```cpp
      // a2
      "  mov.f32 %2, mxa1b1;\n"                        // a2 = max(a1, b1) ** most likely case
      "  selp.f32 %2, mxa2b0, %2, pa1b0;\n"            // a0 > a1 > b0
      "  selp.f32 %2, mxa0b2, %2, pb1a0;\n"            // b0 > b1 > a0
```

**EN:** This method block implements `max`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `max`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 200-211

```cpp
      // a3
      "  mov.f32 %3, mxa1b2;\n"                        // a3 = max(a1, b2) ** one of the most likely cases
      "  selp.f32 %3, mxa2b1, %3, pa1b1;\n"            // a3 = a1 > b1 ? max(a2, b1) ** second most likely case
      "  selp.f32 %3, mxa3b0, %3, pa2b0;\n"            // a0 > a1 > a2 > b0
      "  selp.f32 %3, mxa0b3, %3, pb2a0;\n"            // b0 > b1 > b2 > a0
      "}\n" :
      "=f"(out[0]), "=f"(out[1]), "=f"(out[2]), "=f"(out[3]) :
      "f"(a[0]), "f"(a[1]), "f"(a[2]), "f"(a[3]),
      "f"(b[0]), "f"(b[1]), "f"(b[2]), "f"(b[3]));
#endif
  return out;
}
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 213-239

```cpp
// Assumption: array elements are sorted in descending order
// (a[0] is the largest element in a[].)
template <typename Element, int N>
CUTLASS_DEVICE
void add_element_to_desc_sorted_array(cutlass::Array<Element, N>& a, Element b) {
  if constexpr (IS_NVIDIA_GPU && N == 2 && is_same_v<Element, float>) {
    a = top_2_reduce_scalar(a, b);
  }
  else if constexpr (IS_NVIDIA_GPU && N == 4 && is_same_v<Element, float>) {
    a = top_4_reduce_scalar(a, b);
  }
  else {
    // slower generic path with branching, slower, and can cause register spill
    CUTLASS_PRAGMA_UNROLL
    for (int k = 0; k < N; ++k) {
      if (a[k] < b) {
        // Shift down
        CUTLASS_PRAGMA_UNROLL
        for (int l = N - 1; l > k; --l) {
          a[l] = a[l-1];
        }
        a[k] = b;
        break;
      }
    }
  }
}
```

**EN:** This method block implements `add_element_to_desc_sorted_array`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `add_element_to_desc_sorted_array`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 241-268

```cpp
// Assumption: array elements are sorted in descending order
// (a[0] and b[0] are the largest elements in a[] and b[].)
template <typename Element, int N>
CUTLASS_DEVICE
void merge_desc_sorted_arrays(cutlass::Array<Element, N>& a, const cutlass::Array<Element, N>& b) {
  if constexpr (IS_NVIDIA_GPU && N == 2 && is_same_v<Element, float>) {
    a = top_2_reduce(a, b);
  }
  else if constexpr (IS_NVIDIA_GPU && N == 4 && is_same_v<Element, float>) {
    a = top_4_reduce(a, b);
  }
  else {
    // slower generic path with branching, slower, and can cause register spill
    int j = 0;
    CUTLASS_PRAGMA_UNROLL
    for (int k = 0; k < N; ++k) {
      if (a[k] < b[j]) {
        // Shift down
        CUTLASS_PRAGMA_UNROLL
        for (int l = N - 1; l > k; --l) {
          a[l] = a[l-1];
        }
        a[k] = b[j];
        ++j;
      }
    }
  }
}
```

**EN:** This method block implements `merge_desc_sorted_arrays`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `merge_desc_sorted_arrays`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 270-285

```cpp
// Assumption: array elements are sorted in descending order
// (a[0] is the largest element in a[].)
template <typename Element, int N>
CUTLASS_DEVICE
Element topk_logsumexp(cutlass::Array<Element, N> a) {
  // Do one less `exp`, because we know what its result will be.
  // Assume x is a set of `x_i`s, and `x_m` is the maximum of that set.
  // logsumexp(x) = log(sum(x_i)) = m + log(sum(x_i - m)) = m + log(1 + sum_{i != m}(x_i - x_m))
  // Compute m + log(1 + sum_{i != m}(x_i - x_m))
  Element sum = Element(1.0);
  CUTLASS_PRAGMA_UNROLL
  for (int i = 1; i < N; ++i) {
    sum += fast_exp(a[i] - a[0]);
  }
  return a[0] + fast_log(sum);
}
```

**EN:** This method block implements `topk_logsumexp`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `topk_logsumexp`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 287-295

```cpp
CUTLASS_DEVICE
float fast_masked_softmax(float value, float minimum, float logsumexp) {
  float new_value;
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .pred p0;\n"
      // value >= minimum
      "  setp.geu.f32 p0, %1, %2;\n"
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 297-299

```cpp
      "  .reg .f32 x_lse;\n"
      "  .reg .f32 %%f<11>;\n"
      "  .reg .b32 %%r<3>;\n"
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 301-302

```cpp
      // x_lse = value - minimum
      "  sub.rn.f32  x_lse, %1, %3;\n"
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 304-318

```cpp
      // exp(x_lse)
      // The following is derived from a ptx dump of expf.
      // exp requires a base conversion from exp2.
      "  fma.rn.f32 %%f1, x_lse, 0f3BBB989D, 0f3F000000;\n"
      "  cvt.sat.f32.f32 %%f2, %%f1;\n"
      "  fma.rm.f32 %%f3, %%f2, 0f437C0000, 0f4B400001;\n"
      "  add.f32 %%f4, %%f3, 0fCB40007F;\n"
      "  neg.f32 %%f5, %%f4;\n"
      "  fma.rn.f32 %%f6, x_lse, 0f3FB8AA3B, %%f5;\n"
      "  fma.rn.f32 %%f7, x_lse, 0f32A57060, %%f6;\n"
      "  mov.b32 %%r1, %%f3;\n"
      "  shl.b32 %%r2, %%r1, 23;\n"
      "  mov.b32 %%f8, %%r2;\n"
      "  ex2.approx.ftz.f32 %%f9, %%f7;\n"
      "  mul.f32 %%f10, %%f9, %%f8;\n"
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 320-325

```cpp
      // Mask or softmax
      "  selp.f32 %0, %%f10, 0f00000000, p0;\n"
      "}\n" : "=f"(new_value) : "f"(value), "f"(minimum), "f"(logsumexp));
#endif
  return new_value;
}
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 327-338

```cpp
template <typename Element>
CUTLASS_DEVICE
Element masked_softmax(Element value, Element minimum, Element logsumexp) {
  if constexpr (IS_NVIDIA_GPU && is_same_v<Element, float>) {
    // Inline PTX implementation
    // Significantly reduces register requirements
    return fast_masked_softmax(value, minimum, logsumexp);
  }
  else {
    return value < minimum ? Element(0.0) : fast_exp(value - logsumexp);
  }
}
```

**EN:** This method block implements `masked_softmax`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `masked_softmax`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 342-359

```cpp
template <
  int TopK,
  int FragmentSize,
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  FloatRoundStyle RoundStyle,
  int Alignment = 128 / sizeof_bits_v<ElementOutput>,
  bool UseButterflyReduce = true
>
struct Sm90TopKSoftmaxColReduction {
private:
  static_assert(is_same_v<ElementCompute, float>, "Fused Top-K + Softmax reduction requires FP32 accumulation.");
  static_assert(TopK == 2 || TopK == 4,
  "Fused Top-K + Softmax reduction only allows K=2 and K=4, because those cases have been performance-optimized. Other values of K can be enabled by removing this assertion, but they may come with serious performance implications."
  );
  static_assert(Alignment * sizeof_bits_v<ElementOutput> % 128 == 0, "sub-16B alignment not supported yet");
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 383-385

```cpp
  struct ReductionResult {
    ElementCompute min_;
    ElementCompute logsumexp_;
```

**EN:** Defines `ReductionResult`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `ReductionResult`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 387-388

```cpp
    CUTLASS_DEVICE
    ReductionResult() { }
```

**EN:** This method block implements `ReductionResult`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ReductionResult`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 390-392

```cpp
    CUTLASS_DEVICE
    ReductionResult(ElementCompute min, ElementCompute logsumexp):
      logsumexp_(logsumexp), min_(min) { }
```

**EN:** This method block implements `ReductionResult`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ReductionResult`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 394-402

```cpp
    // Warp shuffle broadcast
    CUTLASS_DEVICE
    void shuffle_up_sync(uint32_t delta, int lane_id) {
      static_assert(sizeof(ReductionResult) == sizeof(uint64_t));
      uint64_t r = reinterpret_cast<uint64_t&>(*this);
      r = shfl_up_sync(0xFFFFFFFF, r, delta);
      *this = (lane_id - static_cast<int>(delta) >= 0) ? reinterpret_cast<ReductionResult&>(r) : *this;
    }
  };
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 404-405

```cpp
  struct TopKResult {
    Array<ElementCompute, TopK> top_k_;
```

**EN:** Defines `TopKResult`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `TopKResult`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 407-410

```cpp
    CUTLASS_DEVICE
    TopKResult() {
      top_k_.fill(-cutlass::platform::numeric_limits<ElementCompute>::infinity());
    }
```

**EN:** This method block implements `TopKResult`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `TopKResult`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 412-418

```cpp
    // This is where we do the "final" reduction, where we compute
    // the logsumexp for softmax, keep the smallest value in top-K,
    // and discard the rest.
    CUTLASS_DEVICE
    ReductionResult reduce_final() const {
      return ReductionResult(top_k_[TopK - 1], topk_logsumexp(top_k_));
    }
```

**EN:** This method block implements `reduce_final`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `reduce_final`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 420-449

```cpp
    // Butterfly reduction
    CUTLASS_DEVICE
    void shuffle_xor_sync(int laneMask) {
      if constexpr (IS_NVIDIA_GPU && TopK == 2) {
        static_assert(sizeof(TopKResult) == sizeof(uint64_t));
        uint64_t top_k = reinterpret_cast<uint64_t&>(*this);
        top_k = shfl_xor_sync(0xFFFFFFFF, top_k, laneMask);
        auto synced_v = reinterpret_cast<TopKResult&>(top_k);
        detail::merge_desc_sorted_arrays(top_k_, synced_v.top_k_);
      }
      else if constexpr (IS_NVIDIA_GPU && TopK == 4) {
        static_assert(sizeof(TopKResult) == 2 * sizeof(uint64_t));
        uint64_t* top_k_ptr = reinterpret_cast<uint64_t*>(this);
        uint64_t top_k_arr[2];
        top_k_arr[0] = top_k_ptr[0];
        top_k_arr[1] = top_k_ptr[1];
        top_k_arr[0] = shfl_xor_sync(0xFFFFFFFF, top_k_arr[0], laneMask);
        top_k_arr[1] = shfl_xor_sync(0xFFFFFFFF, top_k_arr[1], laneMask);
        auto synced_v = reinterpret_cast<TopKResult&>(top_k_arr);
        detail::merge_desc_sorted_arrays(top_k_, synced_v.top_k_);
      }
      else {
        TopKResult synced_v;
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < TopK; ++i) {
          synced_v.top_k_[i] = shfl_xor_sync(0xFFFFFFFF, top_k_[i], laneMask);
        }
        detail::merge_desc_sorted_arrays(top_k_, synced_v.top_k_);
      }
    }
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 451-481

```cpp
    // Warp shuffle reduction
    CUTLASS_DEVICE
    void shuffle_down_sync(uint32_t delta) {
      if constexpr (IS_NVIDIA_GPU && TopK == 2) {
        static_assert(sizeof(TopKResult) == sizeof(uint64_t));
        uint64_t top_k = reinterpret_cast<uint64_t&>(*this);
        top_k = shfl_down_sync(0xFFFFFFFF, top_k, delta);
        auto synced_v = reinterpret_cast<TopKResult&>(top_k);
        detail::merge_desc_sorted_arrays(top_k_, synced_v.top_k_);
      }
      else if constexpr (IS_NVIDIA_GPU && TopK == 4) {
        static_assert(sizeof(TopKResult) == 2 * sizeof(uint64_t));
        uint64_t* top_k_ptr = reinterpret_cast<uint64_t*>(this);
        uint64_t top_k_arr[2];
        top_k_arr[0] = top_k_ptr[0];
        top_k_arr[1] = top_k_ptr[1];
        top_k_arr[0] = shfl_down_sync(0xFFFFFFFF, top_k_arr[0], delta);
        top_k_arr[1] = shfl_down_sync(0xFFFFFFFF, top_k_arr[1], delta);
        auto synced_v = reinterpret_cast<TopKResult&>(top_k_arr);
        detail::merge_desc_sorted_arrays(top_k_, synced_v.top_k_);
      }
      else {
        TopKResult synced_v;
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < TopK; ++i) {
          synced_v.top_k_[i] = shfl_down_sync(0xFFFFFFFF, top_k_[i], delta);
        }
        detail::merge_desc_sorted_arrays(top_k_, synced_v.top_k_);
      }
    }
  };
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 483-484

```cpp
public:
  struct SharedStorage { };
```

**EN:** This access-specifier block switches the following declarations into the `public` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `public` 区域。


### Line 486

```cpp
  struct Arguments { };
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Line 488

```cpp
  struct Params { };
```

**EN:** Defines `Params`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Params`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 490-494

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(ProblemShape const& problem_shape, Arguments const& args, void* workspace) {
    return {};
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 496-510

```cpp
  template <class ProblemShape>
  static bool
  can_implement(ProblemShape const& problem_shape, Arguments const& args) {
    auto [M, N, K, L] = problem_shape;
    auto [tile_M, tile_N, tile_K] = CtaTileShapeMNK{};
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    // Cross CTA reduction is not possible because there is no guarantee that all CTAs run
    // concurrently.
    // Cross epilogue tile reduction is possible, but re-visiting and applying reduction
    // to accumulators is only possible for the current epilogue tile.
    auto [epi_M, epi_N] = EpilogueTile{};
    return N <= tile_N && N <= epi_N && N >= TopK;
#endif
    return true;
  }
```

**EN:** This preprocessor block gates code on architecture, backend, or platform macros so only valid implementations participate in the build.

**CN:** 这个预处理块根据架构、后端或平台宏控制代码是否参与编译，只保留当前环境有效的实现。


### Lines 512-516

```cpp
  template <class ProblemShape>
  static size_t
  get_workspace_size(ProblemShape const& problem_shape, Arguments const& args) {
    return 0;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 518-523

```cpp
  template <class ProblemShape>
  static cutlass::Status
  initialize_workspace(ProblemShape const& problem_shape, Arguments const& args, void* workspace, cudaStream_t stream,
    CudaHostAdapter* cuda_adapter = nullptr) {
    return Status::kSuccess;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 525-528

```cpp
  CUTLASS_DEVICE bool
  is_producer_load_needed() const {
    return false;
  }
```

**EN:** This method block implements `is_producer_load_needed`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `is_producer_load_needed`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 530-533

```cpp
  CUTLASS_DEVICE bool
  is_C_load_needed() const {
    return false;
  }
```

**EN:** This method block implements `is_C_load_needed`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `is_C_load_needed`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 535-536

```cpp
  CUTLASS_HOST_DEVICE
  Sm90TopKSoftmaxColReduction() { }
```

**EN:** This method block implements `Sm90TopKSoftmaxColReduction`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `Sm90TopKSoftmaxColReduction`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 538-540

```cpp
  CUTLASS_HOST_DEVICE
  Sm90TopKSoftmaxColReduction(Params const& params, SharedStorage const& shared_storage)
      : params(params) { }
```

**EN:** This method block implements `Sm90TopKSoftmaxColReduction`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `Sm90TopKSoftmaxColReduction`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 542

```cpp
  Params params;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 544-548

```cpp
  template <class... Args>
  CUTLASS_DEVICE auto
  get_producer_load_callbacks(ProducerLoadArgs<Args...> const& args) {
    return EmptyProducerLoadCallbacks{};
  }
```

**EN:** This method block implements `get_producer_load_callbacks`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_producer_load_callbacks`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 550-555

```cpp
  template<class ArgsTuple>
  struct ConsumerStoreCallbacks : EmptyConsumerStoreCallbacks {
    CUTLASS_DEVICE
    ConsumerStoreCallbacks(ArgsTuple&& args_tuple, Params const& params)
      : args_tuple(cute::forward<ArgsTuple>(args_tuple)),
        params(params) {}
```

**EN:** Defines `ArgsTuple`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `ArgsTuple`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 557-558

```cpp
    ArgsTuple args_tuple;
    Params const& params;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 560-563

```cpp
    template <typename ElementAccumulator, typename ElementInput>
    CUTLASS_DEVICE auto
    visit(Array<ElementAccumulator, FragmentSize> const& frg_acc, int epi_v, int epi_m, int epi_n,
          Array<ElementInput, FragmentSize> const& frg_input) {
```

**EN:** This method block implements `visit`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `visit`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 565-568

```cpp
      auto& [tCrTopK, tCrSoftmax, tCcCol, cCol,
              lane_layout_MN, lane_mn,
              residue_cCol, residue_tCcCol] = args_tuple;
      Tensor tCcCol_mn = tCcCol(_,_,_,epi_m,epi_n);
```

**EN:** This method block implements `tCcCol`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `tCcCol`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 570-571

```cpp
      using ConvertInput = NumericArrayConverter<ElementCompute, ElementInput, FragmentSize, RoundStyle>;
      ConvertInput convert_input{};
```

**EN:** This block establishes type aliases or compile-time constants that control scaling, rounding, and fragment layout.

**CN:** 该代码块建立类型别名或编译期常量，用来控制缩放、舍入以及片段布局。


### Lines 573-581

```cpp
      Array frg_I = convert_input(frg_input);
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < FragmentSize; ++i) {
        auto thread_crd = tCcCol_mn(epi_v * FragmentSize + i);
        if (elem_less(thread_crd, residue_tCcCol)) {
          TopKResult& tCrCol_vmn = tCrTopK(epi_v * FragmentSize + i);
          detail::add_element_to_desc_sorted_array(tCrCol_vmn.top_k_, frg_I[i]);
        }
      }
```

**EN:** This method block implements `convert_input`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `convert_input`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 583-584

```cpp
      return frg_input;
    }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 586-588

```cpp
    template <class STensor, class SyncFn, class VTensor>
    CUTLASS_DEVICE void
    reduce(STensor&& smem_buffer, SyncFn const& sync_fn, int epi_m, int epi_n, bool is_last_iteration, VTensor visit_results) {
```

**EN:** Declares the templated `STensor` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `STensor`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 590-592

```cpp
      auto& [tCrTopK, tCrSoftmax, tCcCol, cCol,
              lane_layout_MN, lane_mn,
              residue_cCol, residue_tCcCol] = args_tuple;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 594-598

```cpp
      // fully OOB CTA in partially OOB cluster
      if (not elem_less(cCol(_0{},_0{}), residue_cCol)) {
        return;
      }
      Tensor tCcCol_mn = tCcCol(_,_,_,epi_m,epi_n);
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 600-605

```cpp
      // `tCrTopK` and `tCrSoftmax` have 0-strides along modes that correspond to N,
      // in order to reduce along modes in the `R2S` sublayout that correspond to N.
      // This means we should modify and warp-reduce them according to their co-domain instead of
      // their domain. Therefore we keep a filtered view of both and use them as necessary.
      auto tCrTopK_f = filter(tCrTopK);
      auto tCrSoftmax_f = filter(tCrSoftmax);
```

**EN:** This method block implements `filter`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `filter`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 607-627

```cpp
      // The pattern here is: reduce Top-K first, then compute logsumexp, keep it and the
      // last element of Top-K, use the latter to mask the visited results, and the former
      // to apply softmax.
      //
      // This gives us two options: reduce the Top-K with warp shuffles, have the reduced
      // lanes compute logsumexp and pair it with the last Top-K element, and broadcast
      // the result back using warp shuffles.
      //
      // Alternatively, we can do a butterfly reduction over Top-K, and have all lanes
      // compute their own logsumexp and skip the broadcast.
      if constexpr (UseButterflyReduce) {
        //
        // 1. Butterfly reduction
        //
        CUTLASS_PRAGMA_UNROLL
        for (int j = 1; j < size<1>(lane_layout_MN); j *= 2) {
          CUTLASS_PRAGMA_UNROLL
          for (int i = 0; i < size(tCrTopK_f); ++i) {
            tCrTopK_f(i).shuffle_xor_sync(j);
          }
        }
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 629-647

```cpp
        //
        // 2. Strip down reduced value and compute sum of exps
        //
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(tCrSoftmax_f); ++i) {
          tCrSoftmax_f(i) = tCrTopK_f(i).reduce_final();
        }
      }
      else {
        //
        // 1. Warp shuffle reduction
        //
        CUTLASS_PRAGMA_UNROLL
        for (int reduction_cols = size<1>(lane_layout_MN) / 2; reduction_cols > 0; reduction_cols /= 2) {
          CUTLASS_PRAGMA_UNROLL
          for (int i = 0; i < size(tCrTopK_f); ++i) {
            tCrTopK_f(i).shuffle_down_sync(lane_layout_MN(_0{},reduction_cols));
          }
        }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 649-658

```cpp
        //
        // 2. Strip down reduced value and compute sum of exps
        //
        bool is_reduced_lane = get<1>(lane_mn) == 0;
        if (is_reduced_lane) {
          CUTLASS_PRAGMA_UNROLL
          for (int i = 0; i < size(tCrSoftmax_f); ++i) {
            tCrSoftmax_f(i) = tCrTopK_f(i).reduce_final();
          }
        }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 660-670

```cpp
        //
        // 3. Broadcast reduced values to all participants
        //
        CUTLASS_PRAGMA_UNROLL
        for (int broadcast_cols = 1; broadcast_cols <= size<1>(lane_layout_MN) / 2; broadcast_cols *= 2) {
          CUTLASS_PRAGMA_UNROLL
          for (int i = 0; i < size(tCrSoftmax_f); ++i) {
            tCrSoftmax_f(i).shuffle_up_sync(lane_layout_MN(_0{},broadcast_cols), get<1>(lane_mn));
          }
        }
      }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 672-686

```cpp
      //
      // 4. Re-visit and apply top-K and softmax
      //
      CUTLASS_PRAGMA_UNROLL
      for (int epi_v = 0; epi_v < size(visit_results); ++epi_v) {
        auto& visit_frag = visit_results(epi_v);
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < FragmentSize; ++i) {
          visit_frag[i] = detail::masked_softmax(
            visit_frag[i],
            tCrSoftmax(epi_v * FragmentSize + i).min_,
            tCrSoftmax(epi_v * FragmentSize + i).logsumexp_
          );
        }
      }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 690-694

```cpp
    CUTLASS_DEVICE void
    end_loop(int epi_m, int epi_n) {
      auto& [tCrTopK, tCrSoftmax, tCcCol, cCol,
              lane_layout_MN, lane_mn,
              residue_cCol, residue_tCcCol] = args_tuple;
```

**EN:** This method block implements `end_loop`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `end_loop`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 696-700

```cpp
      // Reset reduced top-K values for next tile
      // This must be done because we only assume a single epilogue tile across N,
      // but not M.
      fill(tCrTopK, TopKResult());
    }
```

**EN:** This method block implements `fill`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `fill`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 702-703

```cpp
    CUTLASS_DEVICE void
    end() { }
```

**EN:** This method block implements `end`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `end`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 707-717

```cpp
  template <
    bool ReferenceSrc, // do register tensors reference the src or dst layout of the tiled copy
    class... Args
  >
  CUTLASS_DEVICE auto
  get_consumer_store_callbacks(ConsumerStoreArgs<Args...> const& args) {
    Layout ref_layout_MN = [&] () {
      auto mn_shape = shape(typename decltype(args.tiled_copy)::Tiler_MN{});
      if constexpr (ReferenceSrc) { return right_inverse(args.tiled_copy.get_layoutS_TV()).with_shape(mn_shape); }
      else                        { return right_inverse(args.tiled_copy.get_layoutD_TV()).with_shape(mn_shape); }
    }();                                                                                         // tile_mn -> tv_idx
```

**EN:** This method block implements `get_consumer_store_callbacks`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_consumer_store_callbacks`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 719-726

```cpp
    // Get the MN layout + coord of lanes to determine shuffle reduction iterations
    using _W = Int<decltype(args.tiled_copy)::TiledNumThr::value / NumThreadsPerWarp>;
    Layout tv2lane = Layout<Shape<Int<NumThreadsPerWarp>,_W,_1>,Stride<_1,_0,_0>>{};            //   tv_idx -> lane_idx
    Layout ref2lane = composition(tv2lane, ref_layout_MN);                                      //  tile_mn -> lane_idx
    Layout lane_layout_MN = make_layout(filter(get<0>(ref2lane)), filter(get<1>(ref2lane)));    //  lane_mn -> lane_idx
    Layout inv_lane_layout_MN = right_inverse(lane_layout_MN);                                  // lane_idx -> lane_mn
    int lane_idx = canonical_lane_idx();
    auto lane_mn = idx2crd(inv_lane_layout_MN(lane_idx), shape(lane_layout_MN));
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 728-731

```cpp
    // Get the MN layout + coord of warps to determine smem reduction iterations
    Layout tv2warp = Layout<Shape<Int<NumThreadsPerWarp>,_W,_1>,Stride<_0,_1,_0>>{};            //   tv_idx -> warp_idx
    Layout ref2warp = composition(tv2warp, ref_layout_MN);                                      //  tile_mn -> warp_idx
    Layout warp_layout_MN = make_layout(filter(get<0>(ref2warp)), filter(get<1>(ref2warp)));    //  warp_mn -> warp_idx
```

**EN:** This method block implements `composition`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `composition`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 733-734

```cpp
    // Make sure there's only one warp across N so we can use warp shuffle intrinsics for reduction.
    static_assert(decltype(size<1>(warp_layout_MN))::value <= 1);
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 751-757

```cpp
    // Create new accumulator layout with column broadcast
    auto [M, N, K] = args.tile_shape_mnk;
    auto thr_mma = args.tiled_mma.get_thread_slice(args.thread_idx);
    auto gColReduce = make_tensor<ElementCompute>(
        make_layout(make_shape(M, N), make_stride(_1{}, 0_c)));                                                // (M,N)
    auto tCrColReduce = make_tensor_like<ElementCompute>(                                       // (FrgV, MMA_M, MMA_N)
        thr_mma.partition_C(gColReduce).layout());
```

**EN:** This method block implements `get_thread_slice`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_thread_slice`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 759-762

```cpp
    // Tile the new accumulator tensor according to R2S
    ThrCopy thread_r2s = args.tiled_copy.get_slice(args.thread_idx);
    Tensor tRS_rSoftmax = thread_r2s.retile_S(tCrColReduce);                               // ((R2S,R2S_V),MMA_M,MMA_N)
    auto tCrC_layout = args.tCrC.layout();                                                         // (R2S,R2S_M,R2S_N)
```

**EN:** This method block implements `get_slice`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_slice`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 764-766

```cpp
    // Compose the new accumulator R2S layout with the expected tCrC layout to get final
    // reduction tensor layout.
    auto tCrSoftmax_layout = take<0, 3>(tRS_rSoftmax.layout()).compose(tCrC_layout); // (R2S,R2S_V) o (R2S,R2S_M,R2S_N)
```

**EN:** This method block implements `layout`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `layout`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 768-770

```cpp
    Tensor tCrTopK = make_tensor<TopKResult>(tCrSoftmax_layout);                                   // (R2S,R2S_M,R2S_N)
    Tensor tCrSoftmax = make_tensor<ReductionResult>(tCrSoftmax_layout);                           // (R2S,R2S_M,R2S_N)
    fill(tCrTopK, TopKResult());
```

**EN:** This method block implements `fill`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `fill`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 772-778

```cpp
    auto args_tuple = make_tuple(
        cute::move(tCrTopK), cute::move(tCrSoftmax), args.tCcD, args.cD,
        lane_layout_MN, lane_mn,
        args.residue_cD, args.residue_tCcD);
    return ConsumerStoreCallbacks<decltype(args_tuple)>(std::move(args_tuple), params);
  }
};
```

**EN:** This method block implements `make_tuple`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tuple`，为外围 epilogue 组件提供一个聚焦的行为片段。


## Key Concepts / 关键概念

- **Fusion framework / 融合框架:** Uses callbacks, visitors, or operation tags to compose multiple post-processing steps into the epilogue. / 通过回调、访问者或操作标签把多个后处理步骤组合进 epilogue。

- **Post-ops / 后处理算子:** Fuses nonlinear activation or other elementwise transforms directly into the epilogue. / 把非线性激活或其他逐元素变换直接融合进 epilogue。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cutlass/workspace.h`, `cute/tensor.hpp`, `sm90_visitor_tma_warpspecialized.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::fusion`

- **Cute/CuTe dependencies / Cute 依赖:** `cute/tensor.hpp`

- **Feature macros / 特性宏:** `CUTLASS_DEVICE`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`
