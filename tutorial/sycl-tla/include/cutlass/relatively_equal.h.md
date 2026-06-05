# relatively_equal.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/relatively_equal.h`

- **EN:** Performs comparison between two elements with support for floating-point comparisons.

- **CN:** 该头文件属于 CUTLASS 的核心类型或工具定义。文件级摘要：Performs comparison between two elements with support for floating-point comparisons.

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

**EN:** This block records the file copyright, SPDX identifier, and redistribution disclaimer.

**CN:** 该代码块记录了文件的版权信息、SPDX 标识以及再分发免责声明。

### Lines 31-33

```cpp
/* \file
  \brief Performs comparison between two elements with support for floating-point comparisons.
*/
```

**EN:** This file-level comment explains the purpose of the header and introduces the abstractions defined below.

**CN:** 这个文件级注释说明了头文件的用途，并引出了后续定义的抽象。

### Lines 35-35

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 37-38

```cpp
#include "numeric_types.h"
#include "complex.h"
```

**EN:** This block imports dependencies such as `numeric_types.h`, `complex.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `numeric_types.h`, `complex.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 40-40

```cpp
namespace cutlass {
```

**EN:** This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 42-45

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
template <typename T, typename U = T>
CUTLASS_HOST_DEVICE
bool relatively_equal(T a, T b, U epsilon, U nonzero_floor);
```

**EN:** The preceding comment documents this block. The function `relatively_equal` implements a concrete operation in this abstraction. It packages a focused unit of behavior for the surrounding class, specialization, or namespace.

**CN:** 前面的注释说明了这个代码块。`relatively_equal` 函数实现了该抽象中的一个具体操作。它为周围的类、特化或命名空间封装了一个聚焦的行为单元。

### Lines 48-49

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace detail {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `detail` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `detail` 命名空间作用域，以容纳后续声明。

### Lines 52-79

```cpp
// This floating-point comparison function implements the method described in
//
// https://floating-point-gui.de/errors/comparison/
//
template <typename T>
CUTLASS_HOST_DEVICE
bool relatively_equal_float(T a, T b, T epsilon, T nonzero_floor) {

#if defined(__CUDACC_RTC__)
  using cuda::std::abs;
#else
  using std::abs;
#endif

  T abs_A = abs(a);
  T abs_B = abs(b);
  T diff = abs(a - b);
  T zero = T(0);

  if (a == b) {
    return true;
  }
  else if (a == zero || b == zero || (abs_A + abs_B) < nonzero_floor) {
    return diff < epsilon * nonzero_floor;
  }
  
  return diff < epsilon * (abs_A + abs_B);
}
```

**EN:** The preceding comment documents this block. The function `abs_A` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`abs_A` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 83-88

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<bool>(bool a, bool b, bool, bool) {
  return (a == b);
}
```

