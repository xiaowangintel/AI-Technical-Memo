# mma_xe_legacy.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/mma_xe_legacy.hpp`
- **EN:** Provides legacy Intel Xe MMA builtins or SPIR-V wrappers.
- **CN:** 提供传统 Intel Xe MMA 内建或 SPIR-V 封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
 * Copyright (C) 2025 Intel Corporation, All rights reserved.
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
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 34-36
```cpp
#if defined(__SYCL_DEVICE_ONLY__) && defined(SYCL_INTEL_TARGET)
#define CUTE_ARCH_MMA_XE_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (__SYCL_DEVICE_ONLY__, CUTE_ARCH_MMA_XE_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__SYCL_DEVICE_ONLY__, CUTE_ARCH_MMA_XE_ENABLED）只启用当前目标有效的构建路径。

### Lines 38-42
```cpp
#if defined(CUTE_ARCH_MMA_XE_ENABLED) && ((defined(__INTEL_LLVM_COMPILER) && (__INTEL_LLVM_COMPILER < 20250200)) || defined(CUTLASS_SYCL_BUILTIN_ENABLE))
#include <cute/arch/mma_xe_legacy_builtin.hpp>
#elif defined(CUTE_ARCH_MMA_XE_ENABLED)
#include <cute/arch/mma_xe_legacy_spirv.hpp>
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_MMA_XE_ENABLED, CUTLASS_SYCL_BUILTIN_ENABLE) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_MMA_XE_ENABLED, CUTLASS_SYCL_BUILTIN_ENABLE）只启用当前目标有效的构建路径。

