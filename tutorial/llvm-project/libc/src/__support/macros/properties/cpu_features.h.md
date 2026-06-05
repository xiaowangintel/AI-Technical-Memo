# cpu_features.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `libc/src/__support/macros/properties/cpu_features.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Compile time cpu feature detection.
  - **CN**: 声明在 llvm-libc 各处共享的编译期配置、ABI 与属性宏。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14

````cpp
//===-- Compile time cpu feature detection ----------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This file lists target cpu features by introspecting compiler enabled
// preprocessor definitions.
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIBC_SRC___SUPPORT_MACROS_PROPERTIES_CPU_FEATURES_H
#define LLVM_LIBC_SRC___SUPPORT_MACROS_PROPERTIES_CPU_FEATURES_H

````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 分隔注释，用于视觉分组。
- **L3 EN**: Comment documents nearby intent or constraints: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的意图或约束：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment documents nearby intent or constraints: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的意图或约束：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents nearby intent or constraints: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的意图或约束：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 分隔注释，用于视觉分组。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Comment documents nearby intent or constraints: `This file lists target cpu features by introspecting compiler enabled`.
  **L8 CN**: 注释说明附近代码的意图或约束：`This file lists target cpu features by introspecting compiler enabled`。
- **L9 EN**: Comment documents nearby intent or constraints: `preprocessor definitions.`.
  **L9 CN**: 注释说明附近代码的意图或约束：`preprocessor definitions.`。
- **L10 EN**: Banner comment marking a file or section boundary.
  **L10 CN**: 横幅注释，用于标记文件或章节边界。
- **L11 EN**: Blank line separating nearby declarations or logic.
  **L11 CN**: 空行，用于分隔相邻声明或逻辑。
- **L12 EN**: Starts a header guard condition: `#ifndef LLVM_LIBC_SRC___SUPPORT_MACROS_PROPERTIES_CPU_FEATURES_H`.
  **L12 CN**: 开始头文件保护条件：`#ifndef LLVM_LIBC_SRC___SUPPORT_MACROS_PROPERTIES_CPU_FEATURES_H`。
- **L13 EN**: Defines macro `LLVM_LIBC_SRC___SUPPORT_MACROS_PROPERTIES_CPU_FEATURES_H` for compile-time control or shorthand.
  **L13 CN**: 定义宏 `LLVM_LIBC_SRC___SUPPORT_MACROS_PROPERTIES_CPU_FEATURES_H`，用于编译期控制或简写。
- **L14 EN**: Blank line separating nearby declarations or logic.
  **L14 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 15-28

````cpp
#include "architectures.h"

#if defined(__ARM_FEATURE_FP16_SCALAR_ARITHMETIC)
#define LIBC_TARGET_CPU_HAS_FULLFP16
#endif

#if defined(__ARM_FEATURE_SVE)
#define LIBC_TARGET_CPU_HAS_SVE
#endif

#if defined(__ARM_FEATURE_SVE2)
#define LIBC_TARGET_CPU_HAS_SVE2
#endif

````
- **L15 EN**: Includes "architectures.h" to access nearby local declarations.
  **L15 CN**: 引入 "architectures.h" 以使用附近的本地声明。
- **L16 EN**: Blank line separating nearby declarations or logic.
  **L16 CN**: 空行，用于分隔相邻声明或逻辑。
- **L17 EN**: Starts a preprocessor conditional block: `#if defined(__ARM_FEATURE_FP16_SCALAR_ARITHMETIC)`.
  **L17 CN**: 开始一个预处理条件块：`#if defined(__ARM_FEATURE_FP16_SCALAR_ARITHMETIC)`。
- **L18 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FULLFP16` for compile-time control or shorthand.
  **L18 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FULLFP16`，用于编译期控制或简写。
- **L19 EN**: Closes the current preprocessor conditional block or header guard.
  **L19 CN**: 结束当前预处理条件块或头文件保护。
- **L20 EN**: Blank line separating nearby declarations or logic.
  **L20 CN**: 空行，用于分隔相邻声明或逻辑。
- **L21 EN**: Starts a preprocessor conditional block: `#if defined(__ARM_FEATURE_SVE)`.
  **L21 CN**: 开始一个预处理条件块：`#if defined(__ARM_FEATURE_SVE)`。
