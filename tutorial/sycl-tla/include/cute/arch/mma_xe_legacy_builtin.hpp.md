# mma_xe_legacy_builtin.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/mma_xe_legacy_builtin.hpp`
- **EN:** Provides legacy Intel Xe MMA builtins or SPIR-V wrappers.
- **CN:** 提供传统 Intel Xe MMA 内建或 SPIR-V 封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
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
#include <cute/util/sycl_vec.hpp>
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 34-63
```cpp
// mma_bf16
SYCL_EXTERNAL cute::intel::float8 intel_sub_group_bf16_bf16_matrix_mad_k16(cute::intel::short8 a, cute::intel::int8 b, cute::intel::float8 acc);
SYCL_EXTERNAL cute::intel::float4 intel_sub_group_bf16_bf16_matrix_mad_k16(cute::intel::short4 a, cute::intel::int8 b, cute::intel::float4 acc);
SYCL_EXTERNAL cute::intel::float2 intel_sub_group_bf16_bf16_matrix_mad_k16(cute::intel::short2 a, cute::intel::int8 b, cute::intel::float2 acc);
SYCL_EXTERNAL               float intel_sub_group_bf16_bf16_matrix_mad_k16(              short a, cute::intel::int8 b,               float acc);
// mma_half
SYCL_EXTERNAL cute::intel::float8 intel_sub_group_f16_f16_matrix_mad_k16(cute::intel::short8 a, cute::intel::int8 b, cute::intel::float8 acc);
SYCL_EXTERNAL cute::intel::float4 intel_sub_group_f16_f16_matrix_mad_k16(cute::intel::short4 a, cute::intel::int8 b, cute::intel::float4 acc);
SYCL_EXTERNAL cute::intel::float2 intel_sub_group_f16_f16_matrix_mad_k16(cute::intel::short2 a, cute::intel::int8 b, cute::intel::float2 acc);
SYCL_EXTERNAL               float intel_sub_group_f16_f16_matrix_mad_k16(              short a, cute::intel::int8 b,               float acc);
// mma_s8
SYCL_EXTERNAL cute::intel::int8 intel_sub_group_i8_i8_matrix_mad_k32(cute::intel::short8 a, cute::intel::int8 b, cute::intel::int8 acc);
SYCL_EXTERNAL cute::intel::int4 intel_sub_group_i8_i8_matrix_mad_k32(cute::intel::short4 a, cute::intel::int8 b, cute::intel::int4 acc);
SYCL_EXTERNAL cute::intel::int2 intel_sub_group_i8_i8_matrix_mad_k32(cute::intel::short2 a, cute::intel::int8 b, cute::intel::int2 acc);
SYCL_EXTERNAL               int intel_sub_group_i8_i8_matrix_mad_k32(              short a, cute::intel::int8 b,               int acc);
// mma_u8
SYCL_EXTERNAL cute::intel::int8 intel_sub_group_u8_u8_matrix_mad_k32(cute::intel::ushort8 a, cute::intel::uint8 b, cute::intel::int8 acc);
SYCL_EXTERNAL cute::intel::int4 intel_sub_group_u8_u8_matrix_mad_k32(cute::intel::ushort4 a, cute::intel::uint8 b, cute::intel::int4 acc);
SYCL_EXTERNAL cute::intel::int2 intel_sub_group_u8_u8_matrix_mad_k32(cute::intel::ushort2 a, cute::intel::uint8 b, cute::intel::int2 acc);
SYCL_EXTERNAL               int intel_sub_group_u8_u8_matrix_mad_k32(              ushort a, cute::intel::uint8 b,               int acc);
// mma_tf32
SYCL_EXTERNAL cute::intel::float8 intel_sub_group_tf32_tf32_matrix_mad_k8(cute::intel::float4 a, cute::intel::float8 b, cute::intel::float8 acc);
SYCL_EXTERNAL cute::intel::float4 intel_sub_group_tf32_tf32_matrix_mad_k8(cute::intel::float2 a, cute::intel::float8 b, cute::intel::float4 acc);
SYCL_EXTERNAL cute::intel::float2 intel_sub_group_tf32_tf32_matrix_mad_k8(              float a, cute::intel::float8 b, cute::intel::float2 acc);
SYCL_EXTERNAL               float intel_sub_group_tf32_tf32_matrix_mad_k8(              float a, cute::intel::float8 b,               float acc);
// mma_bfloat16 with bfloat16 accumulator:
SYCL_EXTERNAL cute::intel::short8 intel_sub_group_bf16_bf16_matrix_mad_k16(cute::intel::short8 a, cute::intel::int8 b, cute::intel::short8 acc);
SYCL_EXTERNAL cute::intel::short4 intel_sub_group_bf16_bf16_matrix_mad_k16(cute::intel::short4 a, cute::intel::int8 b, cute::intel::short4 acc);
SYCL_EXTERNAL cute::intel::short2 intel_sub_group_bf16_bf16_matrix_mad_k16(cute::intel::short2 a, cute::intel::int8 b, cute::intel::short2 acc);
SYCL_EXTERNAL               short intel_sub_group_bf16_bf16_matrix_mad_k16(              short a, cute::intel::int8 b,               short acc);
```
- **EN:** Defines or forwards `intel_sub_group_bf16_bf16_matrix_mad_k16` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_bf16_bf16_matrix_mad_k16`，作为该头文件可执行接口的一部分。

### Lines 65-69
```cpp
// Use the spirv functions as the builtins do not work
SYCL_EXTERNAL cute::intel::half8 __spirv_SubgroupMatrixMultiplyAccumulateINTEL(int32_t, cute::intel::short8, cute::intel::int8, cute::intel::half8, int32_t);
SYCL_EXTERNAL cute::intel::half4 __spirv_SubgroupMatrixMultiplyAccumulateINTEL(int32_t, cute::intel::short4, cute::intel::int8, cute::intel::half4, int32_t);
SYCL_EXTERNAL cute::intel::half2 __spirv_SubgroupMatrixMultiplyAccumulateINTEL(int32_t, cute::intel::short2, cute::intel::int8, cute::intel::half2, int32_t);
SYCL_EXTERNAL cute::intel::half  __spirv_SubgroupMatrixMultiplyAccumulateINTEL(int32_t,               short, cute::intel::int8,  cute::intel::half, int32_t);
```
- **EN:** Defines or forwards `__spirv_SubgroupMatrixMultiplyAccumulateINTEL` as part of this header's executable interface.
- **CN:** 定义或转发 `__spirv_SubgroupMatrixMultiplyAccumulateINTEL`，作为该头文件可执行接口的一部分。

### Lines 71-74
```cpp
struct SPIRV_MMAOperands {
  static constexpr int SPIRV_MatrixAFp16 = 0x400;
  static constexpr int SPIRV_MatrixBFp16 = 0x800;
};
```
- **EN:** Defines `SPIRV_MMAOperands` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SPIRV_MMAOperands`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 76-77
```cpp
namespace cute::detail
{
```
- **EN:** Enters or leaves namespace scope `cute::detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute::detail`，以便把相关符号组织在一起。

