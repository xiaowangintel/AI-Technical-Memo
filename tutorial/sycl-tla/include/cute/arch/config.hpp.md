# config.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/config.hpp`
- **EN:** Defines architecture-selection macros and compile-time feature switches.
- **CN:** 定义体系结构选择宏与编译期特性开关。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
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
#pragma once
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 33-33
```cpp
#include <cutlass/arch/config.h> // CUTLASS_ARCH_MMA_SMxx_ENABLED
```
- **EN:** Imports `cutlass/arch/config.h` (related definitions from `cutlass/arch/config.h`).
- **CN:** 引入 `cutlass/arch/config.h`（来自 `cutlass/arch/config.h` 的相关定义）。

### Lines 35-38
```cpp
// MMA SM90A
#if defined(CUTLASS_ARCH_MMA_SM90A_ENABLED)
#  define CUTE_ARCH_MMA_SM90A_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM90A_ENABLED, CUTE_ARCH_MMA_SM90A_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM90A_ENABLED, CUTE_ARCH_MMA_SM90A_ENABLED）只启用当前目标有效的构建路径。

### Lines 40-43
```cpp
// TMA instructions
#if defined(CUTLASS_ARCH_MMA_SM90_ENABLED)
#  define CUTE_ARCH_TMA_SM90_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM90_ENABLED, CUTE_ARCH_TMA_SM90_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM90_ENABLED, CUTE_ARCH_TMA_SM90_ENABLED）只启用当前目标有效的构建路径。

### Lines 45-47
```cpp
#if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_ENABLED)
#  define CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_ENABLED, CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_ENABLED, CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED）只启用当前目标有效的构建路径。

### Lines 49-52
```cpp
// STSM
#if defined(CUTLASS_ARCH_MMA_SM90_ENABLED)
#  define CUTE_ARCH_STSM_SM90_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM90_ENABLED, CUTE_ARCH_STSM_SM90_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM90_ENABLED, CUTE_ARCH_STSM_SM90_ENABLED）只启用当前目标有效的构建路径。

### Lines 54-62
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

#if (defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101A_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM103A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM120A_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM121A_ENABLED))
#  define CUTE_ARCH_TMA_SM90_ENABLED
#  define CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED
#  define CUTE_ARCH_STSM_SM90_ENABLED
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 64-71
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101A_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM103A_ENABLED))
#  define CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 73-75
```cpp
#if defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM103A_ENABLED)
#  define CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM100A_ENABLED, CUTLASS_ARCH_MMA_SM103A_ENABLED, CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM100A_ENABLED, CUTLASS_ARCH_MMA_SM103A_ENABLED, CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED）只启用当前目标有效的构建路径。

### Lines 77-87
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM100F_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101F_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM103F_ENABLED))
#  define CUTE_ARCH_TMA_SM90_ENABLED 
#  define CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED
#  define CUTE_ARCH_STSM_SM90_ENABLED
#  define CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 89-91
```cpp
#if defined(CUTLASS_ARCH_MMA_SM100F_ENABLED) || defined(CUTLASS_ARCH_MMA_SM103F_ENABLED)
#  define CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM100F_ENABLED, CUTLASS_ARCH_MMA_SM103F_ENABLED, CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM100F_ENABLED, CUTLASS_ARCH_MMA_SM103F_ENABLED, CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED）只启用当前目标有效的构建路径。

### Lines 93-97
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM120F_ENABLED) || defined(CUTLASS_ARCH_MMA_SM121F_ENABLED))
#  define CUTE_ARCH_TMA_SM90_ENABLED
#  define CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED
#  define CUTE_ARCH_STSM_SM90_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM120F_ENABLED, CUTLASS_ARCH_MMA_SM121F_ENABLED, CUTE_ARCH_TMA_SM90_ENABLED, CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED, CUTE_ARCH_STSM_SM90_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM120F_ENABLED, CUTLASS_ARCH_MMA_SM121F_ENABLED, CUTE_ARCH_TMA_SM90_ENABLED, CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED, CUTE_ARCH_STSM_SM90_ENABLED）只启用当前目标有效的构建路径。