- **L22 EN**: Defines macro `LIBC_TARGET_CPU_HAS_SVE` for compile-time control or shorthand.
  **L22 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_SVE`，用于编译期控制或简写。
- **L23 EN**: Closes the current preprocessor conditional block or header guard.
  **L23 CN**: 结束当前预处理条件块或头文件保护。
- **L24 EN**: Blank line separating nearby declarations or logic.
  **L24 CN**: 空行，用于分隔相邻声明或逻辑。
- **L25 EN**: Starts a preprocessor conditional block: `#if defined(__ARM_FEATURE_SVE2)`.
  **L25 CN**: 开始一个预处理条件块：`#if defined(__ARM_FEATURE_SVE2)`。
- **L26 EN**: Defines macro `LIBC_TARGET_CPU_HAS_SVE2` for compile-time control or shorthand.
  **L26 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_SVE2`，用于编译期控制或简写。
- **L27 EN**: Closes the current preprocessor conditional block or header guard.
  **L27 CN**: 结束当前预处理条件块或头文件保护。
- **L28 EN**: Blank line separating nearby declarations or logic.
  **L28 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 29-42

````cpp
#if defined(__ARM_FEATURE_MOPS)
#define LIBC_TARGET_CPU_HAS_MOPS
#endif

#if defined(__SSE2__)
#define LIBC_TARGET_CPU_HAS_SSE2
#define LIBC_TARGET_CPU_HAS_FPU_FLOAT
#define LIBC_TARGET_CPU_HAS_FPU_DOUBLE
#endif

#if defined(__SSE4_2__)
#define LIBC_TARGET_CPU_HAS_SSE4_2
#endif

````
- **L29 EN**: Starts a preprocessor conditional block: `#if defined(__ARM_FEATURE_MOPS)`.
  **L29 CN**: 开始一个预处理条件块：`#if defined(__ARM_FEATURE_MOPS)`。
- **L30 EN**: Defines macro `LIBC_TARGET_CPU_HAS_MOPS` for compile-time control or shorthand.
  **L30 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_MOPS`，用于编译期控制或简写。
- **L31 EN**: Closes the current preprocessor conditional block or header guard.
  **L31 CN**: 结束当前预处理条件块或头文件保护。
- **L32 EN**: Blank line separating nearby declarations or logic.
  **L32 CN**: 空行，用于分隔相邻声明或逻辑。
- **L33 EN**: Starts a preprocessor conditional block: `#if defined(__SSE2__)`.
  **L33 CN**: 开始一个预处理条件块：`#if defined(__SSE2__)`。
- **L34 EN**: Defines macro `LIBC_TARGET_CPU_HAS_SSE2` for compile-time control or shorthand.
  **L34 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_SSE2`，用于编译期控制或简写。
- **L35 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_FLOAT` for compile-time control or shorthand.
  **L35 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_FLOAT`，用于编译期控制或简写。
- **L36 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_DOUBLE` for compile-time control or shorthand.
  **L36 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_DOUBLE`，用于编译期控制或简写。
- **L37 EN**: Closes the current preprocessor conditional block or header guard.
  **L37 CN**: 结束当前预处理条件块或头文件保护。
- **L38 EN**: Blank line separating nearby declarations or logic.
  **L38 CN**: 空行，用于分隔相邻声明或逻辑。
- **L39 EN**: Starts a preprocessor conditional block: `#if defined(__SSE4_2__)`.
  **L39 CN**: 开始一个预处理条件块：`#if defined(__SSE4_2__)`。
- **L40 EN**: Defines macro `LIBC_TARGET_CPU_HAS_SSE4_2` for compile-time control or shorthand.
  **L40 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_SSE4_2`，用于编译期控制或简写。
- **L41 EN**: Closes the current preprocessor conditional block or header guard.
  **L41 CN**: 结束当前预处理条件块或头文件保护。
- **L42 EN**: Blank line separating nearby declarations or logic.
  **L42 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 43-56

````cpp
#if defined(__AVX__)
#define LIBC_TARGET_CPU_HAS_AVX
#endif

#if defined(__AVX2__)
#define LIBC_TARGET_CPU_HAS_AVX2
#endif

#if defined(__AVX512F__)
#define LIBC_TARGET_CPU_HAS_AVX512F
#endif

#if defined(__AVX512BW__)
#define LIBC_TARGET_CPU_HAS_AVX512BW
````
- **L43 EN**: Starts a preprocessor conditional block: `#if defined(__AVX__)`.
  **L43 CN**: 开始一个预处理条件块：`#if defined(__AVX__)`。