### Lines 79-82
```cpp
template <class ElementD, class ElementA, class ElementB, class ElementC>
struct XeSubgroupMatrixMultiplyAccumulate {
    static_assert(dependent_false<>, "Unsupported MMA Configuration.");
};
```
- **EN:** Defines `XeSubgroupMatrixMultiplyAccumulate` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroupMatrixMultiplyAccumulate`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 84-91
```cpp
template<>
struct XeSubgroupMatrixMultiplyAccumulate<float, bfloat16_t, bfloat16_t, float> {
    template<typename ARegisters, typename BRegisters, typename CRegisters>
    CUTE_HOST_DEVICE
    auto operator()(ARegisters a, BRegisters b, CRegisters c) {
      return intel_sub_group_bf16_bf16_matrix_mad_k16(a, b, c);
    }
};
```
- **EN:** Defines `XeSubgroupMatrixMultiplyAccumulate` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroupMatrixMultiplyAccumulate`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 93-100
```cpp
template<>
struct XeSubgroupMatrixMultiplyAccumulate<float, half_t, half_t, float> {
    template<typename ARegisters, typename BRegisters, typename CRegisters>
    CUTE_HOST_DEVICE
    auto operator()(ARegisters a, BRegisters b, CRegisters c) {
      return intel_sub_group_f16_f16_matrix_mad_k16(a, b, c);
    }
};
```
- **EN:** Defines `XeSubgroupMatrixMultiplyAccumulate` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroupMatrixMultiplyAccumulate`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 102-109
```cpp
template<>
struct XeSubgroupMatrixMultiplyAccumulate<bfloat16_t, bfloat16_t, bfloat16_t, bfloat16_t> {
    template<typename ARegisters, typename BRegisters, typename CRegisters>
    CUTE_HOST_DEVICE
    auto operator()(ARegisters a, BRegisters b, CRegisters c) {
      return intel_sub_group_bf16_bf16_matrix_mad_k16(a, b, c);
    }
};
```
- **EN:** Defines `XeSubgroupMatrixMultiplyAccumulate` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroupMatrixMultiplyAccumulate`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 111-119
```cpp
template<>
struct XeSubgroupMatrixMultiplyAccumulate<half_t, half_t, half_t, half_t> {
  template<typename ARegisters, typename BRegisters, typename CRegisters>
  CUTE_HOST_DEVICE
  auto operator()(ARegisters a, BRegisters b, CRegisters c) {
    return __spirv_SubgroupMatrixMultiplyAccumulateINTEL(16, a, b, c,
             SPIRV_MMAOperands::SPIRV_MatrixAFp16 | SPIRV_MMAOperands::SPIRV_MatrixBFp16);
  }
};
```
- **EN:** Defines `XeSubgroupMatrixMultiplyAccumulate` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroupMatrixMultiplyAccumulate`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 121-128
```cpp
template<>
struct XeSubgroupMatrixMultiplyAccumulate<int32_t, int8_t, int8_t, int32_t> {
    template<typename ARegisters, typename BRegisters, typename CRegisters>
    CUTE_HOST_DEVICE
    auto operator()(ARegisters a, BRegisters b, CRegisters c) {
      return intel_sub_group_i8_i8_matrix_mad_k32(a, b, c);
    }
};
```
- **EN:** Defines `XeSubgroupMatrixMultiplyAccumulate` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroupMatrixMultiplyAccumulate`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 130-137
```cpp
template<>
struct XeSubgroupMatrixMultiplyAccumulate<int32_t, uint8_t, uint8_t, int32_t> {
    template<typename ARegisters, typename BRegisters, typename CRegisters>
    CUTE_HOST_DEVICE
    auto operator()(ARegisters a, BRegisters b, CRegisters c) {
      return intel_sub_group_u8_u8_matrix_mad_k32(a, b, c);
    }
};
```
- **EN:** Defines `XeSubgroupMatrixMultiplyAccumulate` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroupMatrixMultiplyAccumulate`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 139-147
```cpp
template<>
struct XeSubgroupMatrixMultiplyAccumulate<float, tfloat32_t, tfloat32_t, float> {
    template<typename ARegisters, typename BRegisters, typename CRegisters>
    CUTE_HOST_DEVICE
    auto operator()(ARegisters a, BRegisters b, CRegisters c) {
      return intel_sub_group_tf32_tf32_matrix_mad_k8(a, b, c);
    }
};
} // namespace cute::detail end
```
- **EN:** Defines `XeSubgroupMatrixMultiplyAccumulate` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroupMatrixMultiplyAccumulate`，把该头文件这一部分所需的类型、常量与行为组织在一起。

## Key Concepts / 关键概念

- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/util/sycl_vec.hpp` supplies related definitions from `cute/util/sycl_vec.hpp`.
  **CN:** `cute/util/sycl_vec.hpp` 提供了来自 `cute/util/sycl_vec.hpp` 的相关定义。