### Lines 100-118
```cpp
// SM110 specific configs
#if (defined(CUTLASS_ARCH_MMA_SM110A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM110F_ENABLED))
#  define CUTE_ARCH_TMA_SM90_ENABLED
#  define CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED
#  define CUTE_ARCH_STSM_SM90_ENABLED
#  define CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED
#  define CUTE_ARCH_TCGEN05_S8_MMA_ENABLED
#  define CUTE_ARCH_LDSM_SM100A_ENABLED
#  define CUTE_ARCH_STSM_SM100A_ENABLED
#  define CUTE_ARCH_TCGEN05_TMEM_ENABLED
#  define CUTE_ARCH_TMA_SM100_ENABLED
#  define CUTE_ARCH_LOAD256_SM100A_ENABLED
#  define CUTE_ARCH_STORE256_SM100A_ENABLED
#  define CUTE_ARCH_FLOAT2_MATH_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM110A_ENABLED, CUTLASS_ARCH_MMA_SM110F_ENABLED, CUTE_ARCH_TMA_SM90_ENABLED, CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED, CUTE_ARCH_STSM_SM90_ENABLED, CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED, CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED, CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED, CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED, CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED, CUTE_ARCH_TCGEN05_S8_MMA_ENABLED, CUTE_ARCH_LDSM_SM100A_ENABLED, CUTE_ARCH_STSM_SM100A_ENABLED, CUTE_ARCH_TCGEN05_TMEM_ENABLED, CUTE_ARCH_TMA_SM100_ENABLED, CUTE_ARCH_LOAD256_SM100A_ENABLED, CUTE_ARCH_STORE256_SM100A_ENABLED, CUTE_ARCH_FLOAT2_MATH_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM110A_ENABLED, CUTLASS_ARCH_MMA_SM110F_ENABLED, CUTE_ARCH_TMA_SM90_ENABLED, CUTE_ARCH_DEVICE_MODIFIABLE_TMA_SM90_ENABLED, CUTE_ARCH_STSM_SM90_ENABLED, CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED, CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED, CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED, CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED, CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED, CUTE_ARCH_TCGEN05_S8_MMA_ENABLED, CUTE_ARCH_LDSM_SM100A_ENABLED, CUTE_ARCH_STSM_SM100A_ENABLED, CUTE_ARCH_TCGEN05_TMEM_ENABLED, CUTE_ARCH_TMA_SM100_ENABLED, CUTE_ARCH_LOAD256_SM100A_ENABLED, CUTE_ARCH_STORE256_SM100A_ENABLED, CUTE_ARCH_FLOAT2_MATH_ENABLED）只启用当前目标有效的构建路径。

### Lines 120-122
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM110A_ENABLED))
#  define CUTE_ARCH_TCGEN05_S8_MMA_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM110A_ENABLED, CUTE_ARCH_TCGEN05_S8_MMA_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM110A_ENABLED, CUTE_ARCH_TCGEN05_S8_MMA_ENABLED）只启用当前目标有效的构建路径。

### Lines 124-126
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101A_ENABLED))
#  define CUTE_ARCH_TCGEN05_S8_MMA_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM100A_ENABLED, CUTLASS_ARCH_MMA_SM101A_ENABLED, CUTE_ARCH_TCGEN05_S8_MMA_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM100A_ENABLED, CUTLASS_ARCH_MMA_SM101A_ENABLED, CUTE_ARCH_TCGEN05_S8_MMA_ENABLED）只启用当前目标有效的构建路径。

### Lines 128-133
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101A_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM103A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM120A_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM120A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM121A_ENABLED))
#  define CUTE_ARCH_LDSM_SM100A_ENABLED
#  define CUTE_ARCH_STSM_SM100A_ENABLED
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 135-138
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101A_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM103A_ENABLED))
#  define CUTE_ARCH_TCGEN05_TMEM_ENABLED
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 140-143
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101A_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM103A_ENABLED))
#  define CUTE_ARCH_TMA_SM100_ENABLED
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 145-151
```cpp
// {add, mul, fma}.f32x2 PTX
#if defined(CUTLASS_ARCH_MMA_SM100_ENABLED) || defined(CUTLASS_ARCH_MMA_SM100A_ENABLED)
   // Enable CuTe MMA Atoms