- **L44 EN**: Defines macro `LIBC_TARGET_CPU_HAS_AVX` for compile-time control or shorthand.
  **L44 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_AVX`，用于编译期控制或简写。
- **L45 EN**: Closes the current preprocessor conditional block or header guard.
  **L45 CN**: 结束当前预处理条件块或头文件保护。
- **L46 EN**: Blank line separating nearby declarations or logic.
  **L46 CN**: 空行，用于分隔相邻声明或逻辑。
- **L47 EN**: Starts a preprocessor conditional block: `#if defined(__AVX2__)`.
  **L47 CN**: 开始一个预处理条件块：`#if defined(__AVX2__)`。
- **L48 EN**: Defines macro `LIBC_TARGET_CPU_HAS_AVX2` for compile-time control or shorthand.
  **L48 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_AVX2`，用于编译期控制或简写。
- **L49 EN**: Closes the current preprocessor conditional block or header guard.
  **L49 CN**: 结束当前预处理条件块或头文件保护。
- **L50 EN**: Blank line separating nearby declarations or logic.
  **L50 CN**: 空行，用于分隔相邻声明或逻辑。
- **L51 EN**: Starts a preprocessor conditional block: `#if defined(__AVX512F__)`.
  **L51 CN**: 开始一个预处理条件块：`#if defined(__AVX512F__)`。
- **L52 EN**: Defines macro `LIBC_TARGET_CPU_HAS_AVX512F` for compile-time control or shorthand.
  **L52 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_AVX512F`，用于编译期控制或简写。
- **L53 EN**: Closes the current preprocessor conditional block or header guard.
  **L53 CN**: 结束当前预处理条件块或头文件保护。
- **L54 EN**: Blank line separating nearby declarations or logic.
  **L54 CN**: 空行，用于分隔相邻声明或逻辑。
- **L55 EN**: Starts a preprocessor conditional block: `#if defined(__AVX512BW__)`.
  **L55 CN**: 开始一个预处理条件块：`#if defined(__AVX512BW__)`。
- **L56 EN**: Defines macro `LIBC_TARGET_CPU_HAS_AVX512BW` for compile-time control or shorthand.
  **L56 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_AVX512BW`，用于编译期控制或简写。

### Lines 57-70

````cpp
#endif

#if defined(__AVX512F__) || defined(__AVX2__)
#define LIBC_TARGET_CPU_HAS_GATHER
#endif

#if defined(__ARM_FP)
#if (__ARM_FP & 0x2)
#define LIBC_TARGET_CPU_HAS_ARM_FPU_HALF
#define LIBC_TARGET_CPU_HAS_FPU_HALF
#endif // LIBC_TARGET_CPU_HAS_ARM_FPU_HALF
#if (__ARM_FP & 0x4)
#define LIBC_TARGET_CPU_HAS_ARM_FPU_FLOAT
#define LIBC_TARGET_CPU_HAS_FPU_FLOAT
````
- **L57 EN**: Closes the current preprocessor conditional block or header guard.
  **L57 CN**: 结束当前预处理条件块或头文件保护。
- **L58 EN**: Blank line separating nearby declarations or logic.
  **L58 CN**: 空行，用于分隔相邻声明或逻辑。
- **L59 EN**: Starts a preprocessor conditional block: `#if defined(__AVX512F__) || defined(__AVX2__)`.
  **L59 CN**: 开始一个预处理条件块：`#if defined(__AVX512F__) || defined(__AVX2__)`。
- **L60 EN**: Defines macro `LIBC_TARGET_CPU_HAS_GATHER` for compile-time control or shorthand.
  **L60 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_GATHER`，用于编译期控制或简写。
- **L61 EN**: Closes the current preprocessor conditional block or header guard.
  **L61 CN**: 结束当前预处理条件块或头文件保护。
- **L62 EN**: Blank line separating nearby declarations or logic.
  **L62 CN**: 空行，用于分隔相邻声明或逻辑。
- **L63 EN**: Starts a preprocessor conditional block: `#if defined(__ARM_FP)`.
  **L63 CN**: 开始一个预处理条件块：`#if defined(__ARM_FP)`。