### Lines 44-46
```cpp
#include <cute/config.hpp>
#include <cute/arch/mma.hpp>
#include <cute/util/sycl_vec.hpp>
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/arch/mma.hpp` (generic low-level MMA operation tags and wrappers); `cute/util/sycl_vec.hpp` (related definitions from `cute/util/sycl_vec.hpp`).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/arch/mma.hpp`（通用底层 MMA 操作标签与封装）；`cute/util/sycl_vec.hpp`（来自 `cute/util/sycl_vec.hpp` 的相关定义）。

### Lines 48-58
```cpp
namespace cute {
//MxNxK_D,A,B,C
//# of vector component of a x subgroup-size x function name
//float8 intel_sub_group_bf16_bf16_matrix_mad_k16(short8 a, int8 b, float8 acc);
//TODO: Is A really not transposed? Maybe better a macro than separate define for 1,2,4,8
struct XE_8x16x16_F32BF16BF16F32_TT
{
  using DRegisters = intel::float8[1];
  using ARegisters = intel::short8[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::float8[1];
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 60-78
```cpp
  CUTE_HOST_DEVICE static void
  fma(intel::float8      & d,
      intel::short8 const& a,
      intel::int8   const& b,
      intel::float8 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, bfloat16_t, bfloat16_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x16_F32BF16BF16F32_TT on non-Xe hardware");
#endif
  }
};
struct XE_4x16x16_F32BF16BF16F32_TT
{
  using DRegisters = intel::float4[1];
  using ARegisters = intel::short4[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::float4[1];
```
- **EN:** Implements `fma`, combining source fragments and accumulators into the destination fragment.
- **CN:** 实现 `fma`，把源片段与累加器片段组合到目标片段中。

### Lines 80-98
```cpp
  CUTE_HOST_DEVICE static void
  fma(intel::float4      & d,
      intel::short4 const& a,
      intel::int8   const& b,
      intel::float4 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, bfloat16_t, bfloat16_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x16_F32BF16BF16F32_TT on non-Xe hardware");
#endif
  }
};
struct XE_2x16x16_F32BF16BF16F32_TT
{
  using DRegisters = intel::float2[1];
  using ARegisters = intel::short2[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::float2[1];
```
- **EN:** Implements `fma`, combining source fragments and accumulators into the destination fragment.
- **CN:** 实现 `fma`，把源片段与累加器片段组合到目标片段中。

### Lines 100-119
```cpp
  CUTE_HOST_DEVICE static void
  fma(intel::float2      & d,
      intel::short2 const& a,
      intel::int8   const& b,
      intel::float2 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, bfloat16_t, bfloat16_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x16_F32BF16BF16F32_TT on non-Xe hardware");
#endif
  }
};
//float  intel_sub_group_bf16_bf16_matrix_mad_k16(short  a, int8 b, float  acc)
struct XE_1x16x16_F32BF16BF16F32_TT
{
  using DRegisters = float[1];
  using ARegisters = short[1];
  using BRegisters = intel::int8[1];
  using CRegisters = float[1];
```
- **EN:** Implements `fma`, combining source fragments and accumulators into the destination fragment.
- **CN:** 实现 `fma`，把源片段与累加器片段组合到目标片段中。

### Lines 121-133
```cpp
  CUTE_HOST_DEVICE static void
  fma(float      & d,
      short const& a,
      intel::int8  const& b,
      float const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, bfloat16_t, bfloat16_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_1x16x16_F32BF16BF16F32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Implements `fma`, combining source fragments and accumulators into the destination fragment.
- **CN:** 实现 `fma`，把源片段与累加器片段组合到目标片段中。

### Lines 135-238
```cpp
struct XE_8x16x16_BF16BF16BF16BF16_TT
{
  using DRegisters = intel::short8[1];
  using ARegisters = intel::short8[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::short8[1];

  CUTE_HOST_DEVICE static void
  fma(intel::short8      & d,
      intel::short8 const& a,
      intel::int8   const& b,
      intel::short8 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<bfloat16_t, bfloat16_t, bfloat16_t, bfloat16_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x16_BF16BF16BF16BF16_TT on non-PVC hardware");
#endif
  }
};
struct XE_4x16x16_BF16BF16BF16BF16_TT
{
  using DRegisters = intel::short4[1];
  using ARegisters = intel::short4[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::short4[1];

  CUTE_HOST_DEVICE static void
  fma(intel::short4      & d,
      intel::short4 const& a,
      intel::int8   const& b,
      intel::short4 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<bfloat16_t, bfloat16_t, bfloat16_t, bfloat16_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_4x16x16_BF16BF16BF16BF16_TT on non-PVC hardware");
#endif
  }
};
struct XE_2x16x16_BF16BF16BF16BF16_TT
{
  using DRegisters = intel::short2[1];
  using ARegisters = intel::short2[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::short2[1];

  CUTE_HOST_DEVICE static void
  fma(intel::short2      & d,
      intel::short2 const& a,
      intel::int8   const& b,
      intel::short2 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<bfloat16_t, bfloat16_t, bfloat16_t, bfloat16_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_2x16x16_BF16BF16BF16BF16_TT on non-PVC hardware");
#endif
  }
};
struct XE_1x16x16_BF16BF16BF16BF16_TT
{
  using DRegisters = short[1];
  using ARegisters = short[1];
  using BRegisters = intel::int8[1];
  using CRegisters = short[1];

  CUTE_HOST_DEVICE static void
  fma(short            & d,
      short       const& a,
      intel::int8 const& b,
            short const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<bfloat16_t, bfloat16_t, bfloat16_t, bfloat16_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_1x16x16_BF16BF16BF16BF16_TT on non-PVC hardware");
#endif
  }
};
//MxNxK_A,B,C,D
//# of vector component of a x subgroup-size x function name
//float8 intel_sub_group_f16_f16_matrix_mad_k16(short8 a, int8 b, int8 acc);
//TODO: Is A really not transposed? Maybe better a macro than separate define for 1,2,4,8
struct XE_8x16x16_F32F16F16F32_TT
{
  using DRegisters = intel::float8[1];
  using ARegisters = intel::short8[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::float8[1];

  CUTE_HOST_DEVICE static void
  fma(intel::float8      & d,
      intel::short8 const& a,
      intel::int8   const& b,
      intel::float8 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, half_t, half_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x16_F32F16F16F32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_8x16x16_BF16BF16BF16BF16_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_8x16x16_BF16BF16BF16BF16_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 240-259
```cpp
struct XE_4x16x16_F32F16F16F32_TT
{
  using DRegisters = intel::float4[1];
  using ARegisters = intel::short4[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::float4[1];

  CUTE_HOST_DEVICE static void
  fma(intel::float4      & d,
      intel::short4 const& a,
      intel::int8   const& b,
      intel::float4 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, half_t, half_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_4x16x16_F32F16F16F32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_4x16x16_F32F16F16F32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_4x16x16_F32F16F16F32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 261-280
```cpp
struct XE_2x16x16_F32F16F16F32_TT
{
  using DRegisters = intel::float2[1];
  using ARegisters = intel::short2[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::float2[1];

  CUTE_HOST_DEVICE static void
  fma(intel::float2      & d,
      intel::short2 const& a,
      intel::int8   const& b,
      intel::float2 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
  d = detail::XeSubgroupMatrixMultiplyAccumulate<float, half_t, half_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_2x16x16_F32F16F16F32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2x16x16_F32F16F16F32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2x16x16_F32F16F16F32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 282-301
```cpp
struct XE_1x16x16_F32F16F16F32_TT
{
  using DRegisters = float[1];
  using ARegisters = short[1];
  using BRegisters = intel::int8[1];
  using CRegisters = float[1];

  CUTE_HOST_DEVICE static void
  fma(float            & d,
      short       const& a,
      intel::int8 const& b,
      float       const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, half_t, half_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_1x16x16_F32F16F16F32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_1x16x16_F32F16F16F32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_1x16x16_F32F16F16F32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 303-322
```cpp
struct XE_8x16x16_F16F16F16F16_TT
{
  using DRegisters = intel::half8[1];
  using ARegisters = intel::short8[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::half8[1];

  CUTE_HOST_DEVICE static void
  fma(intel::half8       & d,
      intel::short8 const& a,
      intel::int8   const& b,
      intel::half8  const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<half_t, half_t, half_t, half_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x16_F16F16F16F16_TT on non-PVC hardware");
#endif
  }
};
```
- **EN:** Defines `XE_8x16x16_F16F16F16F16_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_8x16x16_F16F16F16F16_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 324-343
```cpp
struct XE_4x16x16_F16F16F16F16_TT
{
  using DRegisters = intel::half4[1];
  using ARegisters = intel::short4[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::half4[1];

  CUTE_HOST_DEVICE static void
  fma(intel::half4       & d,
      intel::short4 const& a,
      intel::int8   const& b,
      intel::half4  const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<half_t, half_t, half_t, half_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_4x16x16_F16F16F16F16_TT on non-PVC hardware");
#endif
  }
};
```
- **EN:** Defines `XE_4x16x16_F16F16F16F16_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_4x16x16_F16F16F16F16_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 345-364
```cpp
struct XE_2x16x16_F16F16F16F16_TT
{
  using DRegisters = intel::half2[1];
  using ARegisters = intel::short2[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::half2[1];

  CUTE_HOST_DEVICE static void
  fma(intel::half2       & d,
      intel::short2 const& a,
      intel::int8   const& b,
      intel::half2  const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
  d = detail::XeSubgroupMatrixMultiplyAccumulate<half_t, half_t, half_t, half_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_2x16x16_F16F16F16F16_TT on non-PVC hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2x16x16_F16F16F16F16_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2x16x16_F16F16F16F16_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 366-409
```cpp
struct XE_1x16x16_F16F16F16F16_TT
{
  using DRegisters = intel::half[1];
  using ARegisters = short[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::half[1];

  CUTE_HOST_DEVICE static void
  fma(intel::half      & d,
      short       const& a,
      intel::int8 const& b,
      intel::half const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<half_t, half_t, half_t, half_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_1x16x16_F16F16F16F16_TT on non-PVC hardware");
#endif
  }
};
//MxNxK_A,B,C,D
//# of vector component of a x subgroup-size x function name
//float8 intel_sub_group_i8_i8_matrix_mad_k16(short8 a, int8 b, float8 acc);
//TODO: Is A really not transposed? Maybe better a macro than separate define for 1,2,4,8
struct XE_8x16x32_S32S8S8S32_TT
{
  using DRegisters = intel::int8[1];
  using ARegisters = intel::short8[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::int8[1];

  CUTE_HOST_DEVICE static void
  fma(intel::int8      & d,
      intel::short8 const& a,
      intel::int8   const& b,
      intel::int8 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<int32_t, int8_t, int8_t, int32_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x32_S32S8S8S32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_1x16x16_F16F16F16F16_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_1x16x16_F16F16F16F16_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 411-430
```cpp
struct XE_4x16x32_S32S8S8S32_TT
{
  using DRegisters = intel::int4[1];
  using ARegisters = intel::short4[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::int4[1];

  CUTE_HOST_DEVICE static void
  fma(intel::int4      & d,
      intel::short4 const& a,
      intel::int8   const& b,
      intel::int4 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<int32_t, int8_t, int8_t, int32_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_4x16x32_S32S8S8S32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_4x16x32_S32S8S8S32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_4x16x32_S32S8S8S32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 432-451
```cpp
struct XE_2x16x32_S32S8S8S32_TT
{
  using DRegisters = intel::int2[1];
  using ARegisters = intel::short2[1];
  using BRegisters = intel::int8[1];
  using CRegisters = intel::int2[1];

  CUTE_HOST_DEVICE static void
  fma(intel::int2      & d,
      intel::short2 const& a,
      intel::int8   const& b,
      intel::int2 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<int32_t, int8_t, int8_t, int32_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_2x16x32_S32S8S8S32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2x16x32_S32S8S8S32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2x16x32_S32S8S8S32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 453-472
```cpp
struct XE_1x16x32_S32S8S8S32_TT
{
  using DRegisters = int[1];
  using ARegisters = short[1];
  using BRegisters = intel::int8[1];
  using CRegisters = int[1];

  CUTE_HOST_DEVICE static void
  fma(int      & d,
      short const& a,
      intel::int8  const& b,
      int const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<int32_t, int8_t, int8_t, int32_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_1x16x32_S32S8S8S32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_1x16x32_S32S8S8S32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_1x16x32_S32S8S8S32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 474-493
```cpp
struct XE_8x16x32_S32U8U8S32_TT
{
  using DRegisters = intel::int8[1];
  using ARegisters = intel::ushort8[1];
  using BRegisters = intel::uint8[1];
  using CRegisters = intel::int8[1];

  CUTE_HOST_DEVICE static void
  fma(intel::int8      & d,
      intel::ushort8 const& a,
      intel::uint8   const& b,
      intel::int8 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<int32_t, uint8_t, uint8_t, int32_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x32_S32U8U8S32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_8x16x32_S32U8U8S32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_8x16x32_S32U8U8S32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 495-514
```cpp
struct XE_4x16x32_S32U8U8S32_TT
{
  using DRegisters = intel::int4[1];
  using ARegisters = intel::ushort4[1];
  using BRegisters = intel::uint8[1];
  using CRegisters = intel::int4[1];

  CUTE_HOST_DEVICE static void
  fma(intel::int4      & d,
      intel::ushort4 const& a,
      intel::uint8   const& b,
      intel::int4 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<int32_t, uint8_t, uint8_t, int32_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_4x16x32_S32U8U8S32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_4x16x32_S32U8U8S32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_4x16x32_S32U8U8S32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 516-535
```cpp
struct XE_2x16x32_S32U8U8S32_TT
{
  using DRegisters = intel::int2[1];
  using ARegisters = intel::ushort2[1];
  using BRegisters = intel::uint8[1];
  using CRegisters = intel::int2[1];

  CUTE_HOST_DEVICE static void
  fma(intel::int2      & d,
      intel::ushort2 const& a,
      intel::uint8   const& b,
      intel::int2 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<int32_t, uint8_t, uint8_t, int32_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_2x16x32_S32U8U8S32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2x16x32_S32U8U8S32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2x16x32_S32U8U8S32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 537-556
```cpp
struct XE_1x16x32_S32U8U8S32_TT
{
  using DRegisters = int[1];
  using ARegisters = cute::intel::ushort[1];
  using BRegisters = intel::uint8[1];
  using CRegisters = int[1];

  CUTE_HOST_DEVICE static void
  fma(int      & d,
      cute::intel::ushort const& a,
      intel::uint8  const& b,
      int const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<int32_t, uint8_t, uint8_t, int32_t>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_1x16x32_S32U8U8S32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_1x16x32_S32U8U8S32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_1x16x32_S32U8U8S32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 558-577
```cpp
struct XE_8x16x8_F32TF32TF32F32_TT
{
  using DRegisters = intel::float8[1];
  using ARegisters = intel::float4[1];
  using BRegisters = intel::float8[1];
  using CRegisters = intel::float8[1];

  CUTE_HOST_DEVICE static void
  fma(intel::float8      & d,
      intel::float4 const& a,
      intel::float8   const& b,
      intel::float8 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, tfloat32_t, tfloat32_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_8x16x8_F32TF32TF32F32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_8x16x8_F32TF32TF32F32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_8x16x8_F32TF32TF32F32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 579-598
```cpp
struct XE_4x16x8_F32TF32TF32F32_TT
{
  using DRegisters = intel::float4[1];
  using ARegisters = intel::float2[1];
  using BRegisters = intel::float8[1];
  using CRegisters = intel::float4[1];

  CUTE_HOST_DEVICE static void
  fma(intel::float4      & d,
      intel::float2 const& a,
      intel::float8   const& b,
      intel::float4 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, tfloat32_t, tfloat32_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_4x16x8_F32TF32TF32F32_TT on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_4x16x8_F32TF32TF32F32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_4x16x8_F32TF32TF32F32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 600-641
```cpp
struct XE_2x16x8_F32TF32TF32F32_TT
{
  using DRegisters = intel::float2[1];
  using ARegisters = float[1];
  using BRegisters = intel::float8[1];
  using CRegisters = intel::float2[1];

  CUTE_HOST_DEVICE static void
  fma(intel::float2      & d,
      float const& a,
      intel::float8   const& b,
      intel::float2 const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, tfloat32_t, tfloat32_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_2x16x8_F32TF32TF32F32_TT on non-Xe hardware");
#endif
  }
};
//float  intel_sub_group_bf16_bf16_matrix_mad_k16(short  a, int8 b, float  acc)
struct XE_1x16x8_F32TF32TF32F32_TT
{
  using DRegisters = float[1];
  using ARegisters = float[1];
  using BRegisters = intel::float8[1];
  using CRegisters = float[1];

  CUTE_HOST_DEVICE static void
  fma(float      & d,
      float const& a,
      intel::float8  const& b,
      float const& c)
  {
#if defined(CUTE_ARCH_MMA_XE_ENABLED)
    d = detail::XeSubgroupMatrixMultiplyAccumulate<float, tfloat32_t, tfloat32_t, float>{}(a, b, c);
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use XE_1x16x8_F32TF32TF32F32_TT on non-Xe hardware");
#endif
  }
};
} //namespace cute
```
- **EN:** Defines `XE_2x16x8_F32TF32TF32F32_TT` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2x16x8_F32TF32TF32F32_TT`，把该头文件这一部分所需的类型、常量与行为组织在一起。

## Key Concepts / 关键概念

- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/arch/mma_xe_legacy_builtin.hpp` supplies related definitions from `cute/arch/mma_xe_legacy_builtin.hpp`.
  **CN:** `cute/arch/mma_xe_legacy_builtin.hpp` 提供了来自 `cute/arch/mma_xe_legacy_builtin.hpp` 的相关定义。
- **EN:** `cute/arch/mma_xe_legacy_spirv.hpp` supplies related definitions from `cute/arch/mma_xe_legacy_spirv.hpp`.
  **CN:** `cute/arch/mma_xe_legacy_spirv.hpp` 提供了来自 `cute/arch/mma_xe_legacy_spirv.hpp` 的相关定义。
- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/arch/mma.hpp` supplies generic low-level MMA operation tags and wrappers.
  **CN:** `cute/arch/mma.hpp` 提供了通用底层 MMA 操作标签与封装。
- **EN:** `cute/util/sycl_vec.hpp` supplies related definitions from `cute/util/sycl_vec.hpp`.
  **CN:** `cute/util/sycl_vec.hpp` 提供了来自 `cute/util/sycl_vec.hpp` 的相关定义。
- **EN:** SYCL/SPIR-V feature macros select alternate code paths for Intel/Xe-style backends.
  **CN:** SYCL/SPIR-V 特性宏会为 Intel/Xe 风格后端选择替代代码路径。