#  define CUTE_ARCH_FFMA2_SM100_ENABLED
   // Enable f32x2 PTX generation
#  define CUTE_ARCH_FLOAT2_MATH_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM100_ENABLED, CUTLASS_ARCH_MMA_SM100A_ENABLED, CUTE_ARCH_FFMA2_SM100_ENABLED, CUTE_ARCH_FLOAT2_MATH_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM100_ENABLED, CUTLASS_ARCH_MMA_SM100A_ENABLED, CUTE_ARCH_FFMA2_SM100_ENABLED, CUTE_ARCH_FLOAT2_MATH_ENABLED）只启用当前目标有效的构建路径。

### Lines 153-157
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM120_ENABLED) || defined(CUTLASS_ARCH_MMA_SM120A_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM121_ENABLED) || defined(CUTLASS_ARCH_MMA_SM121A_ENABLED))
#  define CUTE_ARCH_MMA_SM120_ENABLED
#  define CUTE_ARCH_TMA_SM120_ENABLED
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 159-166
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM120_ENABLED) || defined(CUTLASS_ARCH_MMA_SM120A_ENABLED))
#  if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 8))
#    define CUTE_ARCH_F8F6F4_MMA_ENABLED
#    define CUTE_ARCH_MXF8F6F4_MMA_ENABLED
#    define CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED
#    define CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED
#  endif
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM120_ENABLED, CUTLASS_ARCH_MMA_SM120A_ENABLED, CUTE_ARCH_F8F6F4_MMA_ENABLED, CUTE_ARCH_MXF8F6F4_MMA_ENABLED, CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED, CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM120_ENABLED, CUTLASS_ARCH_MMA_SM120A_ENABLED, CUTE_ARCH_F8F6F4_MMA_ENABLED, CUTE_ARCH_MXF8F6F4_MMA_ENABLED, CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED, CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED）只启用当前目标有效的构建路径。

### Lines 168-175
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM121_ENABLED) || defined(CUTLASS_ARCH_MMA_SM121A_ENABLED))
#  if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9))
#    define CUTE_ARCH_F8F6F4_MMA_ENABLED
#    define CUTE_ARCH_MXF8F6F4_MMA_ENABLED
#    define CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED
#    define CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED
#  endif
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM121_ENABLED, CUTLASS_ARCH_MMA_SM121A_ENABLED, CUTE_ARCH_F8F6F4_MMA_ENABLED, CUTE_ARCH_MXF8F6F4_MMA_ENABLED, CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED, CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM121_ENABLED, CUTLASS_ARCH_MMA_SM121A_ENABLED, CUTE_ARCH_F8F6F4_MMA_ENABLED, CUTE_ARCH_MXF8F6F4_MMA_ENABLED, CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED, CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED）只启用当前目标有效的构建路径。

### Lines 177-183
```cpp
#if defined(CUTLASS_ARCH_MMA_SM100F_ENABLED) || defined(CUTLASS_ARCH_MMA_SM103F_ENABLED)
#  define CUTE_ARCH_LDSM_SM100A_ENABLED
#  define CUTE_ARCH_STSM_SM100A_ENABLED
#  define CUTE_ARCH_TCGEN05_TMEM_ENABLED
#  define CUTE_ARCH_TMA_SM100_ENABLED
#  define CUTE_ARCH_FLOAT2_MATH_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM100F_ENABLED, CUTLASS_ARCH_MMA_SM103F_ENABLED, CUTE_ARCH_LDSM_SM100A_ENABLED, CUTE_ARCH_STSM_SM100A_ENABLED, CUTE_ARCH_TCGEN05_TMEM_ENABLED, CUTE_ARCH_TMA_SM100_ENABLED, CUTE_ARCH_FLOAT2_MATH_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM100F_ENABLED, CUTLASS_ARCH_MMA_SM103F_ENABLED, CUTE_ARCH_LDSM_SM100A_ENABLED, CUTE_ARCH_STSM_SM100A_ENABLED, CUTE_ARCH_TCGEN05_TMEM_ENABLED, CUTE_ARCH_TMA_SM100_ENABLED, CUTE_ARCH_FLOAT2_MATH_ENABLED）只启用当前目标有效的构建路径。

### Lines 185-190
```cpp
#if defined(CUTLASS_ARCH_MMA_SM101F_ENABLED) 
#  define CUTE_ARCH_LDSM_SM100A_ENABLED
#  define CUTE_ARCH_STSM_SM100A_ENABLED
#  define CUTE_ARCH_TCGEN05_TMEM_ENABLED
#  define CUTE_ARCH_TMA_SM100_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM101F_ENABLED, CUTE_ARCH_LDSM_SM100A_ENABLED, CUTE_ARCH_STSM_SM100A_ENABLED, CUTE_ARCH_TCGEN05_TMEM_ENABLED, CUTE_ARCH_TMA_SM100_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM101F_ENABLED, CUTE_ARCH_LDSM_SM100A_ENABLED, CUTE_ARCH_STSM_SM100A_ENABLED, CUTE_ARCH_TCGEN05_TMEM_ENABLED, CUTE_ARCH_TMA_SM100_ENABLED）只启用当前目标有效的构建路径。