- **L64 EN**: Starts a preprocessor conditional block: `#if (__ARM_FP & 0x2)`.
  **L64 CN**: 开始一个预处理条件块：`#if (__ARM_FP & 0x2)`。
- **L65 EN**: Defines macro `LIBC_TARGET_CPU_HAS_ARM_FPU_HALF` for compile-time control or shorthand.
  **L65 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_ARM_FPU_HALF`，用于编译期控制或简写。
- **L66 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_HALF` for compile-time control or shorthand.
  **L66 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_HALF`，用于编译期控制或简写。
- **L67 EN**: Closes the current preprocessor conditional block or header guard.
  **L67 CN**: 结束当前预处理条件块或头文件保护。
- **L68 EN**: Starts a preprocessor conditional block: `#if (__ARM_FP & 0x4)`.
  **L68 CN**: 开始一个预处理条件块：`#if (__ARM_FP & 0x4)`。
- **L69 EN**: Defines macro `LIBC_TARGET_CPU_HAS_ARM_FPU_FLOAT` for compile-time control or shorthand.
  **L69 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_ARM_FPU_FLOAT`，用于编译期控制或简写。
- **L70 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_FLOAT` for compile-time control or shorthand.
  **L70 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_FLOAT`，用于编译期控制或简写。

### Lines 71-84

````cpp
#endif // LIBC_TARGET_CPU_HAS_ARM_FPU_FLOAT
#if (__ARM_FP & 0x8)
#define LIBC_TARGET_CPU_HAS_ARM_FPU_DOUBLE
#define LIBC_TARGET_CPU_HAS_FPU_DOUBLE
#endif // LIBC_TARGET_CPU_HAS_ARM_FPU_DOUBLE
#endif // __ARM_FP

#if defined(__ARM_NEON)
#define LIBC_TARGET_CPU_HAS_ARM_NEON
#endif

#if defined(__riscv_flen)
// https://github.com/riscv-non-isa/riscv-c-api-doc/blob/main/src/c-api.adoc
#if defined(__riscv_zfhmin)
````
- **L71 EN**: Closes the current preprocessor conditional block or header guard.
  **L71 CN**: 结束当前预处理条件块或头文件保护。
- **L72 EN**: Starts a preprocessor conditional block: `#if (__ARM_FP & 0x8)`.
  **L72 CN**: 开始一个预处理条件块：`#if (__ARM_FP & 0x8)`。
- **L73 EN**: Defines macro `LIBC_TARGET_CPU_HAS_ARM_FPU_DOUBLE` for compile-time control or shorthand.
  **L73 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_ARM_FPU_DOUBLE`，用于编译期控制或简写。
- **L74 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_DOUBLE` for compile-time control or shorthand.
  **L74 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_DOUBLE`，用于编译期控制或简写。
- **L75 EN**: Closes the current preprocessor conditional block or header guard.
  **L75 CN**: 结束当前预处理条件块或头文件保护。
- **L76 EN**: Closes the current preprocessor conditional block or header guard.
  **L76 CN**: 结束当前预处理条件块或头文件保护。
- **L77 EN**: Blank line separating nearby declarations or logic.
  **L77 CN**: 空行，用于分隔相邻声明或逻辑。
- **L78 EN**: Starts a preprocessor conditional block: `#if defined(__ARM_NEON)`.
  **L78 CN**: 开始一个预处理条件块：`#if defined(__ARM_NEON)`。
- **L79 EN**: Defines macro `LIBC_TARGET_CPU_HAS_ARM_NEON` for compile-time control or shorthand.
  **L79 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_ARM_NEON`，用于编译期控制或简写。
- **L80 EN**: Closes the current preprocessor conditional block or header guard.
  **L80 CN**: 结束当前预处理条件块或头文件保护。
- **L81 EN**: Blank line separating nearby declarations or logic.
  **L81 CN**: 空行，用于分隔相邻声明或逻辑。
- **L82 EN**: Starts a preprocessor conditional block: `#if defined(__riscv_flen)`.
  **L82 CN**: 开始一个预处理条件块：`#if defined(__riscv_flen)`。