**EN:** The preceding comment documents this block. The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 91-95

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<uint1b_t>(uint1b_t a, uint1b_t b, uint1b_t, uint1b_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 97-101

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<int2b_t>(int2b_t a, int2b_t b, int2b_t, int2b_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 103-107

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<uint2b_t>(uint2b_t a, uint2b_t b, uint2b_t, uint2b_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 109-113

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<int4b_t>(int4b_t a, int4b_t b, int4b_t, int4b_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 115-119

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<uint4b_t>(uint4b_t a, uint4b_t b, uint4b_t, uint4b_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 121-125

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<int8_t>(int8_t a, int8_t b, int8_t, int8_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 127-131

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<uint8_t>(uint8_t a, uint8_t b, uint8_t, uint8_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 133-137

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<int16_t>(int16_t a, int16_t b, int16_t, int16_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 139-143

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<uint16_t>(uint16_t a, uint16_t b, uint16_t, uint16_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 145-149

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<int32_t>(int32_t a, int32_t b, int32_t, int32_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 151-155

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<uint32_t>(uint32_t a, uint32_t b, uint32_t, uint32_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 157-161

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<int64_t>(int64_t a, int64_t b, int64_t, int64_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 163-167

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<uint64_t>(uint64_t a, uint64_t b, uint64_t, uint64_t) {
  return (a == b);
}
```

**EN:** The function `a` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `a` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 169-174

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<float_e4m3_t>(float_e4m3_t a, float_e4m3_t b, float_e4m3_t epsilon, float_e4m3_t nonzero_floor) {
  return detail::relatively_equal_float<float>(a, b, epsilon, nonzero_floor);
}
```

**EN:** The preceding comment documents this block. The function `relatively_equal<float_e4m3_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`relatively_equal<float_e4m3_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 177-181

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<float_e5m2_t>(float_e5m2_t a, float_e5m2_t b, float_e5m2_t epsilon, float_e5m2_t nonzero_floor) {
  return detail::relatively_equal_float<float>(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<float_e5m2_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<float_e5m2_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 183-187

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<half_t>(half_t a, half_t b, half_t epsilon, half_t nonzero_floor) {
  return detail::relatively_equal_float(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<half_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<half_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 189-198

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<bfloat16_t>(
  bfloat16_t a, 
  bfloat16_t b, 
  bfloat16_t epsilon, 
  bfloat16_t nonzero_floor) {
  
  return detail::relatively_equal_float(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<bfloat16_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<bfloat16_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 200-209

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<tfloat32_t>(
  tfloat32_t a, 
  tfloat32_t b, 
  tfloat32_t epsilon, 
  tfloat32_t nonzero_floor) {
  
  return detail::relatively_equal_float(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<tfloat32_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<tfloat32_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 211-215

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<float>(float a, float b, float epsilon, float nonzero_floor) {
  return detail::relatively_equal_float(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<float>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<float>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 218-222

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<double>(double a, double b, double epsilon, double nonzero_floor) {
  return detail::relatively_equal_float(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<double>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<double>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 224-246

```cpp
template<typename T>
CUTLASS_HOST_DEVICE
bool relatively_equal(complex<T> a, complex<T> b, T epsilon, T nonzero_floor) {
#if defined(__CUDACC_RTC__)
  using cuda::std::abs;
#else
  using std::abs;
#endif

  T abs_A = abs(a);
  T abs_B = abs(b);
  T diff = abs(a - b);
  complex<T> zero = complex<T>{T{}, T{}};

  if (a == b) {
    return true;
  }
  else if (a == zero || b == zero || diff < nonzero_floor) {
    return diff < epsilon * nonzero_floor;
  }

  return diff < epsilon * (abs_A + abs_B);
}
```

**EN:** The function `abs_A` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `abs_A` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 248-271

```cpp
template <typename T>
CUTLASS_HOST_DEVICE 
bool relatively_equal(complex<T> a,  complex<T> b, complex<T> epsilon, complex<T> nonzero_floor) {
#if defined(__CUDACC_RTC__)
  using cuda::std::abs;
#else
  using std::abs;
#endif

  T abs_A = abs(a);
  T abs_B = abs(b);
  complex<T> diff = a - b;
  T abs_diff = abs(diff);
  complex<T> zero = complex<T>{T{}, T{}};

  if (a == b) {
    return true;
  }
  else if (a == zero || b == zero || abs_diff < abs(nonzero_floor)) {
    return abs_diff < abs(epsilon * nonzero_floor);
  }

  return abs_diff < abs(epsilon) * (abs_A + abs_B);
}
```

**EN:** The function `abs_A` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `abs_A` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 274-278

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<float_e2m3_t>(float_e2m3_t a, float_e2m3_t b, float_e2m3_t epsilon, float_e2m3_t nonzero_floor) {
  return detail::relatively_equal_float<float>(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<float_e2m3_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<float_e2m3_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 280-284

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<float_e3m2_t>(float_e3m2_t a, float_e3m2_t b, float_e3m2_t epsilon, float_e3m2_t nonzero_floor) {
  return detail::relatively_equal_float<float>(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<float_e3m2_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<float_e3m2_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 286-290

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<float_e2m1_t>(float_e2m1_t a, float_e2m1_t b, float_e2m1_t epsilon, float_e2m1_t nonzero_floor) {
  return detail::relatively_equal_float<float>(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<float_e2m1_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<float_e2m1_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 291-295

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<float_ue8m0_t>(float_ue8m0_t a, float_ue8m0_t b, float_ue8m0_t epsilon, float_ue8m0_t nonzero_floor) {
  return detail::relatively_equal_float<float>(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<float_ue8m0_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<float_ue8m0_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 297-301

```cpp
template <>
CUTLASS_HOST_DEVICE
bool relatively_equal<float_ue4m3_t>(float_ue4m3_t a, float_ue4m3_t b, float_ue4m3_t epsilon, float_ue4m3_t nonzero_floor) {
  return detail::relatively_equal_float<float>(a, b, epsilon, nonzero_floor);
}
```

**EN:** The function `relatively_equal<float_ue4m3_t>` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `relatively_equal<float_ue4m3_t>` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** The file handles specialized numeric formats or packed data representations used by accelerators.
  **CN:** 该文件处理加速器常用的特种数值格式或打包数据表示。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `numeric_types.h`, `complex.h`.
  **CN:** 直接包含：`numeric_types.h`, `complex.h`。

- **EN:** Primary namespaces: `cutlass`, `detail`.
  **CN:** 主要命名空间：`cutlass`, `detail`。

- **EN:** Important macros or compile flags: `CUTLASS_HOST_DEVICE`.
  **CN:** 重要宏或编译开关：`CUTLASS_HOST_DEVICE`。