### Lines 192-195
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM120F_ENABLED) || defined(CUTLASS_ARCH_MMA_SM121F_ENABLED))
#  define CUTE_ARCH_LDSM_SM100A_ENABLED
#  define CUTE_ARCH_STSM_SM100A_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM120F_ENABLED, CUTLASS_ARCH_MMA_SM121F_ENABLED, CUTE_ARCH_LDSM_SM100A_ENABLED, CUTE_ARCH_STSM_SM100A_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM120F_ENABLED, CUTLASS_ARCH_MMA_SM121F_ENABLED, CUTE_ARCH_LDSM_SM100A_ENABLED, CUTE_ARCH_STSM_SM100A_ENABLED）只启用当前目标有效的构建路径。

### Lines 197-206
```cpp
#if (defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM100F_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM101A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM101F_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM103A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM103F_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM120A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM120F_ENABLED) ||\
     defined(CUTLASS_ARCH_MMA_SM121A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM121F_ENABLED))
#  if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9))
#    define CUTE_ARCH_LOAD256_SM100A_ENABLED
#    define CUTE_ARCH_STORE256_SM100A_ENABLED
#  endif
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 208-211
```cpp
// {add, mul, fma}.f32x2 PTX
#if defined(CUTLASS_ARCH_MMA_SM100A_ENABLED) || defined(CUTLASS_ARCH_MMA_SM100F_ENABLED)
  #define CUTE_ARCH_FLOAT2_MATH_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM100A_ENABLED, CUTLASS_ARCH_MMA_SM100F_ENABLED, CUTE_ARCH_FLOAT2_MATH_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM100A_ENABLED, CUTLASS_ARCH_MMA_SM100F_ENABLED, CUTE_ARCH_FLOAT2_MATH_ENABLED）只启用当前目标有效的构建路径。

### Lines 213-215
```cpp
#if defined(CUTLASS_ARCH_MMA_SM103_ENABLED) || defined(CUTLASS_ARCH_MMA_SM100F_ENABLED)
#  define CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ULTRA_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTLASS_ARCH_MMA_SM103_ENABLED, CUTLASS_ARCH_MMA_SM100F_ENABLED, CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ULTRA_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTLASS_ARCH_MMA_SM103_ENABLED, CUTLASS_ARCH_MMA_SM100F_ENABLED, CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ULTRA_ENABLED）只启用当前目标有效的构建路径。

## Key Concepts / 关键概念

- **EN:** TMA-related code packages descriptors, barriers, swizzles, and coordinates for bulk memory movement.
  **CN:** TMA 相关代码会封装描述符、屏障、swizzle 与坐标，以支持批量内存搬运。
- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。

## Dependencies / 依赖关系

- **EN:** `cutlass/arch/config.h` supplies related definitions from `cutlass/arch/config.h`.
  **CN:** `cutlass/arch/config.h` 提供了来自 `cutlass/arch/config.h` 的相关定义。
- **EN:** CUTLASS headers provide adapter types, synchronization hooks, or shared low-level GPU infrastructure.
  **CN:** CUTLASS 头文件提供适配类型、同步钩子或共享的底层 GPU 基础设施。