- **L83 EN**: Comment documents nearby intent or constraints: `https://github.com/riscv-non-isa/riscv-c-api-doc/blob/main/src/c-api.adoc`.
  **L83 CN**: 注释说明附近代码的意图或约束：`https://github.com/riscv-non-isa/riscv-c-api-doc/blob/main/src/c-api.adoc`。
- **L84 EN**: Starts a preprocessor conditional block: `#if defined(__riscv_zfhmin)`.
  **L84 CN**: 开始一个预处理条件块：`#if defined(__riscv_zfhmin)`。

### Lines 85-98

````cpp
#define LIBC_TARGET_CPU_HAS_RISCV_FPU_HALF
#define LIBC_TARGET_CPU_HAS_FPU_HALF
#endif // LIBC_TARGET_CPU_HAS_RISCV_FPU_HALF
#if (__riscv_flen >= 32)
#define LIBC_TARGET_CPU_HAS_RISCV_FPU_FLOAT
#define LIBC_TARGET_CPU_HAS_FPU_FLOAT
#endif // LIBC_TARGET_CPU_HAS_RISCV_FPU_FLOAT
#if (__riscv_flen >= 64)
#define LIBC_TARGET_CPU_HAS_RISCV_FPU_DOUBLE
#define LIBC_TARGET_CPU_HAS_FPU_DOUBLE
#endif // LIBC_TARGET_CPU_HAS_RISCV_FPU_DOUBLE
#endif // __riscv_flen

#if defined(__NVPTX__) || defined(__AMDGPU__) || defined(__SPIRV__)
````
- **L85 EN**: Defines macro `LIBC_TARGET_CPU_HAS_RISCV_FPU_HALF` for compile-time control or shorthand.
  **L85 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_RISCV_FPU_HALF`，用于编译期控制或简写。
- **L86 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_HALF` for compile-time control or shorthand.
  **L86 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_HALF`，用于编译期控制或简写。
- **L87 EN**: Closes the current preprocessor conditional block or header guard.
  **L87 CN**: 结束当前预处理条件块或头文件保护。
- **L88 EN**: Starts a preprocessor conditional block: `#if (__riscv_flen >= 32)`.
  **L88 CN**: 开始一个预处理条件块：`#if (__riscv_flen >= 32)`。
- **L89 EN**: Defines macro `LIBC_TARGET_CPU_HAS_RISCV_FPU_FLOAT` for compile-time control or shorthand.
  **L89 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_RISCV_FPU_FLOAT`，用于编译期控制或简写。
- **L90 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_FLOAT` for compile-time control or shorthand.
  **L90 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_FLOAT`，用于编译期控制或简写。
- **L91 EN**: Closes the current preprocessor conditional block or header guard.
  **L91 CN**: 结束当前预处理条件块或头文件保护。
- **L92 EN**: Starts a preprocessor conditional block: `#if (__riscv_flen >= 64)`.
  **L92 CN**: 开始一个预处理条件块：`#if (__riscv_flen >= 64)`。
- **L93 EN**: Defines macro `LIBC_TARGET_CPU_HAS_RISCV_FPU_DOUBLE` for compile-time control or shorthand.
  **L93 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_RISCV_FPU_DOUBLE`，用于编译期控制或简写。
- **L94 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_DOUBLE` for compile-time control or shorthand.
  **L94 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_DOUBLE`，用于编译期控制或简写。
- **L95 EN**: Closes the current preprocessor conditional block or header guard.
  **L95 CN**: 结束当前预处理条件块或头文件保护。
- **L96 EN**: Closes the current preprocessor conditional block or header guard.
  **L96 CN**: 结束当前预处理条件块或头文件保护。
- **L97 EN**: Blank line separating nearby declarations or logic.
  **L97 CN**: 空行，用于分隔相邻声明或逻辑。
- **L98 EN**: Starts a preprocessor conditional block: `#if defined(__NVPTX__) || defined(__AMDGPU__) || defined(__SPIRV__)`.
  **L98 CN**: 开始一个预处理条件块：`#if defined(__NVPTX__) || defined(__AMDGPU__) || defined(__SPIRV__)`。

