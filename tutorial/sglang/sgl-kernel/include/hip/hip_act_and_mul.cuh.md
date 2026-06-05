# hip_act_and_mul.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/hip/hip_act_and_mul.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Headers and compile-time setup
```cpp
/* Copyright 2025 SGLang Team. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
==============================================================================*/

#pragma once

#include "utils.h"
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 20-36: Namespace and shared declarations
```cpp
#define kBitsToLoad 128
#define kBytesToLoad (kBitsToLoad / 8)

// Adapted from
// [flashinfer::activation::act_and_mul_kernel](https://github.com/flashinfer-ai/flashinfer/blob/4e8eb1879f9c3ba6d75511e5893183bf8f289a62/include/flashinfer/activation.cuh#L29)

namespace sgl_hip {
namespace activation {

template <typename T, T (*Activation)(const T&)>
__global__ void act_and_mul_kernel(T* __restrict__ out, const T* __restrict__ input, const int d) {
  constexpr uint32_t vec_size = kBytesToLoad / sizeof(T);
  const int64_t token_idx = blockIdx.x;
  const int64_t thread_idx = threadIdx.x;
  const int64_t stride = blockDim.x;
  const int64_t offset = token_idx * 2 * d;
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 37-55: Control flow and branching
```cpp
#pragma unroll 1
  for (uint32_t idx = thread_idx; idx < d / vec_size; idx += stride) {
    sgl_hip::vec_t<T, vec_size> x_vec, y_vec, out_vec;
    x_vec.cast_load(input + offset + idx * vec_size);
    y_vec.cast_load(input + offset + d + idx * vec_size);
#pragma unroll
    for (uint32_t i = 0; i < vec_size; ++i) {
      out_vec[i] = Activation(x_vec[i]) * y_vec[i];
    }
    out_vec.cast_store(out + token_idx * d + idx * vec_size);
  }

  const int64_t remaining_offset = d - d % (stride * vec_size);
  // process the remaining elements
#pragma unroll 1
  for (int64_t idx = thread_idx; idx < d % (stride * vec_size); idx += stride) {
    T x = input[offset + remaining_offset + idx], y = input[offset + remaining_offset + d + idx];
    out[token_idx * d + remaining_offset + idx] = Activation(x) * y;
  }
```
**EN:** This section drives `cast_load`, `cast_store`, `remaining_offset` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`cast_load`、`cast_store`、`remaining_offset`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 56-73: Kernel implementation
```cpp
}

template <typename T, T (*Activation)(const T&)>
__global__ void act_only_kernel(T* __restrict__ out, const T* __restrict__ input, const int d) {
  constexpr uint32_t vec_size = kBytesToLoad / sizeof(T);
  const int64_t token_idx = blockIdx.x;
  const int64_t thread_idx = threadIdx.x;
  const int64_t stride = blockDim.x;
  const int64_t offset = token_idx * d;

#pragma unroll 1
  for (uint32_t idx = thread_idx; idx < d / vec_size; idx += stride) {
    sgl_hip::vec_t<T, vec_size> x_vec, y_vec, out_vec;
    x_vec.cast_load(input + offset + idx * vec_size);
#pragma unroll
    for (uint32_t i = 0; i < vec_size; ++i) {
      out_vec[i] = Activation(x_vec[i]);
    }
```
**EN:** This section implements `T`, `cast_load`, `Activation`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`T`、`cast_load`、`Activation`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 74-85: Control flow and branching
```cpp
    out_vec.cast_store(out + token_idx * d + idx * vec_size);
  }

  const int64_t remaining_offset = d - d % (stride * vec_size);
  // process the remaining elements
#pragma unroll 1
  for (int64_t idx = thread_idx; idx < d % (stride * vec_size); idx += stride) {
    T x = input[offset + remaining_offset + idx];
    out[token_idx * d + remaining_offset + idx] = Activation(x);
  }
}
```
**EN:** This section drives `cast_store`, `Activation`, `remaining_offset` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`cast_store`、`Activation`、`remaining_offset`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 86-87: Local implementation details
```cpp
}  // namespace activation
}  // namespace sgl_hip
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。
- **Validation / 校验测试**: Contains checks or scaffolding for correctness verification. / 包含正确性验证所需的检查或脚手架。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `utils.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: include / hip / hip_act_and_mul.cuh