### Lines 99-112

````cpp
#define LIBC_TARGET_CPU_HAS_FPU_FLOAT
#define LIBC_TARGET_CPU_HAS_FPU_DOUBLE
#endif

#if defined(__ARM_FEATURE_FMA) || (defined(__AVX2__) && defined(__FMA__)) ||   \
    defined(__NVPTX__) || defined(__AMDGPU__) || defined(__riscv_flen) ||      \
    defined(__SPIRV__)
#define LIBC_TARGET_CPU_HAS_FMA
// Provide a more fine-grained control of FMA instruction for ARM targets.
#if defined(LIBC_TARGET_CPU_HAS_FPU_HALF)
#define LIBC_TARGET_CPU_HAS_FMA_HALF
#endif // LIBC_TARGET_CPU_HAS_FMA_HALF
#if defined(LIBC_TARGET_CPU_HAS_FPU_FLOAT)
#define LIBC_TARGET_CPU_HAS_FMA_FLOAT
````
- **L99 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_FLOAT` for compile-time control or shorthand.
  **L99 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_FLOAT`，用于编译期控制或简写。
- **L100 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FPU_DOUBLE` for compile-time control or shorthand.
  **L100 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FPU_DOUBLE`，用于编译期控制或简写。
- **L101 EN**: Closes the current preprocessor conditional block or header guard.
  **L101 CN**: 结束当前预处理条件块或头文件保护。
- **L102 EN**: Blank line separating nearby declarations or logic.
  **L102 CN**: 空行，用于分隔相邻声明或逻辑。
- **L103 EN**: Starts a preprocessor conditional block: `#if defined(__ARM_FEATURE_FMA) || (defined(__AVX2__) && defined(__FMA__)) ||   \`.
  **L103 CN**: 开始一个预处理条件块：`#if defined(__ARM_FEATURE_FMA) || (defined(__AVX2__) && defined(__FMA__)) ||   \`。
- **L104 EN**: Continues logic associated with callable symbol `defined`.
  **L104 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L105 EN**: Continues logic associated with callable symbol `defined`.
  **L105 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L106 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FMA` for compile-time control or shorthand.
  **L106 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FMA`，用于编译期控制或简写。
- **L107 EN**: Comment documents nearby intent or constraints: `Provide a more fine-grained control of FMA instruction for ARM targets.`.
  **L107 CN**: 注释说明附近代码的意图或约束：`Provide a more fine-grained control of FMA instruction for ARM targets.`。
- **L108 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TARGET_CPU_HAS_FPU_HALF)`.
  **L108 CN**: 开始一个预处理条件块：`#if defined(LIBC_TARGET_CPU_HAS_FPU_HALF)`。
- **L109 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FMA_HALF` for compile-time control or shorthand.
  **L109 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FMA_HALF`，用于编译期控制或简写。
- **L110 EN**: Closes the current preprocessor conditional block or header guard.
  **L110 CN**: 结束当前预处理条件块或头文件保护。
- **L111 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TARGET_CPU_HAS_FPU_FLOAT)`.
  **L111 CN**: 开始一个预处理条件块：`#if defined(LIBC_TARGET_CPU_HAS_FPU_FLOAT)`。
- **L112 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FMA_FLOAT` for compile-time control or shorthand.
  **L112 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FMA_FLOAT`，用于编译期控制或简写。

### Lines 113-126

````cpp
#endif // LIBC_TARGET_CPU_HAS_FMA_FLOAT
#if defined(LIBC_TARGET_CPU_HAS_FPU_DOUBLE)
#define LIBC_TARGET_CPU_HAS_FMA_DOUBLE
#endif // LIBC_TARGET_CPU_HAS_FMA_DOUBLE
#endif

#if defined(LIBC_TARGET_ARCH_IS_AARCH64) ||                                    \
    (defined(LIBC_TARGET_ARCH_IS_X86_64) &&                                    \
     defined(LIBC_TARGET_CPU_HAS_SSE4_2))
#define LIBC_TARGET_CPU_HAS_NEAREST_INT
#endif

#if defined(LIBC_TARGET_ARCH_IS_AARCH64) || defined(LIBC_TARGET_ARCH_IS_GPU)
#define LIBC_TARGET_CPU_HAS_FAST_FLOAT16_OPS
````
- **L113 EN**: Closes the current preprocessor conditional block or header guard.
  **L113 CN**: 结束当前预处理条件块或头文件保护。
- **L114 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TARGET_CPU_HAS_FPU_DOUBLE)`.
  **L114 CN**: 开始一个预处理条件块：`#if defined(LIBC_TARGET_CPU_HAS_FPU_DOUBLE)`。
- **L115 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FMA_DOUBLE` for compile-time control or shorthand.
  **L115 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FMA_DOUBLE`，用于编译期控制或简写。
- **L116 EN**: Closes the current preprocessor conditional block or header guard.
  **L116 CN**: 结束当前预处理条件块或头文件保护。
- **L117 EN**: Closes the current preprocessor conditional block or header guard.
  **L117 CN**: 结束当前预处理条件块或头文件保护。
- **L118 EN**: Blank line separating nearby declarations or logic.
  **L118 CN**: 空行，用于分隔相邻声明或逻辑。
- **L119 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TARGET_ARCH_IS_AARCH64) ||                                    \`.
  **L119 CN**: 开始一个预处理条件块：`#if defined(LIBC_TARGET_ARCH_IS_AARCH64) ||                                    \`。
- **L120 EN**: Continues logic associated with callable symbol `defined`.
  **L120 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L121 EN**: Continues logic associated with callable symbol `defined`.
  **L121 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L122 EN**: Defines macro `LIBC_TARGET_CPU_HAS_NEAREST_INT` for compile-time control or shorthand.
  **L122 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_NEAREST_INT`，用于编译期控制或简写。
- **L123 EN**: Closes the current preprocessor conditional block or header guard.
  **L123 CN**: 结束当前预处理条件块或头文件保护。
- **L124 EN**: Blank line separating nearby declarations or logic.
  **L124 CN**: 空行，用于分隔相邻声明或逻辑。
- **L125 EN**: Starts a preprocessor conditional block: `#if defined(LIBC_TARGET_ARCH_IS_AARCH64) || defined(LIBC_TARGET_ARCH_IS_GPU)`.
  **L125 CN**: 开始一个预处理条件块：`#if defined(LIBC_TARGET_ARCH_IS_AARCH64) || defined(LIBC_TARGET_ARCH_IS_GPU)`。
- **L126 EN**: Defines macro `LIBC_TARGET_CPU_HAS_FAST_FLOAT16_OPS` for compile-time control or shorthand.
  **L126 CN**: 定义宏 `LIBC_TARGET_CPU_HAS_FAST_FLOAT16_OPS`，用于编译期控制或简写。

### Lines 127-129

````cpp
#endif

#endif // LLVM_LIBC_SRC___SUPPORT_MACROS_PROPERTIES_CPU_FEATURES_H
````
- **L127 EN**: Closes the current preprocessor conditional block or header guard.
  **L127 CN**: 结束当前预处理条件块或头文件保护。
- **L128 EN**: Blank line separating nearby declarations or logic.
  **L128 CN**: 空行，用于分隔相邻声明或逻辑。
- **L129 EN**: Closes the current preprocessor conditional block or header guard.
  **L129 CN**: 结束当前预处理条件块或头文件保护。

## Key Concepts / 关键概念

- **Compile-time configuration / 编译期配置**: Centralizes macros that describe compiler attributes, platform knobs, and internal ABI conventions. / 集中管理描述编译器属性、平台开关与内部 ABI 约定的宏。
- **Low-level libc support / 底层 libc 支撑**: Provides reusable building blocks such as allocation helpers, numeric formatting, or internal data structures. / 提供可复用的基础构件，例如分配辅助逻辑、数值格式化或内部数据结构。
- **Header contracts / 头文件契约**: Provides declarations, templates, or inline logic consumed by other translation units. / 提供供其他编译单元使用的声明、模板或内联逻辑。
- **Multiple-inclusion protection / 防重复包含保护**: Guards header contents against accidental repeated inclusion. / 保护头文件内容，防止被意外重复包含。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `architectures.h`
- **Dependency categories / 依赖类别**: nearby local declarations / 附近的本地声明 (1)

- `architectures.h`: Provides nearby local declarations. / 提供附近的本地声明。
