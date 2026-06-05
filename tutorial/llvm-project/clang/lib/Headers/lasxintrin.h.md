# lasxintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/lasxintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: LoongArch LASX intrinsics.
- **Purpose (CN)**: 提供 LoongArch LASX intrinsic 接口。
- **Line Count / 行数**: 3999

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===------------ lasxintrin.h - LoongArch LASX intrinsics -----------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef _LOONGSON_ASXINTRIN_H
#define _LOONGSON_ASXINTRIN_H 1

#include <lsxintrin.h>

#if defined(__loongarch_asx)

typedef signed char v32i8 __attribute__((vector_size(32), aligned(32)));
typedef signed char v32i8_b __attribute__((vector_size(32), aligned(1)));
typedef unsigned char v32u8 __attribute__((vector_size(32), aligned(32)));
typedef unsigned char v32u8_b __attribute__((vector_size(32), aligned(1)));
typedef short v16i16 __attribute__((vector_size(32), aligned(32)));
typedef short v16i16_h __attribute__((vector_size(32), aligned(2)));
typedef unsigned short v16u16 __attribute__((vector_size(32), aligned(32)));
typedef unsigned short v16u16_h __attribute__((vector_size(32), aligned(2)));
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Blank line separating nearby declarations or logic blocks.
  **L9 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef _LOONGSON_ASXINTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef _LOONGSON_ASXINTRIN_H`。
- **L11 EN**: Defines macro `_LOONGSON_ASXINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `_LOONGSON_ASXINTRIN_H`，用于条件编译、简写或 API 生成。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Includes <lsxintrin.h> to access related header declarations.
  **L13 CN**: 引入 <lsxintrin.h> 以使用相关头文件声明。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Starts a preprocessor conditional block: `#if defined(__loongarch_asx)`.
  **L15 CN**: 开始一个预处理条件块：`#if defined(__loongarch_asx)`。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Introduces an alias or helper declaration: `typedef signed char v32i8 __attribute__((vector_size(32), aligned(32)));`.
  **L17 CN**: 引入一条别名或辅助声明：`typedef signed char v32i8 __attribute__((vector_size(32), aligned(32)));`。
- **L18 EN**: Introduces an alias or helper declaration: `typedef signed char v32i8_b __attribute__((vector_size(32), aligned(1)));`.
  **L18 CN**: 引入一条别名或辅助声明：`typedef signed char v32i8_b __attribute__((vector_size(32), aligned(1)));`。
- **L19 EN**: Introduces an alias or helper declaration: `typedef unsigned char v32u8 __attribute__((vector_size(32), aligned(32)));`.
  **L19 CN**: 引入一条别名或辅助声明：`typedef unsigned char v32u8 __attribute__((vector_size(32), aligned(32)));`。
- **L20 EN**: Introduces an alias or helper declaration: `typedef unsigned char v32u8_b __attribute__((vector_size(32), aligned(1)));`.
  **L20 CN**: 引入一条别名或辅助声明：`typedef unsigned char v32u8_b __attribute__((vector_size(32), aligned(1)));`。
- **L21 EN**: Introduces an alias or helper declaration: `typedef short v16i16 __attribute__((vector_size(32), aligned(32)));`.
  **L21 CN**: 引入一条别名或辅助声明：`typedef short v16i16 __attribute__((vector_size(32), aligned(32)));`。
- **L22 EN**: Introduces an alias or helper declaration: `typedef short v16i16_h __attribute__((vector_size(32), aligned(2)));`.
  **L22 CN**: 引入一条别名或辅助声明：`typedef short v16i16_h __attribute__((vector_size(32), aligned(2)));`。
- **L23 EN**: Introduces an alias or helper declaration: `typedef unsigned short v16u16 __attribute__((vector_size(32), aligned(32)));`.
  **L23 CN**: 引入一条别名或辅助声明：`typedef unsigned short v16u16 __attribute__((vector_size(32), aligned(32)));`。
- **L24 EN**: Introduces an alias or helper declaration: `typedef unsigned short v16u16_h __attribute__((vector_size(32), aligned(2)));`.
  **L24 CN**: 引入一条别名或辅助声明：`typedef unsigned short v16u16_h __attribute__((vector_size(32), aligned(2)));`。

### Lines 25-48

````c
typedef int v8i32 __attribute__((vector_size(32), aligned(32)));
typedef int v8i32_w __attribute__((vector_size(32), aligned(4)));
typedef unsigned int v8u32 __attribute__((vector_size(32), aligned(32)));
typedef unsigned int v8u32_w __attribute__((vector_size(32), aligned(4)));
typedef long long v4i64 __attribute__((vector_size(32), aligned(32)));
typedef long long v4i64_d __attribute__((vector_size(32), aligned(8)));
typedef unsigned long long v4u64 __attribute__((vector_size(32), aligned(32)));
typedef unsigned long long v4u64_d __attribute__((vector_size(32), aligned(8)));
typedef float v8f32 __attribute__((vector_size(32), aligned(32)));
typedef float v8f32_w __attribute__((vector_size(32), aligned(4)));
typedef double v4f64 __attribute__((vector_size(32), aligned(32)));
typedef double v4f64_d __attribute__((vector_size(32), aligned(8)));

typedef double v4f64 __attribute__((vector_size(32), aligned(32)));
typedef double v4f64_d __attribute__((vector_size(32), aligned(8)));

typedef float __m256 __attribute__((__vector_size__(32), __may_alias__));
typedef long long __m256i __attribute__((__vector_size__(32), __may_alias__));
typedef double __m256d __attribute__((__vector_size__(32), __may_alias__));

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsll_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsll_b((v32i8)_1, (v32i8)_2);
````
- **L25 EN**: Introduces an alias or helper declaration: `typedef int v8i32 __attribute__((vector_size(32), aligned(32)));`.
  **L25 CN**: 引入一条别名或辅助声明：`typedef int v8i32 __attribute__((vector_size(32), aligned(32)));`。
- **L26 EN**: Introduces an alias or helper declaration: `typedef int v8i32_w __attribute__((vector_size(32), aligned(4)));`.
  **L26 CN**: 引入一条别名或辅助声明：`typedef int v8i32_w __attribute__((vector_size(32), aligned(4)));`。
- **L27 EN**: Introduces an alias or helper declaration: `typedef unsigned int v8u32 __attribute__((vector_size(32), aligned(32)));`.
  **L27 CN**: 引入一条别名或辅助声明：`typedef unsigned int v8u32 __attribute__((vector_size(32), aligned(32)));`。
- **L28 EN**: Introduces an alias or helper declaration: `typedef unsigned int v8u32_w __attribute__((vector_size(32), aligned(4)));`.
  **L28 CN**: 引入一条别名或辅助声明：`typedef unsigned int v8u32_w __attribute__((vector_size(32), aligned(4)));`。
- **L29 EN**: Introduces an alias or helper declaration: `typedef long long v4i64 __attribute__((vector_size(32), aligned(32)));`.
  **L29 CN**: 引入一条别名或辅助声明：`typedef long long v4i64 __attribute__((vector_size(32), aligned(32)));`。
- **L30 EN**: Introduces an alias or helper declaration: `typedef long long v4i64_d __attribute__((vector_size(32), aligned(8)));`.
  **L30 CN**: 引入一条别名或辅助声明：`typedef long long v4i64_d __attribute__((vector_size(32), aligned(8)));`。
- **L31 EN**: Introduces an alias or helper declaration: `typedef unsigned long long v4u64 __attribute__((vector_size(32), aligned(32)));`.
  **L31 CN**: 引入一条别名或辅助声明：`typedef unsigned long long v4u64 __attribute__((vector_size(32), aligned(32)));`。
- **L32 EN**: Introduces an alias or helper declaration: `typedef unsigned long long v4u64_d __attribute__((vector_size(32), aligned(8)));`.
  **L32 CN**: 引入一条别名或辅助声明：`typedef unsigned long long v4u64_d __attribute__((vector_size(32), aligned(8)));`。
- **L33 EN**: Introduces an alias or helper declaration: `typedef float v8f32 __attribute__((vector_size(32), aligned(32)));`.
  **L33 CN**: 引入一条别名或辅助声明：`typedef float v8f32 __attribute__((vector_size(32), aligned(32)));`。
- **L34 EN**: Introduces an alias or helper declaration: `typedef float v8f32_w __attribute__((vector_size(32), aligned(4)));`.
  **L34 CN**: 引入一条别名或辅助声明：`typedef float v8f32_w __attribute__((vector_size(32), aligned(4)));`。
- **L35 EN**: Introduces an alias or helper declaration: `typedef double v4f64 __attribute__((vector_size(32), aligned(32)));`.
  **L35 CN**: 引入一条别名或辅助声明：`typedef double v4f64 __attribute__((vector_size(32), aligned(32)));`。
- **L36 EN**: Introduces an alias or helper declaration: `typedef double v4f64_d __attribute__((vector_size(32), aligned(8)));`.
  **L36 CN**: 引入一条别名或辅助声明：`typedef double v4f64_d __attribute__((vector_size(32), aligned(8)));`。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Introduces an alias or helper declaration: `typedef double v4f64 __attribute__((vector_size(32), aligned(32)));`.
  **L38 CN**: 引入一条别名或辅助声明：`typedef double v4f64 __attribute__((vector_size(32), aligned(32)));`。
- **L39 EN**: Introduces an alias or helper declaration: `typedef double v4f64_d __attribute__((vector_size(32), aligned(8)));`.
  **L39 CN**: 引入一条别名或辅助声明：`typedef double v4f64_d __attribute__((vector_size(32), aligned(8)));`。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L41 EN**: Introduces an alias or helper declaration: `typedef float __m256 __attribute__((__vector_size__(32), __may_alias__));`.
  **L41 CN**: 引入一条别名或辅助声明：`typedef float __m256 __attribute__((__vector_size__(32), __may_alias__));`。
- **L42 EN**: Introduces an alias or helper declaration: `typedef long long __m256i __attribute__((__vector_size__(32), __may_alias__));`.
  **L42 CN**: 引入一条别名或辅助声明：`typedef long long __m256i __attribute__((__vector_size__(32), __may_alias__));`。
- **L43 EN**: Introduces an alias or helper declaration: `typedef double __m256d __attribute__((__vector_size__(32), __may_alias__));`.
  **L43 CN**: 引入一条别名或辅助声明：`typedef double __m256d __attribute__((__vector_size__(32), __may_alias__));`。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L45 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L45 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L46 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L46 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L47 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsll_b(__m256i _1, __m256i _2) {`.
  **L47 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsll_b(__m256i _1, __m256i _2) {`。
- **L48 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsll_b((v32i8)_1, (v32i8)_2)`.
  **L48 CN**: 以 `(__m256i)__builtin_lasx_xvsll_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 49-72

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsll_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsll_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsll_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsll_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsll_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsll_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvslli_b(/*__m256i*/ _1, /*ui3*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslli_b((v32i8)(_1), (_2)))

#define __lasx_xvslli_h(/*__m256i*/ _1, /*ui4*/ _2)                            \
````
- **L49 EN**: Closes the current lexical scope or compound statement.
  **L49 CN**: 结束当前词法作用域或复合语句块。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L51 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L51 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L52 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L52 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L53 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsll_h(__m256i _1, __m256i _2) {`.
  **L53 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsll_h(__m256i _1, __m256i _2) {`。
- **L54 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsll_h((v16i16)_1, (v16i16)_2)`.
  **L54 CN**: 以 `(__m256i)__builtin_lasx_xvsll_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L57 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L57 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L58 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L58 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L59 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsll_w(__m256i _1, __m256i _2) {`.
  **L59 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsll_w(__m256i _1, __m256i _2) {`。
- **L60 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsll_w((v8i32)_1, (v8i32)_2)`.
  **L60 CN**: 以 `(__m256i)__builtin_lasx_xvsll_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L63 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L64 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L64 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L65 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsll_d(__m256i _1, __m256i _2) {`.
  **L65 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsll_d(__m256i _1, __m256i _2) {`。
- **L66 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsll_d((v4i64)_1, (v4i64)_2)`.
  **L66 CN**: 以 `(__m256i)__builtin_lasx_xvsll_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L69 EN**: Defines macro `__lasx_xvslli_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L69 CN**: 定义宏 `__lasx_xvslli_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L70 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslli_b`.
  **L70 CN**: 继续与可调用符号 `__builtin_lasx_xvslli_b` 相关的逻辑。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Defines macro `__lasx_xvslli_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L72 CN**: 定义宏 `__lasx_xvslli_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 73-96

````c
  ((__m256i)__builtin_lasx_xvslli_h((v16i16)(_1), (_2)))

#define __lasx_xvslli_w(/*__m256i*/ _1, /*ui5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslli_w((v8i32)(_1), (_2)))

#define __lasx_xvslli_d(/*__m256i*/ _1, /*ui6*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslli_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsra_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsra_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsra_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsra_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsra_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsra_w((v8i32)_1, (v8i32)_2);
````
- **L73 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslli_h`.
  **L73 CN**: 继续与可调用符号 `__builtin_lasx_xvslli_h` 相关的逻辑。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L75 EN**: Defines macro `__lasx_xvslli_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L75 CN**: 定义宏 `__lasx_xvslli_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L76 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslli_w`.
  **L76 CN**: 继续与可调用符号 `__builtin_lasx_xvslli_w` 相关的逻辑。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L78 EN**: Defines macro `__lasx_xvslli_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L78 CN**: 定义宏 `__lasx_xvslli_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L79 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslli_d`.
  **L79 CN**: 继续与可调用符号 `__builtin_lasx_xvslli_d` 相关的逻辑。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L81 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L81 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L82 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L82 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L83 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsra_b(__m256i _1, __m256i _2) {`.
  **L83 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsra_b(__m256i _1, __m256i _2) {`。
- **L84 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsra_b((v32i8)_1, (v32i8)_2)`.
  **L84 CN**: 以 `(__m256i)__builtin_lasx_xvsra_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L85 EN**: Closes the current lexical scope or compound statement.
  **L85 CN**: 结束当前词法作用域或复合语句块。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L87 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L87 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L88 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L88 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L89 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsra_h(__m256i _1, __m256i _2) {`.
  **L89 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsra_h(__m256i _1, __m256i _2) {`。
- **L90 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsra_h((v16i16)_1, (v16i16)_2)`.
  **L90 CN**: 以 `(__m256i)__builtin_lasx_xvsra_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L93 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L93 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L94 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L94 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L95 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsra_w(__m256i _1, __m256i _2) {`.
  **L95 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsra_w(__m256i _1, __m256i _2) {`。
- **L96 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsra_w((v8i32)_1, (v8i32)_2)`.
  **L96 CN**: 以 `(__m256i)__builtin_lasx_xvsra_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 97-120

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsra_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsra_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvsrai_b(/*__m256i*/ _1, /*ui3*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsrai_b((v32i8)(_1), (_2)))

#define __lasx_xvsrai_h(/*__m256i*/ _1, /*ui4*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsrai_h((v16i16)(_1), (_2)))

#define __lasx_xvsrai_w(/*__m256i*/ _1, /*ui5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsrai_w((v8i32)(_1), (_2)))

#define __lasx_xvsrai_d(/*__m256i*/ _1, /*ui6*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsrai_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrar_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrar_b((v32i8)_1, (v32i8)_2);
````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L99 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L99 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L100 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L100 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L101 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsra_d(__m256i _1, __m256i _2) {`.
  **L101 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsra_d(__m256i _1, __m256i _2) {`。
- **L102 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsra_d((v4i64)_1, (v4i64)_2)`.
  **L102 CN**: 以 `(__m256i)__builtin_lasx_xvsra_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Defines macro `__lasx_xvsrai_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L105 CN**: 定义宏 `__lasx_xvsrai_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L106 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrai_b`.
  **L106 CN**: 继续与可调用符号 `__builtin_lasx_xvsrai_b` 相关的逻辑。
- **L107 EN**: Blank line separating nearby declarations or logic blocks.
  **L107 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L108 EN**: Defines macro `__lasx_xvsrai_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L108 CN**: 定义宏 `__lasx_xvsrai_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L109 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrai_h`.
  **L109 CN**: 继续与可调用符号 `__builtin_lasx_xvsrai_h` 相关的逻辑。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L111 EN**: Defines macro `__lasx_xvsrai_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L111 CN**: 定义宏 `__lasx_xvsrai_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L112 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrai_w`.
  **L112 CN**: 继续与可调用符号 `__builtin_lasx_xvsrai_w` 相关的逻辑。
- **L113 EN**: Blank line separating nearby declarations or logic blocks.
  **L113 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L114 EN**: Defines macro `__lasx_xvsrai_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L114 CN**: 定义宏 `__lasx_xvsrai_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L115 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrai_d`.
  **L115 CN**: 继续与可调用符号 `__builtin_lasx_xvsrai_d` 相关的逻辑。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L117 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L117 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L118 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L118 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L119 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrar_b(__m256i _1, __m256i _2) {`.
  **L119 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrar_b(__m256i _1, __m256i _2) {`。
- **L120 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrar_b((v32i8)_1, (v32i8)_2)`.
  **L120 CN**: 以 `(__m256i)__builtin_lasx_xvsrar_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 121-144

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrar_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrar_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrar_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrar_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrar_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrar_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvsrari_b(/*__m256i*/ _1, /*ui3*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsrari_b((v32i8)(_1), (_2)))

#define __lasx_xvsrari_h(/*__m256i*/ _1, /*ui4*/ _2)                           \
````
- **L121 EN**: Closes the current lexical scope or compound statement.
  **L121 CN**: 结束当前词法作用域或复合语句块。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L123 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L123 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L124 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L124 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrar_h(__m256i _1, __m256i _2) {`.
  **L125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrar_h(__m256i _1, __m256i _2) {`。
- **L126 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrar_h((v16i16)_1, (v16i16)_2)`.
  **L126 CN**: 以 `(__m256i)__builtin_lasx_xvsrar_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L127 EN**: Closes the current lexical scope or compound statement.
  **L127 CN**: 结束当前词法作用域或复合语句块。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L129 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L129 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L130 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L130 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L131 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrar_w(__m256i _1, __m256i _2) {`.
  **L131 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrar_w(__m256i _1, __m256i _2) {`。
- **L132 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrar_w((v8i32)_1, (v8i32)_2)`.
  **L132 CN**: 以 `(__m256i)__builtin_lasx_xvsrar_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Blank line separating nearby declarations or logic blocks.
  **L134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L135 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L135 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L136 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L136 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L137 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrar_d(__m256i _1, __m256i _2) {`.
  **L137 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrar_d(__m256i _1, __m256i _2) {`。
- **L138 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrar_d((v4i64)_1, (v4i64)_2)`.
  **L138 CN**: 以 `(__m256i)__builtin_lasx_xvsrar_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L141 EN**: Defines macro `__lasx_xvsrari_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L141 CN**: 定义宏 `__lasx_xvsrari_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L142 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrari_b`.
  **L142 CN**: 继续与可调用符号 `__builtin_lasx_xvsrari_b` 相关的逻辑。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L144 EN**: Defines macro `__lasx_xvsrari_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L144 CN**: 定义宏 `__lasx_xvsrari_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 145-168

````c
  ((__m256i)__builtin_lasx_xvsrari_h((v16i16)(_1), (_2)))

#define __lasx_xvsrari_w(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsrari_w((v8i32)(_1), (_2)))

#define __lasx_xvsrari_d(/*__m256i*/ _1, /*ui6*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsrari_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrl_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrl_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrl_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrl_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrl_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrl_w((v8i32)_1, (v8i32)_2);
````
- **L145 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrari_h`.
  **L145 CN**: 继续与可调用符号 `__builtin_lasx_xvsrari_h` 相关的逻辑。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L147 EN**: Defines macro `__lasx_xvsrari_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L147 CN**: 定义宏 `__lasx_xvsrari_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L148 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrari_w`.
  **L148 CN**: 继续与可调用符号 `__builtin_lasx_xvsrari_w` 相关的逻辑。
- **L149 EN**: Blank line separating nearby declarations or logic blocks.
  **L149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L150 EN**: Defines macro `__lasx_xvsrari_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L150 CN**: 定义宏 `__lasx_xvsrari_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L151 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrari_d`.
  **L151 CN**: 继续与可调用符号 `__builtin_lasx_xvsrari_d` 相关的逻辑。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L153 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L153 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L154 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L154 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L155 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrl_b(__m256i _1, __m256i _2) {`.
  **L155 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrl_b(__m256i _1, __m256i _2) {`。
- **L156 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrl_b((v32i8)_1, (v32i8)_2)`.
  **L156 CN**: 以 `(__m256i)__builtin_lasx_xvsrl_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L157 EN**: Closes the current lexical scope or compound statement.
  **L157 CN**: 结束当前词法作用域或复合语句块。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L159 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L159 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L160 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L160 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L161 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrl_h(__m256i _1, __m256i _2) {`.
  **L161 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrl_h(__m256i _1, __m256i _2) {`。
- **L162 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrl_h((v16i16)_1, (v16i16)_2)`.
  **L162 CN**: 以 `(__m256i)__builtin_lasx_xvsrl_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L165 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L166 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L166 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L167 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrl_w(__m256i _1, __m256i _2) {`.
  **L167 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrl_w(__m256i _1, __m256i _2) {`。
- **L168 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrl_w((v8i32)_1, (v8i32)_2)`.
  **L168 CN**: 以 `(__m256i)__builtin_lasx_xvsrl_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 169-192

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrl_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrl_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvsrli_b(/*__m256i*/ _1, /*ui3*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsrli_b((v32i8)(_1), (_2)))

#define __lasx_xvsrli_h(/*__m256i*/ _1, /*ui4*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsrli_h((v16i16)(_1), (_2)))

#define __lasx_xvsrli_w(/*__m256i*/ _1, /*ui5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsrli_w((v8i32)(_1), (_2)))

#define __lasx_xvsrli_d(/*__m256i*/ _1, /*ui6*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsrli_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrlr_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrlr_b((v32i8)_1, (v32i8)_2);
````
- **L169 EN**: Closes the current lexical scope or compound statement.
  **L169 CN**: 结束当前词法作用域或复合语句块。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L171 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L171 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L172 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L172 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L173 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrl_d(__m256i _1, __m256i _2) {`.
  **L173 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrl_d(__m256i _1, __m256i _2) {`。
- **L174 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrl_d((v4i64)_1, (v4i64)_2)`.
  **L174 CN**: 以 `(__m256i)__builtin_lasx_xvsrl_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L175 EN**: Closes the current lexical scope or compound statement.
  **L175 CN**: 结束当前词法作用域或复合语句块。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L177 EN**: Defines macro `__lasx_xvsrli_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L177 CN**: 定义宏 `__lasx_xvsrli_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L178 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrli_b`.
  **L178 CN**: 继续与可调用符号 `__builtin_lasx_xvsrli_b` 相关的逻辑。
- **L179 EN**: Blank line separating nearby declarations or logic blocks.
  **L179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L180 EN**: Defines macro `__lasx_xvsrli_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L180 CN**: 定义宏 `__lasx_xvsrli_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L181 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrli_h`.
  **L181 CN**: 继续与可调用符号 `__builtin_lasx_xvsrli_h` 相关的逻辑。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L183 EN**: Defines macro `__lasx_xvsrli_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L183 CN**: 定义宏 `__lasx_xvsrli_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L184 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrli_w`.
  **L184 CN**: 继续与可调用符号 `__builtin_lasx_xvsrli_w` 相关的逻辑。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Defines macro `__lasx_xvsrli_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L186 CN**: 定义宏 `__lasx_xvsrli_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L187 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrli_d`.
  **L187 CN**: 继续与可调用符号 `__builtin_lasx_xvsrli_d` 相关的逻辑。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L189 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L189 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L190 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L190 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L191 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrlr_b(__m256i _1, __m256i _2) {`.
  **L191 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrlr_b(__m256i _1, __m256i _2) {`。
- **L192 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrlr_b((v32i8)_1, (v32i8)_2)`.
  **L192 CN**: 以 `(__m256i)__builtin_lasx_xvsrlr_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 193-216

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrlr_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrlr_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrlr_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrlr_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrlr_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrlr_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvsrlri_b(/*__m256i*/ _1, /*ui3*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsrlri_b((v32i8)(_1), (_2)))

#define __lasx_xvsrlri_h(/*__m256i*/ _1, /*ui4*/ _2)                           \
````
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L195 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L195 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L196 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L196 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L197 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrlr_h(__m256i _1, __m256i _2) {`.
  **L197 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrlr_h(__m256i _1, __m256i _2) {`。
- **L198 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrlr_h((v16i16)_1, (v16i16)_2)`.
  **L198 CN**: 以 `(__m256i)__builtin_lasx_xvsrlr_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L201 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L201 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L202 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L202 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L203 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrlr_w(__m256i _1, __m256i _2) {`.
  **L203 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrlr_w(__m256i _1, __m256i _2) {`。
- **L204 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrlr_w((v8i32)_1, (v8i32)_2)`.
  **L204 CN**: 以 `(__m256i)__builtin_lasx_xvsrlr_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L205 EN**: Closes the current lexical scope or compound statement.
  **L205 CN**: 结束当前词法作用域或复合语句块。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L207 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L207 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L208 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L208 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L209 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrlr_d(__m256i _1, __m256i _2) {`.
  **L209 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrlr_d(__m256i _1, __m256i _2) {`。
- **L210 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrlr_d((v4i64)_1, (v4i64)_2)`.
  **L210 CN**: 以 `(__m256i)__builtin_lasx_xvsrlr_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L211 EN**: Closes the current lexical scope or compound statement.
  **L211 CN**: 结束当前词法作用域或复合语句块。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L213 EN**: Defines macro `__lasx_xvsrlri_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L213 CN**: 定义宏 `__lasx_xvsrlri_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L214 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlri_b`.
  **L214 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlri_b` 相关的逻辑。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L216 EN**: Defines macro `__lasx_xvsrlri_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L216 CN**: 定义宏 `__lasx_xvsrlri_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 217-240

````c
  ((__m256i)__builtin_lasx_xvsrlri_h((v16i16)(_1), (_2)))

#define __lasx_xvsrlri_w(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsrlri_w((v8i32)(_1), (_2)))

#define __lasx_xvsrlri_d(/*__m256i*/ _1, /*ui6*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsrlri_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitclr_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitclr_b((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitclr_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitclr_h((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitclr_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitclr_w((v8u32)_1, (v8u32)_2);
````
- **L217 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlri_h`.
  **L217 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlri_h` 相关的逻辑。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L219 EN**: Defines macro `__lasx_xvsrlri_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L219 CN**: 定义宏 `__lasx_xvsrlri_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L220 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlri_w`.
  **L220 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlri_w` 相关的逻辑。
- **L221 EN**: Blank line separating nearby declarations or logic blocks.
  **L221 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L222 EN**: Defines macro `__lasx_xvsrlri_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L222 CN**: 定义宏 `__lasx_xvsrlri_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L223 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlri_d`.
  **L223 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlri_d` 相关的逻辑。
- **L224 EN**: Blank line separating nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L225 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L225 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L226 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L226 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitclr_b(__m256i _1, __m256i _2) {`.
  **L227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitclr_b(__m256i _1, __m256i _2) {`。
- **L228 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitclr_b((v32u8)_1, (v32u8)_2)`.
  **L228 CN**: 以 `(__m256i)__builtin_lasx_xvbitclr_b((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L229 EN**: Closes the current lexical scope or compound statement.
  **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L231 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L231 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L232 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L232 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L233 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitclr_h(__m256i _1, __m256i _2) {`.
  **L233 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitclr_h(__m256i _1, __m256i _2) {`。
- **L234 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitclr_h((v16u16)_1, (v16u16)_2)`.
  **L234 CN**: 以 `(__m256i)__builtin_lasx_xvbitclr_h((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L235 EN**: Closes the current lexical scope or compound statement.
  **L235 CN**: 结束当前词法作用域或复合语句块。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L237 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L237 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L238 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L238 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L239 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitclr_w(__m256i _1, __m256i _2) {`.
  **L239 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitclr_w(__m256i _1, __m256i _2) {`。
- **L240 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitclr_w((v8u32)_1, (v8u32)_2)`.
  **L240 CN**: 以 `(__m256i)__builtin_lasx_xvbitclr_w((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 241-264

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitclr_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitclr_d((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvbitclri_b(/*__m256i*/ _1, /*ui3*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitclri_b((v32u8)(_1), (_2)))

#define __lasx_xvbitclri_h(/*__m256i*/ _1, /*ui4*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitclri_h((v16u16)(_1), (_2)))

#define __lasx_xvbitclri_w(/*__m256i*/ _1, /*ui5*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitclri_w((v8u32)(_1), (_2)))

#define __lasx_xvbitclri_d(/*__m256i*/ _1, /*ui6*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitclri_d((v4u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitset_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitset_b((v32u8)_1, (v32u8)_2);
````
- **L241 EN**: Closes the current lexical scope or compound statement.
  **L241 CN**: 结束当前词法作用域或复合语句块。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L243 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L243 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L244 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L244 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L245 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitclr_d(__m256i _1, __m256i _2) {`.
  **L245 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitclr_d(__m256i _1, __m256i _2) {`。
- **L246 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitclr_d((v4u64)_1, (v4u64)_2)`.
  **L246 CN**: 以 `(__m256i)__builtin_lasx_xvbitclr_d((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L247 EN**: Closes the current lexical scope or compound statement.
  **L247 CN**: 结束当前词法作用域或复合语句块。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L249 EN**: Defines macro `__lasx_xvbitclri_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L249 CN**: 定义宏 `__lasx_xvbitclri_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L250 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitclri_b`.
  **L250 CN**: 继续与可调用符号 `__builtin_lasx_xvbitclri_b` 相关的逻辑。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L252 EN**: Defines macro `__lasx_xvbitclri_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L252 CN**: 定义宏 `__lasx_xvbitclri_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L253 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitclri_h`.
  **L253 CN**: 继续与可调用符号 `__builtin_lasx_xvbitclri_h` 相关的逻辑。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L255 EN**: Defines macro `__lasx_xvbitclri_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L255 CN**: 定义宏 `__lasx_xvbitclri_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L256 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitclri_w`.
  **L256 CN**: 继续与可调用符号 `__builtin_lasx_xvbitclri_w` 相关的逻辑。
- **L257 EN**: Blank line separating nearby declarations or logic blocks.
  **L257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L258 EN**: Defines macro `__lasx_xvbitclri_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L258 CN**: 定义宏 `__lasx_xvbitclri_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L259 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitclri_d`.
  **L259 CN**: 继续与可调用符号 `__builtin_lasx_xvbitclri_d` 相关的逻辑。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L261 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L261 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L262 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L262 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L263 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitset_b(__m256i _1, __m256i _2) {`.
  **L263 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitset_b(__m256i _1, __m256i _2) {`。
- **L264 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitset_b((v32u8)_1, (v32u8)_2)`.
  **L264 CN**: 以 `(__m256i)__builtin_lasx_xvbitset_b((v32u8)_1, (v32u8)_2)` 从当前函数返回。

### Lines 265-288

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitset_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitset_h((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitset_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitset_w((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitset_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitset_d((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvbitseti_b(/*__m256i*/ _1, /*ui3*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitseti_b((v32u8)(_1), (_2)))

#define __lasx_xvbitseti_h(/*__m256i*/ _1, /*ui4*/ _2)                         \
````
- **L265 EN**: Closes the current lexical scope or compound statement.
  **L265 CN**: 结束当前词法作用域或复合语句块。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L267 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L267 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L268 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L268 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L269 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitset_h(__m256i _1, __m256i _2) {`.
  **L269 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitset_h(__m256i _1, __m256i _2) {`。
- **L270 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitset_h((v16u16)_1, (v16u16)_2)`.
  **L270 CN**: 以 `(__m256i)__builtin_lasx_xvbitset_h((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L273 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L273 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L274 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L274 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L275 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitset_w(__m256i _1, __m256i _2) {`.
  **L275 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitset_w(__m256i _1, __m256i _2) {`。
- **L276 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitset_w((v8u32)_1, (v8u32)_2)`.
  **L276 CN**: 以 `(__m256i)__builtin_lasx_xvbitset_w((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L277 EN**: Closes the current lexical scope or compound statement.
  **L277 CN**: 结束当前词法作用域或复合语句块。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L279 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L279 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L280 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L280 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L281 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitset_d(__m256i _1, __m256i _2) {`.
  **L281 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitset_d(__m256i _1, __m256i _2) {`。
- **L282 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitset_d((v4u64)_1, (v4u64)_2)`.
  **L282 CN**: 以 `(__m256i)__builtin_lasx_xvbitset_d((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L285 EN**: Defines macro `__lasx_xvbitseti_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L285 CN**: 定义宏 `__lasx_xvbitseti_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L286 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitseti_b`.
  **L286 CN**: 继续与可调用符号 `__builtin_lasx_xvbitseti_b` 相关的逻辑。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L288 EN**: Defines macro `__lasx_xvbitseti_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L288 CN**: 定义宏 `__lasx_xvbitseti_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 289-312

````c
  ((__m256i)__builtin_lasx_xvbitseti_h((v16u16)(_1), (_2)))

#define __lasx_xvbitseti_w(/*__m256i*/ _1, /*ui5*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitseti_w((v8u32)(_1), (_2)))

#define __lasx_xvbitseti_d(/*__m256i*/ _1, /*ui6*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitseti_d((v4u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitrev_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitrev_b((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitrev_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitrev_h((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitrev_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitrev_w((v8u32)_1, (v8u32)_2);
````
- **L289 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitseti_h`.
  **L289 CN**: 继续与可调用符号 `__builtin_lasx_xvbitseti_h` 相关的逻辑。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Defines macro `__lasx_xvbitseti_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L291 CN**: 定义宏 `__lasx_xvbitseti_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L292 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitseti_w`.
  **L292 CN**: 继续与可调用符号 `__builtin_lasx_xvbitseti_w` 相关的逻辑。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L294 EN**: Defines macro `__lasx_xvbitseti_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L294 CN**: 定义宏 `__lasx_xvbitseti_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L295 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitseti_d`.
  **L295 CN**: 继续与可调用符号 `__builtin_lasx_xvbitseti_d` 相关的逻辑。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L297 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L298 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L298 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L299 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitrev_b(__m256i _1, __m256i _2) {`.
  **L299 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitrev_b(__m256i _1, __m256i _2) {`。
- **L300 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitrev_b((v32u8)_1, (v32u8)_2)`.
  **L300 CN**: 以 `(__m256i)__builtin_lasx_xvbitrev_b((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L301 EN**: Closes the current lexical scope or compound statement.
  **L301 CN**: 结束当前词法作用域或复合语句块。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L303 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L303 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L304 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L304 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L305 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitrev_h(__m256i _1, __m256i _2) {`.
  **L305 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitrev_h(__m256i _1, __m256i _2) {`。
- **L306 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitrev_h((v16u16)_1, (v16u16)_2)`.
  **L306 CN**: 以 `(__m256i)__builtin_lasx_xvbitrev_h((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L307 EN**: Closes the current lexical scope or compound statement.
  **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L309 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L309 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L310 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L310 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L311 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitrev_w(__m256i _1, __m256i _2) {`.
  **L311 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitrev_w(__m256i _1, __m256i _2) {`。
- **L312 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitrev_w((v8u32)_1, (v8u32)_2)`.
  **L312 CN**: 以 `(__m256i)__builtin_lasx_xvbitrev_w((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 313-336

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitrev_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvbitrev_d((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvbitrevi_b(/*__m256i*/ _1, /*ui3*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitrevi_b((v32u8)(_1), (_2)))

#define __lasx_xvbitrevi_h(/*__m256i*/ _1, /*ui4*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitrevi_h((v16u16)(_1), (_2)))

#define __lasx_xvbitrevi_w(/*__m256i*/ _1, /*ui5*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitrevi_w((v8u32)(_1), (_2)))

#define __lasx_xvbitrevi_d(/*__m256i*/ _1, /*ui6*/ _2)                         \
  ((__m256i)__builtin_lasx_xvbitrevi_d((v4u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadd_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadd_b((v32i8)_1, (v32i8)_2);
````
- **L313 EN**: Closes the current lexical scope or compound statement.
  **L313 CN**: 结束当前词法作用域或复合语句块。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L315 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L316 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L316 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L317 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitrev_d(__m256i _1, __m256i _2) {`.
  **L317 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitrev_d(__m256i _1, __m256i _2) {`。
- **L318 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitrev_d((v4u64)_1, (v4u64)_2)`.
  **L318 CN**: 以 `(__m256i)__builtin_lasx_xvbitrev_d((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L319 EN**: Closes the current lexical scope or compound statement.
  **L319 CN**: 结束当前词法作用域或复合语句块。
- **L320 EN**: Blank line separating nearby declarations or logic blocks.
  **L320 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L321 EN**: Defines macro `__lasx_xvbitrevi_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L321 CN**: 定义宏 `__lasx_xvbitrevi_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L322 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitrevi_b`.
  **L322 CN**: 继续与可调用符号 `__builtin_lasx_xvbitrevi_b` 相关的逻辑。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L324 EN**: Defines macro `__lasx_xvbitrevi_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L324 CN**: 定义宏 `__lasx_xvbitrevi_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L325 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitrevi_h`.
  **L325 CN**: 继续与可调用符号 `__builtin_lasx_xvbitrevi_h` 相关的逻辑。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Defines macro `__lasx_xvbitrevi_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L327 CN**: 定义宏 `__lasx_xvbitrevi_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L328 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitrevi_w`.
  **L328 CN**: 继续与可调用符号 `__builtin_lasx_xvbitrevi_w` 相关的逻辑。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L330 EN**: Defines macro `__lasx_xvbitrevi_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L330 CN**: 定义宏 `__lasx_xvbitrevi_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L331 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitrevi_d`.
  **L331 CN**: 继续与可调用符号 `__builtin_lasx_xvbitrevi_d` 相关的逻辑。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L333 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L334 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L334 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L335 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadd_b(__m256i _1, __m256i _2) {`.
  **L335 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadd_b(__m256i _1, __m256i _2) {`。
- **L336 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadd_b((v32i8)_1, (v32i8)_2)`.
  **L336 CN**: 以 `(__m256i)__builtin_lasx_xvadd_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 337-360

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadd_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadd_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadd_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadd_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadd_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadd_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvaddi_bu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvaddi_bu((v32i8)(_1), (_2)))

#define __lasx_xvaddi_hu(/*__m256i*/ _1, /*ui5*/ _2)                           \
````
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L339 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L340 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L340 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L341 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadd_h(__m256i _1, __m256i _2) {`.
  **L341 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadd_h(__m256i _1, __m256i _2) {`。
- **L342 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadd_h((v16i16)_1, (v16i16)_2)`.
  **L342 CN**: 以 `(__m256i)__builtin_lasx_xvadd_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L343 EN**: Closes the current lexical scope or compound statement.
  **L343 CN**: 结束当前词法作用域或复合语句块。
- **L344 EN**: Blank line separating nearby declarations or logic blocks.
  **L344 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L345 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L345 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L346 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L346 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L347 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadd_w(__m256i _1, __m256i _2) {`.
  **L347 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadd_w(__m256i _1, __m256i _2) {`。
- **L348 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadd_w((v8i32)_1, (v8i32)_2)`.
  **L348 CN**: 以 `(__m256i)__builtin_lasx_xvadd_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L349 EN**: Closes the current lexical scope or compound statement.
  **L349 CN**: 结束当前词法作用域或复合语句块。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L351 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L351 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L352 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L352 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L353 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadd_d(__m256i _1, __m256i _2) {`.
  **L353 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadd_d(__m256i _1, __m256i _2) {`。
- **L354 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadd_d((v4i64)_1, (v4i64)_2)`.
  **L354 CN**: 以 `(__m256i)__builtin_lasx_xvadd_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L357 EN**: Defines macro `__lasx_xvaddi_bu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L357 CN**: 定义宏 `__lasx_xvaddi_bu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L358 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvaddi_bu`.
  **L358 CN**: 继续与可调用符号 `__builtin_lasx_xvaddi_bu` 相关的逻辑。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L360 EN**: Defines macro `__lasx_xvaddi_hu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L360 CN**: 定义宏 `__lasx_xvaddi_hu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 361-384

````c
  ((__m256i)__builtin_lasx_xvaddi_hu((v16i16)(_1), (_2)))

#define __lasx_xvaddi_wu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvaddi_wu((v8i32)(_1), (_2)))

#define __lasx_xvaddi_du(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvaddi_du((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsub_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsub_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsub_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsub_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsub_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsub_w((v8i32)_1, (v8i32)_2);
````
- **L361 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvaddi_hu`.
  **L361 CN**: 继续与可调用符号 `__builtin_lasx_xvaddi_hu` 相关的逻辑。
- **L362 EN**: Blank line separating nearby declarations or logic blocks.
  **L362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L363 EN**: Defines macro `__lasx_xvaddi_wu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L363 CN**: 定义宏 `__lasx_xvaddi_wu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L364 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvaddi_wu`.
  **L364 CN**: 继续与可调用符号 `__builtin_lasx_xvaddi_wu` 相关的逻辑。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L366 EN**: Defines macro `__lasx_xvaddi_du(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L366 CN**: 定义宏 `__lasx_xvaddi_du(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L367 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvaddi_du`.
  **L367 CN**: 继续与可调用符号 `__builtin_lasx_xvaddi_du` 相关的逻辑。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L369 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L369 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L370 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L370 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L371 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsub_b(__m256i _1, __m256i _2) {`.
  **L371 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsub_b(__m256i _1, __m256i _2) {`。
- **L372 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsub_b((v32i8)_1, (v32i8)_2)`.
  **L372 CN**: 以 `(__m256i)__builtin_lasx_xvsub_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L375 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L375 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L376 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L376 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L377 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsub_h(__m256i _1, __m256i _2) {`.
  **L377 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsub_h(__m256i _1, __m256i _2) {`。
- **L378 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsub_h((v16i16)_1, (v16i16)_2)`.
  **L378 CN**: 以 `(__m256i)__builtin_lasx_xvsub_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L379 EN**: Closes the current lexical scope or compound statement.
  **L379 CN**: 结束当前词法作用域或复合语句块。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L381 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L381 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L382 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L382 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L383 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsub_w(__m256i _1, __m256i _2) {`.
  **L383 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsub_w(__m256i _1, __m256i _2) {`。
- **L384 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsub_w((v8i32)_1, (v8i32)_2)`.
  **L384 CN**: 以 `(__m256i)__builtin_lasx_xvsub_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 385-408

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsub_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsub_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvsubi_bu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsubi_bu((v32i8)(_1), (_2)))

#define __lasx_xvsubi_hu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsubi_hu((v16i16)(_1), (_2)))

#define __lasx_xvsubi_wu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsubi_wu((v8i32)(_1), (_2)))

#define __lasx_xvsubi_du(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvsubi_du((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmax_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmax_b((v32i8)_1, (v32i8)_2);
````
- **L385 EN**: Closes the current lexical scope or compound statement.
  **L385 CN**: 结束当前词法作用域或复合语句块。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L387 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L387 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L388 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L388 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L389 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsub_d(__m256i _1, __m256i _2) {`.
  **L389 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsub_d(__m256i _1, __m256i _2) {`。
- **L390 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsub_d((v4i64)_1, (v4i64)_2)`.
  **L390 CN**: 以 `(__m256i)__builtin_lasx_xvsub_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L391 EN**: Closes the current lexical scope or compound statement.
  **L391 CN**: 结束当前词法作用域或复合语句块。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L393 EN**: Defines macro `__lasx_xvsubi_bu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L393 CN**: 定义宏 `__lasx_xvsubi_bu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L394 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsubi_bu`.
  **L394 CN**: 继续与可调用符号 `__builtin_lasx_xvsubi_bu` 相关的逻辑。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Defines macro `__lasx_xvsubi_hu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L396 CN**: 定义宏 `__lasx_xvsubi_hu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L397 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsubi_hu`.
  **L397 CN**: 继续与可调用符号 `__builtin_lasx_xvsubi_hu` 相关的逻辑。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L399 EN**: Defines macro `__lasx_xvsubi_wu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L399 CN**: 定义宏 `__lasx_xvsubi_wu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L400 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsubi_wu`.
  **L400 CN**: 继续与可调用符号 `__builtin_lasx_xvsubi_wu` 相关的逻辑。
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L402 EN**: Defines macro `__lasx_xvsubi_du(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L402 CN**: 定义宏 `__lasx_xvsubi_du(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L403 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsubi_du`.
  **L403 CN**: 继续与可调用符号 `__builtin_lasx_xvsubi_du` 相关的逻辑。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L405 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L405 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L406 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L406 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L407 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmax_b(__m256i _1, __m256i _2) {`.
  **L407 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmax_b(__m256i _1, __m256i _2) {`。
- **L408 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmax_b((v32i8)_1, (v32i8)_2)`.
  **L408 CN**: 以 `(__m256i)__builtin_lasx_xvmax_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 409-432

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmax_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmax_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmax_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmax_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmax_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmax_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvmaxi_b(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvmaxi_b((v32i8)(_1), (_2)))

#define __lasx_xvmaxi_h(/*__m256i*/ _1, /*si5*/ _2)                            \
````
- **L409 EN**: Closes the current lexical scope or compound statement.
  **L409 CN**: 结束当前词法作用域或复合语句块。
- **L410 EN**: Blank line separating nearby declarations or logic blocks.
  **L410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L411 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L411 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L412 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L412 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L413 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmax_h(__m256i _1, __m256i _2) {`.
  **L413 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmax_h(__m256i _1, __m256i _2) {`。
- **L414 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmax_h((v16i16)_1, (v16i16)_2)`.
  **L414 CN**: 以 `(__m256i)__builtin_lasx_xvmax_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L415 EN**: Closes the current lexical scope or compound statement.
  **L415 CN**: 结束当前词法作用域或复合语句块。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L417 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L417 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L418 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L418 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L419 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmax_w(__m256i _1, __m256i _2) {`.
  **L419 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmax_w(__m256i _1, __m256i _2) {`。
- **L420 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmax_w((v8i32)_1, (v8i32)_2)`.
  **L420 CN**: 以 `(__m256i)__builtin_lasx_xvmax_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L421 EN**: Closes the current lexical scope or compound statement.
  **L421 CN**: 结束当前词法作用域或复合语句块。
- **L422 EN**: Blank line separating nearby declarations or logic blocks.
  **L422 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L423 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L423 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L424 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L424 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L425 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmax_d(__m256i _1, __m256i _2) {`.
  **L425 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmax_d(__m256i _1, __m256i _2) {`。
- **L426 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmax_d((v4i64)_1, (v4i64)_2)`.
  **L426 CN**: 以 `(__m256i)__builtin_lasx_xvmax_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L427 EN**: Closes the current lexical scope or compound statement.
  **L427 CN**: 结束当前词法作用域或复合语句块。
- **L428 EN**: Blank line separating nearby declarations or logic blocks.
  **L428 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L429 EN**: Defines macro `__lasx_xvmaxi_b(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L429 CN**: 定义宏 `__lasx_xvmaxi_b(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L430 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmaxi_b`.
  **L430 CN**: 继续与可调用符号 `__builtin_lasx_xvmaxi_b` 相关的逻辑。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L432 EN**: Defines macro `__lasx_xvmaxi_h(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L432 CN**: 定义宏 `__lasx_xvmaxi_h(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 433-456

````c
  ((__m256i)__builtin_lasx_xvmaxi_h((v16i16)(_1), (_2)))

#define __lasx_xvmaxi_w(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvmaxi_w((v8i32)(_1), (_2)))

#define __lasx_xvmaxi_d(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvmaxi_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmax_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmax_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmax_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmax_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmax_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmax_wu((v8u32)_1, (v8u32)_2);
````
- **L433 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmaxi_h`.
  **L433 CN**: 继续与可调用符号 `__builtin_lasx_xvmaxi_h` 相关的逻辑。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L435 EN**: Defines macro `__lasx_xvmaxi_w(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L435 CN**: 定义宏 `__lasx_xvmaxi_w(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L436 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmaxi_w`.
  **L436 CN**: 继续与可调用符号 `__builtin_lasx_xvmaxi_w` 相关的逻辑。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Defines macro `__lasx_xvmaxi_d(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L438 CN**: 定义宏 `__lasx_xvmaxi_d(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L439 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmaxi_d`.
  **L439 CN**: 继续与可调用符号 `__builtin_lasx_xvmaxi_d` 相关的逻辑。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L441 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L441 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L442 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L442 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L443 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmax_bu(__m256i _1, __m256i _2) {`.
  **L443 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmax_bu(__m256i _1, __m256i _2) {`。
- **L444 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmax_bu((v32u8)_1, (v32u8)_2)`.
  **L444 CN**: 以 `(__m256i)__builtin_lasx_xvmax_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L445 EN**: Closes the current lexical scope or compound statement.
  **L445 CN**: 结束当前词法作用域或复合语句块。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L447 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L447 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L448 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L448 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L449 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmax_hu(__m256i _1, __m256i _2) {`.
  **L449 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmax_hu(__m256i _1, __m256i _2) {`。
- **L450 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmax_hu((v16u16)_1, (v16u16)_2)`.
  **L450 CN**: 以 `(__m256i)__builtin_lasx_xvmax_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L451 EN**: Closes the current lexical scope or compound statement.
  **L451 CN**: 结束当前词法作用域或复合语句块。
- **L452 EN**: Blank line separating nearby declarations or logic blocks.
  **L452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L453 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L453 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L454 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L454 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L455 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmax_wu(__m256i _1, __m256i _2) {`.
  **L455 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmax_wu(__m256i _1, __m256i _2) {`。
- **L456 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmax_wu((v8u32)_1, (v8u32)_2)`.
  **L456 CN**: 以 `(__m256i)__builtin_lasx_xvmax_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 457-480

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmax_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmax_du((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvmaxi_bu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvmaxi_bu((v32u8)(_1), (_2)))

#define __lasx_xvmaxi_hu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvmaxi_hu((v16u16)(_1), (_2)))

#define __lasx_xvmaxi_wu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvmaxi_wu((v8u32)(_1), (_2)))

#define __lasx_xvmaxi_du(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvmaxi_du((v4u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmin_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmin_b((v32i8)_1, (v32i8)_2);
````
- **L457 EN**: Closes the current lexical scope or compound statement.
  **L457 CN**: 结束当前词法作用域或复合语句块。
- **L458 EN**: Blank line separating nearby declarations or logic blocks.
  **L458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L459 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L459 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L460 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L460 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L461 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmax_du(__m256i _1, __m256i _2) {`.
  **L461 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmax_du(__m256i _1, __m256i _2) {`。
- **L462 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmax_du((v4u64)_1, (v4u64)_2)`.
  **L462 CN**: 以 `(__m256i)__builtin_lasx_xvmax_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L463 EN**: Closes the current lexical scope or compound statement.
  **L463 CN**: 结束当前词法作用域或复合语句块。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L465 EN**: Defines macro `__lasx_xvmaxi_bu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L465 CN**: 定义宏 `__lasx_xvmaxi_bu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L466 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmaxi_bu`.
  **L466 CN**: 继续与可调用符号 `__builtin_lasx_xvmaxi_bu` 相关的逻辑。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Defines macro `__lasx_xvmaxi_hu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L468 CN**: 定义宏 `__lasx_xvmaxi_hu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L469 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmaxi_hu`.
  **L469 CN**: 继续与可调用符号 `__builtin_lasx_xvmaxi_hu` 相关的逻辑。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Defines macro `__lasx_xvmaxi_wu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L471 CN**: 定义宏 `__lasx_xvmaxi_wu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L472 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmaxi_wu`.
  **L472 CN**: 继续与可调用符号 `__builtin_lasx_xvmaxi_wu` 相关的逻辑。
- **L473 EN**: Blank line separating nearby declarations or logic blocks.
  **L473 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L474 EN**: Defines macro `__lasx_xvmaxi_du(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L474 CN**: 定义宏 `__lasx_xvmaxi_du(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L475 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmaxi_du`.
  **L475 CN**: 继续与可调用符号 `__builtin_lasx_xvmaxi_du` 相关的逻辑。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L477 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L477 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L478 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L478 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L479 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmin_b(__m256i _1, __m256i _2) {`.
  **L479 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmin_b(__m256i _1, __m256i _2) {`。
- **L480 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmin_b((v32i8)_1, (v32i8)_2)`.
  **L480 CN**: 以 `(__m256i)__builtin_lasx_xvmin_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 481-504

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmin_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmin_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmin_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmin_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmin_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmin_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvmini_b(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvmini_b((v32i8)(_1), (_2)))

#define __lasx_xvmini_h(/*__m256i*/ _1, /*si5*/ _2)                            \
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L483 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L483 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L484 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L484 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L485 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmin_h(__m256i _1, __m256i _2) {`.
  **L485 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmin_h(__m256i _1, __m256i _2) {`。
- **L486 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmin_h((v16i16)_1, (v16i16)_2)`.
  **L486 CN**: 以 `(__m256i)__builtin_lasx_xvmin_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L489 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L489 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L490 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L490 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L491 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmin_w(__m256i _1, __m256i _2) {`.
  **L491 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmin_w(__m256i _1, __m256i _2) {`。
- **L492 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmin_w((v8i32)_1, (v8i32)_2)`.
  **L492 CN**: 以 `(__m256i)__builtin_lasx_xvmin_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L493 EN**: Closes the current lexical scope or compound statement.
  **L493 CN**: 结束当前词法作用域或复合语句块。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L495 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L495 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L496 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L496 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L497 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmin_d(__m256i _1, __m256i _2) {`.
  **L497 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmin_d(__m256i _1, __m256i _2) {`。
- **L498 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmin_d((v4i64)_1, (v4i64)_2)`.
  **L498 CN**: 以 `(__m256i)__builtin_lasx_xvmin_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Blank line separating nearby declarations or logic blocks.
  **L500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L501 EN**: Defines macro `__lasx_xvmini_b(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L501 CN**: 定义宏 `__lasx_xvmini_b(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L502 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmini_b`.
  **L502 CN**: 继续与可调用符号 `__builtin_lasx_xvmini_b` 相关的逻辑。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L504 EN**: Defines macro `__lasx_xvmini_h(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L504 CN**: 定义宏 `__lasx_xvmini_h(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 505-528

````c
  ((__m256i)__builtin_lasx_xvmini_h((v16i16)(_1), (_2)))

#define __lasx_xvmini_w(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvmini_w((v8i32)(_1), (_2)))

#define __lasx_xvmini_d(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvmini_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmin_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmin_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmin_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmin_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmin_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmin_wu((v8u32)_1, (v8u32)_2);
````
- **L505 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmini_h`.
  **L505 CN**: 继续与可调用符号 `__builtin_lasx_xvmini_h` 相关的逻辑。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L507 EN**: Defines macro `__lasx_xvmini_w(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L507 CN**: 定义宏 `__lasx_xvmini_w(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L508 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmini_w`.
  **L508 CN**: 继续与可调用符号 `__builtin_lasx_xvmini_w` 相关的逻辑。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L510 EN**: Defines macro `__lasx_xvmini_d(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L510 CN**: 定义宏 `__lasx_xvmini_d(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L511 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmini_d`.
  **L511 CN**: 继续与可调用符号 `__builtin_lasx_xvmini_d` 相关的逻辑。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L513 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L513 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L514 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L514 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L515 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmin_bu(__m256i _1, __m256i _2) {`.
  **L515 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmin_bu(__m256i _1, __m256i _2) {`。
- **L516 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmin_bu((v32u8)_1, (v32u8)_2)`.
  **L516 CN**: 以 `(__m256i)__builtin_lasx_xvmin_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L517 EN**: Closes the current lexical scope or compound statement.
  **L517 CN**: 结束当前词法作用域或复合语句块。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L519 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L520 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L520 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L521 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmin_hu(__m256i _1, __m256i _2) {`.
  **L521 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmin_hu(__m256i _1, __m256i _2) {`。
- **L522 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmin_hu((v16u16)_1, (v16u16)_2)`.
  **L522 CN**: 以 `(__m256i)__builtin_lasx_xvmin_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L523 EN**: Closes the current lexical scope or compound statement.
  **L523 CN**: 结束当前词法作用域或复合语句块。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L525 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L525 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L526 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L526 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L527 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmin_wu(__m256i _1, __m256i _2) {`.
  **L527 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmin_wu(__m256i _1, __m256i _2) {`。
- **L528 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmin_wu((v8u32)_1, (v8u32)_2)`.
  **L528 CN**: 以 `(__m256i)__builtin_lasx_xvmin_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 529-552

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmin_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmin_du((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvmini_bu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvmini_bu((v32u8)(_1), (_2)))

#define __lasx_xvmini_hu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvmini_hu((v16u16)(_1), (_2)))

#define __lasx_xvmini_wu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvmini_wu((v8u32)(_1), (_2)))

#define __lasx_xvmini_du(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvmini_du((v4u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvseq_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvseq_b((v32i8)_1, (v32i8)_2);
````
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L531 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L531 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L532 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L532 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L533 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmin_du(__m256i _1, __m256i _2) {`.
  **L533 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmin_du(__m256i _1, __m256i _2) {`。
- **L534 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmin_du((v4u64)_1, (v4u64)_2)`.
  **L534 CN**: 以 `(__m256i)__builtin_lasx_xvmin_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L535 EN**: Closes the current lexical scope or compound statement.
  **L535 CN**: 结束当前词法作用域或复合语句块。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Defines macro `__lasx_xvmini_bu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L537 CN**: 定义宏 `__lasx_xvmini_bu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L538 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmini_bu`.
  **L538 CN**: 继续与可调用符号 `__builtin_lasx_xvmini_bu` 相关的逻辑。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L540 EN**: Defines macro `__lasx_xvmini_hu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L540 CN**: 定义宏 `__lasx_xvmini_hu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L541 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmini_hu`.
  **L541 CN**: 继续与可调用符号 `__builtin_lasx_xvmini_hu` 相关的逻辑。
- **L542 EN**: Blank line separating nearby declarations or logic blocks.
  **L542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L543 EN**: Defines macro `__lasx_xvmini_wu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L543 CN**: 定义宏 `__lasx_xvmini_wu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L544 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmini_wu`.
  **L544 CN**: 继续与可调用符号 `__builtin_lasx_xvmini_wu` 相关的逻辑。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L546 EN**: Defines macro `__lasx_xvmini_du(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L546 CN**: 定义宏 `__lasx_xvmini_du(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L547 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvmini_du`.
  **L547 CN**: 继续与可调用符号 `__builtin_lasx_xvmini_du` 相关的逻辑。
- **L548 EN**: Blank line separating nearby declarations or logic blocks.
  **L548 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L549 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L549 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L550 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L550 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L551 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvseq_b(__m256i _1, __m256i _2) {`.
  **L551 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvseq_b(__m256i _1, __m256i _2) {`。
- **L552 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvseq_b((v32i8)_1, (v32i8)_2)`.
  **L552 CN**: 以 `(__m256i)__builtin_lasx_xvseq_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 553-576

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvseq_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvseq_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvseq_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvseq_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvseq_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvseq_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvseqi_b(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvseqi_b((v32i8)(_1), (_2)))

#define __lasx_xvseqi_h(/*__m256i*/ _1, /*si5*/ _2)                            \
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L555 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L556 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L556 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L557 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvseq_h(__m256i _1, __m256i _2) {`.
  **L557 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvseq_h(__m256i _1, __m256i _2) {`。
- **L558 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvseq_h((v16i16)_1, (v16i16)_2)`.
  **L558 CN**: 以 `(__m256i)__builtin_lasx_xvseq_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L559 EN**: Closes the current lexical scope or compound statement.
  **L559 CN**: 结束当前词法作用域或复合语句块。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L561 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L561 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L562 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L562 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L563 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvseq_w(__m256i _1, __m256i _2) {`.
  **L563 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvseq_w(__m256i _1, __m256i _2) {`。
- **L564 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvseq_w((v8i32)_1, (v8i32)_2)`.
  **L564 CN**: 以 `(__m256i)__builtin_lasx_xvseq_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L565 EN**: Closes the current lexical scope or compound statement.
  **L565 CN**: 结束当前词法作用域或复合语句块。
- **L566 EN**: Blank line separating nearby declarations or logic blocks.
  **L566 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L567 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L567 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L568 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L568 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L569 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvseq_d(__m256i _1, __m256i _2) {`.
  **L569 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvseq_d(__m256i _1, __m256i _2) {`。
- **L570 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvseq_d((v4i64)_1, (v4i64)_2)`.
  **L570 CN**: 以 `(__m256i)__builtin_lasx_xvseq_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L571 EN**: Closes the current lexical scope or compound statement.
  **L571 CN**: 结束当前词法作用域或复合语句块。
- **L572 EN**: Blank line separating nearby declarations or logic blocks.
  **L572 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L573 EN**: Defines macro `__lasx_xvseqi_b(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L573 CN**: 定义宏 `__lasx_xvseqi_b(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L574 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvseqi_b`.
  **L574 CN**: 继续与可调用符号 `__builtin_lasx_xvseqi_b` 相关的逻辑。
- **L575 EN**: Blank line separating nearby declarations or logic blocks.
  **L575 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L576 EN**: Defines macro `__lasx_xvseqi_h(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L576 CN**: 定义宏 `__lasx_xvseqi_h(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 577-600

````c
  ((__m256i)__builtin_lasx_xvseqi_h((v16i16)(_1), (_2)))

#define __lasx_xvseqi_w(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvseqi_w((v8i32)(_1), (_2)))

#define __lasx_xvseqi_d(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvseqi_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvslt_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvslt_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvslt_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvslt_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvslt_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvslt_w((v8i32)_1, (v8i32)_2);
````
- **L577 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvseqi_h`.
  **L577 CN**: 继续与可调用符号 `__builtin_lasx_xvseqi_h` 相关的逻辑。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L579 EN**: Defines macro `__lasx_xvseqi_w(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L579 CN**: 定义宏 `__lasx_xvseqi_w(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L580 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvseqi_w`.
  **L580 CN**: 继续与可调用符号 `__builtin_lasx_xvseqi_w` 相关的逻辑。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Defines macro `__lasx_xvseqi_d(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L582 CN**: 定义宏 `__lasx_xvseqi_d(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L583 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvseqi_d`.
  **L583 CN**: 继续与可调用符号 `__builtin_lasx_xvseqi_d` 相关的逻辑。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L585 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L585 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L586 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L586 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L587 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvslt_b(__m256i _1, __m256i _2) {`.
  **L587 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvslt_b(__m256i _1, __m256i _2) {`。
- **L588 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvslt_b((v32i8)_1, (v32i8)_2)`.
  **L588 CN**: 以 `(__m256i)__builtin_lasx_xvslt_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L589 EN**: Closes the current lexical scope or compound statement.
  **L589 CN**: 结束当前词法作用域或复合语句块。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L591 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L591 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L592 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L592 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L593 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvslt_h(__m256i _1, __m256i _2) {`.
  **L593 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvslt_h(__m256i _1, __m256i _2) {`。
- **L594 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvslt_h((v16i16)_1, (v16i16)_2)`.
  **L594 CN**: 以 `(__m256i)__builtin_lasx_xvslt_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L597 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L597 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L598 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L598 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L599 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvslt_w(__m256i _1, __m256i _2) {`.
  **L599 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvslt_w(__m256i _1, __m256i _2) {`。
- **L600 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvslt_w((v8i32)_1, (v8i32)_2)`.
  **L600 CN**: 以 `(__m256i)__builtin_lasx_xvslt_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 601-624

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvslt_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvslt_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvslti_b(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslti_b((v32i8)(_1), (_2)))

#define __lasx_xvslti_h(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslti_h((v16i16)(_1), (_2)))

#define __lasx_xvslti_w(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslti_w((v8i32)(_1), (_2)))

#define __lasx_xvslti_d(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslti_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvslt_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvslt_bu((v32u8)_1, (v32u8)_2);
````
- **L601 EN**: Closes the current lexical scope or compound statement.
  **L601 CN**: 结束当前词法作用域或复合语句块。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L603 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L603 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L604 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L604 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L605 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvslt_d(__m256i _1, __m256i _2) {`.
  **L605 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvslt_d(__m256i _1, __m256i _2) {`。
- **L606 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvslt_d((v4i64)_1, (v4i64)_2)`.
  **L606 CN**: 以 `(__m256i)__builtin_lasx_xvslt_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L607 EN**: Closes the current lexical scope or compound statement.
  **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Defines macro `__lasx_xvslti_b(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L609 CN**: 定义宏 `__lasx_xvslti_b(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L610 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslti_b`.
  **L610 CN**: 继续与可调用符号 `__builtin_lasx_xvslti_b` 相关的逻辑。
- **L611 EN**: Blank line separating nearby declarations or logic blocks.
  **L611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L612 EN**: Defines macro `__lasx_xvslti_h(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L612 CN**: 定义宏 `__lasx_xvslti_h(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L613 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslti_h`.
  **L613 CN**: 继续与可调用符号 `__builtin_lasx_xvslti_h` 相关的逻辑。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L615 EN**: Defines macro `__lasx_xvslti_w(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L615 CN**: 定义宏 `__lasx_xvslti_w(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L616 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslti_w`.
  **L616 CN**: 继续与可调用符号 `__builtin_lasx_xvslti_w` 相关的逻辑。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L618 EN**: Defines macro `__lasx_xvslti_d(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L618 CN**: 定义宏 `__lasx_xvslti_d(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L619 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslti_d`.
  **L619 CN**: 继续与可调用符号 `__builtin_lasx_xvslti_d` 相关的逻辑。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L621 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L622 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L622 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L623 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvslt_bu(__m256i _1, __m256i _2) {`.
  **L623 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvslt_bu(__m256i _1, __m256i _2) {`。
- **L624 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvslt_bu((v32u8)_1, (v32u8)_2)`.
  **L624 CN**: 以 `(__m256i)__builtin_lasx_xvslt_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。

### Lines 625-648

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvslt_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvslt_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvslt_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvslt_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvslt_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvslt_du((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvslti_bu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvslti_bu((v32u8)(_1), (_2)))

#define __lasx_xvslti_hu(/*__m256i*/ _1, /*ui5*/ _2)                           \
````
- **L625 EN**: Closes the current lexical scope or compound statement.
  **L625 CN**: 结束当前词法作用域或复合语句块。
- **L626 EN**: Blank line separating nearby declarations or logic blocks.
  **L626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L627 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L627 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L628 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L628 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L629 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvslt_hu(__m256i _1, __m256i _2) {`.
  **L629 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvslt_hu(__m256i _1, __m256i _2) {`。
- **L630 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvslt_hu((v16u16)_1, (v16u16)_2)`.
  **L630 CN**: 以 `(__m256i)__builtin_lasx_xvslt_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L633 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L634 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L634 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L635 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvslt_wu(__m256i _1, __m256i _2) {`.
  **L635 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvslt_wu(__m256i _1, __m256i _2) {`。
- **L636 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvslt_wu((v8u32)_1, (v8u32)_2)`.
  **L636 CN**: 以 `(__m256i)__builtin_lasx_xvslt_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L639 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L639 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L640 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L640 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L641 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvslt_du(__m256i _1, __m256i _2) {`.
  **L641 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvslt_du(__m256i _1, __m256i _2) {`。
- **L642 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvslt_du((v4u64)_1, (v4u64)_2)`.
  **L642 CN**: 以 `(__m256i)__builtin_lasx_xvslt_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L643 EN**: Closes the current lexical scope or compound statement.
  **L643 CN**: 结束当前词法作用域或复合语句块。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L645 EN**: Defines macro `__lasx_xvslti_bu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L645 CN**: 定义宏 `__lasx_xvslti_bu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L646 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslti_bu`.
  **L646 CN**: 继续与可调用符号 `__builtin_lasx_xvslti_bu` 相关的逻辑。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L648 EN**: Defines macro `__lasx_xvslti_hu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L648 CN**: 定义宏 `__lasx_xvslti_hu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 649-672

````c
  ((__m256i)__builtin_lasx_xvslti_hu((v16u16)(_1), (_2)))

#define __lasx_xvslti_wu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvslti_wu((v8u32)(_1), (_2)))

#define __lasx_xvslti_du(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvslti_du((v4u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsle_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsle_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsle_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsle_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsle_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsle_w((v8i32)_1, (v8i32)_2);
````
- **L649 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslti_hu`.
  **L649 CN**: 继续与可调用符号 `__builtin_lasx_xvslti_hu` 相关的逻辑。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L651 EN**: Defines macro `__lasx_xvslti_wu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L651 CN**: 定义宏 `__lasx_xvslti_wu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L652 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslti_wu`.
  **L652 CN**: 继续与可调用符号 `__builtin_lasx_xvslti_wu` 相关的逻辑。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Defines macro `__lasx_xvslti_du(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L654 CN**: 定义宏 `__lasx_xvslti_du(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L655 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslti_du`.
  **L655 CN**: 继续与可调用符号 `__builtin_lasx_xvslti_du` 相关的逻辑。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L657 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L658 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L658 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L659 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsle_b(__m256i _1, __m256i _2) {`.
  **L659 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsle_b(__m256i _1, __m256i _2) {`。
- **L660 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsle_b((v32i8)_1, (v32i8)_2)`.
  **L660 CN**: 以 `(__m256i)__builtin_lasx_xvsle_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Blank line separating nearby declarations or logic blocks.
  **L662 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L663 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L663 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L664 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L664 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L665 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsle_h(__m256i _1, __m256i _2) {`.
  **L665 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsle_h(__m256i _1, __m256i _2) {`。
- **L666 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsle_h((v16i16)_1, (v16i16)_2)`.
  **L666 CN**: 以 `(__m256i)__builtin_lasx_xvsle_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L669 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L670 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L670 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L671 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsle_w(__m256i _1, __m256i _2) {`.
  **L671 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsle_w(__m256i _1, __m256i _2) {`。
- **L672 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsle_w((v8i32)_1, (v8i32)_2)`.
  **L672 CN**: 以 `(__m256i)__builtin_lasx_xvsle_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 673-696

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsle_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsle_d((v4i64)_1, (v4i64)_2);
}

#define __lasx_xvslei_b(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslei_b((v32i8)(_1), (_2)))

#define __lasx_xvslei_h(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslei_h((v16i16)(_1), (_2)))

#define __lasx_xvslei_w(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslei_w((v8i32)(_1), (_2)))

#define __lasx_xvslei_d(/*__m256i*/ _1, /*si5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvslei_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsle_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsle_bu((v32u8)_1, (v32u8)_2);
````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L675 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L675 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L676 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L676 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L677 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsle_d(__m256i _1, __m256i _2) {`.
  **L677 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsle_d(__m256i _1, __m256i _2) {`。
- **L678 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsle_d((v4i64)_1, (v4i64)_2)`.
  **L678 CN**: 以 `(__m256i)__builtin_lasx_xvsle_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L681 EN**: Defines macro `__lasx_xvslei_b(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L681 CN**: 定义宏 `__lasx_xvslei_b(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L682 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslei_b`.
  **L682 CN**: 继续与可调用符号 `__builtin_lasx_xvslei_b` 相关的逻辑。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L684 EN**: Defines macro `__lasx_xvslei_h(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L684 CN**: 定义宏 `__lasx_xvslei_h(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L685 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslei_h`.
  **L685 CN**: 继续与可调用符号 `__builtin_lasx_xvslei_h` 相关的逻辑。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L687 EN**: Defines macro `__lasx_xvslei_w(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L687 CN**: 定义宏 `__lasx_xvslei_w(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L688 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslei_w`.
  **L688 CN**: 继续与可调用符号 `__builtin_lasx_xvslei_w` 相关的逻辑。
- **L689 EN**: Blank line separating nearby declarations or logic blocks.
  **L689 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L690 EN**: Defines macro `__lasx_xvslei_d(/*__m256i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L690 CN**: 定义宏 `__lasx_xvslei_d(/*__m256i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L691 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslei_d`.
  **L691 CN**: 继续与可调用符号 `__builtin_lasx_xvslei_d` 相关的逻辑。
- **L692 EN**: Blank line separating nearby declarations or logic blocks.
  **L692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L693 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L693 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L694 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L694 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L695 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsle_bu(__m256i _1, __m256i _2) {`.
  **L695 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsle_bu(__m256i _1, __m256i _2) {`。
- **L696 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsle_bu((v32u8)_1, (v32u8)_2)`.
  **L696 CN**: 以 `(__m256i)__builtin_lasx_xvsle_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。

### Lines 697-720

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsle_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsle_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsle_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsle_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsle_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsle_du((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvslei_bu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvslei_bu((v32u8)(_1), (_2)))

#define __lasx_xvslei_hu(/*__m256i*/ _1, /*ui5*/ _2)                           \
````
- **L697 EN**: Closes the current lexical scope or compound statement.
  **L697 CN**: 结束当前词法作用域或复合语句块。
- **L698 EN**: Blank line separating nearby declarations or logic blocks.
  **L698 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L699 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L699 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L700 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L700 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L701 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsle_hu(__m256i _1, __m256i _2) {`.
  **L701 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsle_hu(__m256i _1, __m256i _2) {`。
- **L702 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsle_hu((v16u16)_1, (v16u16)_2)`.
  **L702 CN**: 以 `(__m256i)__builtin_lasx_xvsle_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L703 EN**: Closes the current lexical scope or compound statement.
  **L703 CN**: 结束当前词法作用域或复合语句块。
- **L704 EN**: Blank line separating nearby declarations or logic blocks.
  **L704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L705 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L705 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L706 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L706 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L707 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsle_wu(__m256i _1, __m256i _2) {`.
  **L707 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsle_wu(__m256i _1, __m256i _2) {`。
- **L708 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsle_wu((v8u32)_1, (v8u32)_2)`.
  **L708 CN**: 以 `(__m256i)__builtin_lasx_xvsle_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L709 EN**: Closes the current lexical scope or compound statement.
  **L709 CN**: 结束当前词法作用域或复合语句块。
- **L710 EN**: Blank line separating nearby declarations or logic blocks.
  **L710 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L711 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L711 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L712 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L712 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L713 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsle_du(__m256i _1, __m256i _2) {`.
  **L713 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsle_du(__m256i _1, __m256i _2) {`。
- **L714 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsle_du((v4u64)_1, (v4u64)_2)`.
  **L714 CN**: 以 `(__m256i)__builtin_lasx_xvsle_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L715 EN**: Closes the current lexical scope or compound statement.
  **L715 CN**: 结束当前词法作用域或复合语句块。
- **L716 EN**: Blank line separating nearby declarations or logic blocks.
  **L716 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L717 EN**: Defines macro `__lasx_xvslei_bu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L717 CN**: 定义宏 `__lasx_xvslei_bu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L718 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslei_bu`.
  **L718 CN**: 继续与可调用符号 `__builtin_lasx_xvslei_bu` 相关的逻辑。
- **L719 EN**: Blank line separating nearby declarations or logic blocks.
  **L719 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L720 EN**: Defines macro `__lasx_xvslei_hu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L720 CN**: 定义宏 `__lasx_xvslei_hu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 721-744

````c
  ((__m256i)__builtin_lasx_xvslei_hu((v16u16)(_1), (_2)))

#define __lasx_xvslei_wu(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvslei_wu((v8u32)(_1), (_2)))

#define __lasx_xvslei_du(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvslei_du((v4u64)(_1), (_2)))

#define __lasx_xvsat_b(/*__m256i*/ _1, /*ui3*/ _2)                             \
  ((__m256i)__builtin_lasx_xvsat_b((v32i8)(_1), (_2)))

#define __lasx_xvsat_h(/*__m256i*/ _1, /*ui4*/ _2)                             \
  ((__m256i)__builtin_lasx_xvsat_h((v16i16)(_1), (_2)))

#define __lasx_xvsat_w(/*__m256i*/ _1, /*ui5*/ _2)                             \
  ((__m256i)__builtin_lasx_xvsat_w((v8i32)(_1), (_2)))

#define __lasx_xvsat_d(/*__m256i*/ _1, /*ui6*/ _2)                             \
  ((__m256i)__builtin_lasx_xvsat_d((v4i64)(_1), (_2)))

#define __lasx_xvsat_bu(/*__m256i*/ _1, /*ui3*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsat_bu((v32u8)(_1), (_2)))

#define __lasx_xvsat_hu(/*__m256i*/ _1, /*ui4*/ _2)                            \
````
- **L721 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslei_hu`.
  **L721 CN**: 继续与可调用符号 `__builtin_lasx_xvslei_hu` 相关的逻辑。
- **L722 EN**: Blank line separating nearby declarations or logic blocks.
  **L722 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L723 EN**: Defines macro `__lasx_xvslei_wu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L723 CN**: 定义宏 `__lasx_xvslei_wu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L724 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslei_wu`.
  **L724 CN**: 继续与可调用符号 `__builtin_lasx_xvslei_wu` 相关的逻辑。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L726 EN**: Defines macro `__lasx_xvslei_du(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L726 CN**: 定义宏 `__lasx_xvslei_du(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L727 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvslei_du`.
  **L727 CN**: 继续与可调用符号 `__builtin_lasx_xvslei_du` 相关的逻辑。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L729 EN**: Defines macro `__lasx_xvsat_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L729 CN**: 定义宏 `__lasx_xvsat_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L730 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsat_b`.
  **L730 CN**: 继续与可调用符号 `__builtin_lasx_xvsat_b` 相关的逻辑。
- **L731 EN**: Blank line separating nearby declarations or logic blocks.
  **L731 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L732 EN**: Defines macro `__lasx_xvsat_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L732 CN**: 定义宏 `__lasx_xvsat_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L733 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsat_h`.
  **L733 CN**: 继续与可调用符号 `__builtin_lasx_xvsat_h` 相关的逻辑。
- **L734 EN**: Blank line separating nearby declarations or logic blocks.
  **L734 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L735 EN**: Defines macro `__lasx_xvsat_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L735 CN**: 定义宏 `__lasx_xvsat_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L736 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsat_w`.
  **L736 CN**: 继续与可调用符号 `__builtin_lasx_xvsat_w` 相关的逻辑。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L738 EN**: Defines macro `__lasx_xvsat_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L738 CN**: 定义宏 `__lasx_xvsat_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L739 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsat_d`.
  **L739 CN**: 继续与可调用符号 `__builtin_lasx_xvsat_d` 相关的逻辑。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Defines macro `__lasx_xvsat_bu(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L741 CN**: 定义宏 `__lasx_xvsat_bu(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L742 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsat_bu`.
  **L742 CN**: 继续与可调用符号 `__builtin_lasx_xvsat_bu` 相关的逻辑。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L744 EN**: Defines macro `__lasx_xvsat_hu(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L744 CN**: 定义宏 `__lasx_xvsat_hu(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 745-768

````c
  ((__m256i)__builtin_lasx_xvsat_hu((v16u16)(_1), (_2)))

#define __lasx_xvsat_wu(/*__m256i*/ _1, /*ui5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsat_wu((v8u32)(_1), (_2)))

#define __lasx_xvsat_du(/*__m256i*/ _1, /*ui6*/ _2)                            \
  ((__m256i)__builtin_lasx_xvsat_du((v4u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadda_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadda_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadda_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadda_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadda_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadda_w((v8i32)_1, (v8i32)_2);
````
- **L745 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsat_hu`.
  **L745 CN**: 继续与可调用符号 `__builtin_lasx_xvsat_hu` 相关的逻辑。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L747 EN**: Defines macro `__lasx_xvsat_wu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L747 CN**: 定义宏 `__lasx_xvsat_wu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L748 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsat_wu`.
  **L748 CN**: 继续与可调用符号 `__builtin_lasx_xvsat_wu` 相关的逻辑。
- **L749 EN**: Blank line separating nearby declarations or logic blocks.
  **L749 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L750 EN**: Defines macro `__lasx_xvsat_du(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L750 CN**: 定义宏 `__lasx_xvsat_du(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L751 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsat_du`.
  **L751 CN**: 继续与可调用符号 `__builtin_lasx_xvsat_du` 相关的逻辑。
- **L752 EN**: Blank line separating nearby declarations or logic blocks.
  **L752 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L753 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L753 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L754 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L754 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L755 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadda_b(__m256i _1, __m256i _2) {`.
  **L755 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadda_b(__m256i _1, __m256i _2) {`。
- **L756 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadda_b((v32i8)_1, (v32i8)_2)`.
  **L756 CN**: 以 `(__m256i)__builtin_lasx_xvadda_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L757 EN**: Closes the current lexical scope or compound statement.
  **L757 CN**: 结束当前词法作用域或复合语句块。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L759 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L759 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L760 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L760 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L761 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadda_h(__m256i _1, __m256i _2) {`.
  **L761 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadda_h(__m256i _1, __m256i _2) {`。
- **L762 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadda_h((v16i16)_1, (v16i16)_2)`.
  **L762 CN**: 以 `(__m256i)__builtin_lasx_xvadda_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L763 EN**: Closes the current lexical scope or compound statement.
  **L763 CN**: 结束当前词法作用域或复合语句块。
- **L764 EN**: Blank line separating nearby declarations or logic blocks.
  **L764 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L765 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L765 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L766 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L766 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L767 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadda_w(__m256i _1, __m256i _2) {`.
  **L767 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadda_w(__m256i _1, __m256i _2) {`。
- **L768 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadda_w((v8i32)_1, (v8i32)_2)`.
  **L768 CN**: 以 `(__m256i)__builtin_lasx_xvadda_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 769-792

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadda_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadda_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsadd_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsadd_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsadd_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsadd_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsadd_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsadd_w((v8i32)_1, (v8i32)_2);
````
- **L769 EN**: Closes the current lexical scope or compound statement.
  **L769 CN**: 结束当前词法作用域或复合语句块。
- **L770 EN**: Blank line separating nearby declarations or logic blocks.
  **L770 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L771 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L771 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L772 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L772 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L773 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadda_d(__m256i _1, __m256i _2) {`.
  **L773 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadda_d(__m256i _1, __m256i _2) {`。
- **L774 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadda_d((v4i64)_1, (v4i64)_2)`.
  **L774 CN**: 以 `(__m256i)__builtin_lasx_xvadda_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L775 EN**: Closes the current lexical scope or compound statement.
  **L775 CN**: 结束当前词法作用域或复合语句块。
- **L776 EN**: Blank line separating nearby declarations or logic blocks.
  **L776 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L777 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L777 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L778 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L778 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L779 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsadd_b(__m256i _1, __m256i _2) {`.
  **L779 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsadd_b(__m256i _1, __m256i _2) {`。
- **L780 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsadd_b((v32i8)_1, (v32i8)_2)`.
  **L780 CN**: 以 `(__m256i)__builtin_lasx_xvsadd_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L781 EN**: Closes the current lexical scope or compound statement.
  **L781 CN**: 结束当前词法作用域或复合语句块。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L783 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L783 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L784 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L784 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L785 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsadd_h(__m256i _1, __m256i _2) {`.
  **L785 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsadd_h(__m256i _1, __m256i _2) {`。
- **L786 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsadd_h((v16i16)_1, (v16i16)_2)`.
  **L786 CN**: 以 `(__m256i)__builtin_lasx_xvsadd_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L787 EN**: Closes the current lexical scope or compound statement.
  **L787 CN**: 结束当前词法作用域或复合语句块。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L789 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L789 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L790 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L790 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L791 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsadd_w(__m256i _1, __m256i _2) {`.
  **L791 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsadd_w(__m256i _1, __m256i _2) {`。
- **L792 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsadd_w((v8i32)_1, (v8i32)_2)`.
  **L792 CN**: 以 `(__m256i)__builtin_lasx_xvsadd_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 793-816

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsadd_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsadd_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsadd_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsadd_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsadd_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsadd_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsadd_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsadd_wu((v8u32)_1, (v8u32)_2);
````
- **L793 EN**: Closes the current lexical scope or compound statement.
  **L793 CN**: 结束当前词法作用域或复合语句块。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L795 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L795 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L796 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L796 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L797 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsadd_d(__m256i _1, __m256i _2) {`.
  **L797 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsadd_d(__m256i _1, __m256i _2) {`。
- **L798 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsadd_d((v4i64)_1, (v4i64)_2)`.
  **L798 CN**: 以 `(__m256i)__builtin_lasx_xvsadd_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L799 EN**: Closes the current lexical scope or compound statement.
  **L799 CN**: 结束当前词法作用域或复合语句块。
- **L800 EN**: Blank line separating nearby declarations or logic blocks.
  **L800 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L801 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L801 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L802 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L802 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L803 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsadd_bu(__m256i _1, __m256i _2) {`.
  **L803 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsadd_bu(__m256i _1, __m256i _2) {`。
- **L804 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsadd_bu((v32u8)_1, (v32u8)_2)`.
  **L804 CN**: 以 `(__m256i)__builtin_lasx_xvsadd_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L807 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L807 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L808 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L808 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L809 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsadd_hu(__m256i _1, __m256i _2) {`.
  **L809 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsadd_hu(__m256i _1, __m256i _2) {`。
- **L810 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsadd_hu((v16u16)_1, (v16u16)_2)`.
  **L810 CN**: 以 `(__m256i)__builtin_lasx_xvsadd_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L811 EN**: Closes the current lexical scope or compound statement.
  **L811 CN**: 结束当前词法作用域或复合语句块。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L813 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L814 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L814 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L815 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsadd_wu(__m256i _1, __m256i _2) {`.
  **L815 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsadd_wu(__m256i _1, __m256i _2) {`。
- **L816 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsadd_wu((v8u32)_1, (v8u32)_2)`.
  **L816 CN**: 以 `(__m256i)__builtin_lasx_xvsadd_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 817-840

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsadd_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsadd_du((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavg_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavg_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavg_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavg_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavg_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavg_w((v8i32)_1, (v8i32)_2);
````
- **L817 EN**: Closes the current lexical scope or compound statement.
  **L817 CN**: 结束当前词法作用域或复合语句块。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L819 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L819 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L820 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L820 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L821 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsadd_du(__m256i _1, __m256i _2) {`.
  **L821 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsadd_du(__m256i _1, __m256i _2) {`。
- **L822 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsadd_du((v4u64)_1, (v4u64)_2)`.
  **L822 CN**: 以 `(__m256i)__builtin_lasx_xvsadd_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L823 EN**: Closes the current lexical scope or compound statement.
  **L823 CN**: 结束当前词法作用域或复合语句块。
- **L824 EN**: Blank line separating nearby declarations or logic blocks.
  **L824 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L825 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L825 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L826 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L826 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L827 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavg_b(__m256i _1, __m256i _2) {`.
  **L827 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavg_b(__m256i _1, __m256i _2) {`。
- **L828 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavg_b((v32i8)_1, (v32i8)_2)`.
  **L828 CN**: 以 `(__m256i)__builtin_lasx_xvavg_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L829 EN**: Closes the current lexical scope or compound statement.
  **L829 CN**: 结束当前词法作用域或复合语句块。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L831 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L831 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L832 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L832 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L833 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavg_h(__m256i _1, __m256i _2) {`.
  **L833 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavg_h(__m256i _1, __m256i _2) {`。
- **L834 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavg_h((v16i16)_1, (v16i16)_2)`.
  **L834 CN**: 以 `(__m256i)__builtin_lasx_xvavg_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L835 EN**: Closes the current lexical scope or compound statement.
  **L835 CN**: 结束当前词法作用域或复合语句块。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L837 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L837 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L838 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L838 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L839 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavg_w(__m256i _1, __m256i _2) {`.
  **L839 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavg_w(__m256i _1, __m256i _2) {`。
- **L840 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavg_w((v8i32)_1, (v8i32)_2)`.
  **L840 CN**: 以 `(__m256i)__builtin_lasx_xvavg_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 841-864

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavg_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavg_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavg_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavg_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavg_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavg_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavg_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavg_wu((v8u32)_1, (v8u32)_2);
````
- **L841 EN**: Closes the current lexical scope or compound statement.
  **L841 CN**: 结束当前词法作用域或复合语句块。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L843 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L843 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L844 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L844 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L845 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavg_d(__m256i _1, __m256i _2) {`.
  **L845 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavg_d(__m256i _1, __m256i _2) {`。
- **L846 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavg_d((v4i64)_1, (v4i64)_2)`.
  **L846 CN**: 以 `(__m256i)__builtin_lasx_xvavg_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L849 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L850 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L850 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L851 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavg_bu(__m256i _1, __m256i _2) {`.
  **L851 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavg_bu(__m256i _1, __m256i _2) {`。
- **L852 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavg_bu((v32u8)_1, (v32u8)_2)`.
  **L852 CN**: 以 `(__m256i)__builtin_lasx_xvavg_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L853 EN**: Closes the current lexical scope or compound statement.
  **L853 CN**: 结束当前词法作用域或复合语句块。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L855 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L855 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L856 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L856 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L857 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavg_hu(__m256i _1, __m256i _2) {`.
  **L857 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavg_hu(__m256i _1, __m256i _2) {`。
- **L858 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavg_hu((v16u16)_1, (v16u16)_2)`.
  **L858 CN**: 以 `(__m256i)__builtin_lasx_xvavg_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L859 EN**: Closes the current lexical scope or compound statement.
  **L859 CN**: 结束当前词法作用域或复合语句块。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L861 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L861 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L862 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L862 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L863 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavg_wu(__m256i _1, __m256i _2) {`.
  **L863 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavg_wu(__m256i _1, __m256i _2) {`。
- **L864 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavg_wu((v8u32)_1, (v8u32)_2)`.
  **L864 CN**: 以 `(__m256i)__builtin_lasx_xvavg_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 865-888

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavg_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavg_du((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavgr_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavgr_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavgr_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavgr_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavgr_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavgr_w((v8i32)_1, (v8i32)_2);
````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L867 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L868 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L868 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L869 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavg_du(__m256i _1, __m256i _2) {`.
  **L869 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavg_du(__m256i _1, __m256i _2) {`。
- **L870 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavg_du((v4u64)_1, (v4u64)_2)`.
  **L870 CN**: 以 `(__m256i)__builtin_lasx_xvavg_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L871 EN**: Closes the current lexical scope or compound statement.
  **L871 CN**: 结束当前词法作用域或复合语句块。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L873 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L873 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L874 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L874 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L875 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavgr_b(__m256i _1, __m256i _2) {`.
  **L875 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavgr_b(__m256i _1, __m256i _2) {`。
- **L876 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavgr_b((v32i8)_1, (v32i8)_2)`.
  **L876 CN**: 以 `(__m256i)__builtin_lasx_xvavgr_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L877 EN**: Closes the current lexical scope or compound statement.
  **L877 CN**: 结束当前词法作用域或复合语句块。
- **L878 EN**: Blank line separating nearby declarations or logic blocks.
  **L878 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L879 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L879 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L880 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L880 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L881 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavgr_h(__m256i _1, __m256i _2) {`.
  **L881 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavgr_h(__m256i _1, __m256i _2) {`。
- **L882 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavgr_h((v16i16)_1, (v16i16)_2)`.
  **L882 CN**: 以 `(__m256i)__builtin_lasx_xvavgr_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L883 EN**: Closes the current lexical scope or compound statement.
  **L883 CN**: 结束当前词法作用域或复合语句块。
- **L884 EN**: Blank line separating nearby declarations or logic blocks.
  **L884 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L885 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L885 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L886 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L886 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L887 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavgr_w(__m256i _1, __m256i _2) {`.
  **L887 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavgr_w(__m256i _1, __m256i _2) {`。
- **L888 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavgr_w((v8i32)_1, (v8i32)_2)`.
  **L888 CN**: 以 `(__m256i)__builtin_lasx_xvavgr_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 889-912

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavgr_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavgr_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavgr_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavgr_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavgr_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavgr_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavgr_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavgr_wu((v8u32)_1, (v8u32)_2);
````
- **L889 EN**: Closes the current lexical scope or compound statement.
  **L889 CN**: 结束当前词法作用域或复合语句块。
- **L890 EN**: Blank line separating nearby declarations or logic blocks.
  **L890 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L891 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L891 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L892 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L892 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L893 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavgr_d(__m256i _1, __m256i _2) {`.
  **L893 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavgr_d(__m256i _1, __m256i _2) {`。
- **L894 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavgr_d((v4i64)_1, (v4i64)_2)`.
  **L894 CN**: 以 `(__m256i)__builtin_lasx_xvavgr_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L895 EN**: Closes the current lexical scope or compound statement.
  **L895 CN**: 结束当前词法作用域或复合语句块。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L897 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L897 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L898 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L898 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L899 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavgr_bu(__m256i _1, __m256i _2) {`.
  **L899 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavgr_bu(__m256i _1, __m256i _2) {`。
- **L900 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavgr_bu((v32u8)_1, (v32u8)_2)`.
  **L900 CN**: 以 `(__m256i)__builtin_lasx_xvavgr_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L901 EN**: Closes the current lexical scope or compound statement.
  **L901 CN**: 结束当前词法作用域或复合语句块。
- **L902 EN**: Blank line separating nearby declarations or logic blocks.
  **L902 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L903 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L903 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L904 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L904 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L905 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavgr_hu(__m256i _1, __m256i _2) {`.
  **L905 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavgr_hu(__m256i _1, __m256i _2) {`。
- **L906 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavgr_hu((v16u16)_1, (v16u16)_2)`.
  **L906 CN**: 以 `(__m256i)__builtin_lasx_xvavgr_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L907 EN**: Closes the current lexical scope or compound statement.
  **L907 CN**: 结束当前词法作用域或复合语句块。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L909 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L909 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L910 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L910 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L911 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavgr_wu(__m256i _1, __m256i _2) {`.
  **L911 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavgr_wu(__m256i _1, __m256i _2) {`。
- **L912 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavgr_wu((v8u32)_1, (v8u32)_2)`.
  **L912 CN**: 以 `(__m256i)__builtin_lasx_xvavgr_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 913-936

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvavgr_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvavgr_du((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssub_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssub_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssub_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssub_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssub_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssub_w((v8i32)_1, (v8i32)_2);
````
- **L913 EN**: Closes the current lexical scope or compound statement.
  **L913 CN**: 结束当前词法作用域或复合语句块。
- **L914 EN**: Blank line separating nearby declarations or logic blocks.
  **L914 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L915 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L915 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L916 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L916 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L917 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvavgr_du(__m256i _1, __m256i _2) {`.
  **L917 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvavgr_du(__m256i _1, __m256i _2) {`。
- **L918 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvavgr_du((v4u64)_1, (v4u64)_2)`.
  **L918 CN**: 以 `(__m256i)__builtin_lasx_xvavgr_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L919 EN**: Closes the current lexical scope or compound statement.
  **L919 CN**: 结束当前词法作用域或复合语句块。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L921 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L921 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L922 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L922 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L923 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssub_b(__m256i _1, __m256i _2) {`.
  **L923 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssub_b(__m256i _1, __m256i _2) {`。
- **L924 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssub_b((v32i8)_1, (v32i8)_2)`.
  **L924 CN**: 以 `(__m256i)__builtin_lasx_xvssub_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L925 EN**: Closes the current lexical scope or compound statement.
  **L925 CN**: 结束当前词法作用域或复合语句块。
- **L926 EN**: Blank line separating nearby declarations or logic blocks.
  **L926 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L927 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L927 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L928 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L928 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L929 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssub_h(__m256i _1, __m256i _2) {`.
  **L929 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssub_h(__m256i _1, __m256i _2) {`。
- **L930 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssub_h((v16i16)_1, (v16i16)_2)`.
  **L930 CN**: 以 `(__m256i)__builtin_lasx_xvssub_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L931 EN**: Closes the current lexical scope or compound statement.
  **L931 CN**: 结束当前词法作用域或复合语句块。
- **L932 EN**: Blank line separating nearby declarations or logic blocks.
  **L932 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L933 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L933 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L934 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L934 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L935 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssub_w(__m256i _1, __m256i _2) {`.
  **L935 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssub_w(__m256i _1, __m256i _2) {`。
- **L936 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssub_w((v8i32)_1, (v8i32)_2)`.
  **L936 CN**: 以 `(__m256i)__builtin_lasx_xvssub_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 937-960

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssub_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssub_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssub_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssub_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssub_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssub_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssub_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssub_wu((v8u32)_1, (v8u32)_2);
````
- **L937 EN**: Closes the current lexical scope or compound statement.
  **L937 CN**: 结束当前词法作用域或复合语句块。
- **L938 EN**: Blank line separating nearby declarations or logic blocks.
  **L938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L939 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L939 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L940 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L940 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L941 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssub_d(__m256i _1, __m256i _2) {`.
  **L941 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssub_d(__m256i _1, __m256i _2) {`。
- **L942 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssub_d((v4i64)_1, (v4i64)_2)`.
  **L942 CN**: 以 `(__m256i)__builtin_lasx_xvssub_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L943 EN**: Closes the current lexical scope or compound statement.
  **L943 CN**: 结束当前词法作用域或复合语句块。
- **L944 EN**: Blank line separating nearby declarations or logic blocks.
  **L944 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L945 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L945 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L946 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L946 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L947 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssub_bu(__m256i _1, __m256i _2) {`.
  **L947 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssub_bu(__m256i _1, __m256i _2) {`。
- **L948 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssub_bu((v32u8)_1, (v32u8)_2)`.
  **L948 CN**: 以 `(__m256i)__builtin_lasx_xvssub_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Blank line separating nearby declarations or logic blocks.
  **L950 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L951 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L951 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L952 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L952 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L953 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssub_hu(__m256i _1, __m256i _2) {`.
  **L953 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssub_hu(__m256i _1, __m256i _2) {`。
- **L954 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssub_hu((v16u16)_1, (v16u16)_2)`.
  **L954 CN**: 以 `(__m256i)__builtin_lasx_xvssub_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L955 EN**: Closes the current lexical scope or compound statement.
  **L955 CN**: 结束当前词法作用域或复合语句块。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L957 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L957 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L958 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L958 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L959 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssub_wu(__m256i _1, __m256i _2) {`.
  **L959 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssub_wu(__m256i _1, __m256i _2) {`。
- **L960 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssub_wu((v8u32)_1, (v8u32)_2)`.
  **L960 CN**: 以 `(__m256i)__builtin_lasx_xvssub_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 961-984

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssub_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssub_du((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvabsd_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvabsd_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvabsd_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvabsd_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvabsd_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvabsd_w((v8i32)_1, (v8i32)_2);
````
- **L961 EN**: Closes the current lexical scope or compound statement.
  **L961 CN**: 结束当前词法作用域或复合语句块。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L963 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L963 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L964 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L964 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L965 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssub_du(__m256i _1, __m256i _2) {`.
  **L965 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssub_du(__m256i _1, __m256i _2) {`。
- **L966 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssub_du((v4u64)_1, (v4u64)_2)`.
  **L966 CN**: 以 `(__m256i)__builtin_lasx_xvssub_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L967 EN**: Closes the current lexical scope or compound statement.
  **L967 CN**: 结束当前词法作用域或复合语句块。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L969 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L970 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L970 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L971 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvabsd_b(__m256i _1, __m256i _2) {`.
  **L971 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvabsd_b(__m256i _1, __m256i _2) {`。
- **L972 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvabsd_b((v32i8)_1, (v32i8)_2)`.
  **L972 CN**: 以 `(__m256i)__builtin_lasx_xvabsd_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L973 EN**: Closes the current lexical scope or compound statement.
  **L973 CN**: 结束当前词法作用域或复合语句块。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L975 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L976 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L976 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L977 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvabsd_h(__m256i _1, __m256i _2) {`.
  **L977 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvabsd_h(__m256i _1, __m256i _2) {`。
- **L978 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvabsd_h((v16i16)_1, (v16i16)_2)`.
  **L978 CN**: 以 `(__m256i)__builtin_lasx_xvabsd_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L979 EN**: Closes the current lexical scope or compound statement.
  **L979 CN**: 结束当前词法作用域或复合语句块。
- **L980 EN**: Blank line separating nearby declarations or logic blocks.
  **L980 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L981 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L981 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L982 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L982 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L983 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvabsd_w(__m256i _1, __m256i _2) {`.
  **L983 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvabsd_w(__m256i _1, __m256i _2) {`。
- **L984 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvabsd_w((v8i32)_1, (v8i32)_2)`.
  **L984 CN**: 以 `(__m256i)__builtin_lasx_xvabsd_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 985-1008

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvabsd_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvabsd_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvabsd_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvabsd_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvabsd_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvabsd_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvabsd_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvabsd_wu((v8u32)_1, (v8u32)_2);
````
- **L985 EN**: Closes the current lexical scope or compound statement.
  **L985 CN**: 结束当前词法作用域或复合语句块。
- **L986 EN**: Blank line separating nearby declarations or logic blocks.
  **L986 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L987 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L987 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L988 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L988 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L989 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvabsd_d(__m256i _1, __m256i _2) {`.
  **L989 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvabsd_d(__m256i _1, __m256i _2) {`。
- **L990 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvabsd_d((v4i64)_1, (v4i64)_2)`.
  **L990 CN**: 以 `(__m256i)__builtin_lasx_xvabsd_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L991 EN**: Closes the current lexical scope or compound statement.
  **L991 CN**: 结束当前词法作用域或复合语句块。
- **L992 EN**: Blank line separating nearby declarations or logic blocks.
  **L992 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L993 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L993 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L994 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L994 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L995 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvabsd_bu(__m256i _1, __m256i _2) {`.
  **L995 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvabsd_bu(__m256i _1, __m256i _2) {`。
- **L996 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvabsd_bu((v32u8)_1, (v32u8)_2)`.
  **L996 CN**: 以 `(__m256i)__builtin_lasx_xvabsd_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L997 EN**: Closes the current lexical scope or compound statement.
  **L997 CN**: 结束当前词法作用域或复合语句块。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L999 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L999 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1000 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1000 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1001 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvabsd_hu(__m256i _1, __m256i _2) {`.
  **L1001 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvabsd_hu(__m256i _1, __m256i _2) {`。
- **L1002 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvabsd_hu((v16u16)_1, (v16u16)_2)`.
  **L1002 CN**: 以 `(__m256i)__builtin_lasx_xvabsd_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L1003 EN**: Closes the current lexical scope or compound statement.
  **L1003 CN**: 结束当前词法作用域或复合语句块。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1005 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1005 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1006 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1006 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1007 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvabsd_wu(__m256i _1, __m256i _2) {`.
  **L1007 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvabsd_wu(__m256i _1, __m256i _2) {`。
- **L1008 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvabsd_wu((v8u32)_1, (v8u32)_2)`.
  **L1008 CN**: 以 `(__m256i)__builtin_lasx_xvabsd_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 1009-1032

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvabsd_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvabsd_du((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmul_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmul_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmul_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmul_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmul_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmul_w((v8i32)_1, (v8i32)_2);
````
- **L1009 EN**: Closes the current lexical scope or compound statement.
  **L1009 CN**: 结束当前词法作用域或复合语句块。
- **L1010 EN**: Blank line separating nearby declarations or logic blocks.
  **L1010 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1011 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1011 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1012 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1012 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1013 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvabsd_du(__m256i _1, __m256i _2) {`.
  **L1013 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvabsd_du(__m256i _1, __m256i _2) {`。
- **L1014 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvabsd_du((v4u64)_1, (v4u64)_2)`.
  **L1014 CN**: 以 `(__m256i)__builtin_lasx_xvabsd_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L1015 EN**: Closes the current lexical scope or compound statement.
  **L1015 CN**: 结束当前词法作用域或复合语句块。
- **L1016 EN**: Blank line separating nearby declarations or logic blocks.
  **L1016 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1017 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1017 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1018 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1018 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1019 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmul_b(__m256i _1, __m256i _2) {`.
  **L1019 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmul_b(__m256i _1, __m256i _2) {`。
- **L1020 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmul_b((v32i8)_1, (v32i8)_2)`.
  **L1020 CN**: 以 `(__m256i)__builtin_lasx_xvmul_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L1021 EN**: Closes the current lexical scope or compound statement.
  **L1021 CN**: 结束当前词法作用域或复合语句块。
- **L1022 EN**: Blank line separating nearby declarations or logic blocks.
  **L1022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1023 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1023 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1024 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1024 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1025 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmul_h(__m256i _1, __m256i _2) {`.
  **L1025 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmul_h(__m256i _1, __m256i _2) {`。
- **L1026 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmul_h((v16i16)_1, (v16i16)_2)`.
  **L1026 CN**: 以 `(__m256i)__builtin_lasx_xvmul_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1027 EN**: Closes the current lexical scope or compound statement.
  **L1027 CN**: 结束当前词法作用域或复合语句块。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1029 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1029 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1030 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1030 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1031 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmul_w(__m256i _1, __m256i _2) {`.
  **L1031 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmul_w(__m256i _1, __m256i _2) {`。
- **L1032 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmul_w((v8i32)_1, (v8i32)_2)`.
  **L1032 CN**: 以 `(__m256i)__builtin_lasx_xvmul_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 1033-1056

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmul_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmul_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmadd_b(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmadd_b((v32i8)_1, (v32i8)_2, (v32i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmadd_h(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmadd_h((v16i16)_1, (v16i16)_2, (v16i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmadd_w(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmadd_w((v8i32)_1, (v8i32)_2, (v8i32)_3);
````
- **L1033 EN**: Closes the current lexical scope or compound statement.
  **L1033 CN**: 结束当前词法作用域或复合语句块。
- **L1034 EN**: Blank line separating nearby declarations or logic blocks.
  **L1034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1035 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1035 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1036 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1036 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1037 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmul_d(__m256i _1, __m256i _2) {`.
  **L1037 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmul_d(__m256i _1, __m256i _2) {`。
- **L1038 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmul_d((v4i64)_1, (v4i64)_2)`.
  **L1038 CN**: 以 `(__m256i)__builtin_lasx_xvmul_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1039 EN**: Closes the current lexical scope or compound statement.
  **L1039 CN**: 结束当前词法作用域或复合语句块。
- **L1040 EN**: Blank line separating nearby declarations or logic blocks.
  **L1040 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1041 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1041 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1042 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1042 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1043 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmadd_b(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1043 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmadd_b(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1044 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmadd_b((v32i8)_1, (v32i8)_2, (v32i8)_3)`.
  **L1044 CN**: 以 `(__m256i)__builtin_lasx_xvmadd_b((v32i8)_1, (v32i8)_2, (v32i8)_3)` 从当前函数返回。
- **L1045 EN**: Closes the current lexical scope or compound statement.
  **L1045 CN**: 结束当前词法作用域或复合语句块。
- **L1046 EN**: Blank line separating nearby declarations or logic blocks.
  **L1046 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1047 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1047 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1048 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1048 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1049 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmadd_h(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1049 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmadd_h(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1050 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmadd_h((v16i16)_1, (v16i16)_2, (v16i16)_3)`.
  **L1050 CN**: 以 `(__m256i)__builtin_lasx_xvmadd_h((v16i16)_1, (v16i16)_2, (v16i16)_3)` 从当前函数返回。
- **L1051 EN**: Closes the current lexical scope or compound statement.
  **L1051 CN**: 结束当前词法作用域或复合语句块。
- **L1052 EN**: Blank line separating nearby declarations or logic blocks.
  **L1052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1053 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1053 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1054 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1054 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1055 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmadd_w(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1055 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmadd_w(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1056 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmadd_w((v8i32)_1, (v8i32)_2, (v8i32)_3)`.
  **L1056 CN**: 以 `(__m256i)__builtin_lasx_xvmadd_w((v8i32)_1, (v8i32)_2, (v8i32)_3)` 从当前函数返回。

### Lines 1057-1080

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmadd_d(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmadd_d((v4i64)_1, (v4i64)_2, (v4i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmsub_b(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmsub_b((v32i8)_1, (v32i8)_2, (v32i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmsub_h(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmsub_h((v16i16)_1, (v16i16)_2, (v16i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmsub_w(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmsub_w((v8i32)_1, (v8i32)_2, (v8i32)_3);
````
- **L1057 EN**: Closes the current lexical scope or compound statement.
  **L1057 CN**: 结束当前词法作用域或复合语句块。
- **L1058 EN**: Blank line separating nearby declarations or logic blocks.
  **L1058 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1059 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1059 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1060 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1060 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1061 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmadd_d(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1061 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmadd_d(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1062 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmadd_d((v4i64)_1, (v4i64)_2, (v4i64)_3)`.
  **L1062 CN**: 以 `(__m256i)__builtin_lasx_xvmadd_d((v4i64)_1, (v4i64)_2, (v4i64)_3)` 从当前函数返回。
- **L1063 EN**: Closes the current lexical scope or compound statement.
  **L1063 CN**: 结束当前词法作用域或复合语句块。
- **L1064 EN**: Blank line separating nearby declarations or logic blocks.
  **L1064 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1065 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1065 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1066 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1066 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1067 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmsub_b(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1067 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmsub_b(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1068 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmsub_b((v32i8)_1, (v32i8)_2, (v32i8)_3)`.
  **L1068 CN**: 以 `(__m256i)__builtin_lasx_xvmsub_b((v32i8)_1, (v32i8)_2, (v32i8)_3)` 从当前函数返回。
- **L1069 EN**: Closes the current lexical scope or compound statement.
  **L1069 CN**: 结束当前词法作用域或复合语句块。
- **L1070 EN**: Blank line separating nearby declarations or logic blocks.
  **L1070 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1071 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1071 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1072 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1072 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1073 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmsub_h(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1073 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmsub_h(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1074 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmsub_h((v16i16)_1, (v16i16)_2, (v16i16)_3)`.
  **L1074 CN**: 以 `(__m256i)__builtin_lasx_xvmsub_h((v16i16)_1, (v16i16)_2, (v16i16)_3)` 从当前函数返回。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Blank line separating nearby declarations or logic blocks.
  **L1076 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1077 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1077 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1078 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1078 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1079 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmsub_w(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1079 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmsub_w(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1080 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmsub_w((v8i32)_1, (v8i32)_2, (v8i32)_3)`.
  **L1080 CN**: 以 `(__m256i)__builtin_lasx_xvmsub_w((v8i32)_1, (v8i32)_2, (v8i32)_3)` 从当前函数返回。

### Lines 1081-1104

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmsub_d(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmsub_d((v4i64)_1, (v4i64)_2, (v4i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvdiv_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvdiv_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvdiv_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvdiv_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvdiv_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvdiv_w((v8i32)_1, (v8i32)_2);
````
- **L1081 EN**: Closes the current lexical scope or compound statement.
  **L1081 CN**: 结束当前词法作用域或复合语句块。
- **L1082 EN**: Blank line separating nearby declarations or logic blocks.
  **L1082 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1083 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1083 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1084 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1084 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1085 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmsub_d(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1085 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmsub_d(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1086 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmsub_d((v4i64)_1, (v4i64)_2, (v4i64)_3)`.
  **L1086 CN**: 以 `(__m256i)__builtin_lasx_xvmsub_d((v4i64)_1, (v4i64)_2, (v4i64)_3)` 从当前函数返回。
- **L1087 EN**: Closes the current lexical scope or compound statement.
  **L1087 CN**: 结束当前词法作用域或复合语句块。
- **L1088 EN**: Blank line separating nearby declarations or logic blocks.
  **L1088 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1089 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1089 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1090 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1090 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1091 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvdiv_b(__m256i _1, __m256i _2) {`.
  **L1091 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvdiv_b(__m256i _1, __m256i _2) {`。
- **L1092 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvdiv_b((v32i8)_1, (v32i8)_2)`.
  **L1092 CN**: 以 `(__m256i)__builtin_lasx_xvdiv_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L1093 EN**: Closes the current lexical scope or compound statement.
  **L1093 CN**: 结束当前词法作用域或复合语句块。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1095 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1095 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1096 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1096 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1097 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvdiv_h(__m256i _1, __m256i _2) {`.
  **L1097 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvdiv_h(__m256i _1, __m256i _2) {`。
- **L1098 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvdiv_h((v16i16)_1, (v16i16)_2)`.
  **L1098 CN**: 以 `(__m256i)__builtin_lasx_xvdiv_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1099 EN**: Closes the current lexical scope or compound statement.
  **L1099 CN**: 结束当前词法作用域或复合语句块。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1101 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1101 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1102 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1102 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1103 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvdiv_w(__m256i _1, __m256i _2) {`.
  **L1103 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvdiv_w(__m256i _1, __m256i _2) {`。
- **L1104 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvdiv_w((v8i32)_1, (v8i32)_2)`.
  **L1104 CN**: 以 `(__m256i)__builtin_lasx_xvdiv_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 1105-1128

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvdiv_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvdiv_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvdiv_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvdiv_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvdiv_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvdiv_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvdiv_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvdiv_wu((v8u32)_1, (v8u32)_2);
````
- **L1105 EN**: Closes the current lexical scope or compound statement.
  **L1105 CN**: 结束当前词法作用域或复合语句块。
- **L1106 EN**: Blank line separating nearby declarations or logic blocks.
  **L1106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1107 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1107 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1108 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1108 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1109 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvdiv_d(__m256i _1, __m256i _2) {`.
  **L1109 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvdiv_d(__m256i _1, __m256i _2) {`。
- **L1110 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvdiv_d((v4i64)_1, (v4i64)_2)`.
  **L1110 CN**: 以 `(__m256i)__builtin_lasx_xvdiv_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1111 EN**: Closes the current lexical scope or compound statement.
  **L1111 CN**: 结束当前词法作用域或复合语句块。
- **L1112 EN**: Blank line separating nearby declarations or logic blocks.
  **L1112 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1113 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1113 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1114 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1114 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1115 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvdiv_bu(__m256i _1, __m256i _2) {`.
  **L1115 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvdiv_bu(__m256i _1, __m256i _2) {`。
- **L1116 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvdiv_bu((v32u8)_1, (v32u8)_2)`.
  **L1116 CN**: 以 `(__m256i)__builtin_lasx_xvdiv_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1117 EN**: Closes the current lexical scope or compound statement.
  **L1117 CN**: 结束当前词法作用域或复合语句块。
- **L1118 EN**: Blank line separating nearby declarations or logic blocks.
  **L1118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1119 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1119 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1120 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1120 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1121 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvdiv_hu(__m256i _1, __m256i _2) {`.
  **L1121 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvdiv_hu(__m256i _1, __m256i _2) {`。
- **L1122 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvdiv_hu((v16u16)_1, (v16u16)_2)`.
  **L1122 CN**: 以 `(__m256i)__builtin_lasx_xvdiv_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L1123 EN**: Closes the current lexical scope or compound statement.
  **L1123 CN**: 结束当前词法作用域或复合语句块。
- **L1124 EN**: Blank line separating nearby declarations or logic blocks.
  **L1124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1125 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1125 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1126 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1126 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1127 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvdiv_wu(__m256i _1, __m256i _2) {`.
  **L1127 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvdiv_wu(__m256i _1, __m256i _2) {`。
- **L1128 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvdiv_wu((v8u32)_1, (v8u32)_2)`.
  **L1128 CN**: 以 `(__m256i)__builtin_lasx_xvdiv_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 1129-1152

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvdiv_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvdiv_du((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhaddw_h_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhaddw_h_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhaddw_w_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhaddw_w_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhaddw_d_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhaddw_d_w((v8i32)_1, (v8i32)_2);
````
- **L1129 EN**: Closes the current lexical scope or compound statement.
  **L1129 CN**: 结束当前词法作用域或复合语句块。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1131 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1131 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1132 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1132 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1133 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvdiv_du(__m256i _1, __m256i _2) {`.
  **L1133 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvdiv_du(__m256i _1, __m256i _2) {`。
- **L1134 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvdiv_du((v4u64)_1, (v4u64)_2)`.
  **L1134 CN**: 以 `(__m256i)__builtin_lasx_xvdiv_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L1135 EN**: Closes the current lexical scope or compound statement.
  **L1135 CN**: 结束当前词法作用域或复合语句块。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1137 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1137 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1138 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1138 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1139 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhaddw_h_b(__m256i _1, __m256i _2) {`.
  **L1139 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhaddw_h_b(__m256i _1, __m256i _2) {`。
- **L1140 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhaddw_h_b((v32i8)_1, (v32i8)_2)`.
  **L1140 CN**: 以 `(__m256i)__builtin_lasx_xvhaddw_h_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L1141 EN**: Closes the current lexical scope or compound statement.
  **L1141 CN**: 结束当前词法作用域或复合语句块。
- **L1142 EN**: Blank line separating nearby declarations or logic blocks.
  **L1142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1143 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1143 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1144 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1144 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1145 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhaddw_w_h(__m256i _1, __m256i _2) {`.
  **L1145 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhaddw_w_h(__m256i _1, __m256i _2) {`。
- **L1146 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhaddw_w_h((v16i16)_1, (v16i16)_2)`.
  **L1146 CN**: 以 `(__m256i)__builtin_lasx_xvhaddw_w_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1147 EN**: Closes the current lexical scope or compound statement.
  **L1147 CN**: 结束当前词法作用域或复合语句块。
- **L1148 EN**: Blank line separating nearby declarations or logic blocks.
  **L1148 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1149 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1149 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1150 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1150 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1151 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhaddw_d_w(__m256i _1, __m256i _2) {`.
  **L1151 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhaddw_d_w(__m256i _1, __m256i _2) {`。
- **L1152 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhaddw_d_w((v8i32)_1, (v8i32)_2)`.
  **L1152 CN**: 以 `(__m256i)__builtin_lasx_xvhaddw_d_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 1153-1176

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhaddw_hu_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhaddw_hu_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhaddw_wu_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhaddw_wu_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhaddw_du_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhaddw_du_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhsubw_h_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhsubw_h_b((v32i8)_1, (v32i8)_2);
````
- **L1153 EN**: Closes the current lexical scope or compound statement.
  **L1153 CN**: 结束当前词法作用域或复合语句块。
- **L1154 EN**: Blank line separating nearby declarations or logic blocks.
  **L1154 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1155 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1155 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1156 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1156 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1157 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhaddw_hu_bu(__m256i _1, __m256i _2) {`.
  **L1157 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhaddw_hu_bu(__m256i _1, __m256i _2) {`。
- **L1158 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhaddw_hu_bu((v32u8)_1, (v32u8)_2)`.
  **L1158 CN**: 以 `(__m256i)__builtin_lasx_xvhaddw_hu_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Blank line separating nearby declarations or logic blocks.
  **L1160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1161 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1161 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1162 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1162 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhaddw_wu_hu(__m256i _1, __m256i _2) {`.
  **L1163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhaddw_wu_hu(__m256i _1, __m256i _2) {`。
- **L1164 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhaddw_wu_hu((v16u16)_1, (v16u16)_2)`.
  **L1164 CN**: 以 `(__m256i)__builtin_lasx_xvhaddw_wu_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1167 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1167 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1168 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1168 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1169 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhaddw_du_wu(__m256i _1, __m256i _2) {`.
  **L1169 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhaddw_du_wu(__m256i _1, __m256i _2) {`。
- **L1170 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhaddw_du_wu((v8u32)_1, (v8u32)_2)`.
  **L1170 CN**: 以 `(__m256i)__builtin_lasx_xvhaddw_du_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L1171 EN**: Closes the current lexical scope or compound statement.
  **L1171 CN**: 结束当前词法作用域或复合语句块。
- **L1172 EN**: Blank line separating nearby declarations or logic blocks.
  **L1172 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1173 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1173 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1174 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1174 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1175 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhsubw_h_b(__m256i _1, __m256i _2) {`.
  **L1175 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhsubw_h_b(__m256i _1, __m256i _2) {`。
- **L1176 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhsubw_h_b((v32i8)_1, (v32i8)_2)`.
  **L1176 CN**: 以 `(__m256i)__builtin_lasx_xvhsubw_h_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 1177-1200

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhsubw_w_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhsubw_w_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhsubw_d_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhsubw_d_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhsubw_hu_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhsubw_hu_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhsubw_wu_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhsubw_wu_hu((v16u16)_1, (v16u16)_2);
````
- **L1177 EN**: Closes the current lexical scope or compound statement.
  **L1177 CN**: 结束当前词法作用域或复合语句块。
- **L1178 EN**: Blank line separating nearby declarations or logic blocks.
  **L1178 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1179 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1179 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1180 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1180 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1181 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhsubw_w_h(__m256i _1, __m256i _2) {`.
  **L1181 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhsubw_w_h(__m256i _1, __m256i _2) {`。
- **L1182 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhsubw_w_h((v16i16)_1, (v16i16)_2)`.
  **L1182 CN**: 以 `(__m256i)__builtin_lasx_xvhsubw_w_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1183 EN**: Closes the current lexical scope or compound statement.
  **L1183 CN**: 结束当前词法作用域或复合语句块。
- **L1184 EN**: Blank line separating nearby declarations or logic blocks.
  **L1184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1185 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1185 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1186 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1186 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1187 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhsubw_d_w(__m256i _1, __m256i _2) {`.
  **L1187 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhsubw_d_w(__m256i _1, __m256i _2) {`。
- **L1188 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhsubw_d_w((v8i32)_1, (v8i32)_2)`.
  **L1188 CN**: 以 `(__m256i)__builtin_lasx_xvhsubw_d_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1189 EN**: Closes the current lexical scope or compound statement.
  **L1189 CN**: 结束当前词法作用域或复合语句块。
- **L1190 EN**: Blank line separating nearby declarations or logic blocks.
  **L1190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1191 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1191 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1192 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1192 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1193 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhsubw_hu_bu(__m256i _1, __m256i _2) {`.
  **L1193 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhsubw_hu_bu(__m256i _1, __m256i _2) {`。
- **L1194 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhsubw_hu_bu((v32u8)_1, (v32u8)_2)`.
  **L1194 CN**: 以 `(__m256i)__builtin_lasx_xvhsubw_hu_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1195 EN**: Closes the current lexical scope or compound statement.
  **L1195 CN**: 结束当前词法作用域或复合语句块。
- **L1196 EN**: Blank line separating nearby declarations or logic blocks.
  **L1196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1197 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1197 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1198 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1198 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1199 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhsubw_wu_hu(__m256i _1, __m256i _2) {`.
  **L1199 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhsubw_wu_hu(__m256i _1, __m256i _2) {`。
- **L1200 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhsubw_wu_hu((v16u16)_1, (v16u16)_2)`.
  **L1200 CN**: 以 `(__m256i)__builtin_lasx_xvhsubw_wu_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。

### Lines 1201-1224

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhsubw_du_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhsubw_du_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmod_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmod_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmod_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmod_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmod_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmod_w((v8i32)_1, (v8i32)_2);
````
- **L1201 EN**: Closes the current lexical scope or compound statement.
  **L1201 CN**: 结束当前词法作用域或复合语句块。
- **L1202 EN**: Blank line separating nearby declarations or logic blocks.
  **L1202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1203 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1203 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1204 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1204 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1205 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhsubw_du_wu(__m256i _1, __m256i _2) {`.
  **L1205 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhsubw_du_wu(__m256i _1, __m256i _2) {`。
- **L1206 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhsubw_du_wu((v8u32)_1, (v8u32)_2)`.
  **L1206 CN**: 以 `(__m256i)__builtin_lasx_xvhsubw_du_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1209 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1209 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1210 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1210 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1211 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmod_b(__m256i _1, __m256i _2) {`.
  **L1211 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmod_b(__m256i _1, __m256i _2) {`。
- **L1212 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmod_b((v32i8)_1, (v32i8)_2)`.
  **L1212 CN**: 以 `(__m256i)__builtin_lasx_xvmod_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L1213 EN**: Closes the current lexical scope or compound statement.
  **L1213 CN**: 结束当前词法作用域或复合语句块。
- **L1214 EN**: Blank line separating nearby declarations or logic blocks.
  **L1214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1215 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1215 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1216 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1216 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1217 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmod_h(__m256i _1, __m256i _2) {`.
  **L1217 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmod_h(__m256i _1, __m256i _2) {`。
- **L1218 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmod_h((v16i16)_1, (v16i16)_2)`.
  **L1218 CN**: 以 `(__m256i)__builtin_lasx_xvmod_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1219 EN**: Closes the current lexical scope or compound statement.
  **L1219 CN**: 结束当前词法作用域或复合语句块。
- **L1220 EN**: Blank line separating nearby declarations or logic blocks.
  **L1220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1221 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1221 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1222 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1222 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1223 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmod_w(__m256i _1, __m256i _2) {`.
  **L1223 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmod_w(__m256i _1, __m256i _2) {`。
- **L1224 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmod_w((v8i32)_1, (v8i32)_2)`.
  **L1224 CN**: 以 `(__m256i)__builtin_lasx_xvmod_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 1225-1248

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmod_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmod_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmod_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmod_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmod_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmod_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmod_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmod_wu((v8u32)_1, (v8u32)_2);
````
- **L1225 EN**: Closes the current lexical scope or compound statement.
  **L1225 CN**: 结束当前词法作用域或复合语句块。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1227 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1227 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1228 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1228 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1229 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmod_d(__m256i _1, __m256i _2) {`.
  **L1229 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmod_d(__m256i _1, __m256i _2) {`。
- **L1230 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmod_d((v4i64)_1, (v4i64)_2)`.
  **L1230 CN**: 以 `(__m256i)__builtin_lasx_xvmod_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1231 EN**: Closes the current lexical scope or compound statement.
  **L1231 CN**: 结束当前词法作用域或复合语句块。
- **L1232 EN**: Blank line separating nearby declarations or logic blocks.
  **L1232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1233 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1233 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1234 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1234 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1235 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmod_bu(__m256i _1, __m256i _2) {`.
  **L1235 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmod_bu(__m256i _1, __m256i _2) {`。
- **L1236 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmod_bu((v32u8)_1, (v32u8)_2)`.
  **L1236 CN**: 以 `(__m256i)__builtin_lasx_xvmod_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Blank line separating nearby declarations or logic blocks.
  **L1238 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1239 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1239 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1240 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1240 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1241 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmod_hu(__m256i _1, __m256i _2) {`.
  **L1241 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmod_hu(__m256i _1, __m256i _2) {`。
- **L1242 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmod_hu((v16u16)_1, (v16u16)_2)`.
  **L1242 CN**: 以 `(__m256i)__builtin_lasx_xvmod_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L1243 EN**: Closes the current lexical scope or compound statement.
  **L1243 CN**: 结束当前词法作用域或复合语句块。
- **L1244 EN**: Blank line separating nearby declarations or logic blocks.
  **L1244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1245 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1245 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1246 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1246 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1247 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmod_wu(__m256i _1, __m256i _2) {`.
  **L1247 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmod_wu(__m256i _1, __m256i _2) {`。
- **L1248 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmod_wu((v8u32)_1, (v8u32)_2)`.
  **L1248 CN**: 以 `(__m256i)__builtin_lasx_xvmod_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。

### Lines 1249-1272

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmod_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmod_du((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvrepl128vei_b(/*__m256i*/ _1, /*ui4*/ _2)                      \
  ((__m256i)__builtin_lasx_xvrepl128vei_b((v32i8)(_1), (_2)))

#define __lasx_xvrepl128vei_h(/*__m256i*/ _1, /*ui3*/ _2)                      \
  ((__m256i)__builtin_lasx_xvrepl128vei_h((v16i16)(_1), (_2)))

#define __lasx_xvrepl128vei_w(/*__m256i*/ _1, /*ui2*/ _2)                      \
  ((__m256i)__builtin_lasx_xvrepl128vei_w((v8i32)(_1), (_2)))

#define __lasx_xvrepl128vei_d(/*__m256i*/ _1, /*ui1*/ _2)                      \
  ((__m256i)__builtin_lasx_xvrepl128vei_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpickev_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpickev_b((v32i8)_1, (v32i8)_2);
````
- **L1249 EN**: Closes the current lexical scope or compound statement.
  **L1249 CN**: 结束当前词法作用域或复合语句块。
- **L1250 EN**: Blank line separating nearby declarations or logic blocks.
  **L1250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1251 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1251 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1252 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1252 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1253 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmod_du(__m256i _1, __m256i _2) {`.
  **L1253 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmod_du(__m256i _1, __m256i _2) {`。
- **L1254 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmod_du((v4u64)_1, (v4u64)_2)`.
  **L1254 CN**: 以 `(__m256i)__builtin_lasx_xvmod_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L1255 EN**: Closes the current lexical scope or compound statement.
  **L1255 CN**: 结束当前词法作用域或复合语句块。
- **L1256 EN**: Blank line separating nearby declarations or logic blocks.
  **L1256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1257 EN**: Defines macro `__lasx_xvrepl128vei_b(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1257 CN**: 定义宏 `__lasx_xvrepl128vei_b(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L1258 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvrepl128vei_b`.
  **L1258 CN**: 继续与可调用符号 `__builtin_lasx_xvrepl128vei_b` 相关的逻辑。
- **L1259 EN**: Blank line separating nearby declarations or logic blocks.
  **L1259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1260 EN**: Defines macro `__lasx_xvrepl128vei_h(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1260 CN**: 定义宏 `__lasx_xvrepl128vei_h(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L1261 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvrepl128vei_h`.
  **L1261 CN**: 继续与可调用符号 `__builtin_lasx_xvrepl128vei_h` 相关的逻辑。
- **L1262 EN**: Blank line separating nearby declarations or logic blocks.
  **L1262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1263 EN**: Defines macro `__lasx_xvrepl128vei_w(/*__m256i*/ _1, /*ui2*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1263 CN**: 定义宏 `__lasx_xvrepl128vei_w(/*__m256i*/ _1, /*ui2*/ _2)`，用于条件编译、简写或 API 生成。
- **L1264 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvrepl128vei_w`.
  **L1264 CN**: 继续与可调用符号 `__builtin_lasx_xvrepl128vei_w` 相关的逻辑。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Defines macro `__lasx_xvrepl128vei_d(/*__m256i*/ _1, /*ui1*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1266 CN**: 定义宏 `__lasx_xvrepl128vei_d(/*__m256i*/ _1, /*ui1*/ _2)`，用于条件编译、简写或 API 生成。
- **L1267 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvrepl128vei_d`.
  **L1267 CN**: 继续与可调用符号 `__builtin_lasx_xvrepl128vei_d` 相关的逻辑。
- **L1268 EN**: Blank line separating nearby declarations or logic blocks.
  **L1268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1269 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1269 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1270 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1270 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1271 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpickev_b(__m256i _1, __m256i _2) {`.
  **L1271 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpickev_b(__m256i _1, __m256i _2) {`。
- **L1272 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpickev_b((v32i8)_1, (v32i8)_2)`.
  **L1272 CN**: 以 `(__m256i)__builtin_lasx_xvpickev_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 1273-1296

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpickev_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpickev_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpickev_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpickev_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpickev_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpickev_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpickod_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpickod_b((v32i8)_1, (v32i8)_2);
````
- **L1273 EN**: Closes the current lexical scope or compound statement.
  **L1273 CN**: 结束当前词法作用域或复合语句块。
- **L1274 EN**: Blank line separating nearby declarations or logic blocks.
  **L1274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1275 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1275 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1276 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1276 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1277 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpickev_h(__m256i _1, __m256i _2) {`.
  **L1277 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpickev_h(__m256i _1, __m256i _2) {`。
- **L1278 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpickev_h((v16i16)_1, (v16i16)_2)`.
  **L1278 CN**: 以 `(__m256i)__builtin_lasx_xvpickev_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1279 EN**: Closes the current lexical scope or compound statement.
  **L1279 CN**: 结束当前词法作用域或复合语句块。
- **L1280 EN**: Blank line separating nearby declarations or logic blocks.
  **L1280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1281 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1281 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1282 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1282 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1283 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpickev_w(__m256i _1, __m256i _2) {`.
  **L1283 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpickev_w(__m256i _1, __m256i _2) {`。
- **L1284 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpickev_w((v8i32)_1, (v8i32)_2)`.
  **L1284 CN**: 以 `(__m256i)__builtin_lasx_xvpickev_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1285 EN**: Closes the current lexical scope or compound statement.
  **L1285 CN**: 结束当前词法作用域或复合语句块。
- **L1286 EN**: Blank line separating nearby declarations or logic blocks.
  **L1286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1287 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1287 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1288 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1288 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1289 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpickev_d(__m256i _1, __m256i _2) {`.
  **L1289 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpickev_d(__m256i _1, __m256i _2) {`。
- **L1290 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpickev_d((v4i64)_1, (v4i64)_2)`.
  **L1290 CN**: 以 `(__m256i)__builtin_lasx_xvpickev_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1291 EN**: Closes the current lexical scope or compound statement.
  **L1291 CN**: 结束当前词法作用域或复合语句块。
- **L1292 EN**: Blank line separating nearby declarations or logic blocks.
  **L1292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1293 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1293 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1294 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1294 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1295 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpickod_b(__m256i _1, __m256i _2) {`.
  **L1295 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpickod_b(__m256i _1, __m256i _2) {`。
- **L1296 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpickod_b((v32i8)_1, (v32i8)_2)`.
  **L1296 CN**: 以 `(__m256i)__builtin_lasx_xvpickod_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 1297-1320

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpickod_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpickod_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpickod_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpickod_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpickod_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpickod_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvilvh_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvilvh_b((v32i8)_1, (v32i8)_2);
````
- **L1297 EN**: Closes the current lexical scope or compound statement.
  **L1297 CN**: 结束当前词法作用域或复合语句块。
- **L1298 EN**: Blank line separating nearby declarations or logic blocks.
  **L1298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1299 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1299 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1300 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1300 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1301 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpickod_h(__m256i _1, __m256i _2) {`.
  **L1301 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpickod_h(__m256i _1, __m256i _2) {`。
- **L1302 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpickod_h((v16i16)_1, (v16i16)_2)`.
  **L1302 CN**: 以 `(__m256i)__builtin_lasx_xvpickod_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1303 EN**: Closes the current lexical scope or compound statement.
  **L1303 CN**: 结束当前词法作用域或复合语句块。
- **L1304 EN**: Blank line separating nearby declarations or logic blocks.
  **L1304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1305 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1305 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1306 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1306 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1307 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpickod_w(__m256i _1, __m256i _2) {`.
  **L1307 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpickod_w(__m256i _1, __m256i _2) {`。
- **L1308 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpickod_w((v8i32)_1, (v8i32)_2)`.
  **L1308 CN**: 以 `(__m256i)__builtin_lasx_xvpickod_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1309 EN**: Closes the current lexical scope or compound statement.
  **L1309 CN**: 结束当前词法作用域或复合语句块。
- **L1310 EN**: Blank line separating nearby declarations or logic blocks.
  **L1310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1311 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1311 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1312 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1312 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1313 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpickod_d(__m256i _1, __m256i _2) {`.
  **L1313 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpickod_d(__m256i _1, __m256i _2) {`。
- **L1314 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpickod_d((v4i64)_1, (v4i64)_2)`.
  **L1314 CN**: 以 `(__m256i)__builtin_lasx_xvpickod_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1315 EN**: Closes the current lexical scope or compound statement.
  **L1315 CN**: 结束当前词法作用域或复合语句块。
- **L1316 EN**: Blank line separating nearby declarations or logic blocks.
  **L1316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1317 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1317 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1318 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1318 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1319 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvilvh_b(__m256i _1, __m256i _2) {`.
  **L1319 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvilvh_b(__m256i _1, __m256i _2) {`。
- **L1320 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvilvh_b((v32i8)_1, (v32i8)_2)`.
  **L1320 CN**: 以 `(__m256i)__builtin_lasx_xvilvh_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 1321-1344

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvilvh_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvilvh_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvilvh_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvilvh_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvilvh_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvilvh_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvilvl_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvilvl_b((v32i8)_1, (v32i8)_2);
````
- **L1321 EN**: Closes the current lexical scope or compound statement.
  **L1321 CN**: 结束当前词法作用域或复合语句块。
- **L1322 EN**: Blank line separating nearby declarations or logic blocks.
  **L1322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1323 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1323 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1324 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1324 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1325 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvilvh_h(__m256i _1, __m256i _2) {`.
  **L1325 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvilvh_h(__m256i _1, __m256i _2) {`。
- **L1326 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvilvh_h((v16i16)_1, (v16i16)_2)`.
  **L1326 CN**: 以 `(__m256i)__builtin_lasx_xvilvh_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1327 EN**: Closes the current lexical scope or compound statement.
  **L1327 CN**: 结束当前词法作用域或复合语句块。
- **L1328 EN**: Blank line separating nearby declarations or logic blocks.
  **L1328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1329 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1329 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1330 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1330 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1331 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvilvh_w(__m256i _1, __m256i _2) {`.
  **L1331 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvilvh_w(__m256i _1, __m256i _2) {`。
- **L1332 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvilvh_w((v8i32)_1, (v8i32)_2)`.
  **L1332 CN**: 以 `(__m256i)__builtin_lasx_xvilvh_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1333 EN**: Closes the current lexical scope or compound statement.
  **L1333 CN**: 结束当前词法作用域或复合语句块。
- **L1334 EN**: Blank line separating nearby declarations or logic blocks.
  **L1334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1335 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1335 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1336 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1336 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1337 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvilvh_d(__m256i _1, __m256i _2) {`.
  **L1337 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvilvh_d(__m256i _1, __m256i _2) {`。
- **L1338 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvilvh_d((v4i64)_1, (v4i64)_2)`.
  **L1338 CN**: 以 `(__m256i)__builtin_lasx_xvilvh_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1339 EN**: Closes the current lexical scope or compound statement.
  **L1339 CN**: 结束当前词法作用域或复合语句块。
- **L1340 EN**: Blank line separating nearby declarations or logic blocks.
  **L1340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1341 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1341 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1342 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1342 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1343 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvilvl_b(__m256i _1, __m256i _2) {`.
  **L1343 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvilvl_b(__m256i _1, __m256i _2) {`。
- **L1344 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvilvl_b((v32i8)_1, (v32i8)_2)`.
  **L1344 CN**: 以 `(__m256i)__builtin_lasx_xvilvl_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 1345-1368

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvilvl_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvilvl_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvilvl_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvilvl_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvilvl_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvilvl_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpackev_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpackev_b((v32i8)_1, (v32i8)_2);
````
- **L1345 EN**: Closes the current lexical scope or compound statement.
  **L1345 CN**: 结束当前词法作用域或复合语句块。
- **L1346 EN**: Blank line separating nearby declarations or logic blocks.
  **L1346 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1347 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1347 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1348 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1348 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1349 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvilvl_h(__m256i _1, __m256i _2) {`.
  **L1349 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvilvl_h(__m256i _1, __m256i _2) {`。
- **L1350 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvilvl_h((v16i16)_1, (v16i16)_2)`.
  **L1350 CN**: 以 `(__m256i)__builtin_lasx_xvilvl_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1351 EN**: Closes the current lexical scope or compound statement.
  **L1351 CN**: 结束当前词法作用域或复合语句块。
- **L1352 EN**: Blank line separating nearby declarations or logic blocks.
  **L1352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1353 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1353 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1354 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1354 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1355 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvilvl_w(__m256i _1, __m256i _2) {`.
  **L1355 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvilvl_w(__m256i _1, __m256i _2) {`。
- **L1356 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvilvl_w((v8i32)_1, (v8i32)_2)`.
  **L1356 CN**: 以 `(__m256i)__builtin_lasx_xvilvl_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1357 EN**: Closes the current lexical scope or compound statement.
  **L1357 CN**: 结束当前词法作用域或复合语句块。
- **L1358 EN**: Blank line separating nearby declarations or logic blocks.
  **L1358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1359 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1359 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1360 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1360 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1361 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvilvl_d(__m256i _1, __m256i _2) {`.
  **L1361 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvilvl_d(__m256i _1, __m256i _2) {`。
- **L1362 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvilvl_d((v4i64)_1, (v4i64)_2)`.
  **L1362 CN**: 以 `(__m256i)__builtin_lasx_xvilvl_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1363 EN**: Closes the current lexical scope or compound statement.
  **L1363 CN**: 结束当前词法作用域或复合语句块。
- **L1364 EN**: Blank line separating nearby declarations or logic blocks.
  **L1364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1365 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1365 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1366 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1366 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1367 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpackev_b(__m256i _1, __m256i _2) {`.
  **L1367 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpackev_b(__m256i _1, __m256i _2) {`。
- **L1368 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpackev_b((v32i8)_1, (v32i8)_2)`.
  **L1368 CN**: 以 `(__m256i)__builtin_lasx_xvpackev_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 1369-1392

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpackev_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpackev_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpackev_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpackev_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpackev_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpackev_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpackod_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpackod_b((v32i8)_1, (v32i8)_2);
````
- **L1369 EN**: Closes the current lexical scope or compound statement.
  **L1369 CN**: 结束当前词法作用域或复合语句块。
- **L1370 EN**: Blank line separating nearby declarations or logic blocks.
  **L1370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1371 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1371 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1372 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1372 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1373 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpackev_h(__m256i _1, __m256i _2) {`.
  **L1373 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpackev_h(__m256i _1, __m256i _2) {`。
- **L1374 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpackev_h((v16i16)_1, (v16i16)_2)`.
  **L1374 CN**: 以 `(__m256i)__builtin_lasx_xvpackev_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1375 EN**: Closes the current lexical scope or compound statement.
  **L1375 CN**: 结束当前词法作用域或复合语句块。
- **L1376 EN**: Blank line separating nearby declarations or logic blocks.
  **L1376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1377 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1377 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1378 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1378 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpackev_w(__m256i _1, __m256i _2) {`.
  **L1379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpackev_w(__m256i _1, __m256i _2) {`。
- **L1380 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpackev_w((v8i32)_1, (v8i32)_2)`.
  **L1380 CN**: 以 `(__m256i)__builtin_lasx_xvpackev_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1381 EN**: Closes the current lexical scope or compound statement.
  **L1381 CN**: 结束当前词法作用域或复合语句块。
- **L1382 EN**: Blank line separating nearby declarations or logic blocks.
  **L1382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1383 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1383 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1384 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1384 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpackev_d(__m256i _1, __m256i _2) {`.
  **L1385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpackev_d(__m256i _1, __m256i _2) {`。
- **L1386 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpackev_d((v4i64)_1, (v4i64)_2)`.
  **L1386 CN**: 以 `(__m256i)__builtin_lasx_xvpackev_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1387 EN**: Closes the current lexical scope or compound statement.
  **L1387 CN**: 结束当前词法作用域或复合语句块。
- **L1388 EN**: Blank line separating nearby declarations or logic blocks.
  **L1388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1389 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1389 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1390 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1390 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpackod_b(__m256i _1, __m256i _2) {`.
  **L1391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpackod_b(__m256i _1, __m256i _2) {`。
- **L1392 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpackod_b((v32i8)_1, (v32i8)_2)`.
  **L1392 CN**: 以 `(__m256i)__builtin_lasx_xvpackod_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 1393-1416

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpackod_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpackod_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpackod_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpackod_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpackod_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvpackod_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvshuf_b(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvshuf_b((v32i8)_1, (v32i8)_2, (v32i8)_3);
````
- **L1393 EN**: Closes the current lexical scope or compound statement.
  **L1393 CN**: 结束当前词法作用域或复合语句块。
- **L1394 EN**: Blank line separating nearby declarations or logic blocks.
  **L1394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1395 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1395 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1396 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1396 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1397 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpackod_h(__m256i _1, __m256i _2) {`.
  **L1397 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpackod_h(__m256i _1, __m256i _2) {`。
- **L1398 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpackod_h((v16i16)_1, (v16i16)_2)`.
  **L1398 CN**: 以 `(__m256i)__builtin_lasx_xvpackod_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1399 EN**: Closes the current lexical scope or compound statement.
  **L1399 CN**: 结束当前词法作用域或复合语句块。
- **L1400 EN**: Blank line separating nearby declarations or logic blocks.
  **L1400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1401 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1401 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1402 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1402 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1403 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpackod_w(__m256i _1, __m256i _2) {`.
  **L1403 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpackod_w(__m256i _1, __m256i _2) {`。
- **L1404 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpackod_w((v8i32)_1, (v8i32)_2)`.
  **L1404 CN**: 以 `(__m256i)__builtin_lasx_xvpackod_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1405 EN**: Closes the current lexical scope or compound statement.
  **L1405 CN**: 结束当前词法作用域或复合语句块。
- **L1406 EN**: Blank line separating nearby declarations or logic blocks.
  **L1406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1407 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1407 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1408 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1408 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1409 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpackod_d(__m256i _1, __m256i _2) {`.
  **L1409 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpackod_d(__m256i _1, __m256i _2) {`。
- **L1410 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpackod_d((v4i64)_1, (v4i64)_2)`.
  **L1410 CN**: 以 `(__m256i)__builtin_lasx_xvpackod_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1411 EN**: Closes the current lexical scope or compound statement.
  **L1411 CN**: 结束当前词法作用域或复合语句块。
- **L1412 EN**: Blank line separating nearby declarations or logic blocks.
  **L1412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1413 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1413 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1414 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1414 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1415 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvshuf_b(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1415 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvshuf_b(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1416 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvshuf_b((v32i8)_1, (v32i8)_2, (v32i8)_3)`.
  **L1416 CN**: 以 `(__m256i)__builtin_lasx_xvshuf_b((v32i8)_1, (v32i8)_2, (v32i8)_3)` 从当前函数返回。

### Lines 1417-1440

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvshuf_h(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvshuf_h((v16i16)_1, (v16i16)_2, (v16i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvshuf_w(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvshuf_w((v8i32)_1, (v8i32)_2, (v8i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvshuf_d(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvshuf_d((v4i64)_1, (v4i64)_2, (v4i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvand_v(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvand_v((v32u8)_1, (v32u8)_2);
````
- **L1417 EN**: Closes the current lexical scope or compound statement.
  **L1417 CN**: 结束当前词法作用域或复合语句块。
- **L1418 EN**: Blank line separating nearby declarations or logic blocks.
  **L1418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1419 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1419 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1420 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1420 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1421 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvshuf_h(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1421 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvshuf_h(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1422 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvshuf_h((v16i16)_1, (v16i16)_2, (v16i16)_3)`.
  **L1422 CN**: 以 `(__m256i)__builtin_lasx_xvshuf_h((v16i16)_1, (v16i16)_2, (v16i16)_3)` 从当前函数返回。
- **L1423 EN**: Closes the current lexical scope or compound statement.
  **L1423 CN**: 结束当前词法作用域或复合语句块。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1425 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1425 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1426 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1426 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1427 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvshuf_w(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1427 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvshuf_w(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1428 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvshuf_w((v8i32)_1, (v8i32)_2, (v8i32)_3)`.
  **L1428 CN**: 以 `(__m256i)__builtin_lasx_xvshuf_w((v8i32)_1, (v8i32)_2, (v8i32)_3)` 从当前函数返回。
- **L1429 EN**: Closes the current lexical scope or compound statement.
  **L1429 CN**: 结束当前词法作用域或复合语句块。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1431 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1431 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1432 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1432 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1433 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvshuf_d(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1433 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvshuf_d(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1434 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvshuf_d((v4i64)_1, (v4i64)_2, (v4i64)_3)`.
  **L1434 CN**: 以 `(__m256i)__builtin_lasx_xvshuf_d((v4i64)_1, (v4i64)_2, (v4i64)_3)` 从当前函数返回。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Blank line separating nearby declarations or logic blocks.
  **L1436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1437 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1437 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1438 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1438 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1439 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvand_v(__m256i _1, __m256i _2) {`.
  **L1439 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvand_v(__m256i _1, __m256i _2) {`。
- **L1440 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvand_v((v32u8)_1, (v32u8)_2)`.
  **L1440 CN**: 以 `(__m256i)__builtin_lasx_xvand_v((v32u8)_1, (v32u8)_2)` 从当前函数返回。

### Lines 1441-1464

````c
}

#define __lasx_xvandi_b(/*__m256i*/ _1, /*ui8*/ _2)                            \
  ((__m256i)__builtin_lasx_xvandi_b((v32u8)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvor_v(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvor_v((v32u8)_1, (v32u8)_2);
}

#define __lasx_xvori_b(/*__m256i*/ _1, /*ui8*/ _2)                             \
  ((__m256i)__builtin_lasx_xvori_b((v32u8)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvnor_v(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvnor_v((v32u8)_1, (v32u8)_2);
}

#define __lasx_xvnori_b(/*__m256i*/ _1, /*ui8*/ _2)                            \
  ((__m256i)__builtin_lasx_xvnori_b((v32u8)(_1), (_2)))

extern __inline
````
- **L1441 EN**: Closes the current lexical scope or compound statement.
  **L1441 CN**: 结束当前词法作用域或复合语句块。
- **L1442 EN**: Blank line separating nearby declarations or logic blocks.
  **L1442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1443 EN**: Defines macro `__lasx_xvandi_b(/*__m256i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1443 CN**: 定义宏 `__lasx_xvandi_b(/*__m256i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1444 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvandi_b`.
  **L1444 CN**: 继续与可调用符号 `__builtin_lasx_xvandi_b` 相关的逻辑。
- **L1445 EN**: Blank line separating nearby declarations or logic blocks.
  **L1445 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1446 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1446 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1447 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1447 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1448 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvor_v(__m256i _1, __m256i _2) {`.
  **L1448 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvor_v(__m256i _1, __m256i _2) {`。
- **L1449 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvor_v((v32u8)_1, (v32u8)_2)`.
  **L1449 CN**: 以 `(__m256i)__builtin_lasx_xvor_v((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1450 EN**: Closes the current lexical scope or compound statement.
  **L1450 CN**: 结束当前词法作用域或复合语句块。
- **L1451 EN**: Blank line separating nearby declarations or logic blocks.
  **L1451 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1452 EN**: Defines macro `__lasx_xvori_b(/*__m256i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1452 CN**: 定义宏 `__lasx_xvori_b(/*__m256i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1453 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvori_b`.
  **L1453 CN**: 继续与可调用符号 `__builtin_lasx_xvori_b` 相关的逻辑。
- **L1454 EN**: Blank line separating nearby declarations or logic blocks.
  **L1454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1455 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1455 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1456 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1456 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1457 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvnor_v(__m256i _1, __m256i _2) {`.
  **L1457 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvnor_v(__m256i _1, __m256i _2) {`。
- **L1458 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvnor_v((v32u8)_1, (v32u8)_2)`.
  **L1458 CN**: 以 `(__m256i)__builtin_lasx_xvnor_v((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1459 EN**: Closes the current lexical scope or compound statement.
  **L1459 CN**: 结束当前词法作用域或复合语句块。
- **L1460 EN**: Blank line separating nearby declarations or logic blocks.
  **L1460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1461 EN**: Defines macro `__lasx_xvnori_b(/*__m256i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1461 CN**: 定义宏 `__lasx_xvnori_b(/*__m256i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1462 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvnori_b`.
  **L1462 CN**: 继续与可调用符号 `__builtin_lasx_xvnori_b` 相关的逻辑。
- **L1463 EN**: Blank line separating nearby declarations or logic blocks.
  **L1463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1464 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1464 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 1465-1488

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvxor_v(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvxor_v((v32u8)_1, (v32u8)_2);
}

#define __lasx_xvxori_b(/*__m256i*/ _1, /*ui8*/ _2)                            \
  ((__m256i)__builtin_lasx_xvxori_b((v32u8)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvbitsel_v(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvbitsel_v((v32u8)_1, (v32u8)_2, (v32u8)_3);
}

#define __lasx_xvbitseli_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)         \
  ((__m256i)__builtin_lasx_xvbitseli_b((v32u8)(_1), (v32u8)(_2), (_3)))

#define __lasx_xvshuf4i_b(/*__m256i*/ _1, /*ui8*/ _2)                          \
  ((__m256i)__builtin_lasx_xvshuf4i_b((v32i8)(_1), (_2)))

#define __lasx_xvshuf4i_h(/*__m256i*/ _1, /*ui8*/ _2)                          \
  ((__m256i)__builtin_lasx_xvshuf4i_h((v16i16)(_1), (_2)))

#define __lasx_xvshuf4i_w(/*__m256i*/ _1, /*ui8*/ _2)                          \
````
- **L1465 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1465 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1466 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvxor_v(__m256i _1, __m256i _2) {`.
  **L1466 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvxor_v(__m256i _1, __m256i _2) {`。
- **L1467 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvxor_v((v32u8)_1, (v32u8)_2)`.
  **L1467 CN**: 以 `(__m256i)__builtin_lasx_xvxor_v((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1468 EN**: Closes the current lexical scope or compound statement.
  **L1468 CN**: 结束当前词法作用域或复合语句块。
- **L1469 EN**: Blank line separating nearby declarations or logic blocks.
  **L1469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1470 EN**: Defines macro `__lasx_xvxori_b(/*__m256i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1470 CN**: 定义宏 `__lasx_xvxori_b(/*__m256i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1471 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvxori_b`.
  **L1471 CN**: 继续与可调用符号 `__builtin_lasx_xvxori_b` 相关的逻辑。
- **L1472 EN**: Blank line separating nearby declarations or logic blocks.
  **L1472 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1473 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1473 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1474 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1474 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1475 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvbitsel_v(__m256i _1, __m256i _2, __m256i _3) {`.
  **L1475 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvbitsel_v(__m256i _1, __m256i _2, __m256i _3) {`。
- **L1476 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvbitsel_v((v32u8)_1, (v32u8)_2, (v32u8)_3)`.
  **L1476 CN**: 以 `(__m256i)__builtin_lasx_xvbitsel_v((v32u8)_1, (v32u8)_2, (v32u8)_3)` 从当前函数返回。
- **L1477 EN**: Closes the current lexical scope or compound statement.
  **L1477 CN**: 结束当前词法作用域或复合语句块。
- **L1478 EN**: Blank line separating nearby declarations or logic blocks.
  **L1478 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1479 EN**: Defines macro `__lasx_xvbitseli_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L1479 CN**: 定义宏 `__lasx_xvbitseli_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L1480 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbitseli_b`.
  **L1480 CN**: 继续与可调用符号 `__builtin_lasx_xvbitseli_b` 相关的逻辑。
- **L1481 EN**: Blank line separating nearby declarations or logic blocks.
  **L1481 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1482 EN**: Defines macro `__lasx_xvshuf4i_b(/*__m256i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1482 CN**: 定义宏 `__lasx_xvshuf4i_b(/*__m256i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1483 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvshuf4i_b`.
  **L1483 CN**: 继续与可调用符号 `__builtin_lasx_xvshuf4i_b` 相关的逻辑。
- **L1484 EN**: Blank line separating nearby declarations or logic blocks.
  **L1484 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1485 EN**: Defines macro `__lasx_xvshuf4i_h(/*__m256i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1485 CN**: 定义宏 `__lasx_xvshuf4i_h(/*__m256i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1486 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvshuf4i_h`.
  **L1486 CN**: 继续与可调用符号 `__builtin_lasx_xvshuf4i_h` 相关的逻辑。
- **L1487 EN**: Blank line separating nearby declarations or logic blocks.
  **L1487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1488 EN**: Defines macro `__lasx_xvshuf4i_w(/*__m256i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1488 CN**: 定义宏 `__lasx_xvshuf4i_w(/*__m256i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 1489-1512

````c
  ((__m256i)__builtin_lasx_xvshuf4i_w((v8i32)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplgr2vr_b(int _1) {
  return (__m256i)__builtin_lasx_xvreplgr2vr_b((int)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplgr2vr_h(int _1) {
  return (__m256i)__builtin_lasx_xvreplgr2vr_h((int)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplgr2vr_w(int _1) {
  return (__m256i)__builtin_lasx_xvreplgr2vr_w((int)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplgr2vr_d(long int _1) {
  return (__m256i)__builtin_lasx_xvreplgr2vr_d((long int)_1);
````
- **L1489 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvshuf4i_w`.
  **L1489 CN**: 继续与可调用符号 `__builtin_lasx_xvshuf4i_w` 相关的逻辑。
- **L1490 EN**: Blank line separating nearby declarations or logic blocks.
  **L1490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1491 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1491 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1492 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1492 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1493 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplgr2vr_b(int _1) {`.
  **L1493 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplgr2vr_b(int _1) {`。
- **L1494 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplgr2vr_b((int)_1)`.
  **L1494 CN**: 以 `(__m256i)__builtin_lasx_xvreplgr2vr_b((int)_1)` 从当前函数返回。
- **L1495 EN**: Closes the current lexical scope or compound statement.
  **L1495 CN**: 结束当前词法作用域或复合语句块。
- **L1496 EN**: Blank line separating nearby declarations or logic blocks.
  **L1496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1497 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1497 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1498 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1498 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1499 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplgr2vr_h(int _1) {`.
  **L1499 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplgr2vr_h(int _1) {`。
- **L1500 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplgr2vr_h((int)_1)`.
  **L1500 CN**: 以 `(__m256i)__builtin_lasx_xvreplgr2vr_h((int)_1)` 从当前函数返回。
- **L1501 EN**: Closes the current lexical scope or compound statement.
  **L1501 CN**: 结束当前词法作用域或复合语句块。
- **L1502 EN**: Blank line separating nearby declarations or logic blocks.
  **L1502 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1503 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1503 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1504 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1504 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1505 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplgr2vr_w(int _1) {`.
  **L1505 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplgr2vr_w(int _1) {`。
- **L1506 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplgr2vr_w((int)_1)`.
  **L1506 CN**: 以 `(__m256i)__builtin_lasx_xvreplgr2vr_w((int)_1)` 从当前函数返回。
- **L1507 EN**: Closes the current lexical scope or compound statement.
  **L1507 CN**: 结束当前词法作用域或复合语句块。
- **L1508 EN**: Blank line separating nearby declarations or logic blocks.
  **L1508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1509 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1509 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1510 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1510 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1511 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplgr2vr_d(long int _1) {`.
  **L1511 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplgr2vr_d(long int _1) {`。
- **L1512 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplgr2vr_d((long int)_1)`.
  **L1512 CN**: 以 `(__m256i)__builtin_lasx_xvreplgr2vr_d((long int)_1)` 从当前函数返回。

### Lines 1513-1536

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpcnt_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvpcnt_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpcnt_h(__m256i _1) {
  return (__m256i)__builtin_lasx_xvpcnt_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpcnt_w(__m256i _1) {
  return (__m256i)__builtin_lasx_xvpcnt_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvpcnt_d(__m256i _1) {
  return (__m256i)__builtin_lasx_xvpcnt_d((v4i64)_1);
````
- **L1513 EN**: Closes the current lexical scope or compound statement.
  **L1513 CN**: 结束当前词法作用域或复合语句块。
- **L1514 EN**: Blank line separating nearby declarations or logic blocks.
  **L1514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1515 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1515 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1516 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1516 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1517 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpcnt_b(__m256i _1) {`.
  **L1517 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpcnt_b(__m256i _1) {`。
- **L1518 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpcnt_b((v32i8)_1)`.
  **L1518 CN**: 以 `(__m256i)__builtin_lasx_xvpcnt_b((v32i8)_1)` 从当前函数返回。
- **L1519 EN**: Closes the current lexical scope or compound statement.
  **L1519 CN**: 结束当前词法作用域或复合语句块。
- **L1520 EN**: Blank line separating nearby declarations or logic blocks.
  **L1520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1521 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1521 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1522 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1522 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1523 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpcnt_h(__m256i _1) {`.
  **L1523 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpcnt_h(__m256i _1) {`。
- **L1524 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpcnt_h((v16i16)_1)`.
  **L1524 CN**: 以 `(__m256i)__builtin_lasx_xvpcnt_h((v16i16)_1)` 从当前函数返回。
- **L1525 EN**: Closes the current lexical scope or compound statement.
  **L1525 CN**: 结束当前词法作用域或复合语句块。
- **L1526 EN**: Blank line separating nearby declarations or logic blocks.
  **L1526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1527 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1527 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1528 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1528 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1529 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpcnt_w(__m256i _1) {`.
  **L1529 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpcnt_w(__m256i _1) {`。
- **L1530 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpcnt_w((v8i32)_1)`.
  **L1530 CN**: 以 `(__m256i)__builtin_lasx_xvpcnt_w((v8i32)_1)` 从当前函数返回。
- **L1531 EN**: Closes the current lexical scope or compound statement.
  **L1531 CN**: 结束当前词法作用域或复合语句块。
- **L1532 EN**: Blank line separating nearby declarations or logic blocks.
  **L1532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1533 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1533 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1534 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1534 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1535 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvpcnt_d(__m256i _1) {`.
  **L1535 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvpcnt_d(__m256i _1) {`。
- **L1536 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvpcnt_d((v4i64)_1)`.
  **L1536 CN**: 以 `(__m256i)__builtin_lasx_xvpcnt_d((v4i64)_1)` 从当前函数返回。

### Lines 1537-1560

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvclo_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvclo_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvclo_h(__m256i _1) {
  return (__m256i)__builtin_lasx_xvclo_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvclo_w(__m256i _1) {
  return (__m256i)__builtin_lasx_xvclo_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvclo_d(__m256i _1) {
  return (__m256i)__builtin_lasx_xvclo_d((v4i64)_1);
````
- **L1537 EN**: Closes the current lexical scope or compound statement.
  **L1537 CN**: 结束当前词法作用域或复合语句块。
- **L1538 EN**: Blank line separating nearby declarations or logic blocks.
  **L1538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1539 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1539 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1540 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1540 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1541 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvclo_b(__m256i _1) {`.
  **L1541 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvclo_b(__m256i _1) {`。
- **L1542 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvclo_b((v32i8)_1)`.
  **L1542 CN**: 以 `(__m256i)__builtin_lasx_xvclo_b((v32i8)_1)` 从当前函数返回。
- **L1543 EN**: Closes the current lexical scope or compound statement.
  **L1543 CN**: 结束当前词法作用域或复合语句块。
- **L1544 EN**: Blank line separating nearby declarations or logic blocks.
  **L1544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1545 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1545 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1546 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1546 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1547 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvclo_h(__m256i _1) {`.
  **L1547 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvclo_h(__m256i _1) {`。
- **L1548 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvclo_h((v16i16)_1)`.
  **L1548 CN**: 以 `(__m256i)__builtin_lasx_xvclo_h((v16i16)_1)` 从当前函数返回。
- **L1549 EN**: Closes the current lexical scope or compound statement.
  **L1549 CN**: 结束当前词法作用域或复合语句块。
- **L1550 EN**: Blank line separating nearby declarations or logic blocks.
  **L1550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1551 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1551 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1552 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1552 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1553 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvclo_w(__m256i _1) {`.
  **L1553 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvclo_w(__m256i _1) {`。
- **L1554 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvclo_w((v8i32)_1)`.
  **L1554 CN**: 以 `(__m256i)__builtin_lasx_xvclo_w((v8i32)_1)` 从当前函数返回。
- **L1555 EN**: Closes the current lexical scope or compound statement.
  **L1555 CN**: 结束当前词法作用域或复合语句块。
- **L1556 EN**: Blank line separating nearby declarations or logic blocks.
  **L1556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1557 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1557 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1558 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1558 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1559 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvclo_d(__m256i _1) {`.
  **L1559 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvclo_d(__m256i _1) {`。
- **L1560 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvclo_d((v4i64)_1)`.
  **L1560 CN**: 以 `(__m256i)__builtin_lasx_xvclo_d((v4i64)_1)` 从当前函数返回。

### Lines 1561-1584

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvclz_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvclz_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvclz_h(__m256i _1) {
  return (__m256i)__builtin_lasx_xvclz_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvclz_w(__m256i _1) {
  return (__m256i)__builtin_lasx_xvclz_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvclz_d(__m256i _1) {
  return (__m256i)__builtin_lasx_xvclz_d((v4i64)_1);
````
- **L1561 EN**: Closes the current lexical scope or compound statement.
  **L1561 CN**: 结束当前词法作用域或复合语句块。
- **L1562 EN**: Blank line separating nearby declarations or logic blocks.
  **L1562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1563 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1563 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1564 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1564 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1565 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvclz_b(__m256i _1) {`.
  **L1565 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvclz_b(__m256i _1) {`。
- **L1566 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvclz_b((v32i8)_1)`.
  **L1566 CN**: 以 `(__m256i)__builtin_lasx_xvclz_b((v32i8)_1)` 从当前函数返回。
- **L1567 EN**: Closes the current lexical scope or compound statement.
  **L1567 CN**: 结束当前词法作用域或复合语句块。
- **L1568 EN**: Blank line separating nearby declarations or logic blocks.
  **L1568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1569 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1569 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1570 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1570 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1571 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvclz_h(__m256i _1) {`.
  **L1571 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvclz_h(__m256i _1) {`。
- **L1572 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvclz_h((v16i16)_1)`.
  **L1572 CN**: 以 `(__m256i)__builtin_lasx_xvclz_h((v16i16)_1)` 从当前函数返回。
- **L1573 EN**: Closes the current lexical scope or compound statement.
  **L1573 CN**: 结束当前词法作用域或复合语句块。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1575 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1575 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1576 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1576 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1577 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvclz_w(__m256i _1) {`.
  **L1577 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvclz_w(__m256i _1) {`。
- **L1578 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvclz_w((v8i32)_1)`.
  **L1578 CN**: 以 `(__m256i)__builtin_lasx_xvclz_w((v8i32)_1)` 从当前函数返回。
- **L1579 EN**: Closes the current lexical scope or compound statement.
  **L1579 CN**: 结束当前词法作用域或复合语句块。
- **L1580 EN**: Blank line separating nearby declarations or logic blocks.
  **L1580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1581 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1581 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1582 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1582 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1583 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvclz_d(__m256i _1) {`.
  **L1583 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvclz_d(__m256i _1) {`。
- **L1584 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvclz_d((v4i64)_1)`.
  **L1584 CN**: 以 `(__m256i)__builtin_lasx_xvclz_d((v4i64)_1)` 从当前函数返回。

### Lines 1585-1608

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfadd_s(__m256 _1, __m256 _2) {
  return (__m256)__builtin_lasx_xvfadd_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfadd_d(__m256d _1, __m256d _2) {
  return (__m256d)__builtin_lasx_xvfadd_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfsub_s(__m256 _1, __m256 _2) {
  return (__m256)__builtin_lasx_xvfsub_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfsub_d(__m256d _1, __m256d _2) {
  return (__m256d)__builtin_lasx_xvfsub_d((v4f64)_1, (v4f64)_2);
````
- **L1585 EN**: Closes the current lexical scope or compound statement.
  **L1585 CN**: 结束当前词法作用域或复合语句块。
- **L1586 EN**: Blank line separating nearby declarations or logic blocks.
  **L1586 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1587 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1587 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1588 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1588 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1589 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfadd_s(__m256 _1, __m256 _2) {`.
  **L1589 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfadd_s(__m256 _1, __m256 _2) {`。
- **L1590 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfadd_s((v8f32)_1, (v8f32)_2)`.
  **L1590 CN**: 以 `(__m256)__builtin_lasx_xvfadd_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1591 EN**: Closes the current lexical scope or compound statement.
  **L1591 CN**: 结束当前词法作用域或复合语句块。
- **L1592 EN**: Blank line separating nearby declarations or logic blocks.
  **L1592 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1593 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1593 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1594 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1594 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1595 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfadd_d(__m256d _1, __m256d _2) {`.
  **L1595 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfadd_d(__m256d _1, __m256d _2) {`。
- **L1596 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfadd_d((v4f64)_1, (v4f64)_2)`.
  **L1596 CN**: 以 `(__m256d)__builtin_lasx_xvfadd_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L1597 EN**: Closes the current lexical scope or compound statement.
  **L1597 CN**: 结束当前词法作用域或复合语句块。
- **L1598 EN**: Blank line separating nearby declarations or logic blocks.
  **L1598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1599 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1599 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1600 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1600 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1601 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfsub_s(__m256 _1, __m256 _2) {`.
  **L1601 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfsub_s(__m256 _1, __m256 _2) {`。
- **L1602 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfsub_s((v8f32)_1, (v8f32)_2)`.
  **L1602 CN**: 以 `(__m256)__builtin_lasx_xvfsub_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1603 EN**: Closes the current lexical scope or compound statement.
  **L1603 CN**: 结束当前词法作用域或复合语句块。
- **L1604 EN**: Blank line separating nearby declarations or logic blocks.
  **L1604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1605 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1605 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1606 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1606 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1607 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfsub_d(__m256d _1, __m256d _2) {`.
  **L1607 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfsub_d(__m256d _1, __m256d _2) {`。
- **L1608 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfsub_d((v4f64)_1, (v4f64)_2)`.
  **L1608 CN**: 以 `(__m256d)__builtin_lasx_xvfsub_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。

### Lines 1609-1632

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfmul_s(__m256 _1, __m256 _2) {
  return (__m256)__builtin_lasx_xvfmul_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfmul_d(__m256d _1, __m256d _2) {
  return (__m256d)__builtin_lasx_xvfmul_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfdiv_s(__m256 _1, __m256 _2) {
  return (__m256)__builtin_lasx_xvfdiv_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfdiv_d(__m256d _1, __m256d _2) {
  return (__m256d)__builtin_lasx_xvfdiv_d((v4f64)_1, (v4f64)_2);
````
- **L1609 EN**: Closes the current lexical scope or compound statement.
  **L1609 CN**: 结束当前词法作用域或复合语句块。
- **L1610 EN**: Blank line separating nearby declarations or logic blocks.
  **L1610 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1611 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1611 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1612 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1612 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1613 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmul_s(__m256 _1, __m256 _2) {`.
  **L1613 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmul_s(__m256 _1, __m256 _2) {`。
- **L1614 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfmul_s((v8f32)_1, (v8f32)_2)`.
  **L1614 CN**: 以 `(__m256)__builtin_lasx_xvfmul_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1615 EN**: Closes the current lexical scope or compound statement.
  **L1615 CN**: 结束当前词法作用域或复合语句块。
- **L1616 EN**: Blank line separating nearby declarations or logic blocks.
  **L1616 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1617 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1617 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1618 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1618 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1619 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmul_d(__m256d _1, __m256d _2) {`.
  **L1619 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmul_d(__m256d _1, __m256d _2) {`。
- **L1620 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfmul_d((v4f64)_1, (v4f64)_2)`.
  **L1620 CN**: 以 `(__m256d)__builtin_lasx_xvfmul_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L1621 EN**: Closes the current lexical scope or compound statement.
  **L1621 CN**: 结束当前词法作用域或复合语句块。
- **L1622 EN**: Blank line separating nearby declarations or logic blocks.
  **L1622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1623 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1623 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1624 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1624 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1625 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfdiv_s(__m256 _1, __m256 _2) {`.
  **L1625 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfdiv_s(__m256 _1, __m256 _2) {`。
- **L1626 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfdiv_s((v8f32)_1, (v8f32)_2)`.
  **L1626 CN**: 以 `(__m256)__builtin_lasx_xvfdiv_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1627 EN**: Closes the current lexical scope or compound statement.
  **L1627 CN**: 结束当前词法作用域或复合语句块。
- **L1628 EN**: Blank line separating nearby declarations or logic blocks.
  **L1628 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1629 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1629 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1630 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1630 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1631 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfdiv_d(__m256d _1, __m256d _2) {`.
  **L1631 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfdiv_d(__m256d _1, __m256d _2) {`。
- **L1632 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfdiv_d((v4f64)_1, (v4f64)_2)`.
  **L1632 CN**: 以 `(__m256d)__builtin_lasx_xvfdiv_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。

### Lines 1633-1656

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcvt_h_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcvt_h_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfcvt_s_d(__m256d _1, __m256d _2) {
  return (__m256)__builtin_lasx_xvfcvt_s_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfmin_s(__m256 _1, __m256 _2) {
  return (__m256)__builtin_lasx_xvfmin_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfmin_d(__m256d _1, __m256d _2) {
  return (__m256d)__builtin_lasx_xvfmin_d((v4f64)_1, (v4f64)_2);
````
- **L1633 EN**: Closes the current lexical scope or compound statement.
  **L1633 CN**: 结束当前词法作用域或复合语句块。
- **L1634 EN**: Blank line separating nearby declarations or logic blocks.
  **L1634 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1635 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1635 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1636 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1636 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1637 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcvt_h_s(__m256 _1, __m256 _2) {`.
  **L1637 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcvt_h_s(__m256 _1, __m256 _2) {`。
- **L1638 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcvt_h_s((v8f32)_1, (v8f32)_2)`.
  **L1638 CN**: 以 `(__m256i)__builtin_lasx_xvfcvt_h_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1639 EN**: Closes the current lexical scope or compound statement.
  **L1639 CN**: 结束当前词法作用域或复合语句块。
- **L1640 EN**: Blank line separating nearby declarations or logic blocks.
  **L1640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1641 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1641 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1642 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1642 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1643 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcvt_s_d(__m256d _1, __m256d _2) {`.
  **L1643 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcvt_s_d(__m256d _1, __m256d _2) {`。
- **L1644 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfcvt_s_d((v4f64)_1, (v4f64)_2)`.
  **L1644 CN**: 以 `(__m256)__builtin_lasx_xvfcvt_s_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L1645 EN**: Closes the current lexical scope or compound statement.
  **L1645 CN**: 结束当前词法作用域或复合语句块。
- **L1646 EN**: Blank line separating nearby declarations or logic blocks.
  **L1646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1647 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1647 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1648 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1648 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1649 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmin_s(__m256 _1, __m256 _2) {`.
  **L1649 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmin_s(__m256 _1, __m256 _2) {`。
- **L1650 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfmin_s((v8f32)_1, (v8f32)_2)`.
  **L1650 CN**: 以 `(__m256)__builtin_lasx_xvfmin_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1651 EN**: Closes the current lexical scope or compound statement.
  **L1651 CN**: 结束当前词法作用域或复合语句块。
- **L1652 EN**: Blank line separating nearby declarations or logic blocks.
  **L1652 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1653 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1653 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1654 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1654 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1655 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmin_d(__m256d _1, __m256d _2) {`.
  **L1655 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmin_d(__m256d _1, __m256d _2) {`。
- **L1656 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfmin_d((v4f64)_1, (v4f64)_2)`.
  **L1656 CN**: 以 `(__m256d)__builtin_lasx_xvfmin_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。

### Lines 1657-1680

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfmina_s(__m256 _1, __m256 _2) {
  return (__m256)__builtin_lasx_xvfmina_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfmina_d(__m256d _1, __m256d _2) {
  return (__m256d)__builtin_lasx_xvfmina_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfmax_s(__m256 _1, __m256 _2) {
  return (__m256)__builtin_lasx_xvfmax_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfmax_d(__m256d _1, __m256d _2) {
  return (__m256d)__builtin_lasx_xvfmax_d((v4f64)_1, (v4f64)_2);
````
- **L1657 EN**: Closes the current lexical scope or compound statement.
  **L1657 CN**: 结束当前词法作用域或复合语句块。
- **L1658 EN**: Blank line separating nearby declarations or logic blocks.
  **L1658 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1659 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1659 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1660 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1660 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1661 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmina_s(__m256 _1, __m256 _2) {`.
  **L1661 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmina_s(__m256 _1, __m256 _2) {`。
- **L1662 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfmina_s((v8f32)_1, (v8f32)_2)`.
  **L1662 CN**: 以 `(__m256)__builtin_lasx_xvfmina_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1663 EN**: Closes the current lexical scope or compound statement.
  **L1663 CN**: 结束当前词法作用域或复合语句块。
- **L1664 EN**: Blank line separating nearby declarations or logic blocks.
  **L1664 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1665 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1665 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1666 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1666 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1667 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmina_d(__m256d _1, __m256d _2) {`.
  **L1667 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmina_d(__m256d _1, __m256d _2) {`。
- **L1668 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfmina_d((v4f64)_1, (v4f64)_2)`.
  **L1668 CN**: 以 `(__m256d)__builtin_lasx_xvfmina_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L1669 EN**: Closes the current lexical scope or compound statement.
  **L1669 CN**: 结束当前词法作用域或复合语句块。
- **L1670 EN**: Blank line separating nearby declarations or logic blocks.
  **L1670 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1671 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1671 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1672 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1672 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1673 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmax_s(__m256 _1, __m256 _2) {`.
  **L1673 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmax_s(__m256 _1, __m256 _2) {`。
- **L1674 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfmax_s((v8f32)_1, (v8f32)_2)`.
  **L1674 CN**: 以 `(__m256)__builtin_lasx_xvfmax_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1675 EN**: Closes the current lexical scope or compound statement.
  **L1675 CN**: 结束当前词法作用域或复合语句块。
- **L1676 EN**: Blank line separating nearby declarations or logic blocks.
  **L1676 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1677 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1677 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1678 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1678 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1679 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmax_d(__m256d _1, __m256d _2) {`.
  **L1679 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmax_d(__m256d _1, __m256d _2) {`。
- **L1680 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfmax_d((v4f64)_1, (v4f64)_2)`.
  **L1680 CN**: 以 `(__m256d)__builtin_lasx_xvfmax_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。

### Lines 1681-1704

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfmaxa_s(__m256 _1, __m256 _2) {
  return (__m256)__builtin_lasx_xvfmaxa_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfmaxa_d(__m256d _1, __m256d _2) {
  return (__m256d)__builtin_lasx_xvfmaxa_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfclass_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvfclass_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfclass_d(__m256d _1) {
  return (__m256i)__builtin_lasx_xvfclass_d((v4f64)_1);
````
- **L1681 EN**: Closes the current lexical scope or compound statement.
  **L1681 CN**: 结束当前词法作用域或复合语句块。
- **L1682 EN**: Blank line separating nearby declarations or logic blocks.
  **L1682 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1683 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1683 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1684 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1684 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1685 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmaxa_s(__m256 _1, __m256 _2) {`.
  **L1685 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmaxa_s(__m256 _1, __m256 _2) {`。
- **L1686 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfmaxa_s((v8f32)_1, (v8f32)_2)`.
  **L1686 CN**: 以 `(__m256)__builtin_lasx_xvfmaxa_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L1687 EN**: Closes the current lexical scope or compound statement.
  **L1687 CN**: 结束当前词法作用域或复合语句块。
- **L1688 EN**: Blank line separating nearby declarations or logic blocks.
  **L1688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1689 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1689 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1690 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1690 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1691 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmaxa_d(__m256d _1, __m256d _2) {`.
  **L1691 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmaxa_d(__m256d _1, __m256d _2) {`。
- **L1692 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfmaxa_d((v4f64)_1, (v4f64)_2)`.
  **L1692 CN**: 以 `(__m256d)__builtin_lasx_xvfmaxa_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L1693 EN**: Closes the current lexical scope or compound statement.
  **L1693 CN**: 结束当前词法作用域或复合语句块。
- **L1694 EN**: Blank line separating nearby declarations or logic blocks.
  **L1694 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1695 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1695 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1696 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1696 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1697 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfclass_s(__m256 _1) {`.
  **L1697 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfclass_s(__m256 _1) {`。
- **L1698 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfclass_s((v8f32)_1)`.
  **L1698 CN**: 以 `(__m256i)__builtin_lasx_xvfclass_s((v8f32)_1)` 从当前函数返回。
- **L1699 EN**: Closes the current lexical scope or compound statement.
  **L1699 CN**: 结束当前词法作用域或复合语句块。
- **L1700 EN**: Blank line separating nearby declarations or logic blocks.
  **L1700 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1701 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1701 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1702 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1702 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1703 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfclass_d(__m256d _1) {`.
  **L1703 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfclass_d(__m256d _1) {`。
- **L1704 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfclass_d((v4f64)_1)`.
  **L1704 CN**: 以 `(__m256i)__builtin_lasx_xvfclass_d((v4f64)_1)` 从当前函数返回。

### Lines 1705-1728

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfsqrt_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfsqrt_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfsqrt_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfsqrt_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrecip_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrecip_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrecip_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrecip_d((v4f64)_1);
````
- **L1705 EN**: Closes the current lexical scope or compound statement.
  **L1705 CN**: 结束当前词法作用域或复合语句块。
- **L1706 EN**: Blank line separating nearby declarations or logic blocks.
  **L1706 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1707 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1707 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1708 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1708 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1709 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfsqrt_s(__m256 _1) {`.
  **L1709 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfsqrt_s(__m256 _1) {`。
- **L1710 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfsqrt_s((v8f32)_1)`.
  **L1710 CN**: 以 `(__m256)__builtin_lasx_xvfsqrt_s((v8f32)_1)` 从当前函数返回。
- **L1711 EN**: Closes the current lexical scope or compound statement.
  **L1711 CN**: 结束当前词法作用域或复合语句块。
- **L1712 EN**: Blank line separating nearby declarations or logic blocks.
  **L1712 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1713 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1713 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1714 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1714 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1715 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfsqrt_d(__m256d _1) {`.
  **L1715 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfsqrt_d(__m256d _1) {`。
- **L1716 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfsqrt_d((v4f64)_1)`.
  **L1716 CN**: 以 `(__m256d)__builtin_lasx_xvfsqrt_d((v4f64)_1)` 从当前函数返回。
- **L1717 EN**: Closes the current lexical scope or compound statement.
  **L1717 CN**: 结束当前词法作用域或复合语句块。
- **L1718 EN**: Blank line separating nearby declarations or logic blocks.
  **L1718 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1719 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1719 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1720 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1720 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1721 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrecip_s(__m256 _1) {`.
  **L1721 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrecip_s(__m256 _1) {`。
- **L1722 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrecip_s((v8f32)_1)`.
  **L1722 CN**: 以 `(__m256)__builtin_lasx_xvfrecip_s((v8f32)_1)` 从当前函数返回。
- **L1723 EN**: Closes the current lexical scope or compound statement.
  **L1723 CN**: 结束当前词法作用域或复合语句块。
- **L1724 EN**: Blank line separating nearby declarations or logic blocks.
  **L1724 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1725 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1725 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1726 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1726 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1727 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrecip_d(__m256d _1) {`.
  **L1727 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrecip_d(__m256d _1) {`。
- **L1728 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrecip_d((v4f64)_1)`.
  **L1728 CN**: 以 `(__m256d)__builtin_lasx_xvfrecip_d((v4f64)_1)` 从当前函数返回。

### Lines 1729-1752

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrint_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrint_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrint_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrint_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrsqrt_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrsqrt_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrsqrt_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrsqrt_d((v4f64)_1);
````
- **L1729 EN**: Closes the current lexical scope or compound statement.
  **L1729 CN**: 结束当前词法作用域或复合语句块。
- **L1730 EN**: Blank line separating nearby declarations or logic blocks.
  **L1730 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1731 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1731 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1732 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1732 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1733 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrint_s(__m256 _1) {`.
  **L1733 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrint_s(__m256 _1) {`。
- **L1734 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrint_s((v8f32)_1)`.
  **L1734 CN**: 以 `(__m256)__builtin_lasx_xvfrint_s((v8f32)_1)` 从当前函数返回。
- **L1735 EN**: Closes the current lexical scope or compound statement.
  **L1735 CN**: 结束当前词法作用域或复合语句块。
- **L1736 EN**: Blank line separating nearby declarations or logic blocks.
  **L1736 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1737 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1737 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1738 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1738 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1739 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrint_d(__m256d _1) {`.
  **L1739 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrint_d(__m256d _1) {`。
- **L1740 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrint_d((v4f64)_1)`.
  **L1740 CN**: 以 `(__m256d)__builtin_lasx_xvfrint_d((v4f64)_1)` 从当前函数返回。
- **L1741 EN**: Closes the current lexical scope or compound statement.
  **L1741 CN**: 结束当前词法作用域或复合语句块。
- **L1742 EN**: Blank line separating nearby declarations or logic blocks.
  **L1742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1743 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1743 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1744 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1744 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1745 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrsqrt_s(__m256 _1) {`.
  **L1745 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrsqrt_s(__m256 _1) {`。
- **L1746 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrsqrt_s((v8f32)_1)`.
  **L1746 CN**: 以 `(__m256)__builtin_lasx_xvfrsqrt_s((v8f32)_1)` 从当前函数返回。
- **L1747 EN**: Closes the current lexical scope or compound statement.
  **L1747 CN**: 结束当前词法作用域或复合语句块。
- **L1748 EN**: Blank line separating nearby declarations or logic blocks.
  **L1748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1749 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1749 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1750 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1750 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1751 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrsqrt_d(__m256d _1) {`.
  **L1751 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrsqrt_d(__m256d _1) {`。
- **L1752 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrsqrt_d((v4f64)_1)`.
  **L1752 CN**: 以 `(__m256d)__builtin_lasx_xvfrsqrt_d((v4f64)_1)` 从当前函数返回。

### Lines 1753-1776

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvflogb_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvflogb_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvflogb_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvflogb_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfcvth_s_h(__m256i _1) {
  return (__m256)__builtin_lasx_xvfcvth_s_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfcvth_d_s(__m256 _1) {
  return (__m256d)__builtin_lasx_xvfcvth_d_s((v8f32)_1);
````
- **L1753 EN**: Closes the current lexical scope or compound statement.
  **L1753 CN**: 结束当前词法作用域或复合语句块。
- **L1754 EN**: Blank line separating nearby declarations or logic blocks.
  **L1754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1755 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1755 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1756 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1756 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1757 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvflogb_s(__m256 _1) {`.
  **L1757 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvflogb_s(__m256 _1) {`。
- **L1758 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvflogb_s((v8f32)_1)`.
  **L1758 CN**: 以 `(__m256)__builtin_lasx_xvflogb_s((v8f32)_1)` 从当前函数返回。
- **L1759 EN**: Closes the current lexical scope or compound statement.
  **L1759 CN**: 结束当前词法作用域或复合语句块。
- **L1760 EN**: Blank line separating nearby declarations or logic blocks.
  **L1760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1761 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1761 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1762 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1762 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1763 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvflogb_d(__m256d _1) {`.
  **L1763 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvflogb_d(__m256d _1) {`。
- **L1764 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvflogb_d((v4f64)_1)`.
  **L1764 CN**: 以 `(__m256d)__builtin_lasx_xvflogb_d((v4f64)_1)` 从当前函数返回。
- **L1765 EN**: Closes the current lexical scope or compound statement.
  **L1765 CN**: 结束当前词法作用域或复合语句块。
- **L1766 EN**: Blank line separating nearby declarations or logic blocks.
  **L1766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1767 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1767 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1768 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1768 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1769 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcvth_s_h(__m256i _1) {`.
  **L1769 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcvth_s_h(__m256i _1) {`。
- **L1770 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfcvth_s_h((v16i16)_1)`.
  **L1770 CN**: 以 `(__m256)__builtin_lasx_xvfcvth_s_h((v16i16)_1)` 从当前函数返回。
- **L1771 EN**: Closes the current lexical scope or compound statement.
  **L1771 CN**: 结束当前词法作用域或复合语句块。
- **L1772 EN**: Blank line separating nearby declarations or logic blocks.
  **L1772 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1773 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1773 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1774 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1774 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1775 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcvth_d_s(__m256 _1) {`.
  **L1775 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcvth_d_s(__m256 _1) {`。
- **L1776 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfcvth_d_s((v8f32)_1)`.
  **L1776 CN**: 以 `(__m256d)__builtin_lasx_xvfcvth_d_s((v8f32)_1)` 从当前函数返回。

### Lines 1777-1800

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfcvtl_s_h(__m256i _1) {
  return (__m256)__builtin_lasx_xvfcvtl_s_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfcvtl_d_s(__m256 _1) {
  return (__m256d)__builtin_lasx_xvfcvtl_d_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftint_w_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftint_w_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftint_l_d(__m256d _1) {
  return (__m256i)__builtin_lasx_xvftint_l_d((v4f64)_1);
````
- **L1777 EN**: Closes the current lexical scope or compound statement.
  **L1777 CN**: 结束当前词法作用域或复合语句块。
- **L1778 EN**: Blank line separating nearby declarations or logic blocks.
  **L1778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1779 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1779 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1780 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1780 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1781 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcvtl_s_h(__m256i _1) {`.
  **L1781 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcvtl_s_h(__m256i _1) {`。
- **L1782 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfcvtl_s_h((v16i16)_1)`.
  **L1782 CN**: 以 `(__m256)__builtin_lasx_xvfcvtl_s_h((v16i16)_1)` 从当前函数返回。
- **L1783 EN**: Closes the current lexical scope or compound statement.
  **L1783 CN**: 结束当前词法作用域或复合语句块。
- **L1784 EN**: Blank line separating nearby declarations or logic blocks.
  **L1784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1785 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1785 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1786 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1786 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1787 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcvtl_d_s(__m256 _1) {`.
  **L1787 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcvtl_d_s(__m256 _1) {`。
- **L1788 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfcvtl_d_s((v8f32)_1)`.
  **L1788 CN**: 以 `(__m256d)__builtin_lasx_xvfcvtl_d_s((v8f32)_1)` 从当前函数返回。
- **L1789 EN**: Closes the current lexical scope or compound statement.
  **L1789 CN**: 结束当前词法作用域或复合语句块。
- **L1790 EN**: Blank line separating nearby declarations or logic blocks.
  **L1790 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1791 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1791 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1792 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1792 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1793 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftint_w_s(__m256 _1) {`.
  **L1793 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftint_w_s(__m256 _1) {`。
- **L1794 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftint_w_s((v8f32)_1)`.
  **L1794 CN**: 以 `(__m256i)__builtin_lasx_xvftint_w_s((v8f32)_1)` 从当前函数返回。
- **L1795 EN**: Closes the current lexical scope or compound statement.
  **L1795 CN**: 结束当前词法作用域或复合语句块。
- **L1796 EN**: Blank line separating nearby declarations or logic blocks.
  **L1796 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1797 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1797 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1798 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1798 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1799 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftint_l_d(__m256d _1) {`.
  **L1799 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftint_l_d(__m256d _1) {`。
- **L1800 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftint_l_d((v4f64)_1)`.
  **L1800 CN**: 以 `(__m256i)__builtin_lasx_xvftint_l_d((v4f64)_1)` 从当前函数返回。

### Lines 1801-1824

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftint_wu_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftint_wu_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftint_lu_d(__m256d _1) {
  return (__m256i)__builtin_lasx_xvftint_lu_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrz_w_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrz_w_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrz_l_d(__m256d _1) {
  return (__m256i)__builtin_lasx_xvftintrz_l_d((v4f64)_1);
````
- **L1801 EN**: Closes the current lexical scope or compound statement.
  **L1801 CN**: 结束当前词法作用域或复合语句块。
- **L1802 EN**: Blank line separating nearby declarations or logic blocks.
  **L1802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1803 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1803 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1804 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1804 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1805 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftint_wu_s(__m256 _1) {`.
  **L1805 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftint_wu_s(__m256 _1) {`。
- **L1806 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftint_wu_s((v8f32)_1)`.
  **L1806 CN**: 以 `(__m256i)__builtin_lasx_xvftint_wu_s((v8f32)_1)` 从当前函数返回。
- **L1807 EN**: Closes the current lexical scope or compound statement.
  **L1807 CN**: 结束当前词法作用域或复合语句块。
- **L1808 EN**: Blank line separating nearby declarations or logic blocks.
  **L1808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1809 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1809 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1810 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1810 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1811 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftint_lu_d(__m256d _1) {`.
  **L1811 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftint_lu_d(__m256d _1) {`。
- **L1812 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftint_lu_d((v4f64)_1)`.
  **L1812 CN**: 以 `(__m256i)__builtin_lasx_xvftint_lu_d((v4f64)_1)` 从当前函数返回。
- **L1813 EN**: Closes the current lexical scope or compound statement.
  **L1813 CN**: 结束当前词法作用域或复合语句块。
- **L1814 EN**: Blank line separating nearby declarations or logic blocks.
  **L1814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1815 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1815 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1816 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1816 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1817 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrz_w_s(__m256 _1) {`.
  **L1817 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrz_w_s(__m256 _1) {`。
- **L1818 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrz_w_s((v8f32)_1)`.
  **L1818 CN**: 以 `(__m256i)__builtin_lasx_xvftintrz_w_s((v8f32)_1)` 从当前函数返回。
- **L1819 EN**: Closes the current lexical scope or compound statement.
  **L1819 CN**: 结束当前词法作用域或复合语句块。
- **L1820 EN**: Blank line separating nearby declarations or logic blocks.
  **L1820 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1821 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1821 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1822 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1822 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1823 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrz_l_d(__m256d _1) {`.
  **L1823 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrz_l_d(__m256d _1) {`。
- **L1824 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrz_l_d((v4f64)_1)`.
  **L1824 CN**: 以 `(__m256i)__builtin_lasx_xvftintrz_l_d((v4f64)_1)` 从当前函数返回。

### Lines 1825-1848

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrz_wu_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrz_wu_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrz_lu_d(__m256d _1) {
  return (__m256i)__builtin_lasx_xvftintrz_lu_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvffint_s_w(__m256i _1) {
  return (__m256)__builtin_lasx_xvffint_s_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvffint_d_l(__m256i _1) {
  return (__m256d)__builtin_lasx_xvffint_d_l((v4i64)_1);
````
- **L1825 EN**: Closes the current lexical scope or compound statement.
  **L1825 CN**: 结束当前词法作用域或复合语句块。
- **L1826 EN**: Blank line separating nearby declarations or logic blocks.
  **L1826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1827 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1827 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1828 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1828 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1829 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrz_wu_s(__m256 _1) {`.
  **L1829 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrz_wu_s(__m256 _1) {`。
- **L1830 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrz_wu_s((v8f32)_1)`.
  **L1830 CN**: 以 `(__m256i)__builtin_lasx_xvftintrz_wu_s((v8f32)_1)` 从当前函数返回。
- **L1831 EN**: Closes the current lexical scope or compound statement.
  **L1831 CN**: 结束当前词法作用域或复合语句块。
- **L1832 EN**: Blank line separating nearby declarations or logic blocks.
  **L1832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1833 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1833 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1834 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1834 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1835 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrz_lu_d(__m256d _1) {`.
  **L1835 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrz_lu_d(__m256d _1) {`。
- **L1836 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrz_lu_d((v4f64)_1)`.
  **L1836 CN**: 以 `(__m256i)__builtin_lasx_xvftintrz_lu_d((v4f64)_1)` 从当前函数返回。
- **L1837 EN**: Closes the current lexical scope or compound statement.
  **L1837 CN**: 结束当前词法作用域或复合语句块。
- **L1838 EN**: Blank line separating nearby declarations or logic blocks.
  **L1838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1839 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1839 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1840 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1840 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1841 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvffint_s_w(__m256i _1) {`.
  **L1841 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvffint_s_w(__m256i _1) {`。
- **L1842 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvffint_s_w((v8i32)_1)`.
  **L1842 CN**: 以 `(__m256)__builtin_lasx_xvffint_s_w((v8i32)_1)` 从当前函数返回。
- **L1843 EN**: Closes the current lexical scope or compound statement.
  **L1843 CN**: 结束当前词法作用域或复合语句块。
- **L1844 EN**: Blank line separating nearby declarations or logic blocks.
  **L1844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1845 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1845 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1846 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1846 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1847 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvffint_d_l(__m256i _1) {`.
  **L1847 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvffint_d_l(__m256i _1) {`。
- **L1848 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvffint_d_l((v4i64)_1)`.
  **L1848 CN**: 以 `(__m256d)__builtin_lasx_xvffint_d_l((v4i64)_1)` 从当前函数返回。

### Lines 1849-1872

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvffint_s_wu(__m256i _1) {
  return (__m256)__builtin_lasx_xvffint_s_wu((v8u32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvffint_d_lu(__m256i _1) {
  return (__m256d)__builtin_lasx_xvffint_d_lu((v4u64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve_b(__m256i _1, int _2) {
  return (__m256i)__builtin_lasx_xvreplve_b((v32i8)_1, (int)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve_h(__m256i _1, int _2) {
  return (__m256i)__builtin_lasx_xvreplve_h((v16i16)_1, (int)_2);
````
- **L1849 EN**: Closes the current lexical scope or compound statement.
  **L1849 CN**: 结束当前词法作用域或复合语句块。
- **L1850 EN**: Blank line separating nearby declarations or logic blocks.
  **L1850 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1851 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1851 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1852 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L1852 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L1853 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvffint_s_wu(__m256i _1) {`.
  **L1853 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvffint_s_wu(__m256i _1) {`。
- **L1854 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvffint_s_wu((v8u32)_1)`.
  **L1854 CN**: 以 `(__m256)__builtin_lasx_xvffint_s_wu((v8u32)_1)` 从当前函数返回。
- **L1855 EN**: Closes the current lexical scope or compound statement.
  **L1855 CN**: 结束当前词法作用域或复合语句块。
- **L1856 EN**: Blank line separating nearby declarations or logic blocks.
  **L1856 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1857 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1857 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1858 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L1858 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L1859 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvffint_d_lu(__m256i _1) {`.
  **L1859 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvffint_d_lu(__m256i _1) {`。
- **L1860 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvffint_d_lu((v4u64)_1)`.
  **L1860 CN**: 以 `(__m256d)__builtin_lasx_xvffint_d_lu((v4u64)_1)` 从当前函数返回。
- **L1861 EN**: Closes the current lexical scope or compound statement.
  **L1861 CN**: 结束当前词法作用域或复合语句块。
- **L1862 EN**: Blank line separating nearby declarations or logic blocks.
  **L1862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1863 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1863 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1864 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1864 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1865 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve_b(__m256i _1, int _2) {`.
  **L1865 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve_b(__m256i _1, int _2) {`。
- **L1866 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve_b((v32i8)_1, (int)_2)`.
  **L1866 CN**: 以 `(__m256i)__builtin_lasx_xvreplve_b((v32i8)_1, (int)_2)` 从当前函数返回。
- **L1867 EN**: Closes the current lexical scope or compound statement.
  **L1867 CN**: 结束当前词法作用域或复合语句块。
- **L1868 EN**: Blank line separating nearby declarations or logic blocks.
  **L1868 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1869 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1869 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1870 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1870 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1871 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve_h(__m256i _1, int _2) {`.
  **L1871 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve_h(__m256i _1, int _2) {`。
- **L1872 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve_h((v16i16)_1, (int)_2)`.
  **L1872 CN**: 以 `(__m256i)__builtin_lasx_xvreplve_h((v16i16)_1, (int)_2)` 从当前函数返回。

### Lines 1873-1896

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve_w(__m256i _1, int _2) {
  return (__m256i)__builtin_lasx_xvreplve_w((v8i32)_1, (int)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve_d(__m256i _1, int _2) {
  return (__m256i)__builtin_lasx_xvreplve_d((v4i64)_1, (int)_2);
}

#define __lasx_xvpermi_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)           \
  ((__m256i)__builtin_lasx_xvpermi_w((v8i32)(_1), (v8i32)(_2), (_3)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvandn_v(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvandn_v((v32u8)_1, (v32u8)_2);
}

extern __inline
````
- **L1873 EN**: Closes the current lexical scope or compound statement.
  **L1873 CN**: 结束当前词法作用域或复合语句块。
- **L1874 EN**: Blank line separating nearby declarations or logic blocks.
  **L1874 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1875 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1875 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1876 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1876 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1877 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve_w(__m256i _1, int _2) {`.
  **L1877 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve_w(__m256i _1, int _2) {`。
- **L1878 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve_w((v8i32)_1, (int)_2)`.
  **L1878 CN**: 以 `(__m256i)__builtin_lasx_xvreplve_w((v8i32)_1, (int)_2)` 从当前函数返回。
- **L1879 EN**: Closes the current lexical scope or compound statement.
  **L1879 CN**: 结束当前词法作用域或复合语句块。
- **L1880 EN**: Blank line separating nearby declarations or logic blocks.
  **L1880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1881 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1881 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1882 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1882 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1883 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve_d(__m256i _1, int _2) {`.
  **L1883 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve_d(__m256i _1, int _2) {`。
- **L1884 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve_d((v4i64)_1, (int)_2)`.
  **L1884 CN**: 以 `(__m256i)__builtin_lasx_xvreplve_d((v4i64)_1, (int)_2)` 从当前函数返回。
- **L1885 EN**: Closes the current lexical scope or compound statement.
  **L1885 CN**: 结束当前词法作用域或复合语句块。
- **L1886 EN**: Blank line separating nearby declarations or logic blocks.
  **L1886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1887 EN**: Defines macro `__lasx_xvpermi_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L1887 CN**: 定义宏 `__lasx_xvpermi_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L1888 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpermi_w`.
  **L1888 CN**: 继续与可调用符号 `__builtin_lasx_xvpermi_w` 相关的逻辑。
- **L1889 EN**: Blank line separating nearby declarations or logic blocks.
  **L1889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1890 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1890 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1891 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1891 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1892 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvandn_v(__m256i _1, __m256i _2) {`.
  **L1892 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvandn_v(__m256i _1, __m256i _2) {`。
- **L1893 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvandn_v((v32u8)_1, (v32u8)_2)`.
  **L1893 CN**: 以 `(__m256i)__builtin_lasx_xvandn_v((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1894 EN**: Closes the current lexical scope or compound statement.
  **L1894 CN**: 结束当前词法作用域或复合语句块。
- **L1895 EN**: Blank line separating nearby declarations or logic blocks.
  **L1895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1896 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1896 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 1897-1920

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvneg_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvneg_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvneg_h(__m256i _1) {
  return (__m256i)__builtin_lasx_xvneg_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvneg_w(__m256i _1) {
  return (__m256i)__builtin_lasx_xvneg_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvneg_d(__m256i _1) {
  return (__m256i)__builtin_lasx_xvneg_d((v4i64)_1);
}

extern __inline
````
- **L1897 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1897 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1898 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvneg_b(__m256i _1) {`.
  **L1898 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvneg_b(__m256i _1) {`。
- **L1899 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvneg_b((v32i8)_1)`.
  **L1899 CN**: 以 `(__m256i)__builtin_lasx_xvneg_b((v32i8)_1)` 从当前函数返回。
- **L1900 EN**: Closes the current lexical scope or compound statement.
  **L1900 CN**: 结束当前词法作用域或复合语句块。
- **L1901 EN**: Blank line separating nearby declarations or logic blocks.
  **L1901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1902 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1902 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1903 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1903 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1904 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvneg_h(__m256i _1) {`.
  **L1904 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvneg_h(__m256i _1) {`。
- **L1905 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvneg_h((v16i16)_1)`.
  **L1905 CN**: 以 `(__m256i)__builtin_lasx_xvneg_h((v16i16)_1)` 从当前函数返回。
- **L1906 EN**: Closes the current lexical scope or compound statement.
  **L1906 CN**: 结束当前词法作用域或复合语句块。
- **L1907 EN**: Blank line separating nearby declarations or logic blocks.
  **L1907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1908 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1908 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1909 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1909 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1910 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvneg_w(__m256i _1) {`.
  **L1910 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvneg_w(__m256i _1) {`。
- **L1911 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvneg_w((v8i32)_1)`.
  **L1911 CN**: 以 `(__m256i)__builtin_lasx_xvneg_w((v8i32)_1)` 从当前函数返回。
- **L1912 EN**: Closes the current lexical scope or compound statement.
  **L1912 CN**: 结束当前词法作用域或复合语句块。
- **L1913 EN**: Blank line separating nearby declarations or logic blocks.
  **L1913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1914 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1914 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1915 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1915 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1916 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvneg_d(__m256i _1) {`.
  **L1916 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvneg_d(__m256i _1) {`。
- **L1917 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvneg_d((v4i64)_1)`.
  **L1917 CN**: 以 `(__m256i)__builtin_lasx_xvneg_d((v4i64)_1)` 从当前函数返回。
- **L1918 EN**: Closes the current lexical scope or compound statement.
  **L1918 CN**: 结束当前词法作用域或复合语句块。
- **L1919 EN**: Blank line separating nearby declarations or logic blocks.
  **L1919 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1920 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1920 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 1921-1944

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmuh_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmuh_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmuh_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmuh_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmuh_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmuh_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmuh_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmuh_d((v4i64)_1, (v4i64)_2);
}

extern __inline
````
- **L1921 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1921 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1922 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmuh_b(__m256i _1, __m256i _2) {`.
  **L1922 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmuh_b(__m256i _1, __m256i _2) {`。
- **L1923 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmuh_b((v32i8)_1, (v32i8)_2)`.
  **L1923 CN**: 以 `(__m256i)__builtin_lasx_xvmuh_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L1924 EN**: Closes the current lexical scope or compound statement.
  **L1924 CN**: 结束当前词法作用域或复合语句块。
- **L1925 EN**: Blank line separating nearby declarations or logic blocks.
  **L1925 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1926 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1926 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1927 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1927 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1928 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmuh_h(__m256i _1, __m256i _2) {`.
  **L1928 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmuh_h(__m256i _1, __m256i _2) {`。
- **L1929 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmuh_h((v16i16)_1, (v16i16)_2)`.
  **L1929 CN**: 以 `(__m256i)__builtin_lasx_xvmuh_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1930 EN**: Closes the current lexical scope or compound statement.
  **L1930 CN**: 结束当前词法作用域或复合语句块。
- **L1931 EN**: Blank line separating nearby declarations or logic blocks.
  **L1931 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1932 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1932 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1933 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1933 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1934 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmuh_w(__m256i _1, __m256i _2) {`.
  **L1934 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmuh_w(__m256i _1, __m256i _2) {`。
- **L1935 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmuh_w((v8i32)_1, (v8i32)_2)`.
  **L1935 CN**: 以 `(__m256i)__builtin_lasx_xvmuh_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1936 EN**: Closes the current lexical scope or compound statement.
  **L1936 CN**: 结束当前词法作用域或复合语句块。
- **L1937 EN**: Blank line separating nearby declarations or logic blocks.
  **L1937 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1938 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1938 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1939 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1939 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1940 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmuh_d(__m256i _1, __m256i _2) {`.
  **L1940 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmuh_d(__m256i _1, __m256i _2) {`。
- **L1941 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmuh_d((v4i64)_1, (v4i64)_2)`.
  **L1941 CN**: 以 `(__m256i)__builtin_lasx_xvmuh_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L1942 EN**: Closes the current lexical scope or compound statement.
  **L1942 CN**: 结束当前词法作用域或复合语句块。
- **L1943 EN**: Blank line separating nearby declarations or logic blocks.
  **L1943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1944 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1944 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 1945-1968

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmuh_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmuh_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmuh_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmuh_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmuh_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmuh_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmuh_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmuh_du((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvsllwil_h_b(/*__m256i*/ _1, /*ui3*/ _2)                        \
````
- **L1945 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1945 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1946 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmuh_bu(__m256i _1, __m256i _2) {`.
  **L1946 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmuh_bu(__m256i _1, __m256i _2) {`。
- **L1947 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmuh_bu((v32u8)_1, (v32u8)_2)`.
  **L1947 CN**: 以 `(__m256i)__builtin_lasx_xvmuh_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L1948 EN**: Closes the current lexical scope or compound statement.
  **L1948 CN**: 结束当前词法作用域或复合语句块。
- **L1949 EN**: Blank line separating nearby declarations or logic blocks.
  **L1949 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1950 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1950 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1951 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1951 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmuh_hu(__m256i _1, __m256i _2) {`.
  **L1952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmuh_hu(__m256i _1, __m256i _2) {`。
- **L1953 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmuh_hu((v16u16)_1, (v16u16)_2)`.
  **L1953 CN**: 以 `(__m256i)__builtin_lasx_xvmuh_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L1954 EN**: Closes the current lexical scope or compound statement.
  **L1954 CN**: 结束当前词法作用域或复合语句块。
- **L1955 EN**: Blank line separating nearby declarations or logic blocks.
  **L1955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1956 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1956 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1957 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1957 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1958 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmuh_wu(__m256i _1, __m256i _2) {`.
  **L1958 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmuh_wu(__m256i _1, __m256i _2) {`。
- **L1959 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmuh_wu((v8u32)_1, (v8u32)_2)`.
  **L1959 CN**: 以 `(__m256i)__builtin_lasx_xvmuh_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L1960 EN**: Closes the current lexical scope or compound statement.
  **L1960 CN**: 结束当前词法作用域或复合语句块。
- **L1961 EN**: Blank line separating nearby declarations or logic blocks.
  **L1961 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1962 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1962 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1963 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1963 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1964 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmuh_du(__m256i _1, __m256i _2) {`.
  **L1964 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmuh_du(__m256i _1, __m256i _2) {`。
- **L1965 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmuh_du((v4u64)_1, (v4u64)_2)`.
  **L1965 CN**: 以 `(__m256i)__builtin_lasx_xvmuh_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L1966 EN**: Closes the current lexical scope or compound statement.
  **L1966 CN**: 结束当前词法作用域或复合语句块。
- **L1967 EN**: Blank line separating nearby declarations or logic blocks.
  **L1967 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1968 EN**: Defines macro `__lasx_xvsllwil_h_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1968 CN**: 定义宏 `__lasx_xvsllwil_h_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 1969-1992

````c
  ((__m256i)__builtin_lasx_xvsllwil_h_b((v32i8)(_1), (_2)))

#define __lasx_xvsllwil_w_h(/*__m256i*/ _1, /*ui4*/ _2)                        \
  ((__m256i)__builtin_lasx_xvsllwil_w_h((v16i16)(_1), (_2)))

#define __lasx_xvsllwil_d_w(/*__m256i*/ _1, /*ui5*/ _2)                        \
  ((__m256i)__builtin_lasx_xvsllwil_d_w((v8i32)(_1), (_2)))

#define __lasx_xvsllwil_hu_bu(/*__m256i*/ _1, /*ui3*/ _2)                      \
  ((__m256i)__builtin_lasx_xvsllwil_hu_bu((v32u8)(_1), (_2)))

#define __lasx_xvsllwil_wu_hu(/*__m256i*/ _1, /*ui4*/ _2)                      \
  ((__m256i)__builtin_lasx_xvsllwil_wu_hu((v16u16)(_1), (_2)))

#define __lasx_xvsllwil_du_wu(/*__m256i*/ _1, /*ui5*/ _2)                      \
  ((__m256i)__builtin_lasx_xvsllwil_du_wu((v8u32)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsran_b_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsran_b_h((v16i16)_1, (v16i16)_2);
}

extern __inline
````
- **L1969 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsllwil_h_b`.
  **L1969 CN**: 继续与可调用符号 `__builtin_lasx_xvsllwil_h_b` 相关的逻辑。
- **L1970 EN**: Blank line separating nearby declarations or logic blocks.
  **L1970 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1971 EN**: Defines macro `__lasx_xvsllwil_w_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1971 CN**: 定义宏 `__lasx_xvsllwil_w_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L1972 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsllwil_w_h`.
  **L1972 CN**: 继续与可调用符号 `__builtin_lasx_xvsllwil_w_h` 相关的逻辑。
- **L1973 EN**: Blank line separating nearby declarations or logic blocks.
  **L1973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1974 EN**: Defines macro `__lasx_xvsllwil_d_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1974 CN**: 定义宏 `__lasx_xvsllwil_d_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L1975 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsllwil_d_w`.
  **L1975 CN**: 继续与可调用符号 `__builtin_lasx_xvsllwil_d_w` 相关的逻辑。
- **L1976 EN**: Blank line separating nearby declarations or logic blocks.
  **L1976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1977 EN**: Defines macro `__lasx_xvsllwil_hu_bu(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1977 CN**: 定义宏 `__lasx_xvsllwil_hu_bu(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L1978 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsllwil_hu_bu`.
  **L1978 CN**: 继续与可调用符号 `__builtin_lasx_xvsllwil_hu_bu` 相关的逻辑。
- **L1979 EN**: Blank line separating nearby declarations or logic blocks.
  **L1979 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1980 EN**: Defines macro `__lasx_xvsllwil_wu_hu(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1980 CN**: 定义宏 `__lasx_xvsllwil_wu_hu(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L1981 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsllwil_wu_hu`.
  **L1981 CN**: 继续与可调用符号 `__builtin_lasx_xvsllwil_wu_hu` 相关的逻辑。
- **L1982 EN**: Blank line separating nearby declarations or logic blocks.
  **L1982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1983 EN**: Defines macro `__lasx_xvsllwil_du_wu(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1983 CN**: 定义宏 `__lasx_xvsllwil_du_wu(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L1984 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsllwil_du_wu`.
  **L1984 CN**: 继续与可调用符号 `__builtin_lasx_xvsllwil_du_wu` 相关的逻辑。
- **L1985 EN**: Blank line separating nearby declarations or logic blocks.
  **L1985 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1986 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1986 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1987 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1987 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1988 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsran_b_h(__m256i _1, __m256i _2) {`.
  **L1988 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsran_b_h(__m256i _1, __m256i _2) {`。
- **L1989 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsran_b_h((v16i16)_1, (v16i16)_2)`.
  **L1989 CN**: 以 `(__m256i)__builtin_lasx_xvsran_b_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L1990 EN**: Closes the current lexical scope or compound statement.
  **L1990 CN**: 结束当前词法作用域或复合语句块。
- **L1991 EN**: Blank line separating nearby declarations or logic blocks.
  **L1991 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1992 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1992 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 1993-2016

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsran_h_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsran_h_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsran_w_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsran_w_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssran_b_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssran_b_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssran_h_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssran_h_w((v8i32)_1, (v8i32)_2);
}

extern __inline
````
- **L1993 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1993 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L1994 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsran_h_w(__m256i _1, __m256i _2) {`.
  **L1994 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsran_h_w(__m256i _1, __m256i _2) {`。
- **L1995 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsran_h_w((v8i32)_1, (v8i32)_2)`.
  **L1995 CN**: 以 `(__m256i)__builtin_lasx_xvsran_h_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L1996 EN**: Closes the current lexical scope or compound statement.
  **L1996 CN**: 结束当前词法作用域或复合语句块。
- **L1997 EN**: Blank line separating nearby declarations or logic blocks.
  **L1997 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1998 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1998 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1999 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L1999 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2000 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsran_w_d(__m256i _1, __m256i _2) {`.
  **L2000 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsran_w_d(__m256i _1, __m256i _2) {`。
- **L2001 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsran_w_d((v4i64)_1, (v4i64)_2)`.
  **L2001 CN**: 以 `(__m256i)__builtin_lasx_xvsran_w_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2002 EN**: Closes the current lexical scope or compound statement.
  **L2002 CN**: 结束当前词法作用域或复合语句块。
- **L2003 EN**: Blank line separating nearby declarations or logic blocks.
  **L2003 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2004 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2004 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2005 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2005 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2006 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssran_b_h(__m256i _1, __m256i _2) {`.
  **L2006 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssran_b_h(__m256i _1, __m256i _2) {`。
- **L2007 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssran_b_h((v16i16)_1, (v16i16)_2)`.
  **L2007 CN**: 以 `(__m256i)__builtin_lasx_xvssran_b_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2008 EN**: Closes the current lexical scope or compound statement.
  **L2008 CN**: 结束当前词法作用域或复合语句块。
- **L2009 EN**: Blank line separating nearby declarations or logic blocks.
  **L2009 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2010 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2010 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2011 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2011 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2012 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssran_h_w(__m256i _1, __m256i _2) {`.
  **L2012 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssran_h_w(__m256i _1, __m256i _2) {`。
- **L2013 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssran_h_w((v8i32)_1, (v8i32)_2)`.
  **L2013 CN**: 以 `(__m256i)__builtin_lasx_xvssran_h_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2014 EN**: Closes the current lexical scope or compound statement.
  **L2014 CN**: 结束当前词法作用域或复合语句块。
- **L2015 EN**: Blank line separating nearby declarations or logic blocks.
  **L2015 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2016 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2016 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2017-2040

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssran_w_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssran_w_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssran_bu_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssran_bu_h((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssran_hu_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssran_hu_w((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssran_wu_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssran_wu_d((v4u64)_1, (v4u64)_2);
}

extern __inline
````
- **L2017 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2017 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2018 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssran_w_d(__m256i _1, __m256i _2) {`.
  **L2018 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssran_w_d(__m256i _1, __m256i _2) {`。
- **L2019 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssran_w_d((v4i64)_1, (v4i64)_2)`.
  **L2019 CN**: 以 `(__m256i)__builtin_lasx_xvssran_w_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2020 EN**: Closes the current lexical scope or compound statement.
  **L2020 CN**: 结束当前词法作用域或复合语句块。
- **L2021 EN**: Blank line separating nearby declarations or logic blocks.
  **L2021 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2022 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2022 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2023 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2023 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2024 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssran_bu_h(__m256i _1, __m256i _2) {`.
  **L2024 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssran_bu_h(__m256i _1, __m256i _2) {`。
- **L2025 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssran_bu_h((v16u16)_1, (v16u16)_2)`.
  **L2025 CN**: 以 `(__m256i)__builtin_lasx_xvssran_bu_h((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2026 EN**: Closes the current lexical scope or compound statement.
  **L2026 CN**: 结束当前词法作用域或复合语句块。
- **L2027 EN**: Blank line separating nearby declarations or logic blocks.
  **L2027 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2028 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2028 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2029 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2029 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2030 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssran_hu_w(__m256i _1, __m256i _2) {`.
  **L2030 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssran_hu_w(__m256i _1, __m256i _2) {`。
- **L2031 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssran_hu_w((v8u32)_1, (v8u32)_2)`.
  **L2031 CN**: 以 `(__m256i)__builtin_lasx_xvssran_hu_w((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2032 EN**: Closes the current lexical scope or compound statement.
  **L2032 CN**: 结束当前词法作用域或复合语句块。
- **L2033 EN**: Blank line separating nearby declarations or logic blocks.
  **L2033 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2034 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2034 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2035 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2035 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2036 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssran_wu_d(__m256i _1, __m256i _2) {`.
  **L2036 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssran_wu_d(__m256i _1, __m256i _2) {`。
- **L2037 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssran_wu_d((v4u64)_1, (v4u64)_2)`.
  **L2037 CN**: 以 `(__m256i)__builtin_lasx_xvssran_wu_d((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L2038 EN**: Closes the current lexical scope or compound statement.
  **L2038 CN**: 结束当前词法作用域或复合语句块。
- **L2039 EN**: Blank line separating nearby declarations or logic blocks.
  **L2039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2040 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2040 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2041-2064

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrarn_b_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrarn_b_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrarn_h_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrarn_h_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrarn_w_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrarn_w_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrarn_b_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrarn_b_h((v16i16)_1, (v16i16)_2);
}

extern __inline
````
- **L2041 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2041 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2042 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrarn_b_h(__m256i _1, __m256i _2) {`.
  **L2042 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrarn_b_h(__m256i _1, __m256i _2) {`。
- **L2043 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrarn_b_h((v16i16)_1, (v16i16)_2)`.
  **L2043 CN**: 以 `(__m256i)__builtin_lasx_xvsrarn_b_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2044 EN**: Closes the current lexical scope or compound statement.
  **L2044 CN**: 结束当前词法作用域或复合语句块。
- **L2045 EN**: Blank line separating nearby declarations or logic blocks.
  **L2045 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2046 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2046 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2047 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2047 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2048 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrarn_h_w(__m256i _1, __m256i _2) {`.
  **L2048 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrarn_h_w(__m256i _1, __m256i _2) {`。
- **L2049 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrarn_h_w((v8i32)_1, (v8i32)_2)`.
  **L2049 CN**: 以 `(__m256i)__builtin_lasx_xvsrarn_h_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2050 EN**: Closes the current lexical scope or compound statement.
  **L2050 CN**: 结束当前词法作用域或复合语句块。
- **L2051 EN**: Blank line separating nearby declarations or logic blocks.
  **L2051 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2052 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2052 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2053 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2053 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2054 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrarn_w_d(__m256i _1, __m256i _2) {`.
  **L2054 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrarn_w_d(__m256i _1, __m256i _2) {`。
- **L2055 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrarn_w_d((v4i64)_1, (v4i64)_2)`.
  **L2055 CN**: 以 `(__m256i)__builtin_lasx_xvsrarn_w_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2056 EN**: Closes the current lexical scope or compound statement.
  **L2056 CN**: 结束当前词法作用域或复合语句块。
- **L2057 EN**: Blank line separating nearby declarations or logic blocks.
  **L2057 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2058 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2058 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2059 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2059 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2060 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrarn_b_h(__m256i _1, __m256i _2) {`.
  **L2060 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrarn_b_h(__m256i _1, __m256i _2) {`。
- **L2061 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrarn_b_h((v16i16)_1, (v16i16)_2)`.
  **L2061 CN**: 以 `(__m256i)__builtin_lasx_xvssrarn_b_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2062 EN**: Closes the current lexical scope or compound statement.
  **L2062 CN**: 结束当前词法作用域或复合语句块。
- **L2063 EN**: Blank line separating nearby declarations or logic blocks.
  **L2063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2064 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2064 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2065-2088

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrarn_h_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrarn_h_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrarn_w_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrarn_w_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrarn_bu_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrarn_bu_h((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrarn_hu_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrarn_hu_w((v8u32)_1, (v8u32)_2);
}

extern __inline
````
- **L2065 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2065 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2066 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrarn_h_w(__m256i _1, __m256i _2) {`.
  **L2066 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrarn_h_w(__m256i _1, __m256i _2) {`。
- **L2067 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrarn_h_w((v8i32)_1, (v8i32)_2)`.
  **L2067 CN**: 以 `(__m256i)__builtin_lasx_xvssrarn_h_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2068 EN**: Closes the current lexical scope or compound statement.
  **L2068 CN**: 结束当前词法作用域或复合语句块。
- **L2069 EN**: Blank line separating nearby declarations or logic blocks.
  **L2069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2070 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2070 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2071 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2071 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2072 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrarn_w_d(__m256i _1, __m256i _2) {`.
  **L2072 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrarn_w_d(__m256i _1, __m256i _2) {`。
- **L2073 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrarn_w_d((v4i64)_1, (v4i64)_2)`.
  **L2073 CN**: 以 `(__m256i)__builtin_lasx_xvssrarn_w_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2074 EN**: Closes the current lexical scope or compound statement.
  **L2074 CN**: 结束当前词法作用域或复合语句块。
- **L2075 EN**: Blank line separating nearby declarations or logic blocks.
  **L2075 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2076 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2076 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2077 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2077 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2078 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrarn_bu_h(__m256i _1, __m256i _2) {`.
  **L2078 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrarn_bu_h(__m256i _1, __m256i _2) {`。
- **L2079 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrarn_bu_h((v16u16)_1, (v16u16)_2)`.
  **L2079 CN**: 以 `(__m256i)__builtin_lasx_xvssrarn_bu_h((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2080 EN**: Closes the current lexical scope or compound statement.
  **L2080 CN**: 结束当前词法作用域或复合语句块。
- **L2081 EN**: Blank line separating nearby declarations or logic blocks.
  **L2081 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2082 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2082 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2083 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2083 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2084 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrarn_hu_w(__m256i _1, __m256i _2) {`.
  **L2084 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrarn_hu_w(__m256i _1, __m256i _2) {`。
- **L2085 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrarn_hu_w((v8u32)_1, (v8u32)_2)`.
  **L2085 CN**: 以 `(__m256i)__builtin_lasx_xvssrarn_hu_w((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2086 EN**: Closes the current lexical scope or compound statement.
  **L2086 CN**: 结束当前词法作用域或复合语句块。
- **L2087 EN**: Blank line separating nearby declarations or logic blocks.
  **L2087 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2088 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2088 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2089-2112

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrarn_wu_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrarn_wu_d((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrln_b_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrln_b_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrln_h_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrln_h_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrln_w_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrln_w_d((v4i64)_1, (v4i64)_2);
}

extern __inline
````
- **L2089 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2089 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2090 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrarn_wu_d(__m256i _1, __m256i _2) {`.
  **L2090 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrarn_wu_d(__m256i _1, __m256i _2) {`。
- **L2091 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrarn_wu_d((v4u64)_1, (v4u64)_2)`.
  **L2091 CN**: 以 `(__m256i)__builtin_lasx_xvssrarn_wu_d((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L2092 EN**: Closes the current lexical scope or compound statement.
  **L2092 CN**: 结束当前词法作用域或复合语句块。
- **L2093 EN**: Blank line separating nearby declarations or logic blocks.
  **L2093 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2094 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2094 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2095 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2095 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2096 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrln_b_h(__m256i _1, __m256i _2) {`.
  **L2096 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrln_b_h(__m256i _1, __m256i _2) {`。
- **L2097 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrln_b_h((v16i16)_1, (v16i16)_2)`.
  **L2097 CN**: 以 `(__m256i)__builtin_lasx_xvsrln_b_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2098 EN**: Closes the current lexical scope or compound statement.
  **L2098 CN**: 结束当前词法作用域或复合语句块。
- **L2099 EN**: Blank line separating nearby declarations or logic blocks.
  **L2099 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2100 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2100 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2101 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2101 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2102 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrln_h_w(__m256i _1, __m256i _2) {`.
  **L2102 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrln_h_w(__m256i _1, __m256i _2) {`。
- **L2103 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrln_h_w((v8i32)_1, (v8i32)_2)`.
  **L2103 CN**: 以 `(__m256i)__builtin_lasx_xvsrln_h_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2104 EN**: Closes the current lexical scope or compound statement.
  **L2104 CN**: 结束当前词法作用域或复合语句块。
- **L2105 EN**: Blank line separating nearby declarations or logic blocks.
  **L2105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2106 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2106 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2107 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2107 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2108 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrln_w_d(__m256i _1, __m256i _2) {`.
  **L2108 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrln_w_d(__m256i _1, __m256i _2) {`。
- **L2109 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrln_w_d((v4i64)_1, (v4i64)_2)`.
  **L2109 CN**: 以 `(__m256i)__builtin_lasx_xvsrln_w_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2110 EN**: Closes the current lexical scope or compound statement.
  **L2110 CN**: 结束当前词法作用域或复合语句块。
- **L2111 EN**: Blank line separating nearby declarations or logic blocks.
  **L2111 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2112 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2112 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2113-2136

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrln_bu_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrln_bu_h((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrln_hu_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrln_hu_w((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrln_wu_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrln_wu_d((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrlrn_b_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrlrn_b_h((v16i16)_1, (v16i16)_2);
}

extern __inline
````
- **L2113 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2113 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2114 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrln_bu_h(__m256i _1, __m256i _2) {`.
  **L2114 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrln_bu_h(__m256i _1, __m256i _2) {`。
- **L2115 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrln_bu_h((v16u16)_1, (v16u16)_2)`.
  **L2115 CN**: 以 `(__m256i)__builtin_lasx_xvssrln_bu_h((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2116 EN**: Closes the current lexical scope or compound statement.
  **L2116 CN**: 结束当前词法作用域或复合语句块。
- **L2117 EN**: Blank line separating nearby declarations or logic blocks.
  **L2117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2118 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2118 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2119 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2119 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2120 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrln_hu_w(__m256i _1, __m256i _2) {`.
  **L2120 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrln_hu_w(__m256i _1, __m256i _2) {`。
- **L2121 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrln_hu_w((v8u32)_1, (v8u32)_2)`.
  **L2121 CN**: 以 `(__m256i)__builtin_lasx_xvssrln_hu_w((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2122 EN**: Closes the current lexical scope or compound statement.
  **L2122 CN**: 结束当前词法作用域或复合语句块。
- **L2123 EN**: Blank line separating nearby declarations or logic blocks.
  **L2123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2124 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2124 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2125 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2125 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2126 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrln_wu_d(__m256i _1, __m256i _2) {`.
  **L2126 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrln_wu_d(__m256i _1, __m256i _2) {`。
- **L2127 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrln_wu_d((v4u64)_1, (v4u64)_2)`.
  **L2127 CN**: 以 `(__m256i)__builtin_lasx_xvssrln_wu_d((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L2128 EN**: Closes the current lexical scope or compound statement.
  **L2128 CN**: 结束当前词法作用域或复合语句块。
- **L2129 EN**: Blank line separating nearby declarations or logic blocks.
  **L2129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2130 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2130 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2131 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2131 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2132 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrlrn_b_h(__m256i _1, __m256i _2) {`.
  **L2132 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrlrn_b_h(__m256i _1, __m256i _2) {`。
- **L2133 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrlrn_b_h((v16i16)_1, (v16i16)_2)`.
  **L2133 CN**: 以 `(__m256i)__builtin_lasx_xvsrlrn_b_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2134 EN**: Closes the current lexical scope or compound statement.
  **L2134 CN**: 结束当前词法作用域或复合语句块。
- **L2135 EN**: Blank line separating nearby declarations or logic blocks.
  **L2135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2136 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2136 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2137-2160

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrlrn_h_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrlrn_h_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsrlrn_w_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsrlrn_w_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrlrn_bu_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrlrn_bu_h((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrlrn_hu_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrlrn_hu_w((v8u32)_1, (v8u32)_2);
}

extern __inline
````
- **L2137 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2137 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2138 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrlrn_h_w(__m256i _1, __m256i _2) {`.
  **L2138 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrlrn_h_w(__m256i _1, __m256i _2) {`。
- **L2139 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrlrn_h_w((v8i32)_1, (v8i32)_2)`.
  **L2139 CN**: 以 `(__m256i)__builtin_lasx_xvsrlrn_h_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2140 EN**: Closes the current lexical scope or compound statement.
  **L2140 CN**: 结束当前词法作用域或复合语句块。
- **L2141 EN**: Blank line separating nearby declarations or logic blocks.
  **L2141 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2142 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2142 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2143 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2143 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2144 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsrlrn_w_d(__m256i _1, __m256i _2) {`.
  **L2144 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsrlrn_w_d(__m256i _1, __m256i _2) {`。
- **L2145 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsrlrn_w_d((v4i64)_1, (v4i64)_2)`.
  **L2145 CN**: 以 `(__m256i)__builtin_lasx_xvsrlrn_w_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2146 EN**: Closes the current lexical scope or compound statement.
  **L2146 CN**: 结束当前词法作用域或复合语句块。
- **L2147 EN**: Blank line separating nearby declarations or logic blocks.
  **L2147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2148 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2148 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2149 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2149 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2150 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrlrn_bu_h(__m256i _1, __m256i _2) {`.
  **L2150 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrlrn_bu_h(__m256i _1, __m256i _2) {`。
- **L2151 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrlrn_bu_h((v16u16)_1, (v16u16)_2)`.
  **L2151 CN**: 以 `(__m256i)__builtin_lasx_xvssrlrn_bu_h((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2152 EN**: Closes the current lexical scope or compound statement.
  **L2152 CN**: 结束当前词法作用域或复合语句块。
- **L2153 EN**: Blank line separating nearby declarations or logic blocks.
  **L2153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2154 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2154 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2155 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2155 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrlrn_hu_w(__m256i _1, __m256i _2) {`.
  **L2156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrlrn_hu_w(__m256i _1, __m256i _2) {`。
- **L2157 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrlrn_hu_w((v8u32)_1, (v8u32)_2)`.
  **L2157 CN**: 以 `(__m256i)__builtin_lasx_xvssrlrn_hu_w((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2158 EN**: Closes the current lexical scope or compound statement.
  **L2158 CN**: 结束当前词法作用域或复合语句块。
- **L2159 EN**: Blank line separating nearby declarations or logic blocks.
  **L2159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2160 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2160 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2161-2184

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrlrn_wu_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrlrn_wu_d((v4u64)_1, (v4u64)_2);
}

#define __lasx_xvfrstpi_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)          \
  ((__m256i)__builtin_lasx_xvfrstpi_b((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvfrstpi_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)          \
  ((__m256i)__builtin_lasx_xvfrstpi_h((v16i16)(_1), (v16i16)(_2), (_3)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfrstp_b(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvfrstp_b((v32i8)_1, (v32i8)_2, (v32i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfrstp_h(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvfrstp_h((v16i16)_1, (v16i16)_2, (v16i16)_3);
}

#define __lasx_xvshuf4i_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)          \
````
- **L2161 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2161 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2162 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrlrn_wu_d(__m256i _1, __m256i _2) {`.
  **L2162 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrlrn_wu_d(__m256i _1, __m256i _2) {`。
- **L2163 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrlrn_wu_d((v4u64)_1, (v4u64)_2)`.
  **L2163 CN**: 以 `(__m256i)__builtin_lasx_xvssrlrn_wu_d((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L2164 EN**: Closes the current lexical scope or compound statement.
  **L2164 CN**: 结束当前词法作用域或复合语句块。
- **L2165 EN**: Blank line separating nearby declarations or logic blocks.
  **L2165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2166 EN**: Defines macro `__lasx_xvfrstpi_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2166 CN**: 定义宏 `__lasx_xvfrstpi_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L2167 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvfrstpi_b`.
  **L2167 CN**: 继续与可调用符号 `__builtin_lasx_xvfrstpi_b` 相关的逻辑。
- **L2168 EN**: Blank line separating nearby declarations or logic blocks.
  **L2168 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2169 EN**: Defines macro `__lasx_xvfrstpi_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2169 CN**: 定义宏 `__lasx_xvfrstpi_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L2170 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvfrstpi_h`.
  **L2170 CN**: 继续与可调用符号 `__builtin_lasx_xvfrstpi_h` 相关的逻辑。
- **L2171 EN**: Blank line separating nearby declarations or logic blocks.
  **L2171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2172 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2172 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2173 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2173 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2174 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrstp_b(__m256i _1, __m256i _2, __m256i _3) {`.
  **L2174 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrstp_b(__m256i _1, __m256i _2, __m256i _3) {`。
- **L2175 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfrstp_b((v32i8)_1, (v32i8)_2, (v32i8)_3)`.
  **L2175 CN**: 以 `(__m256i)__builtin_lasx_xvfrstp_b((v32i8)_1, (v32i8)_2, (v32i8)_3)` 从当前函数返回。
- **L2176 EN**: Closes the current lexical scope or compound statement.
  **L2176 CN**: 结束当前词法作用域或复合语句块。
- **L2177 EN**: Blank line separating nearby declarations or logic blocks.
  **L2177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2178 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2178 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2179 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2179 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2180 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrstp_h(__m256i _1, __m256i _2, __m256i _3) {`.
  **L2180 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrstp_h(__m256i _1, __m256i _2, __m256i _3) {`。
- **L2181 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfrstp_h((v16i16)_1, (v16i16)_2, (v16i16)_3)`.
  **L2181 CN**: 以 `(__m256i)__builtin_lasx_xvfrstp_h((v16i16)_1, (v16i16)_2, (v16i16)_3)` 从当前函数返回。
- **L2182 EN**: Closes the current lexical scope or compound statement.
  **L2182 CN**: 结束当前词法作用域或复合语句块。
- **L2183 EN**: Blank line separating nearby declarations or logic blocks.
  **L2183 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2184 EN**: Defines macro `__lasx_xvshuf4i_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2184 CN**: 定义宏 `__lasx_xvshuf4i_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。

### Lines 2185-2208

````c
  ((__m256i)__builtin_lasx_xvshuf4i_d((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvbsrl_v(/*__m256i*/ _1, /*ui5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvbsrl_v((v32i8)(_1), (_2)))

#define __lasx_xvbsll_v(/*__m256i*/ _1, /*ui5*/ _2)                            \
  ((__m256i)__builtin_lasx_xvbsll_v((v32i8)(_1), (_2)))

#define __lasx_xvextrins_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)         \
  ((__m256i)__builtin_lasx_xvextrins_b((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvextrins_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)         \
  ((__m256i)__builtin_lasx_xvextrins_h((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvextrins_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)         \
  ((__m256i)__builtin_lasx_xvextrins_w((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvextrins_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)         \
  ((__m256i)__builtin_lasx_xvextrins_d((v4i64)(_1), (v4i64)(_2), (_3)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmskltz_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvmskltz_b((v32i8)_1);
````
- **L2185 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvshuf4i_d`.
  **L2185 CN**: 继续与可调用符号 `__builtin_lasx_xvshuf4i_d` 相关的逻辑。
- **L2186 EN**: Blank line separating nearby declarations or logic blocks.
  **L2186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2187 EN**: Defines macro `__lasx_xvbsrl_v(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2187 CN**: 定义宏 `__lasx_xvbsrl_v(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L2188 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbsrl_v`.
  **L2188 CN**: 继续与可调用符号 `__builtin_lasx_xvbsrl_v` 相关的逻辑。
- **L2189 EN**: Blank line separating nearby declarations or logic blocks.
  **L2189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2190 EN**: Defines macro `__lasx_xvbsll_v(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2190 CN**: 定义宏 `__lasx_xvbsll_v(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L2191 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvbsll_v`.
  **L2191 CN**: 继续与可调用符号 `__builtin_lasx_xvbsll_v` 相关的逻辑。
- **L2192 EN**: Blank line separating nearby declarations or logic blocks.
  **L2192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2193 EN**: Defines macro `__lasx_xvextrins_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2193 CN**: 定义宏 `__lasx_xvextrins_b(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2194 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvextrins_b`.
  **L2194 CN**: 继续与可调用符号 `__builtin_lasx_xvextrins_b` 相关的逻辑。
- **L2195 EN**: Blank line separating nearby declarations or logic blocks.
  **L2195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2196 EN**: Defines macro `__lasx_xvextrins_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2196 CN**: 定义宏 `__lasx_xvextrins_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2197 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvextrins_h`.
  **L2197 CN**: 继续与可调用符号 `__builtin_lasx_xvextrins_h` 相关的逻辑。
- **L2198 EN**: Blank line separating nearby declarations or logic blocks.
  **L2198 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2199 EN**: Defines macro `__lasx_xvextrins_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2199 CN**: 定义宏 `__lasx_xvextrins_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2200 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvextrins_w`.
  **L2200 CN**: 继续与可调用符号 `__builtin_lasx_xvextrins_w` 相关的逻辑。
- **L2201 EN**: Blank line separating nearby declarations or logic blocks.
  **L2201 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2202 EN**: Defines macro `__lasx_xvextrins_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2202 CN**: 定义宏 `__lasx_xvextrins_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2203 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvextrins_d`.
  **L2203 CN**: 继续与可调用符号 `__builtin_lasx_xvextrins_d` 相关的逻辑。
- **L2204 EN**: Blank line separating nearby declarations or logic blocks.
  **L2204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2205 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2205 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2206 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2206 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2207 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmskltz_b(__m256i _1) {`.
  **L2207 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmskltz_b(__m256i _1) {`。
- **L2208 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmskltz_b((v32i8)_1)`.
  **L2208 CN**: 以 `(__m256i)__builtin_lasx_xvmskltz_b((v32i8)_1)` 从当前函数返回。

### Lines 2209-2232

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmskltz_h(__m256i _1) {
  return (__m256i)__builtin_lasx_xvmskltz_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmskltz_w(__m256i _1) {
  return (__m256i)__builtin_lasx_xvmskltz_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmskltz_d(__m256i _1) {
  return (__m256i)__builtin_lasx_xvmskltz_d((v4i64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsigncov_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsigncov_b((v32i8)_1, (v32i8)_2);
````
- **L2209 EN**: Closes the current lexical scope or compound statement.
  **L2209 CN**: 结束当前词法作用域或复合语句块。
- **L2210 EN**: Blank line separating nearby declarations or logic blocks.
  **L2210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2211 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2211 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2212 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2212 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2213 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmskltz_h(__m256i _1) {`.
  **L2213 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmskltz_h(__m256i _1) {`。
- **L2214 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmskltz_h((v16i16)_1)`.
  **L2214 CN**: 以 `(__m256i)__builtin_lasx_xvmskltz_h((v16i16)_1)` 从当前函数返回。
- **L2215 EN**: Closes the current lexical scope or compound statement.
  **L2215 CN**: 结束当前词法作用域或复合语句块。
- **L2216 EN**: Blank line separating nearby declarations or logic blocks.
  **L2216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2217 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2217 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2218 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2218 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2219 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmskltz_w(__m256i _1) {`.
  **L2219 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmskltz_w(__m256i _1) {`。
- **L2220 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmskltz_w((v8i32)_1)`.
  **L2220 CN**: 以 `(__m256i)__builtin_lasx_xvmskltz_w((v8i32)_1)` 从当前函数返回。
- **L2221 EN**: Closes the current lexical scope or compound statement.
  **L2221 CN**: 结束当前词法作用域或复合语句块。
- **L2222 EN**: Blank line separating nearby declarations or logic blocks.
  **L2222 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2223 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2223 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2224 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2224 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2225 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmskltz_d(__m256i _1) {`.
  **L2225 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmskltz_d(__m256i _1) {`。
- **L2226 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmskltz_d((v4i64)_1)`.
  **L2226 CN**: 以 `(__m256i)__builtin_lasx_xvmskltz_d((v4i64)_1)` 从当前函数返回。
- **L2227 EN**: Closes the current lexical scope or compound statement.
  **L2227 CN**: 结束当前词法作用域或复合语句块。
- **L2228 EN**: Blank line separating nearby declarations or logic blocks.
  **L2228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2229 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2229 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2230 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2230 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2231 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsigncov_b(__m256i _1, __m256i _2) {`.
  **L2231 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsigncov_b(__m256i _1, __m256i _2) {`。
- **L2232 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsigncov_b((v32i8)_1, (v32i8)_2)`.
  **L2232 CN**: 以 `(__m256i)__builtin_lasx_xvsigncov_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 2233-2256

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsigncov_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsigncov_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsigncov_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsigncov_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsigncov_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsigncov_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfmadd_s(__m256 _1, __m256 _2, __m256 _3) {
  return (__m256)__builtin_lasx_xvfmadd_s((v8f32)_1, (v8f32)_2, (v8f32)_3);
````
- **L2233 EN**: Closes the current lexical scope or compound statement.
  **L2233 CN**: 结束当前词法作用域或复合语句块。
- **L2234 EN**: Blank line separating nearby declarations or logic blocks.
  **L2234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2235 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2235 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2236 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2236 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2237 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsigncov_h(__m256i _1, __m256i _2) {`.
  **L2237 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsigncov_h(__m256i _1, __m256i _2) {`。
- **L2238 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsigncov_h((v16i16)_1, (v16i16)_2)`.
  **L2238 CN**: 以 `(__m256i)__builtin_lasx_xvsigncov_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2239 EN**: Closes the current lexical scope or compound statement.
  **L2239 CN**: 结束当前词法作用域或复合语句块。
- **L2240 EN**: Blank line separating nearby declarations or logic blocks.
  **L2240 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2241 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2241 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2242 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2242 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2243 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsigncov_w(__m256i _1, __m256i _2) {`.
  **L2243 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsigncov_w(__m256i _1, __m256i _2) {`。
- **L2244 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsigncov_w((v8i32)_1, (v8i32)_2)`.
  **L2244 CN**: 以 `(__m256i)__builtin_lasx_xvsigncov_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2245 EN**: Closes the current lexical scope or compound statement.
  **L2245 CN**: 结束当前词法作用域或复合语句块。
- **L2246 EN**: Blank line separating nearby declarations or logic blocks.
  **L2246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2247 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2247 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2248 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2248 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2249 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsigncov_d(__m256i _1, __m256i _2) {`.
  **L2249 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsigncov_d(__m256i _1, __m256i _2) {`。
- **L2250 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsigncov_d((v4i64)_1, (v4i64)_2)`.
  **L2250 CN**: 以 `(__m256i)__builtin_lasx_xvsigncov_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2251 EN**: Closes the current lexical scope or compound statement.
  **L2251 CN**: 结束当前词法作用域或复合语句块。
- **L2252 EN**: Blank line separating nearby declarations or logic blocks.
  **L2252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2253 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2253 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2254 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2254 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2255 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmadd_s(__m256 _1, __m256 _2, __m256 _3) {`.
  **L2255 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmadd_s(__m256 _1, __m256 _2, __m256 _3) {`。
- **L2256 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfmadd_s((v8f32)_1, (v8f32)_2, (v8f32)_3)`.
  **L2256 CN**: 以 `(__m256)__builtin_lasx_xvfmadd_s((v8f32)_1, (v8f32)_2, (v8f32)_3)` 从当前函数返回。

### Lines 2257-2280

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfmadd_d(__m256d _1, __m256d _2, __m256d _3) {
  return (__m256d)__builtin_lasx_xvfmadd_d((v4f64)_1, (v4f64)_2, (v4f64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfmsub_s(__m256 _1, __m256 _2, __m256 _3) {
  return (__m256)__builtin_lasx_xvfmsub_s((v8f32)_1, (v8f32)_2, (v8f32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfmsub_d(__m256d _1, __m256d _2, __m256d _3) {
  return (__m256d)__builtin_lasx_xvfmsub_d((v4f64)_1, (v4f64)_2, (v4f64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfnmadd_s(__m256 _1, __m256 _2, __m256 _3) {
  return (__m256)__builtin_lasx_xvfnmadd_s((v8f32)_1, (v8f32)_2, (v8f32)_3);
````
- **L2257 EN**: Closes the current lexical scope or compound statement.
  **L2257 CN**: 结束当前词法作用域或复合语句块。
- **L2258 EN**: Blank line separating nearby declarations or logic blocks.
  **L2258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2259 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2259 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2260 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2260 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2261 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmadd_d(__m256d _1, __m256d _2, __m256d _3) {`.
  **L2261 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmadd_d(__m256d _1, __m256d _2, __m256d _3) {`。
- **L2262 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfmadd_d((v4f64)_1, (v4f64)_2, (v4f64)_3)`.
  **L2262 CN**: 以 `(__m256d)__builtin_lasx_xvfmadd_d((v4f64)_1, (v4f64)_2, (v4f64)_3)` 从当前函数返回。
- **L2263 EN**: Closes the current lexical scope or compound statement.
  **L2263 CN**: 结束当前词法作用域或复合语句块。
- **L2264 EN**: Blank line separating nearby declarations or logic blocks.
  **L2264 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2265 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2265 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2266 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2266 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2267 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmsub_s(__m256 _1, __m256 _2, __m256 _3) {`.
  **L2267 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmsub_s(__m256 _1, __m256 _2, __m256 _3) {`。
- **L2268 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfmsub_s((v8f32)_1, (v8f32)_2, (v8f32)_3)`.
  **L2268 CN**: 以 `(__m256)__builtin_lasx_xvfmsub_s((v8f32)_1, (v8f32)_2, (v8f32)_3)` 从当前函数返回。
- **L2269 EN**: Closes the current lexical scope or compound statement.
  **L2269 CN**: 结束当前词法作用域或复合语句块。
- **L2270 EN**: Blank line separating nearby declarations or logic blocks.
  **L2270 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2271 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2271 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2272 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2272 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2273 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfmsub_d(__m256d _1, __m256d _2, __m256d _3) {`.
  **L2273 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfmsub_d(__m256d _1, __m256d _2, __m256d _3) {`。
- **L2274 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfmsub_d((v4f64)_1, (v4f64)_2, (v4f64)_3)`.
  **L2274 CN**: 以 `(__m256d)__builtin_lasx_xvfmsub_d((v4f64)_1, (v4f64)_2, (v4f64)_3)` 从当前函数返回。
- **L2275 EN**: Closes the current lexical scope or compound statement.
  **L2275 CN**: 结束当前词法作用域或复合语句块。
- **L2276 EN**: Blank line separating nearby declarations or logic blocks.
  **L2276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2277 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2277 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2278 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2278 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2279 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfnmadd_s(__m256 _1, __m256 _2, __m256 _3) {`.
  **L2279 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfnmadd_s(__m256 _1, __m256 _2, __m256 _3) {`。
- **L2280 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfnmadd_s((v8f32)_1, (v8f32)_2, (v8f32)_3)`.
  **L2280 CN**: 以 `(__m256)__builtin_lasx_xvfnmadd_s((v8f32)_1, (v8f32)_2, (v8f32)_3)` 从当前函数返回。

### Lines 2281-2304

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfnmadd_d(__m256d _1, __m256d _2, __m256d _3) {
  return (__m256d)__builtin_lasx_xvfnmadd_d((v4f64)_1, (v4f64)_2, (v4f64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfnmsub_s(__m256 _1, __m256 _2, __m256 _3) {
  return (__m256)__builtin_lasx_xvfnmsub_s((v8f32)_1, (v8f32)_2, (v8f32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfnmsub_d(__m256d _1, __m256d _2, __m256d _3) {
  return (__m256d)__builtin_lasx_xvfnmsub_d((v4f64)_1, (v4f64)_2, (v4f64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrne_w_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrne_w_s((v8f32)_1);
````
- **L2281 EN**: Closes the current lexical scope or compound statement.
  **L2281 CN**: 结束当前词法作用域或复合语句块。
- **L2282 EN**: Blank line separating nearby declarations or logic blocks.
  **L2282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2283 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2283 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2284 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2284 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2285 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfnmadd_d(__m256d _1, __m256d _2, __m256d _3) {`.
  **L2285 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfnmadd_d(__m256d _1, __m256d _2, __m256d _3) {`。
- **L2286 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfnmadd_d((v4f64)_1, (v4f64)_2, (v4f64)_3)`.
  **L2286 CN**: 以 `(__m256d)__builtin_lasx_xvfnmadd_d((v4f64)_1, (v4f64)_2, (v4f64)_3)` 从当前函数返回。
- **L2287 EN**: Closes the current lexical scope or compound statement.
  **L2287 CN**: 结束当前词法作用域或复合语句块。
- **L2288 EN**: Blank line separating nearby declarations or logic blocks.
  **L2288 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2289 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2289 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2290 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2290 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2291 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfnmsub_s(__m256 _1, __m256 _2, __m256 _3) {`.
  **L2291 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfnmsub_s(__m256 _1, __m256 _2, __m256 _3) {`。
- **L2292 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfnmsub_s((v8f32)_1, (v8f32)_2, (v8f32)_3)`.
  **L2292 CN**: 以 `(__m256)__builtin_lasx_xvfnmsub_s((v8f32)_1, (v8f32)_2, (v8f32)_3)` 从当前函数返回。
- **L2293 EN**: Closes the current lexical scope or compound statement.
  **L2293 CN**: 结束当前词法作用域或复合语句块。
- **L2294 EN**: Blank line separating nearby declarations or logic blocks.
  **L2294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2295 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2295 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2296 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2296 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2297 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfnmsub_d(__m256d _1, __m256d _2, __m256d _3) {`.
  **L2297 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfnmsub_d(__m256d _1, __m256d _2, __m256d _3) {`。
- **L2298 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfnmsub_d((v4f64)_1, (v4f64)_2, (v4f64)_3)`.
  **L2298 CN**: 以 `(__m256d)__builtin_lasx_xvfnmsub_d((v4f64)_1, (v4f64)_2, (v4f64)_3)` 从当前函数返回。
- **L2299 EN**: Closes the current lexical scope or compound statement.
  **L2299 CN**: 结束当前词法作用域或复合语句块。
- **L2300 EN**: Blank line separating nearby declarations or logic blocks.
  **L2300 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2301 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2301 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2302 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2302 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2303 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrne_w_s(__m256 _1) {`.
  **L2303 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrne_w_s(__m256 _1) {`。
- **L2304 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrne_w_s((v8f32)_1)`.
  **L2304 CN**: 以 `(__m256i)__builtin_lasx_xvftintrne_w_s((v8f32)_1)` 从当前函数返回。

### Lines 2305-2328

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrne_l_d(__m256d _1) {
  return (__m256i)__builtin_lasx_xvftintrne_l_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrp_w_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrp_w_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrp_l_d(__m256d _1) {
  return (__m256i)__builtin_lasx_xvftintrp_l_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrm_w_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrm_w_s((v8f32)_1);
````
- **L2305 EN**: Closes the current lexical scope or compound statement.
  **L2305 CN**: 结束当前词法作用域或复合语句块。
- **L2306 EN**: Blank line separating nearby declarations or logic blocks.
  **L2306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2307 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2307 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2308 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2308 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2309 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrne_l_d(__m256d _1) {`.
  **L2309 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrne_l_d(__m256d _1) {`。
- **L2310 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrne_l_d((v4f64)_1)`.
  **L2310 CN**: 以 `(__m256i)__builtin_lasx_xvftintrne_l_d((v4f64)_1)` 从当前函数返回。
- **L2311 EN**: Closes the current lexical scope or compound statement.
  **L2311 CN**: 结束当前词法作用域或复合语句块。
- **L2312 EN**: Blank line separating nearby declarations or logic blocks.
  **L2312 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2313 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2313 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2314 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2314 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2315 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrp_w_s(__m256 _1) {`.
  **L2315 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrp_w_s(__m256 _1) {`。
- **L2316 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrp_w_s((v8f32)_1)`.
  **L2316 CN**: 以 `(__m256i)__builtin_lasx_xvftintrp_w_s((v8f32)_1)` 从当前函数返回。
- **L2317 EN**: Closes the current lexical scope or compound statement.
  **L2317 CN**: 结束当前词法作用域或复合语句块。
- **L2318 EN**: Blank line separating nearby declarations or logic blocks.
  **L2318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2319 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2319 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2320 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2320 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2321 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrp_l_d(__m256d _1) {`.
  **L2321 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrp_l_d(__m256d _1) {`。
- **L2322 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrp_l_d((v4f64)_1)`.
  **L2322 CN**: 以 `(__m256i)__builtin_lasx_xvftintrp_l_d((v4f64)_1)` 从当前函数返回。
- **L2323 EN**: Closes the current lexical scope or compound statement.
  **L2323 CN**: 结束当前词法作用域或复合语句块。
- **L2324 EN**: Blank line separating nearby declarations or logic blocks.
  **L2324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2325 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2325 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2326 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2326 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2327 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrm_w_s(__m256 _1) {`.
  **L2327 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrm_w_s(__m256 _1) {`。
- **L2328 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrm_w_s((v8f32)_1)`.
  **L2328 CN**: 以 `(__m256i)__builtin_lasx_xvftintrm_w_s((v8f32)_1)` 从当前函数返回。

### Lines 2329-2352

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrm_l_d(__m256d _1) {
  return (__m256i)__builtin_lasx_xvftintrm_l_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftint_w_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvftint_w_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvffint_s_l(__m256i _1, __m256i _2) {
  return (__m256)__builtin_lasx_xvffint_s_l((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrz_w_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvftintrz_w_d((v4f64)_1, (v4f64)_2);
````
- **L2329 EN**: Closes the current lexical scope or compound statement.
  **L2329 CN**: 结束当前词法作用域或复合语句块。
- **L2330 EN**: Blank line separating nearby declarations or logic blocks.
  **L2330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2331 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2331 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2332 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2332 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2333 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrm_l_d(__m256d _1) {`.
  **L2333 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrm_l_d(__m256d _1) {`。
- **L2334 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrm_l_d((v4f64)_1)`.
  **L2334 CN**: 以 `(__m256i)__builtin_lasx_xvftintrm_l_d((v4f64)_1)` 从当前函数返回。
- **L2335 EN**: Closes the current lexical scope or compound statement.
  **L2335 CN**: 结束当前词法作用域或复合语句块。
- **L2336 EN**: Blank line separating nearby declarations or logic blocks.
  **L2336 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2337 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2337 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2338 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2338 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2339 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftint_w_d(__m256d _1, __m256d _2) {`.
  **L2339 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftint_w_d(__m256d _1, __m256d _2) {`。
- **L2340 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftint_w_d((v4f64)_1, (v4f64)_2)`.
  **L2340 CN**: 以 `(__m256i)__builtin_lasx_xvftint_w_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L2341 EN**: Closes the current lexical scope or compound statement.
  **L2341 CN**: 结束当前词法作用域或复合语句块。
- **L2342 EN**: Blank line separating nearby declarations or logic blocks.
  **L2342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2343 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2343 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2344 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2344 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2345 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvffint_s_l(__m256i _1, __m256i _2) {`.
  **L2345 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvffint_s_l(__m256i _1, __m256i _2) {`。
- **L2346 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvffint_s_l((v4i64)_1, (v4i64)_2)`.
  **L2346 CN**: 以 `(__m256)__builtin_lasx_xvffint_s_l((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2347 EN**: Closes the current lexical scope or compound statement.
  **L2347 CN**: 结束当前词法作用域或复合语句块。
- **L2348 EN**: Blank line separating nearby declarations or logic blocks.
  **L2348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2349 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2349 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2350 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2350 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2351 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrz_w_d(__m256d _1, __m256d _2) {`.
  **L2351 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrz_w_d(__m256d _1, __m256d _2) {`。
- **L2352 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrz_w_d((v4f64)_1, (v4f64)_2)`.
  **L2352 CN**: 以 `(__m256i)__builtin_lasx_xvftintrz_w_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。

### Lines 2353-2376

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrp_w_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvftintrp_w_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrm_w_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvftintrm_w_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrne_w_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvftintrne_w_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftinth_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftinth_l_s((v8f32)_1);
````
- **L2353 EN**: Closes the current lexical scope or compound statement.
  **L2353 CN**: 结束当前词法作用域或复合语句块。
- **L2354 EN**: Blank line separating nearby declarations or logic blocks.
  **L2354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2355 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2355 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2356 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2356 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2357 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrp_w_d(__m256d _1, __m256d _2) {`.
  **L2357 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrp_w_d(__m256d _1, __m256d _2) {`。
- **L2358 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrp_w_d((v4f64)_1, (v4f64)_2)`.
  **L2358 CN**: 以 `(__m256i)__builtin_lasx_xvftintrp_w_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L2359 EN**: Closes the current lexical scope or compound statement.
  **L2359 CN**: 结束当前词法作用域或复合语句块。
- **L2360 EN**: Blank line separating nearby declarations or logic blocks.
  **L2360 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2361 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2361 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2362 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2362 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2363 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrm_w_d(__m256d _1, __m256d _2) {`.
  **L2363 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrm_w_d(__m256d _1, __m256d _2) {`。
- **L2364 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrm_w_d((v4f64)_1, (v4f64)_2)`.
  **L2364 CN**: 以 `(__m256i)__builtin_lasx_xvftintrm_w_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L2365 EN**: Closes the current lexical scope or compound statement.
  **L2365 CN**: 结束当前词法作用域或复合语句块。
- **L2366 EN**: Blank line separating nearby declarations or logic blocks.
  **L2366 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2367 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2367 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2368 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2368 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2369 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrne_w_d(__m256d _1, __m256d _2) {`.
  **L2369 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrne_w_d(__m256d _1, __m256d _2) {`。
- **L2370 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrne_w_d((v4f64)_1, (v4f64)_2)`.
  **L2370 CN**: 以 `(__m256i)__builtin_lasx_xvftintrne_w_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L2371 EN**: Closes the current lexical scope or compound statement.
  **L2371 CN**: 结束当前词法作用域或复合语句块。
- **L2372 EN**: Blank line separating nearby declarations or logic blocks.
  **L2372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2373 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2373 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2374 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2374 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2375 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftinth_l_s(__m256 _1) {`.
  **L2375 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftinth_l_s(__m256 _1) {`。
- **L2376 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftinth_l_s((v8f32)_1)`.
  **L2376 CN**: 以 `(__m256i)__builtin_lasx_xvftinth_l_s((v8f32)_1)` 从当前函数返回。

### Lines 2377-2400

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintl_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintl_l_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvffinth_d_w(__m256i _1) {
  return (__m256d)__builtin_lasx_xvffinth_d_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvffintl_d_w(__m256i _1) {
  return (__m256d)__builtin_lasx_xvffintl_d_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrzh_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrzh_l_s((v8f32)_1);
````
- **L2377 EN**: Closes the current lexical scope or compound statement.
  **L2377 CN**: 结束当前词法作用域或复合语句块。
- **L2378 EN**: Blank line separating nearby declarations or logic blocks.
  **L2378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2379 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2379 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2380 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2380 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2381 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintl_l_s(__m256 _1) {`.
  **L2381 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintl_l_s(__m256 _1) {`。
- **L2382 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintl_l_s((v8f32)_1)`.
  **L2382 CN**: 以 `(__m256i)__builtin_lasx_xvftintl_l_s((v8f32)_1)` 从当前函数返回。
- **L2383 EN**: Closes the current lexical scope or compound statement.
  **L2383 CN**: 结束当前词法作用域或复合语句块。
- **L2384 EN**: Blank line separating nearby declarations or logic blocks.
  **L2384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2385 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2385 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2386 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2386 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2387 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvffinth_d_w(__m256i _1) {`.
  **L2387 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvffinth_d_w(__m256i _1) {`。
- **L2388 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvffinth_d_w((v8i32)_1)`.
  **L2388 CN**: 以 `(__m256d)__builtin_lasx_xvffinth_d_w((v8i32)_1)` 从当前函数返回。
- **L2389 EN**: Closes the current lexical scope or compound statement.
  **L2389 CN**: 结束当前词法作用域或复合语句块。
- **L2390 EN**: Blank line separating nearby declarations or logic blocks.
  **L2390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2391 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2391 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2392 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2392 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2393 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvffintl_d_w(__m256i _1) {`.
  **L2393 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvffintl_d_w(__m256i _1) {`。
- **L2394 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvffintl_d_w((v8i32)_1)`.
  **L2394 CN**: 以 `(__m256d)__builtin_lasx_xvffintl_d_w((v8i32)_1)` 从当前函数返回。
- **L2395 EN**: Closes the current lexical scope or compound statement.
  **L2395 CN**: 结束当前词法作用域或复合语句块。
- **L2396 EN**: Blank line separating nearby declarations or logic blocks.
  **L2396 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2397 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2397 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2398 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2398 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2399 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrzh_l_s(__m256 _1) {`.
  **L2399 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrzh_l_s(__m256 _1) {`。
- **L2400 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrzh_l_s((v8f32)_1)`.
  **L2400 CN**: 以 `(__m256i)__builtin_lasx_xvftintrzh_l_s((v8f32)_1)` 从当前函数返回。

### Lines 2401-2424

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrzl_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrzl_l_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrph_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrph_l_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrpl_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrpl_l_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrmh_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrmh_l_s((v8f32)_1);
````
- **L2401 EN**: Closes the current lexical scope or compound statement.
  **L2401 CN**: 结束当前词法作用域或复合语句块。
- **L2402 EN**: Blank line separating nearby declarations or logic blocks.
  **L2402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2403 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2403 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2404 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2404 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2405 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrzl_l_s(__m256 _1) {`.
  **L2405 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrzl_l_s(__m256 _1) {`。
- **L2406 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrzl_l_s((v8f32)_1)`.
  **L2406 CN**: 以 `(__m256i)__builtin_lasx_xvftintrzl_l_s((v8f32)_1)` 从当前函数返回。
- **L2407 EN**: Closes the current lexical scope or compound statement.
  **L2407 CN**: 结束当前词法作用域或复合语句块。
- **L2408 EN**: Blank line separating nearby declarations or logic blocks.
  **L2408 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2409 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2409 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2410 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2410 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2411 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrph_l_s(__m256 _1) {`.
  **L2411 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrph_l_s(__m256 _1) {`。
- **L2412 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrph_l_s((v8f32)_1)`.
  **L2412 CN**: 以 `(__m256i)__builtin_lasx_xvftintrph_l_s((v8f32)_1)` 从当前函数返回。
- **L2413 EN**: Closes the current lexical scope or compound statement.
  **L2413 CN**: 结束当前词法作用域或复合语句块。
- **L2414 EN**: Blank line separating nearby declarations or logic blocks.
  **L2414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2415 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2415 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2416 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2416 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2417 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrpl_l_s(__m256 _1) {`.
  **L2417 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrpl_l_s(__m256 _1) {`。
- **L2418 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrpl_l_s((v8f32)_1)`.
  **L2418 CN**: 以 `(__m256i)__builtin_lasx_xvftintrpl_l_s((v8f32)_1)` 从当前函数返回。
- **L2419 EN**: Closes the current lexical scope or compound statement.
  **L2419 CN**: 结束当前词法作用域或复合语句块。
- **L2420 EN**: Blank line separating nearby declarations or logic blocks.
  **L2420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2421 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2421 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2422 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2422 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2423 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrmh_l_s(__m256 _1) {`.
  **L2423 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrmh_l_s(__m256 _1) {`。
- **L2424 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrmh_l_s((v8f32)_1)`.
  **L2424 CN**: 以 `(__m256i)__builtin_lasx_xvftintrmh_l_s((v8f32)_1)` 从当前函数返回。

### Lines 2425-2448

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrml_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrml_l_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrneh_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrneh_l_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvftintrnel_l_s(__m256 _1) {
  return (__m256i)__builtin_lasx_xvftintrnel_l_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrintrne_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrintrne_s((v8f32)_1);
````
- **L2425 EN**: Closes the current lexical scope or compound statement.
  **L2425 CN**: 结束当前词法作用域或复合语句块。
- **L2426 EN**: Blank line separating nearby declarations or logic blocks.
  **L2426 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2427 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2427 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2428 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2428 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2429 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrml_l_s(__m256 _1) {`.
  **L2429 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrml_l_s(__m256 _1) {`。
- **L2430 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrml_l_s((v8f32)_1)`.
  **L2430 CN**: 以 `(__m256i)__builtin_lasx_xvftintrml_l_s((v8f32)_1)` 从当前函数返回。
- **L2431 EN**: Closes the current lexical scope or compound statement.
  **L2431 CN**: 结束当前词法作用域或复合语句块。
- **L2432 EN**: Blank line separating nearby declarations or logic blocks.
  **L2432 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2433 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2433 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2434 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2434 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2435 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrneh_l_s(__m256 _1) {`.
  **L2435 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrneh_l_s(__m256 _1) {`。
- **L2436 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrneh_l_s((v8f32)_1)`.
  **L2436 CN**: 以 `(__m256i)__builtin_lasx_xvftintrneh_l_s((v8f32)_1)` 从当前函数返回。
- **L2437 EN**: Closes the current lexical scope or compound statement.
  **L2437 CN**: 结束当前词法作用域或复合语句块。
- **L2438 EN**: Blank line separating nearby declarations or logic blocks.
  **L2438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2439 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2439 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2440 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2440 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2441 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvftintrnel_l_s(__m256 _1) {`.
  **L2441 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvftintrnel_l_s(__m256 _1) {`。
- **L2442 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvftintrnel_l_s((v8f32)_1)`.
  **L2442 CN**: 以 `(__m256i)__builtin_lasx_xvftintrnel_l_s((v8f32)_1)` 从当前函数返回。
- **L2443 EN**: Closes the current lexical scope or compound statement.
  **L2443 CN**: 结束当前词法作用域或复合语句块。
- **L2444 EN**: Blank line separating nearby declarations or logic blocks.
  **L2444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2445 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2445 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2446 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2446 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2447 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrintrne_s(__m256 _1) {`.
  **L2447 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrintrne_s(__m256 _1) {`。
- **L2448 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrintrne_s((v8f32)_1)`.
  **L2448 CN**: 以 `(__m256)__builtin_lasx_xvfrintrne_s((v8f32)_1)` 从当前函数返回。

### Lines 2449-2472

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrintrne_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrintrne_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrintrz_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrintrz_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrintrz_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrintrz_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrintrp_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrintrp_s((v8f32)_1);
````
- **L2449 EN**: Closes the current lexical scope or compound statement.
  **L2449 CN**: 结束当前词法作用域或复合语句块。
- **L2450 EN**: Blank line separating nearby declarations or logic blocks.
  **L2450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2451 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2451 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2452 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2452 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2453 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrintrne_d(__m256d _1) {`.
  **L2453 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrintrne_d(__m256d _1) {`。
- **L2454 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrintrne_d((v4f64)_1)`.
  **L2454 CN**: 以 `(__m256d)__builtin_lasx_xvfrintrne_d((v4f64)_1)` 从当前函数返回。
- **L2455 EN**: Closes the current lexical scope or compound statement.
  **L2455 CN**: 结束当前词法作用域或复合语句块。
- **L2456 EN**: Blank line separating nearby declarations or logic blocks.
  **L2456 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2457 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2457 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2458 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2458 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2459 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrintrz_s(__m256 _1) {`.
  **L2459 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrintrz_s(__m256 _1) {`。
- **L2460 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrintrz_s((v8f32)_1)`.
  **L2460 CN**: 以 `(__m256)__builtin_lasx_xvfrintrz_s((v8f32)_1)` 从当前函数返回。
- **L2461 EN**: Closes the current lexical scope or compound statement.
  **L2461 CN**: 结束当前词法作用域或复合语句块。
- **L2462 EN**: Blank line separating nearby declarations or logic blocks.
  **L2462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2463 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2463 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2464 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2464 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2465 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrintrz_d(__m256d _1) {`.
  **L2465 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrintrz_d(__m256d _1) {`。
- **L2466 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrintrz_d((v4f64)_1)`.
  **L2466 CN**: 以 `(__m256d)__builtin_lasx_xvfrintrz_d((v4f64)_1)` 从当前函数返回。
- **L2467 EN**: Closes the current lexical scope or compound statement.
  **L2467 CN**: 结束当前词法作用域或复合语句块。
- **L2468 EN**: Blank line separating nearby declarations or logic blocks.
  **L2468 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2469 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2469 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2470 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2470 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2471 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrintrp_s(__m256 _1) {`.
  **L2471 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrintrp_s(__m256 _1) {`。
- **L2472 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrintrp_s((v8f32)_1)`.
  **L2472 CN**: 以 `(__m256)__builtin_lasx_xvfrintrp_s((v8f32)_1)` 从当前函数返回。

### Lines 2473-2496

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrintrp_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrintrp_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrintrm_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrintrm_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrintrm_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrintrm_d((v4f64)_1);
}

#define __lasx_xvld(/*void **/ _1, /*si12*/ _2)                                \
  ((__m256i)__builtin_lasx_xvld((void const *)(_1), (_2)))

#define __lasx_xvst(/*__m256i*/ _1, /*void **/ _2, /*si12*/ _3)                \
````
- **L2473 EN**: Closes the current lexical scope or compound statement.
  **L2473 CN**: 结束当前词法作用域或复合语句块。
- **L2474 EN**: Blank line separating nearby declarations or logic blocks.
  **L2474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2475 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2475 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2476 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2476 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2477 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrintrp_d(__m256d _1) {`.
  **L2477 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrintrp_d(__m256d _1) {`。
- **L2478 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrintrp_d((v4f64)_1)`.
  **L2478 CN**: 以 `(__m256d)__builtin_lasx_xvfrintrp_d((v4f64)_1)` 从当前函数返回。
- **L2479 EN**: Closes the current lexical scope or compound statement.
  **L2479 CN**: 结束当前词法作用域或复合语句块。
- **L2480 EN**: Blank line separating nearby declarations or logic blocks.
  **L2480 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2481 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2481 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2482 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L2482 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L2483 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrintrm_s(__m256 _1) {`.
  **L2483 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrintrm_s(__m256 _1) {`。
- **L2484 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrintrm_s((v8f32)_1)`.
  **L2484 CN**: 以 `(__m256)__builtin_lasx_xvfrintrm_s((v8f32)_1)` 从当前函数返回。
- **L2485 EN**: Closes the current lexical scope or compound statement.
  **L2485 CN**: 结束当前词法作用域或复合语句块。
- **L2486 EN**: Blank line separating nearby declarations or logic blocks.
  **L2486 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2487 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2487 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2488 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L2488 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L2489 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrintrm_d(__m256d _1) {`.
  **L2489 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrintrm_d(__m256d _1) {`。
- **L2490 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrintrm_d((v4f64)_1)`.
  **L2490 CN**: 以 `(__m256d)__builtin_lasx_xvfrintrm_d((v4f64)_1)` 从当前函数返回。
- **L2491 EN**: Closes the current lexical scope or compound statement.
  **L2491 CN**: 结束当前词法作用域或复合语句块。
- **L2492 EN**: Blank line separating nearby declarations or logic blocks.
  **L2492 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2493 EN**: Defines macro `__lasx_xvld(/*void **/ _1, /*si12*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2493 CN**: 定义宏 `__lasx_xvld(/*void **/ _1, /*si12*/ _2)`，用于条件编译、简写或 API 生成。
- **L2494 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvld`.
  **L2494 CN**: 继续与可调用符号 `__builtin_lasx_xvld` 相关的逻辑。
- **L2495 EN**: Blank line separating nearby declarations or logic blocks.
  **L2495 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2496 EN**: Defines macro `__lasx_xvst(/*__m256i*/ _1, /*void **/ _2, /*si12*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2496 CN**: 定义宏 `__lasx_xvst(/*__m256i*/ _1, /*void **/ _2, /*si12*/ _3)`，用于条件编译、简写或 API 生成。

### Lines 2497-2520

````c
  ((void)__builtin_lasx_xvst((v32i8)(_1), (void *)(_2), (_3)))

#define __lasx_xvstelm_b(/*__m256i*/ _1, /*void **/ _2, /*si8*/ _3,            \
                         /*idx*/ _4)                                           \
  ((void)__builtin_lasx_xvstelm_b((v32i8)(_1), (void *)(_2), (_3), (_4)))

#define __lasx_xvstelm_h(/*__m256i*/ _1, /*void **/ _2, /*si8*/ _3,            \
                         /*idx*/ _4)                                           \
  ((void)__builtin_lasx_xvstelm_h((v16i16)(_1), (void *)(_2), (_3), (_4)))

#define __lasx_xvstelm_w(/*__m256i*/ _1, /*void **/ _2, /*si8*/ _3,            \
                         /*idx*/ _4)                                           \
  ((void)__builtin_lasx_xvstelm_w((v8i32)(_1), (void *)(_2), (_3), (_4)))

#define __lasx_xvstelm_d(/*__m256i*/ _1, /*void **/ _2, /*si8*/ _3,            \
                         /*idx*/ _4)                                           \
  ((void)__builtin_lasx_xvstelm_d((v4i64)(_1), (void *)(_2), (_3), (_4)))

#define __lasx_xvinsve0_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui3*/ _3)          \
  ((__m256i)__builtin_lasx_xvinsve0_w((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvinsve0_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui2*/ _3)          \
  ((__m256i)__builtin_lasx_xvinsve0_d((v4i64)(_1), (v4i64)(_2), (_3)))

````
- **L2497 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvst`.
  **L2497 CN**: 继续与可调用符号 `__builtin_lasx_xvst` 相关的逻辑。
- **L2498 EN**: Blank line separating nearby declarations or logic blocks.
  **L2498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2499 EN**: Defines macro `__lasx_xvstelm_b` for conditional compilation, shorthand, or API generation.
  **L2499 CN**: 定义宏 `__lasx_xvstelm_b`，用于条件编译、简写或 API 生成。
- **L2500 EN**: Comment explains nearby logic, constraints, or intent: `idx _4)`.
  **L2500 CN**: 注释解释附近代码的逻辑、约束或设计意图：`idx _4)`。
- **L2501 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvstelm_b`.
  **L2501 CN**: 继续与可调用符号 `__builtin_lasx_xvstelm_b` 相关的逻辑。
- **L2502 EN**: Blank line separating nearby declarations or logic blocks.
  **L2502 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2503 EN**: Defines macro `__lasx_xvstelm_h` for conditional compilation, shorthand, or API generation.
  **L2503 CN**: 定义宏 `__lasx_xvstelm_h`，用于条件编译、简写或 API 生成。
- **L2504 EN**: Comment explains nearby logic, constraints, or intent: `idx _4)`.
  **L2504 CN**: 注释解释附近代码的逻辑、约束或设计意图：`idx _4)`。
- **L2505 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvstelm_h`.
  **L2505 CN**: 继续与可调用符号 `__builtin_lasx_xvstelm_h` 相关的逻辑。
- **L2506 EN**: Blank line separating nearby declarations or logic blocks.
  **L2506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2507 EN**: Defines macro `__lasx_xvstelm_w` for conditional compilation, shorthand, or API generation.
  **L2507 CN**: 定义宏 `__lasx_xvstelm_w`，用于条件编译、简写或 API 生成。
- **L2508 EN**: Comment explains nearby logic, constraints, or intent: `idx _4)`.
  **L2508 CN**: 注释解释附近代码的逻辑、约束或设计意图：`idx _4)`。
- **L2509 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvstelm_w`.
  **L2509 CN**: 继续与可调用符号 `__builtin_lasx_xvstelm_w` 相关的逻辑。
- **L2510 EN**: Blank line separating nearby declarations or logic blocks.
  **L2510 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2511 EN**: Defines macro `__lasx_xvstelm_d` for conditional compilation, shorthand, or API generation.
  **L2511 CN**: 定义宏 `__lasx_xvstelm_d`，用于条件编译、简写或 API 生成。
- **L2512 EN**: Comment explains nearby logic, constraints, or intent: `idx _4)`.
  **L2512 CN**: 注释解释附近代码的逻辑、约束或设计意图：`idx _4)`。
- **L2513 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvstelm_d`.
  **L2513 CN**: 继续与可调用符号 `__builtin_lasx_xvstelm_d` 相关的逻辑。
- **L2514 EN**: Blank line separating nearby declarations or logic blocks.
  **L2514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2515 EN**: Defines macro `__lasx_xvinsve0_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui3*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2515 CN**: 定义宏 `__lasx_xvinsve0_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui3*/ _3)`，用于条件编译、简写或 API 生成。
- **L2516 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvinsve0_w`.
  **L2516 CN**: 继续与可调用符号 `__builtin_lasx_xvinsve0_w` 相关的逻辑。
- **L2517 EN**: Blank line separating nearby declarations or logic blocks.
  **L2517 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2518 EN**: Defines macro `__lasx_xvinsve0_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui2*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2518 CN**: 定义宏 `__lasx_xvinsve0_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui2*/ _3)`，用于条件编译、简写或 API 生成。
- **L2519 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvinsve0_d`.
  **L2519 CN**: 继续与可调用符号 `__builtin_lasx_xvinsve0_d` 相关的逻辑。
- **L2520 EN**: Blank line separating nearby declarations or logic blocks.
  **L2520 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2521-2544

````c
#define __lasx_xvpickve_w(/*__m256i*/ _1, /*ui3*/ _2)                          \
  ((__m256i)__builtin_lasx_xvpickve_w((v8i32)(_1), (_2)))

#define __lasx_xvpickve_d(/*__m256i*/ _1, /*ui2*/ _2)                          \
  ((__m256i)__builtin_lasx_xvpickve_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrlrn_b_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrlrn_b_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrlrn_h_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrlrn_h_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrlrn_w_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrlrn_w_d((v4i64)_1, (v4i64)_2);
}

````
- **L2521 EN**: Defines macro `__lasx_xvpickve_w(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2521 CN**: 定义宏 `__lasx_xvpickve_w(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L2522 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpickve_w`.
  **L2522 CN**: 继续与可调用符号 `__builtin_lasx_xvpickve_w` 相关的逻辑。
- **L2523 EN**: Blank line separating nearby declarations or logic blocks.
  **L2523 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2524 EN**: Defines macro `__lasx_xvpickve_d(/*__m256i*/ _1, /*ui2*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2524 CN**: 定义宏 `__lasx_xvpickve_d(/*__m256i*/ _1, /*ui2*/ _2)`，用于条件编译、简写或 API 生成。
- **L2525 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpickve_d`.
  **L2525 CN**: 继续与可调用符号 `__builtin_lasx_xvpickve_d` 相关的逻辑。
- **L2526 EN**: Blank line separating nearby declarations or logic blocks.
  **L2526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2527 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2527 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2528 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2528 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2529 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrlrn_b_h(__m256i _1, __m256i _2) {`.
  **L2529 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrlrn_b_h(__m256i _1, __m256i _2) {`。
- **L2530 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrlrn_b_h((v16i16)_1, (v16i16)_2)`.
  **L2530 CN**: 以 `(__m256i)__builtin_lasx_xvssrlrn_b_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2531 EN**: Closes the current lexical scope or compound statement.
  **L2531 CN**: 结束当前词法作用域或复合语句块。
- **L2532 EN**: Blank line separating nearby declarations or logic blocks.
  **L2532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2533 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2533 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2534 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2534 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2535 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrlrn_h_w(__m256i _1, __m256i _2) {`.
  **L2535 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrlrn_h_w(__m256i _1, __m256i _2) {`。
- **L2536 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrlrn_h_w((v8i32)_1, (v8i32)_2)`.
  **L2536 CN**: 以 `(__m256i)__builtin_lasx_xvssrlrn_h_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2537 EN**: Closes the current lexical scope or compound statement.
  **L2537 CN**: 结束当前词法作用域或复合语句块。
- **L2538 EN**: Blank line separating nearby declarations or logic blocks.
  **L2538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2539 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2539 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2540 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2540 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2541 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrlrn_w_d(__m256i _1, __m256i _2) {`.
  **L2541 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrlrn_w_d(__m256i _1, __m256i _2) {`。
- **L2542 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrlrn_w_d((v4i64)_1, (v4i64)_2)`.
  **L2542 CN**: 以 `(__m256i)__builtin_lasx_xvssrlrn_w_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2543 EN**: Closes the current lexical scope or compound statement.
  **L2543 CN**: 结束当前词法作用域或复合语句块。
- **L2544 EN**: Blank line separating nearby declarations or logic blocks.
  **L2544 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2545-2568

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrln_b_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrln_b_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrln_h_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrln_h_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvssrln_w_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvssrln_w_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvorn_v(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvorn_v((v32u8)_1, (v32u8)_2);
}

````
- **L2545 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2545 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2546 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2546 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2547 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrln_b_h(__m256i _1, __m256i _2) {`.
  **L2547 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrln_b_h(__m256i _1, __m256i _2) {`。
- **L2548 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrln_b_h((v16i16)_1, (v16i16)_2)`.
  **L2548 CN**: 以 `(__m256i)__builtin_lasx_xvssrln_b_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2549 EN**: Closes the current lexical scope or compound statement.
  **L2549 CN**: 结束当前词法作用域或复合语句块。
- **L2550 EN**: Blank line separating nearby declarations or logic blocks.
  **L2550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2551 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2551 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2552 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2552 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2553 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrln_h_w(__m256i _1, __m256i _2) {`.
  **L2553 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrln_h_w(__m256i _1, __m256i _2) {`。
- **L2554 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrln_h_w((v8i32)_1, (v8i32)_2)`.
  **L2554 CN**: 以 `(__m256i)__builtin_lasx_xvssrln_h_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2555 EN**: Closes the current lexical scope or compound statement.
  **L2555 CN**: 结束当前词法作用域或复合语句块。
- **L2556 EN**: Blank line separating nearby declarations or logic blocks.
  **L2556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2557 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2557 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2558 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2558 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2559 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvssrln_w_d(__m256i _1, __m256i _2) {`.
  **L2559 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvssrln_w_d(__m256i _1, __m256i _2) {`。
- **L2560 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvssrln_w_d((v4i64)_1, (v4i64)_2)`.
  **L2560 CN**: 以 `(__m256i)__builtin_lasx_xvssrln_w_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L2561 EN**: Closes the current lexical scope or compound statement.
  **L2561 CN**: 结束当前词法作用域或复合语句块。
- **L2562 EN**: Blank line separating nearby declarations or logic blocks.
  **L2562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2563 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2563 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2564 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2564 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2565 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvorn_v(__m256i _1, __m256i _2) {`.
  **L2565 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvorn_v(__m256i _1, __m256i _2) {`。
- **L2566 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvorn_v((v32u8)_1, (v32u8)_2)`.
  **L2566 CN**: 以 `(__m256i)__builtin_lasx_xvorn_v((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L2567 EN**: Closes the current lexical scope or compound statement.
  **L2567 CN**: 结束当前词法作用域或复合语句块。
- **L2568 EN**: Blank line separating nearby declarations or logic blocks.
  **L2568 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2569-2592

````c
#define __lasx_xvldi(/*i13*/ _1) ((__m256i)__builtin_lasx_xvldi((_1)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvldx(void const *_1, long int _2) {
  return (__m256i)__builtin_lasx_xvldx((void const *)_1, (long int)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) void
    __lasx_xvstx(__m256i _1, void *_2, long int _3) {
  return (void)__builtin_lasx_xvstx((v32i8)_1, (void *)_2, (long int)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvextl_qu_du(__m256i _1) {
  return (__m256i)__builtin_lasx_xvextl_qu_du((v4u64)_1);
}

#define __lasx_xvinsgr2vr_w(/*__m256i*/ _1, /*int*/ _2, /*ui3*/ _3)            \
  ((__m256i)__builtin_lasx_xvinsgr2vr_w((v8i32)(_1), (int)(_2), (_3)))

#define __lasx_xvinsgr2vr_d(/*__m256i*/ _1, /*long int*/ _2, /*ui2*/ _3)       \
````
- **L2569 EN**: Defines macro `__lasx_xvldi(/*i13*/ _1)` for conditional compilation, shorthand, or API generation.
  **L2569 CN**: 定义宏 `__lasx_xvldi(/*i13*/ _1)`，用于条件编译、简写或 API 生成。
- **L2570 EN**: Blank line separating nearby declarations or logic blocks.
  **L2570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2571 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2571 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2572 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2572 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2573 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvldx(void const *_1, long int _2) {`.
  **L2573 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvldx(void const *_1, long int _2) {`。
- **L2574 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvldx((void const *)_1, (long int)_2)`.
  **L2574 CN**: 以 `(__m256i)__builtin_lasx_xvldx((void const *)_1, (long int)_2)` 从当前函数返回。
- **L2575 EN**: Closes the current lexical scope or compound statement.
  **L2575 CN**: 结束当前词法作用域或复合语句块。
- **L2576 EN**: Blank line separating nearby declarations or logic blocks.
  **L2576 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2577 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2577 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2578 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) void`.
  **L2578 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) void`。
- **L2579 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvstx(__m256i _1, void *_2, long int _3) {`.
  **L2579 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvstx(__m256i _1, void *_2, long int _3) {`。
- **L2580 EN**: Returns from the current function with `(void)__builtin_lasx_xvstx((v32i8)_1, (void *)_2, (long int)_3)`.
  **L2580 CN**: 以 `(void)__builtin_lasx_xvstx((v32i8)_1, (void *)_2, (long int)_3)` 从当前函数返回。
- **L2581 EN**: Closes the current lexical scope or compound statement.
  **L2581 CN**: 结束当前词法作用域或复合语句块。
- **L2582 EN**: Blank line separating nearby declarations or logic blocks.
  **L2582 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2583 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2583 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2584 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2584 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2585 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvextl_qu_du(__m256i _1) {`.
  **L2585 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvextl_qu_du(__m256i _1) {`。
- **L2586 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvextl_qu_du((v4u64)_1)`.
  **L2586 CN**: 以 `(__m256i)__builtin_lasx_xvextl_qu_du((v4u64)_1)` 从当前函数返回。
- **L2587 EN**: Closes the current lexical scope or compound statement.
  **L2587 CN**: 结束当前词法作用域或复合语句块。
- **L2588 EN**: Blank line separating nearby declarations or logic blocks.
  **L2588 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2589 EN**: Defines macro `__lasx_xvinsgr2vr_w(/*__m256i*/ _1, /*int*/ _2, /*ui3*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2589 CN**: 定义宏 `__lasx_xvinsgr2vr_w(/*__m256i*/ _1, /*int*/ _2, /*ui3*/ _3)`，用于条件编译、简写或 API 生成。
- **L2590 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvinsgr2vr_w`.
  **L2590 CN**: 继续与可调用符号 `__builtin_lasx_xvinsgr2vr_w` 相关的逻辑。
- **L2591 EN**: Blank line separating nearby declarations or logic blocks.
  **L2591 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2592 EN**: Defines macro `__lasx_xvinsgr2vr_d(/*__m256i*/ _1, /*long int*/ _2, /*ui2*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2592 CN**: 定义宏 `__lasx_xvinsgr2vr_d(/*__m256i*/ _1, /*long int*/ _2, /*ui2*/ _3)`，用于条件编译、简写或 API 生成。

### Lines 2593-2616

````c
  ((__m256i)__builtin_lasx_xvinsgr2vr_d((v4i64)(_1), (long int)(_2), (_3)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve0_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvreplve0_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve0_h(__m256i _1) {
  return (__m256i)__builtin_lasx_xvreplve0_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve0_w(__m256i _1) {
  return (__m256i)__builtin_lasx_xvreplve0_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve0_d(__m256i _1) {
  return (__m256i)__builtin_lasx_xvreplve0_d((v4i64)_1);
````
- **L2593 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvinsgr2vr_d`.
  **L2593 CN**: 继续与可调用符号 `__builtin_lasx_xvinsgr2vr_d` 相关的逻辑。
- **L2594 EN**: Blank line separating nearby declarations or logic blocks.
  **L2594 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2595 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2595 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2596 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2596 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2597 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve0_b(__m256i _1) {`.
  **L2597 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve0_b(__m256i _1) {`。
- **L2598 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve0_b((v32i8)_1)`.
  **L2598 CN**: 以 `(__m256i)__builtin_lasx_xvreplve0_b((v32i8)_1)` 从当前函数返回。
- **L2599 EN**: Closes the current lexical scope or compound statement.
  **L2599 CN**: 结束当前词法作用域或复合语句块。
- **L2600 EN**: Blank line separating nearby declarations or logic blocks.
  **L2600 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2601 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2601 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2602 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2602 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2603 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve0_h(__m256i _1) {`.
  **L2603 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve0_h(__m256i _1) {`。
- **L2604 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve0_h((v16i16)_1)`.
  **L2604 CN**: 以 `(__m256i)__builtin_lasx_xvreplve0_h((v16i16)_1)` 从当前函数返回。
- **L2605 EN**: Closes the current lexical scope or compound statement.
  **L2605 CN**: 结束当前词法作用域或复合语句块。
- **L2606 EN**: Blank line separating nearby declarations or logic blocks.
  **L2606 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2607 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2607 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2608 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2608 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2609 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve0_w(__m256i _1) {`.
  **L2609 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve0_w(__m256i _1) {`。
- **L2610 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve0_w((v8i32)_1)`.
  **L2610 CN**: 以 `(__m256i)__builtin_lasx_xvreplve0_w((v8i32)_1)` 从当前函数返回。
- **L2611 EN**: Closes the current lexical scope or compound statement.
  **L2611 CN**: 结束当前词法作用域或复合语句块。
- **L2612 EN**: Blank line separating nearby declarations or logic blocks.
  **L2612 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2613 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2613 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2614 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2614 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2615 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve0_d(__m256i _1) {`.
  **L2615 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve0_d(__m256i _1) {`。
- **L2616 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve0_d((v4i64)_1)`.
  **L2616 CN**: 以 `(__m256i)__builtin_lasx_xvreplve0_d((v4i64)_1)` 从当前函数返回。

### Lines 2617-2640

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvreplve0_q(__m256i _1) {
  return (__m256i)__builtin_lasx_xvreplve0_q((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_h_b(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_h_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_w_h(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_w_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_d_w(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_d_w((v8i32)_1);
````
- **L2617 EN**: Closes the current lexical scope or compound statement.
  **L2617 CN**: 结束当前词法作用域或复合语句块。
- **L2618 EN**: Blank line separating nearby declarations or logic blocks.
  **L2618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2619 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2619 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2620 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2620 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2621 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvreplve0_q(__m256i _1) {`.
  **L2621 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvreplve0_q(__m256i _1) {`。
- **L2622 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvreplve0_q((v32i8)_1)`.
  **L2622 CN**: 以 `(__m256i)__builtin_lasx_xvreplve0_q((v32i8)_1)` 从当前函数返回。
- **L2623 EN**: Closes the current lexical scope or compound statement.
  **L2623 CN**: 结束当前词法作用域或复合语句块。
- **L2624 EN**: Blank line separating nearby declarations or logic blocks.
  **L2624 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2625 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2625 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2626 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2626 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2627 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_h_b(__m256i _1) {`.
  **L2627 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_h_b(__m256i _1) {`。
- **L2628 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_h_b((v32i8)_1)`.
  **L2628 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_h_b((v32i8)_1)` 从当前函数返回。
- **L2629 EN**: Closes the current lexical scope or compound statement.
  **L2629 CN**: 结束当前词法作用域或复合语句块。
- **L2630 EN**: Blank line separating nearby declarations or logic blocks.
  **L2630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2631 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2631 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2632 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2632 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2633 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_w_h(__m256i _1) {`.
  **L2633 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_w_h(__m256i _1) {`。
- **L2634 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_w_h((v16i16)_1)`.
  **L2634 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_w_h((v16i16)_1)` 从当前函数返回。
- **L2635 EN**: Closes the current lexical scope or compound statement.
  **L2635 CN**: 结束当前词法作用域或复合语句块。
- **L2636 EN**: Blank line separating nearby declarations or logic blocks.
  **L2636 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2637 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2637 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2638 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2638 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2639 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_d_w(__m256i _1) {`.
  **L2639 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_d_w(__m256i _1) {`。
- **L2640 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_d_w((v8i32)_1)`.
  **L2640 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_d_w((v8i32)_1)` 从当前函数返回。

### Lines 2641-2664

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_w_b(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_w_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_d_h(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_d_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_d_b(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_d_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_hu_bu(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_hu_bu((v32i8)_1);
````
- **L2641 EN**: Closes the current lexical scope or compound statement.
  **L2641 CN**: 结束当前词法作用域或复合语句块。
- **L2642 EN**: Blank line separating nearby declarations or logic blocks.
  **L2642 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2643 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2643 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2644 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2644 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2645 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_w_b(__m256i _1) {`.
  **L2645 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_w_b(__m256i _1) {`。
- **L2646 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_w_b((v32i8)_1)`.
  **L2646 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_w_b((v32i8)_1)` 从当前函数返回。
- **L2647 EN**: Closes the current lexical scope or compound statement.
  **L2647 CN**: 结束当前词法作用域或复合语句块。
- **L2648 EN**: Blank line separating nearby declarations or logic blocks.
  **L2648 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2649 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2649 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2650 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2650 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2651 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_d_h(__m256i _1) {`.
  **L2651 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_d_h(__m256i _1) {`。
- **L2652 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_d_h((v16i16)_1)`.
  **L2652 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_d_h((v16i16)_1)` 从当前函数返回。
- **L2653 EN**: Closes the current lexical scope or compound statement.
  **L2653 CN**: 结束当前词法作用域或复合语句块。
- **L2654 EN**: Blank line separating nearby declarations or logic blocks.
  **L2654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2655 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2655 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2656 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2656 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2657 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_d_b(__m256i _1) {`.
  **L2657 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_d_b(__m256i _1) {`。
- **L2658 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_d_b((v32i8)_1)`.
  **L2658 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_d_b((v32i8)_1)` 从当前函数返回。
- **L2659 EN**: Closes the current lexical scope or compound statement.
  **L2659 CN**: 结束当前词法作用域或复合语句块。
- **L2660 EN**: Blank line separating nearby declarations or logic blocks.
  **L2660 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2661 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2661 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2662 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2662 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2663 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_hu_bu(__m256i _1) {`.
  **L2663 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_hu_bu(__m256i _1) {`。
- **L2664 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_hu_bu((v32i8)_1)`.
  **L2664 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_hu_bu((v32i8)_1)` 从当前函数返回。

### Lines 2665-2688

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_wu_hu(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_wu_hu((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_du_wu(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_du_wu((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_wu_bu(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_wu_bu((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_du_hu(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_du_hu((v16i16)_1);
````
- **L2665 EN**: Closes the current lexical scope or compound statement.
  **L2665 CN**: 结束当前词法作用域或复合语句块。
- **L2666 EN**: Blank line separating nearby declarations or logic blocks.
  **L2666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2667 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2667 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2668 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2668 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2669 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_wu_hu(__m256i _1) {`.
  **L2669 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_wu_hu(__m256i _1) {`。
- **L2670 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_wu_hu((v16i16)_1)`.
  **L2670 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_wu_hu((v16i16)_1)` 从当前函数返回。
- **L2671 EN**: Closes the current lexical scope or compound statement.
  **L2671 CN**: 结束当前词法作用域或复合语句块。
- **L2672 EN**: Blank line separating nearby declarations or logic blocks.
  **L2672 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2673 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2673 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2674 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2674 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2675 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_du_wu(__m256i _1) {`.
  **L2675 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_du_wu(__m256i _1) {`。
- **L2676 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_du_wu((v8i32)_1)`.
  **L2676 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_du_wu((v8i32)_1)` 从当前函数返回。
- **L2677 EN**: Closes the current lexical scope or compound statement.
  **L2677 CN**: 结束当前词法作用域或复合语句块。
- **L2678 EN**: Blank line separating nearby declarations or logic blocks.
  **L2678 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2679 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2679 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2680 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2680 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2681 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_wu_bu(__m256i _1) {`.
  **L2681 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_wu_bu(__m256i _1) {`。
- **L2682 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_wu_bu((v32i8)_1)`.
  **L2682 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_wu_bu((v32i8)_1)` 从当前函数返回。
- **L2683 EN**: Closes the current lexical scope or compound statement.
  **L2683 CN**: 结束当前词法作用域或复合语句块。
- **L2684 EN**: Blank line separating nearby declarations or logic blocks.
  **L2684 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2685 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2685 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2686 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2686 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2687 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_du_hu(__m256i _1) {`.
  **L2687 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_du_hu(__m256i _1) {`。
- **L2688 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_du_hu((v16i16)_1)`.
  **L2688 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_du_hu((v16i16)_1)` 从当前函数返回。

### Lines 2689-2712

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_vext2xv_du_bu(__m256i _1) {
  return (__m256i)__builtin_lasx_vext2xv_du_bu((v32i8)_1);
}

#define __lasx_xvpermi_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)           \
  ((__m256i)__builtin_lasx_xvpermi_q((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvpermi_d(/*__m256i*/ _1, /*ui8*/ _2)                           \
  ((__m256i)__builtin_lasx_xvpermi_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvperm_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvperm_w((v8i32)_1, (v8i32)_2);
}

#define __lasx_xvldrepl_b(/*void **/ _1, /*si12*/ _2)                          \
  ((__m256i)__builtin_lasx_xvldrepl_b((void const *)(_1), (_2)))

#define __lasx_xvldrepl_h(/*void **/ _1, /*si11*/ _2)                          \
````
- **L2689 EN**: Closes the current lexical scope or compound statement.
  **L2689 CN**: 结束当前词法作用域或复合语句块。
- **L2690 EN**: Blank line separating nearby declarations or logic blocks.
  **L2690 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2691 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2691 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2692 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2692 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2693 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_vext2xv_du_bu(__m256i _1) {`.
  **L2693 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_vext2xv_du_bu(__m256i _1) {`。
- **L2694 EN**: Returns from the current function with `(__m256i)__builtin_lasx_vext2xv_du_bu((v32i8)_1)`.
  **L2694 CN**: 以 `(__m256i)__builtin_lasx_vext2xv_du_bu((v32i8)_1)` 从当前函数返回。
- **L2695 EN**: Closes the current lexical scope or compound statement.
  **L2695 CN**: 结束当前词法作用域或复合语句块。
- **L2696 EN**: Blank line separating nearby declarations or logic blocks.
  **L2696 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2697 EN**: Defines macro `__lasx_xvpermi_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2697 CN**: 定义宏 `__lasx_xvpermi_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2698 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpermi_q`.
  **L2698 CN**: 继续与可调用符号 `__builtin_lasx_xvpermi_q` 相关的逻辑。
- **L2699 EN**: Blank line separating nearby declarations or logic blocks.
  **L2699 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2700 EN**: Defines macro `__lasx_xvpermi_d(/*__m256i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2700 CN**: 定义宏 `__lasx_xvpermi_d(/*__m256i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L2701 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpermi_d`.
  **L2701 CN**: 继续与可调用符号 `__builtin_lasx_xvpermi_d` 相关的逻辑。
- **L2702 EN**: Blank line separating nearby declarations or logic blocks.
  **L2702 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2703 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2703 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2704 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2704 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2705 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvperm_w(__m256i _1, __m256i _2) {`.
  **L2705 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvperm_w(__m256i _1, __m256i _2) {`。
- **L2706 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvperm_w((v8i32)_1, (v8i32)_2)`.
  **L2706 CN**: 以 `(__m256i)__builtin_lasx_xvperm_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2707 EN**: Closes the current lexical scope or compound statement.
  **L2707 CN**: 结束当前词法作用域或复合语句块。
- **L2708 EN**: Blank line separating nearby declarations or logic blocks.
  **L2708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2709 EN**: Defines macro `__lasx_xvldrepl_b(/*void **/ _1, /*si12*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2709 CN**: 定义宏 `__lasx_xvldrepl_b(/*void **/ _1, /*si12*/ _2)`，用于条件编译、简写或 API 生成。
- **L2710 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvldrepl_b`.
  **L2710 CN**: 继续与可调用符号 `__builtin_lasx_xvldrepl_b` 相关的逻辑。
- **L2711 EN**: Blank line separating nearby declarations or logic blocks.
  **L2711 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2712 EN**: Defines macro `__lasx_xvldrepl_h(/*void **/ _1, /*si11*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2712 CN**: 定义宏 `__lasx_xvldrepl_h(/*void **/ _1, /*si11*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 2713-2736

````c
  ((__m256i)__builtin_lasx_xvldrepl_h((void const *)(_1), (_2)))

#define __lasx_xvldrepl_w(/*void **/ _1, /*si10*/ _2)                          \
  ((__m256i)__builtin_lasx_xvldrepl_w((void const *)(_1), (_2)))

#define __lasx_xvldrepl_d(/*void **/ _1, /*si9*/ _2)                           \
  ((__m256i)__builtin_lasx_xvldrepl_d((void const *)(_1), (_2)))

#define __lasx_xvpickve2gr_w(/*__m256i*/ _1, /*ui3*/ _2)                       \
  ((int)__builtin_lasx_xvpickve2gr_w((v8i32)(_1), (_2)))

#define __lasx_xvpickve2gr_wu(/*__m256i*/ _1, /*ui3*/ _2)                      \
  ((unsigned int)__builtin_lasx_xvpickve2gr_wu((v8i32)(_1), (_2)))

#define __lasx_xvpickve2gr_d(/*__m256i*/ _1, /*ui2*/ _2)                       \
  ((long int)__builtin_lasx_xvpickve2gr_d((v4i64)(_1), (_2)))

#define __lasx_xvpickve2gr_du(/*__m256i*/ _1, /*ui2*/ _2)                      \
  ((unsigned long int)__builtin_lasx_xvpickve2gr_du((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_q_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_q_d((v4i64)_1, (v4i64)_2);
````
- **L2713 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvldrepl_h`.
  **L2713 CN**: 继续与可调用符号 `__builtin_lasx_xvldrepl_h` 相关的逻辑。
- **L2714 EN**: Blank line separating nearby declarations or logic blocks.
  **L2714 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2715 EN**: Defines macro `__lasx_xvldrepl_w(/*void **/ _1, /*si10*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2715 CN**: 定义宏 `__lasx_xvldrepl_w(/*void **/ _1, /*si10*/ _2)`，用于条件编译、简写或 API 生成。
- **L2716 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvldrepl_w`.
  **L2716 CN**: 继续与可调用符号 `__builtin_lasx_xvldrepl_w` 相关的逻辑。
- **L2717 EN**: Blank line separating nearby declarations or logic blocks.
  **L2717 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2718 EN**: Defines macro `__lasx_xvldrepl_d(/*void **/ _1, /*si9*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2718 CN**: 定义宏 `__lasx_xvldrepl_d(/*void **/ _1, /*si9*/ _2)`，用于条件编译、简写或 API 生成。
- **L2719 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvldrepl_d`.
  **L2719 CN**: 继续与可调用符号 `__builtin_lasx_xvldrepl_d` 相关的逻辑。
- **L2720 EN**: Blank line separating nearby declarations or logic blocks.
  **L2720 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2721 EN**: Defines macro `__lasx_xvpickve2gr_w(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2721 CN**: 定义宏 `__lasx_xvpickve2gr_w(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L2722 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpickve2gr_w`.
  **L2722 CN**: 继续与可调用符号 `__builtin_lasx_xvpickve2gr_w` 相关的逻辑。
- **L2723 EN**: Blank line separating nearby declarations or logic blocks.
  **L2723 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2724 EN**: Defines macro `__lasx_xvpickve2gr_wu(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2724 CN**: 定义宏 `__lasx_xvpickve2gr_wu(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L2725 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpickve2gr_wu`.
  **L2725 CN**: 继续与可调用符号 `__builtin_lasx_xvpickve2gr_wu` 相关的逻辑。
- **L2726 EN**: Blank line separating nearby declarations or logic blocks.
  **L2726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2727 EN**: Defines macro `__lasx_xvpickve2gr_d(/*__m256i*/ _1, /*ui2*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2727 CN**: 定义宏 `__lasx_xvpickve2gr_d(/*__m256i*/ _1, /*ui2*/ _2)`，用于条件编译、简写或 API 生成。
- **L2728 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpickve2gr_d`.
  **L2728 CN**: 继续与可调用符号 `__builtin_lasx_xvpickve2gr_d` 相关的逻辑。
- **L2729 EN**: Blank line separating nearby declarations or logic blocks.
  **L2729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2730 EN**: Defines macro `__lasx_xvpickve2gr_du(/*__m256i*/ _1, /*ui2*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2730 CN**: 定义宏 `__lasx_xvpickve2gr_du(/*__m256i*/ _1, /*ui2*/ _2)`，用于条件编译、简写或 API 生成。
- **L2731 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpickve2gr_du`.
  **L2731 CN**: 继续与可调用符号 `__builtin_lasx_xvpickve2gr_du` 相关的逻辑。
- **L2732 EN**: Blank line separating nearby declarations or logic blocks.
  **L2732 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2733 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2733 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2734 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2734 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2735 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_q_d(__m256i _1, __m256i _2) {`.
  **L2735 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_q_d(__m256i _1, __m256i _2) {`。
- **L2736 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_q_d((v4i64)_1, (v4i64)_2)`.
  **L2736 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_q_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。

### Lines 2737-2760

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_d_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_d_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_w_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_w_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_h_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_h_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_q_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_q_du((v4u64)_1, (v4u64)_2);
````
- **L2737 EN**: Closes the current lexical scope or compound statement.
  **L2737 CN**: 结束当前词法作用域或复合语句块。
- **L2738 EN**: Blank line separating nearby declarations or logic blocks.
  **L2738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2739 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2739 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2740 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2740 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2741 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_d_w(__m256i _1, __m256i _2) {`.
  **L2741 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_d_w(__m256i _1, __m256i _2) {`。
- **L2742 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_d_w((v8i32)_1, (v8i32)_2)`.
  **L2742 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_d_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2743 EN**: Closes the current lexical scope or compound statement.
  **L2743 CN**: 结束当前词法作用域或复合语句块。
- **L2744 EN**: Blank line separating nearby declarations or logic blocks.
  **L2744 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2745 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2745 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2746 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2746 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2747 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_w_h(__m256i _1, __m256i _2) {`.
  **L2747 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_w_h(__m256i _1, __m256i _2) {`。
- **L2748 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_w_h((v16i16)_1, (v16i16)_2)`.
  **L2748 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_w_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2749 EN**: Closes the current lexical scope or compound statement.
  **L2749 CN**: 结束当前词法作用域或复合语句块。
- **L2750 EN**: Blank line separating nearby declarations or logic blocks.
  **L2750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2751 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2751 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2752 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2752 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2753 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_h_b(__m256i _1, __m256i _2) {`.
  **L2753 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_h_b(__m256i _1, __m256i _2) {`。
- **L2754 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_h_b((v32i8)_1, (v32i8)_2)`.
  **L2754 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_h_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L2755 EN**: Closes the current lexical scope or compound statement.
  **L2755 CN**: 结束当前词法作用域或复合语句块。
- **L2756 EN**: Blank line separating nearby declarations or logic blocks.
  **L2756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2757 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2757 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2758 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2758 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2759 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_q_du(__m256i _1, __m256i _2) {`.
  **L2759 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_q_du(__m256i _1, __m256i _2) {`。
- **L2760 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_q_du((v4u64)_1, (v4u64)_2)`.
  **L2760 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_q_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。

### Lines 2761-2784

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_d_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_d_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_w_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_w_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_h_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_h_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwev_q_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwev_q_d((v4i64)_1, (v4i64)_2);
````
- **L2761 EN**: Closes the current lexical scope or compound statement.
  **L2761 CN**: 结束当前词法作用域或复合语句块。
- **L2762 EN**: Blank line separating nearby declarations or logic blocks.
  **L2762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2763 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2763 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2764 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2764 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2765 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_d_wu(__m256i _1, __m256i _2) {`.
  **L2765 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_d_wu(__m256i _1, __m256i _2) {`。
- **L2766 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_d_wu((v8u32)_1, (v8u32)_2)`.
  **L2766 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_d_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2767 EN**: Closes the current lexical scope or compound statement.
  **L2767 CN**: 结束当前词法作用域或复合语句块。
- **L2768 EN**: Blank line separating nearby declarations or logic blocks.
  **L2768 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2769 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2769 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2770 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2770 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2771 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_w_hu(__m256i _1, __m256i _2) {`.
  **L2771 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_w_hu(__m256i _1, __m256i _2) {`。
- **L2772 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_w_hu((v16u16)_1, (v16u16)_2)`.
  **L2772 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_w_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2773 EN**: Closes the current lexical scope or compound statement.
  **L2773 CN**: 结束当前词法作用域或复合语句块。
- **L2774 EN**: Blank line separating nearby declarations or logic blocks.
  **L2774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2775 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2775 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2776 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2776 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2777 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_h_bu(__m256i _1, __m256i _2) {`.
  **L2777 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_h_bu(__m256i _1, __m256i _2) {`。
- **L2778 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_h_bu((v32u8)_1, (v32u8)_2)`.
  **L2778 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_h_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L2779 EN**: Closes the current lexical scope or compound statement.
  **L2779 CN**: 结束当前词法作用域或复合语句块。
- **L2780 EN**: Blank line separating nearby declarations or logic blocks.
  **L2780 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2781 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2781 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2782 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2782 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2783 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwev_q_d(__m256i _1, __m256i _2) {`.
  **L2783 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwev_q_d(__m256i _1, __m256i _2) {`。
- **L2784 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwev_q_d((v4i64)_1, (v4i64)_2)`.
  **L2784 CN**: 以 `(__m256i)__builtin_lasx_xvsubwev_q_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。

### Lines 2785-2808

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwev_d_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwev_d_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwev_w_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwev_w_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwev_h_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwev_h_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwev_q_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwev_q_du((v4u64)_1, (v4u64)_2);
````
- **L2785 EN**: Closes the current lexical scope or compound statement.
  **L2785 CN**: 结束当前词法作用域或复合语句块。
- **L2786 EN**: Blank line separating nearby declarations or logic blocks.
  **L2786 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2787 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2787 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2788 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2788 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2789 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwev_d_w(__m256i _1, __m256i _2) {`.
  **L2789 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwev_d_w(__m256i _1, __m256i _2) {`。
- **L2790 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwev_d_w((v8i32)_1, (v8i32)_2)`.
  **L2790 CN**: 以 `(__m256i)__builtin_lasx_xvsubwev_d_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2791 EN**: Closes the current lexical scope or compound statement.
  **L2791 CN**: 结束当前词法作用域或复合语句块。
- **L2792 EN**: Blank line separating nearby declarations or logic blocks.
  **L2792 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2793 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2793 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2794 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2794 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2795 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwev_w_h(__m256i _1, __m256i _2) {`.
  **L2795 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwev_w_h(__m256i _1, __m256i _2) {`。
- **L2796 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwev_w_h((v16i16)_1, (v16i16)_2)`.
  **L2796 CN**: 以 `(__m256i)__builtin_lasx_xvsubwev_w_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2797 EN**: Closes the current lexical scope or compound statement.
  **L2797 CN**: 结束当前词法作用域或复合语句块。
- **L2798 EN**: Blank line separating nearby declarations or logic blocks.
  **L2798 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2799 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2799 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2800 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2800 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2801 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwev_h_b(__m256i _1, __m256i _2) {`.
  **L2801 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwev_h_b(__m256i _1, __m256i _2) {`。
- **L2802 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwev_h_b((v32i8)_1, (v32i8)_2)`.
  **L2802 CN**: 以 `(__m256i)__builtin_lasx_xvsubwev_h_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L2803 EN**: Closes the current lexical scope or compound statement.
  **L2803 CN**: 结束当前词法作用域或复合语句块。
- **L2804 EN**: Blank line separating nearby declarations or logic blocks.
  **L2804 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2805 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2805 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2806 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2806 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2807 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwev_q_du(__m256i _1, __m256i _2) {`.
  **L2807 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwev_q_du(__m256i _1, __m256i _2) {`。
- **L2808 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwev_q_du((v4u64)_1, (v4u64)_2)`.
  **L2808 CN**: 以 `(__m256i)__builtin_lasx_xvsubwev_q_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。

### Lines 2809-2832

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwev_d_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwev_d_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwev_w_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwev_w_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwev_h_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwev_h_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_q_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_q_d((v4i64)_1, (v4i64)_2);
````
- **L2809 EN**: Closes the current lexical scope or compound statement.
  **L2809 CN**: 结束当前词法作用域或复合语句块。
- **L2810 EN**: Blank line separating nearby declarations or logic blocks.
  **L2810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2811 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2811 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2812 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2812 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2813 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwev_d_wu(__m256i _1, __m256i _2) {`.
  **L2813 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwev_d_wu(__m256i _1, __m256i _2) {`。
- **L2814 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwev_d_wu((v8u32)_1, (v8u32)_2)`.
  **L2814 CN**: 以 `(__m256i)__builtin_lasx_xvsubwev_d_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2815 EN**: Closes the current lexical scope or compound statement.
  **L2815 CN**: 结束当前词法作用域或复合语句块。
- **L2816 EN**: Blank line separating nearby declarations or logic blocks.
  **L2816 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2817 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2817 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2818 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2818 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2819 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwev_w_hu(__m256i _1, __m256i _2) {`.
  **L2819 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwev_w_hu(__m256i _1, __m256i _2) {`。
- **L2820 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwev_w_hu((v16u16)_1, (v16u16)_2)`.
  **L2820 CN**: 以 `(__m256i)__builtin_lasx_xvsubwev_w_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2821 EN**: Closes the current lexical scope or compound statement.
  **L2821 CN**: 结束当前词法作用域或复合语句块。
- **L2822 EN**: Blank line separating nearby declarations or logic blocks.
  **L2822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2823 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2823 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2824 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2824 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2825 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwev_h_bu(__m256i _1, __m256i _2) {`.
  **L2825 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwev_h_bu(__m256i _1, __m256i _2) {`。
- **L2826 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwev_h_bu((v32u8)_1, (v32u8)_2)`.
  **L2826 CN**: 以 `(__m256i)__builtin_lasx_xvsubwev_h_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L2827 EN**: Closes the current lexical scope or compound statement.
  **L2827 CN**: 结束当前词法作用域或复合语句块。
- **L2828 EN**: Blank line separating nearby declarations or logic blocks.
  **L2828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2829 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2829 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2830 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2830 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2831 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_q_d(__m256i _1, __m256i _2) {`.
  **L2831 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_q_d(__m256i _1, __m256i _2) {`。
- **L2832 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_q_d((v4i64)_1, (v4i64)_2)`.
  **L2832 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_q_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。

### Lines 2833-2856

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_d_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_d_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_w_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_w_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_h_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_h_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_q_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_q_du((v4u64)_1, (v4u64)_2);
````
- **L2833 EN**: Closes the current lexical scope or compound statement.
  **L2833 CN**: 结束当前词法作用域或复合语句块。
- **L2834 EN**: Blank line separating nearby declarations or logic blocks.
  **L2834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2835 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2835 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2836 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2836 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2837 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_d_w(__m256i _1, __m256i _2) {`.
  **L2837 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_d_w(__m256i _1, __m256i _2) {`。
- **L2838 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_d_w((v8i32)_1, (v8i32)_2)`.
  **L2838 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_d_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2839 EN**: Closes the current lexical scope or compound statement.
  **L2839 CN**: 结束当前词法作用域或复合语句块。
- **L2840 EN**: Blank line separating nearby declarations or logic blocks.
  **L2840 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2841 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2841 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2842 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2842 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2843 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_w_h(__m256i _1, __m256i _2) {`.
  **L2843 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_w_h(__m256i _1, __m256i _2) {`。
- **L2844 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_w_h((v16i16)_1, (v16i16)_2)`.
  **L2844 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_w_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2845 EN**: Closes the current lexical scope or compound statement.
  **L2845 CN**: 结束当前词法作用域或复合语句块。
- **L2846 EN**: Blank line separating nearby declarations or logic blocks.
  **L2846 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2847 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2847 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2848 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2848 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2849 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_h_b(__m256i _1, __m256i _2) {`.
  **L2849 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_h_b(__m256i _1, __m256i _2) {`。
- **L2850 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_h_b((v32i8)_1, (v32i8)_2)`.
  **L2850 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_h_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L2851 EN**: Closes the current lexical scope or compound statement.
  **L2851 CN**: 结束当前词法作用域或复合语句块。
- **L2852 EN**: Blank line separating nearby declarations or logic blocks.
  **L2852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2853 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2853 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2854 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2854 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2855 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_q_du(__m256i _1, __m256i _2) {`.
  **L2855 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_q_du(__m256i _1, __m256i _2) {`。
- **L2856 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_q_du((v4u64)_1, (v4u64)_2)`.
  **L2856 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_q_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。

### Lines 2857-2880

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_d_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_d_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_w_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_w_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_h_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_h_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_q_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_q_d((v4i64)_1, (v4i64)_2);
````
- **L2857 EN**: Closes the current lexical scope or compound statement.
  **L2857 CN**: 结束当前词法作用域或复合语句块。
- **L2858 EN**: Blank line separating nearby declarations or logic blocks.
  **L2858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2859 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2859 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2860 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2860 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2861 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_d_wu(__m256i _1, __m256i _2) {`.
  **L2861 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_d_wu(__m256i _1, __m256i _2) {`。
- **L2862 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_d_wu((v8u32)_1, (v8u32)_2)`.
  **L2862 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_d_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2863 EN**: Closes the current lexical scope or compound statement.
  **L2863 CN**: 结束当前词法作用域或复合语句块。
- **L2864 EN**: Blank line separating nearby declarations or logic blocks.
  **L2864 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2865 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2865 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2866 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2866 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2867 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_w_hu(__m256i _1, __m256i _2) {`.
  **L2867 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_w_hu(__m256i _1, __m256i _2) {`。
- **L2868 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_w_hu((v16u16)_1, (v16u16)_2)`.
  **L2868 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_w_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2869 EN**: Closes the current lexical scope or compound statement.
  **L2869 CN**: 结束当前词法作用域或复合语句块。
- **L2870 EN**: Blank line separating nearby declarations or logic blocks.
  **L2870 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2871 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2871 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2872 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2872 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2873 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_h_bu(__m256i _1, __m256i _2) {`.
  **L2873 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_h_bu(__m256i _1, __m256i _2) {`。
- **L2874 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_h_bu((v32u8)_1, (v32u8)_2)`.
  **L2874 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_h_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L2875 EN**: Closes the current lexical scope or compound statement.
  **L2875 CN**: 结束当前词法作用域或复合语句块。
- **L2876 EN**: Blank line separating nearby declarations or logic blocks.
  **L2876 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2877 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2877 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2878 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2878 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2879 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_q_d(__m256i _1, __m256i _2) {`.
  **L2879 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_q_d(__m256i _1, __m256i _2) {`。
- **L2880 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_q_d((v4i64)_1, (v4i64)_2)`.
  **L2880 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_q_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。

### Lines 2881-2904

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_d_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_d_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_w_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_w_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_h_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_h_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_q_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_q_du((v4u64)_1, (v4u64)_2);
````
- **L2881 EN**: Closes the current lexical scope or compound statement.
  **L2881 CN**: 结束当前词法作用域或复合语句块。
- **L2882 EN**: Blank line separating nearby declarations or logic blocks.
  **L2882 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2883 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2883 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2884 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2884 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2885 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_d_w(__m256i _1, __m256i _2) {`.
  **L2885 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_d_w(__m256i _1, __m256i _2) {`。
- **L2886 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_d_w((v8i32)_1, (v8i32)_2)`.
  **L2886 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_d_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2887 EN**: Closes the current lexical scope or compound statement.
  **L2887 CN**: 结束当前词法作用域或复合语句块。
- **L2888 EN**: Blank line separating nearby declarations or logic blocks.
  **L2888 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2889 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2889 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2890 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2890 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2891 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_w_h(__m256i _1, __m256i _2) {`.
  **L2891 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_w_h(__m256i _1, __m256i _2) {`。
- **L2892 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_w_h((v16i16)_1, (v16i16)_2)`.
  **L2892 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_w_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2893 EN**: Closes the current lexical scope or compound statement.
  **L2893 CN**: 结束当前词法作用域或复合语句块。
- **L2894 EN**: Blank line separating nearby declarations or logic blocks.
  **L2894 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2895 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2895 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2896 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2896 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2897 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_h_b(__m256i _1, __m256i _2) {`.
  **L2897 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_h_b(__m256i _1, __m256i _2) {`。
- **L2898 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_h_b((v32i8)_1, (v32i8)_2)`.
  **L2898 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_h_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L2899 EN**: Closes the current lexical scope or compound statement.
  **L2899 CN**: 结束当前词法作用域或复合语句块。
- **L2900 EN**: Blank line separating nearby declarations or logic blocks.
  **L2900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2901 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2901 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2902 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2902 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2903 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_q_du(__m256i _1, __m256i _2) {`.
  **L2903 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_q_du(__m256i _1, __m256i _2) {`。
- **L2904 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_q_du((v4u64)_1, (v4u64)_2)`.
  **L2904 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_q_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。

### Lines 2905-2928

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_d_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_d_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_w_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_w_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_h_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_h_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwod_q_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwod_q_d((v4i64)_1, (v4i64)_2);
````
- **L2905 EN**: Closes the current lexical scope or compound statement.
  **L2905 CN**: 结束当前词法作用域或复合语句块。
- **L2906 EN**: Blank line separating nearby declarations or logic blocks.
  **L2906 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2907 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2907 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2908 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2908 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2909 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_d_wu(__m256i _1, __m256i _2) {`.
  **L2909 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_d_wu(__m256i _1, __m256i _2) {`。
- **L2910 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_d_wu((v8u32)_1, (v8u32)_2)`.
  **L2910 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_d_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2911 EN**: Closes the current lexical scope or compound statement.
  **L2911 CN**: 结束当前词法作用域或复合语句块。
- **L2912 EN**: Blank line separating nearby declarations or logic blocks.
  **L2912 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2913 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2913 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2914 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2914 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2915 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_w_hu(__m256i _1, __m256i _2) {`.
  **L2915 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_w_hu(__m256i _1, __m256i _2) {`。
- **L2916 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_w_hu((v16u16)_1, (v16u16)_2)`.
  **L2916 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_w_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2917 EN**: Closes the current lexical scope or compound statement.
  **L2917 CN**: 结束当前词法作用域或复合语句块。
- **L2918 EN**: Blank line separating nearby declarations or logic blocks.
  **L2918 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2919 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2919 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2920 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2920 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2921 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_h_bu(__m256i _1, __m256i _2) {`.
  **L2921 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_h_bu(__m256i _1, __m256i _2) {`。
- **L2922 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_h_bu((v32u8)_1, (v32u8)_2)`.
  **L2922 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_h_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L2923 EN**: Closes the current lexical scope or compound statement.
  **L2923 CN**: 结束当前词法作用域或复合语句块。
- **L2924 EN**: Blank line separating nearby declarations or logic blocks.
  **L2924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2925 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2925 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2926 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2926 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2927 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwod_q_d(__m256i _1, __m256i _2) {`.
  **L2927 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwod_q_d(__m256i _1, __m256i _2) {`。
- **L2928 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwod_q_d((v4i64)_1, (v4i64)_2)`.
  **L2928 CN**: 以 `(__m256i)__builtin_lasx_xvsubwod_q_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。

### Lines 2929-2952

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwod_d_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwod_d_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwod_w_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwod_w_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwod_h_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwod_h_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwod_q_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwod_q_du((v4u64)_1, (v4u64)_2);
````
- **L2929 EN**: Closes the current lexical scope or compound statement.
  **L2929 CN**: 结束当前词法作用域或复合语句块。
- **L2930 EN**: Blank line separating nearby declarations or logic blocks.
  **L2930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2931 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2931 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2932 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2932 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2933 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwod_d_w(__m256i _1, __m256i _2) {`.
  **L2933 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwod_d_w(__m256i _1, __m256i _2) {`。
- **L2934 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwod_d_w((v8i32)_1, (v8i32)_2)`.
  **L2934 CN**: 以 `(__m256i)__builtin_lasx_xvsubwod_d_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2935 EN**: Closes the current lexical scope or compound statement.
  **L2935 CN**: 结束当前词法作用域或复合语句块。
- **L2936 EN**: Blank line separating nearby declarations or logic blocks.
  **L2936 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2937 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2937 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2938 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2938 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2939 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwod_w_h(__m256i _1, __m256i _2) {`.
  **L2939 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwod_w_h(__m256i _1, __m256i _2) {`。
- **L2940 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwod_w_h((v16i16)_1, (v16i16)_2)`.
  **L2940 CN**: 以 `(__m256i)__builtin_lasx_xvsubwod_w_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2941 EN**: Closes the current lexical scope or compound statement.
  **L2941 CN**: 结束当前词法作用域或复合语句块。
- **L2942 EN**: Blank line separating nearby declarations or logic blocks.
  **L2942 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2943 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2943 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2944 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2944 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2945 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwod_h_b(__m256i _1, __m256i _2) {`.
  **L2945 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwod_h_b(__m256i _1, __m256i _2) {`。
- **L2946 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwod_h_b((v32i8)_1, (v32i8)_2)`.
  **L2946 CN**: 以 `(__m256i)__builtin_lasx_xvsubwod_h_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L2947 EN**: Closes the current lexical scope or compound statement.
  **L2947 CN**: 结束当前词法作用域或复合语句块。
- **L2948 EN**: Blank line separating nearby declarations or logic blocks.
  **L2948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2949 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2949 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2950 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2950 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2951 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwod_q_du(__m256i _1, __m256i _2) {`.
  **L2951 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwod_q_du(__m256i _1, __m256i _2) {`。
- **L2952 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwod_q_du((v4u64)_1, (v4u64)_2)`.
  **L2952 CN**: 以 `(__m256i)__builtin_lasx_xvsubwod_q_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。

### Lines 2953-2976

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwod_d_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwod_d_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwod_w_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwod_w_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvsubwod_h_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsubwod_h_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_q_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_q_d((v4i64)_1, (v4i64)_2);
````
- **L2953 EN**: Closes the current lexical scope or compound statement.
  **L2953 CN**: 结束当前词法作用域或复合语句块。
- **L2954 EN**: Blank line separating nearby declarations or logic blocks.
  **L2954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2955 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2955 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2956 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2956 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2957 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwod_d_wu(__m256i _1, __m256i _2) {`.
  **L2957 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwod_d_wu(__m256i _1, __m256i _2) {`。
- **L2958 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwod_d_wu((v8u32)_1, (v8u32)_2)`.
  **L2958 CN**: 以 `(__m256i)__builtin_lasx_xvsubwod_d_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L2959 EN**: Closes the current lexical scope or compound statement.
  **L2959 CN**: 结束当前词法作用域或复合语句块。
- **L2960 EN**: Blank line separating nearby declarations or logic blocks.
  **L2960 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2961 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2961 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2962 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2962 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2963 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwod_w_hu(__m256i _1, __m256i _2) {`.
  **L2963 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwod_w_hu(__m256i _1, __m256i _2) {`。
- **L2964 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwod_w_hu((v16u16)_1, (v16u16)_2)`.
  **L2964 CN**: 以 `(__m256i)__builtin_lasx_xvsubwod_w_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L2965 EN**: Closes the current lexical scope or compound statement.
  **L2965 CN**: 结束当前词法作用域或复合语句块。
- **L2966 EN**: Blank line separating nearby declarations or logic blocks.
  **L2966 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2967 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2967 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2968 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2968 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2969 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsubwod_h_bu(__m256i _1, __m256i _2) {`.
  **L2969 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsubwod_h_bu(__m256i _1, __m256i _2) {`。
- **L2970 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsubwod_h_bu((v32u8)_1, (v32u8)_2)`.
  **L2970 CN**: 以 `(__m256i)__builtin_lasx_xvsubwod_h_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L2971 EN**: Closes the current lexical scope or compound statement.
  **L2971 CN**: 结束当前词法作用域或复合语句块。
- **L2972 EN**: Blank line separating nearby declarations or logic blocks.
  **L2972 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2973 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2973 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2974 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2974 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2975 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_q_d(__m256i _1, __m256i _2) {`.
  **L2975 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_q_d(__m256i _1, __m256i _2) {`。
- **L2976 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_q_d((v4i64)_1, (v4i64)_2)`.
  **L2976 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_q_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。

### Lines 2977-3000

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_d_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_d_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_w_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_w_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_h_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_h_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_q_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_q_du((v4u64)_1, (v4u64)_2);
````
- **L2977 EN**: Closes the current lexical scope or compound statement.
  **L2977 CN**: 结束当前词法作用域或复合语句块。
- **L2978 EN**: Blank line separating nearby declarations or logic blocks.
  **L2978 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2979 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2979 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2980 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2980 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2981 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_d_w(__m256i _1, __m256i _2) {`.
  **L2981 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_d_w(__m256i _1, __m256i _2) {`。
- **L2982 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_d_w((v8i32)_1, (v8i32)_2)`.
  **L2982 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_d_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L2983 EN**: Closes the current lexical scope or compound statement.
  **L2983 CN**: 结束当前词法作用域或复合语句块。
- **L2984 EN**: Blank line separating nearby declarations or logic blocks.
  **L2984 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2985 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2985 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2986 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2986 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2987 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_w_h(__m256i _1, __m256i _2) {`.
  **L2987 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_w_h(__m256i _1, __m256i _2) {`。
- **L2988 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_w_h((v16i16)_1, (v16i16)_2)`.
  **L2988 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_w_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L2989 EN**: Closes the current lexical scope or compound statement.
  **L2989 CN**: 结束当前词法作用域或复合语句块。
- **L2990 EN**: Blank line separating nearby declarations or logic blocks.
  **L2990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2991 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2991 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2992 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2992 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2993 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_h_b(__m256i _1, __m256i _2) {`.
  **L2993 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_h_b(__m256i _1, __m256i _2) {`。
- **L2994 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_h_b((v32i8)_1, (v32i8)_2)`.
  **L2994 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_h_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L2995 EN**: Closes the current lexical scope or compound statement.
  **L2995 CN**: 结束当前词法作用域或复合语句块。
- **L2996 EN**: Blank line separating nearby declarations or logic blocks.
  **L2996 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2997 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2997 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2998 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L2998 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L2999 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_q_du(__m256i _1, __m256i _2) {`.
  **L2999 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_q_du(__m256i _1, __m256i _2) {`。
- **L3000 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_q_du((v4u64)_1, (v4u64)_2)`.
  **L3000 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_q_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。

### Lines 3001-3024

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_d_wu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_d_wu((v8u32)_1, (v8u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_w_hu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_w_hu((v16u16)_1, (v16u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_h_bu(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_h_bu((v32u8)_1, (v32u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_d_wu_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_d_wu_w((v8u32)_1, (v8i32)_2);
````
- **L3001 EN**: Closes the current lexical scope or compound statement.
  **L3001 CN**: 结束当前词法作用域或复合语句块。
- **L3002 EN**: Blank line separating nearby declarations or logic blocks.
  **L3002 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3003 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3003 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3004 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3004 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3005 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_d_wu(__m256i _1, __m256i _2) {`.
  **L3005 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_d_wu(__m256i _1, __m256i _2) {`。
- **L3006 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_d_wu((v8u32)_1, (v8u32)_2)`.
  **L3006 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_d_wu((v8u32)_1, (v8u32)_2)` 从当前函数返回。
- **L3007 EN**: Closes the current lexical scope or compound statement.
  **L3007 CN**: 结束当前词法作用域或复合语句块。
- **L3008 EN**: Blank line separating nearby declarations or logic blocks.
  **L3008 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3009 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3009 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3010 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3010 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3011 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_w_hu(__m256i _1, __m256i _2) {`.
  **L3011 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_w_hu(__m256i _1, __m256i _2) {`。
- **L3012 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_w_hu((v16u16)_1, (v16u16)_2)`.
  **L3012 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_w_hu((v16u16)_1, (v16u16)_2)` 从当前函数返回。
- **L3013 EN**: Closes the current lexical scope or compound statement.
  **L3013 CN**: 结束当前词法作用域或复合语句块。
- **L3014 EN**: Blank line separating nearby declarations or logic blocks.
  **L3014 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3015 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3015 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3016 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3016 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3017 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_h_bu(__m256i _1, __m256i _2) {`.
  **L3017 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_h_bu(__m256i _1, __m256i _2) {`。
- **L3018 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_h_bu((v32u8)_1, (v32u8)_2)`.
  **L3018 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_h_bu((v32u8)_1, (v32u8)_2)` 从当前函数返回。
- **L3019 EN**: Closes the current lexical scope or compound statement.
  **L3019 CN**: 结束当前词法作用域或复合语句块。
- **L3020 EN**: Blank line separating nearby declarations or logic blocks.
  **L3020 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3021 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3021 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3022 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3022 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3023 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_d_wu_w(__m256i _1, __m256i _2) {`.
  **L3023 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_d_wu_w(__m256i _1, __m256i _2) {`。
- **L3024 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_d_wu_w((v8u32)_1, (v8i32)_2)`.
  **L3024 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_d_wu_w((v8u32)_1, (v8i32)_2)` 从当前函数返回。

### Lines 3025-3048

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_w_hu_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_w_hu_h((v16u16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_h_bu_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_h_bu_b((v32u8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_d_wu_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_d_wu_w((v8u32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_w_hu_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_w_hu_h((v16u16)_1, (v16i16)_2);
````
- **L3025 EN**: Closes the current lexical scope or compound statement.
  **L3025 CN**: 结束当前词法作用域或复合语句块。
- **L3026 EN**: Blank line separating nearby declarations or logic blocks.
  **L3026 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3027 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3027 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3028 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3028 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3029 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_w_hu_h(__m256i _1, __m256i _2) {`.
  **L3029 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_w_hu_h(__m256i _1, __m256i _2) {`。
- **L3030 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_w_hu_h((v16u16)_1, (v16i16)_2)`.
  **L3030 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_w_hu_h((v16u16)_1, (v16i16)_2)` 从当前函数返回。
- **L3031 EN**: Closes the current lexical scope or compound statement.
  **L3031 CN**: 结束当前词法作用域或复合语句块。
- **L3032 EN**: Blank line separating nearby declarations or logic blocks.
  **L3032 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3033 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3033 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3034 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3034 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3035 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_h_bu_b(__m256i _1, __m256i _2) {`.
  **L3035 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_h_bu_b(__m256i _1, __m256i _2) {`。
- **L3036 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_h_bu_b((v32u8)_1, (v32i8)_2)`.
  **L3036 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_h_bu_b((v32u8)_1, (v32i8)_2)` 从当前函数返回。
- **L3037 EN**: Closes the current lexical scope or compound statement.
  **L3037 CN**: 结束当前词法作用域或复合语句块。
- **L3038 EN**: Blank line separating nearby declarations or logic blocks.
  **L3038 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3039 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3039 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3040 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3040 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3041 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_d_wu_w(__m256i _1, __m256i _2) {`.
  **L3041 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_d_wu_w(__m256i _1, __m256i _2) {`。
- **L3042 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_d_wu_w((v8u32)_1, (v8i32)_2)`.
  **L3042 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_d_wu_w((v8u32)_1, (v8i32)_2)` 从当前函数返回。
- **L3043 EN**: Closes the current lexical scope or compound statement.
  **L3043 CN**: 结束当前词法作用域或复合语句块。
- **L3044 EN**: Blank line separating nearby declarations or logic blocks.
  **L3044 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3045 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3045 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3046 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3046 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3047 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_w_hu_h(__m256i _1, __m256i _2) {`.
  **L3047 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_w_hu_h(__m256i _1, __m256i _2) {`。
- **L3048 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_w_hu_h((v16u16)_1, (v16i16)_2)`.
  **L3048 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_w_hu_h((v16u16)_1, (v16i16)_2)` 从当前函数返回。

### Lines 3049-3072

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_h_bu_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_h_bu_b((v32u8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_d_wu_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_d_wu_w((v8u32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_w_hu_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_w_hu_h((v16u16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_h_bu_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_h_bu_b((v32u8)_1, (v32i8)_2);
````
- **L3049 EN**: Closes the current lexical scope or compound statement.
  **L3049 CN**: 结束当前词法作用域或复合语句块。
- **L3050 EN**: Blank line separating nearby declarations or logic blocks.
  **L3050 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3051 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3051 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3052 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3052 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3053 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_h_bu_b(__m256i _1, __m256i _2) {`.
  **L3053 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_h_bu_b(__m256i _1, __m256i _2) {`。
- **L3054 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_h_bu_b((v32u8)_1, (v32i8)_2)`.
  **L3054 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_h_bu_b((v32u8)_1, (v32i8)_2)` 从当前函数返回。
- **L3055 EN**: Closes the current lexical scope or compound statement.
  **L3055 CN**: 结束当前词法作用域或复合语句块。
- **L3056 EN**: Blank line separating nearby declarations or logic blocks.
  **L3056 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3057 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3057 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3058 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3058 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3059 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_d_wu_w(__m256i _1, __m256i _2) {`.
  **L3059 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_d_wu_w(__m256i _1, __m256i _2) {`。
- **L3060 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_d_wu_w((v8u32)_1, (v8i32)_2)`.
  **L3060 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_d_wu_w((v8u32)_1, (v8i32)_2)` 从当前函数返回。
- **L3061 EN**: Closes the current lexical scope or compound statement.
  **L3061 CN**: 结束当前词法作用域或复合语句块。
- **L3062 EN**: Blank line separating nearby declarations or logic blocks.
  **L3062 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3063 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3063 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3064 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3064 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3065 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_w_hu_h(__m256i _1, __m256i _2) {`.
  **L3065 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_w_hu_h(__m256i _1, __m256i _2) {`。
- **L3066 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_w_hu_h((v16u16)_1, (v16i16)_2)`.
  **L3066 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_w_hu_h((v16u16)_1, (v16i16)_2)` 从当前函数返回。
- **L3067 EN**: Closes the current lexical scope or compound statement.
  **L3067 CN**: 结束当前词法作用域或复合语句块。
- **L3068 EN**: Blank line separating nearby declarations or logic blocks.
  **L3068 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3069 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3069 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3070 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3070 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3071 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_h_bu_b(__m256i _1, __m256i _2) {`.
  **L3071 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_h_bu_b(__m256i _1, __m256i _2) {`。
- **L3072 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_h_bu_b((v32u8)_1, (v32i8)_2)`.
  **L3072 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_h_bu_b((v32u8)_1, (v32i8)_2)` 从当前函数返回。

### Lines 3073-3096

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_d_wu_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_d_wu_w((v8u32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_w_hu_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_w_hu_h((v16u16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwod_h_bu_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_h_bu_b((v32u8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhaddw_q_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhaddw_q_d((v4i64)_1, (v4i64)_2);
````
- **L3073 EN**: Closes the current lexical scope or compound statement.
  **L3073 CN**: 结束当前词法作用域或复合语句块。
- **L3074 EN**: Blank line separating nearby declarations or logic blocks.
  **L3074 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3075 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3075 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3076 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3076 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3077 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_d_wu_w(__m256i _1, __m256i _2) {`.
  **L3077 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_d_wu_w(__m256i _1, __m256i _2) {`。
- **L3078 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_d_wu_w((v8u32)_1, (v8i32)_2)`.
  **L3078 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_d_wu_w((v8u32)_1, (v8i32)_2)` 从当前函数返回。
- **L3079 EN**: Closes the current lexical scope or compound statement.
  **L3079 CN**: 结束当前词法作用域或复合语句块。
- **L3080 EN**: Blank line separating nearby declarations or logic blocks.
  **L3080 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3081 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3081 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3082 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3082 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3083 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_w_hu_h(__m256i _1, __m256i _2) {`.
  **L3083 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_w_hu_h(__m256i _1, __m256i _2) {`。
- **L3084 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_w_hu_h((v16u16)_1, (v16i16)_2)`.
  **L3084 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_w_hu_h((v16u16)_1, (v16i16)_2)` 从当前函数返回。
- **L3085 EN**: Closes the current lexical scope or compound statement.
  **L3085 CN**: 结束当前词法作用域或复合语句块。
- **L3086 EN**: Blank line separating nearby declarations or logic blocks.
  **L3086 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3087 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3087 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3088 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3088 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3089 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_h_bu_b(__m256i _1, __m256i _2) {`.
  **L3089 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_h_bu_b(__m256i _1, __m256i _2) {`。
- **L3090 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_h_bu_b((v32u8)_1, (v32i8)_2)`.
  **L3090 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_h_bu_b((v32u8)_1, (v32i8)_2)` 从当前函数返回。
- **L3091 EN**: Closes the current lexical scope or compound statement.
  **L3091 CN**: 结束当前词法作用域或复合语句块。
- **L3092 EN**: Blank line separating nearby declarations or logic blocks.
  **L3092 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3093 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3093 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3094 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3094 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3095 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhaddw_q_d(__m256i _1, __m256i _2) {`.
  **L3095 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhaddw_q_d(__m256i _1, __m256i _2) {`。
- **L3096 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhaddw_q_d((v4i64)_1, (v4i64)_2)`.
  **L3096 CN**: 以 `(__m256i)__builtin_lasx_xvhaddw_q_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。

### Lines 3097-3120

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhaddw_qu_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhaddw_qu_du((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhsubw_q_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhsubw_q_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvhsubw_qu_du(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvhsubw_qu_du((v4u64)_1, (v4u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_q_d(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_q_d((v4i64)_1, (v4i64)_2, (v4i64)_3);
````
- **L3097 EN**: Closes the current lexical scope or compound statement.
  **L3097 CN**: 结束当前词法作用域或复合语句块。
- **L3098 EN**: Blank line separating nearby declarations or logic blocks.
  **L3098 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3099 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3099 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3100 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3100 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3101 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhaddw_qu_du(__m256i _1, __m256i _2) {`.
  **L3101 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhaddw_qu_du(__m256i _1, __m256i _2) {`。
- **L3102 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhaddw_qu_du((v4u64)_1, (v4u64)_2)`.
  **L3102 CN**: 以 `(__m256i)__builtin_lasx_xvhaddw_qu_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L3103 EN**: Closes the current lexical scope or compound statement.
  **L3103 CN**: 结束当前词法作用域或复合语句块。
- **L3104 EN**: Blank line separating nearby declarations or logic blocks.
  **L3104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3105 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3105 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3106 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3106 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3107 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhsubw_q_d(__m256i _1, __m256i _2) {`.
  **L3107 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhsubw_q_d(__m256i _1, __m256i _2) {`。
- **L3108 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhsubw_q_d((v4i64)_1, (v4i64)_2)`.
  **L3108 CN**: 以 `(__m256i)__builtin_lasx_xvhsubw_q_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L3109 EN**: Closes the current lexical scope or compound statement.
  **L3109 CN**: 结束当前词法作用域或复合语句块。
- **L3110 EN**: Blank line separating nearby declarations or logic blocks.
  **L3110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3111 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3111 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3112 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3112 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3113 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvhsubw_qu_du(__m256i _1, __m256i _2) {`.
  **L3113 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvhsubw_qu_du(__m256i _1, __m256i _2) {`。
- **L3114 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvhsubw_qu_du((v4u64)_1, (v4u64)_2)`.
  **L3114 CN**: 以 `(__m256i)__builtin_lasx_xvhsubw_qu_du((v4u64)_1, (v4u64)_2)` 从当前函数返回。
- **L3115 EN**: Closes the current lexical scope or compound statement.
  **L3115 CN**: 结束当前词法作用域或复合语句块。
- **L3116 EN**: Blank line separating nearby declarations or logic blocks.
  **L3116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3117 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3117 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3118 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3118 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3119 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_q_d(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3119 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_q_d(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3120 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_q_d((v4i64)_1, (v4i64)_2, (v4i64)_3)`.
  **L3120 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_q_d((v4i64)_1, (v4i64)_2, (v4i64)_3)` 从当前函数返回。

### Lines 3121-3144

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_d_w(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_d_w((v4i64)_1, (v8i32)_2, (v8i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_w_h(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_w_h((v8i32)_1, (v16i16)_2,
                                               (v16i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_h_b(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_h_b((v16i16)_1, (v32i8)_2,
                                               (v32i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
````
- **L3121 EN**: Closes the current lexical scope or compound statement.
  **L3121 CN**: 结束当前词法作用域或复合语句块。
- **L3122 EN**: Blank line separating nearby declarations or logic blocks.
  **L3122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3123 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3123 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3124 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3124 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_d_w(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_d_w(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3126 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_d_w((v4i64)_1, (v8i32)_2, (v8i32)_3)`.
  **L3126 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_d_w((v4i64)_1, (v8i32)_2, (v8i32)_3)` 从当前函数返回。
- **L3127 EN**: Closes the current lexical scope or compound statement.
  **L3127 CN**: 结束当前词法作用域或复合语句块。
- **L3128 EN**: Blank line separating nearby declarations or logic blocks.
  **L3128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3129 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3129 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3130 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3130 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3131 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_w_h(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3131 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_w_h(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3132 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_w_h((v8i32)_1, (v16i16)_2,`.
  **L3132 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_w_h((v8i32)_1, (v16i16)_2,` 从当前函数返回。
- **L3133 EN**: Executes a call or declaration centered on `statement`.
  **L3133 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3134 EN**: Closes the current lexical scope or compound statement.
  **L3134 CN**: 结束当前词法作用域或复合语句块。
- **L3135 EN**: Blank line separating nearby declarations or logic blocks.
  **L3135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3136 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3136 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3137 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3137 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3138 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_h_b(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3138 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_h_b(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3139 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_h_b((v16i16)_1, (v32i8)_2,`.
  **L3139 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_h_b((v16i16)_1, (v32i8)_2,` 从当前函数返回。
- **L3140 EN**: Executes a call or declaration centered on `statement`.
  **L3140 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3141 EN**: Closes the current lexical scope or compound statement.
  **L3141 CN**: 结束当前词法作用域或复合语句块。
- **L3142 EN**: Blank line separating nearby declarations or logic blocks.
  **L3142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3143 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3143 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3144 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3144 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。

### Lines 3145-3168

````c
    __lasx_xvmaddwev_q_du(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_q_du((v4u64)_1, (v4u64)_2,
                                                (v4u64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_d_wu(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_d_wu((v4u64)_1, (v8u32)_2,
                                                (v8u32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_w_hu(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_w_hu((v8u32)_1, (v16u16)_2,
                                                (v16u16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_h_bu(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_h_bu((v16u16)_1, (v32u8)_2,
                                                (v32u8)_3);
````
- **L3145 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_q_du(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3145 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_q_du(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3146 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_q_du((v4u64)_1, (v4u64)_2,`.
  **L3146 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_q_du((v4u64)_1, (v4u64)_2,` 从当前函数返回。
- **L3147 EN**: Executes a call or declaration centered on `statement`.
  **L3147 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3148 EN**: Closes the current lexical scope or compound statement.
  **L3148 CN**: 结束当前词法作用域或复合语句块。
- **L3149 EN**: Blank line separating nearby declarations or logic blocks.
  **L3149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3150 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3150 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3151 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3151 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3152 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_d_wu(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3152 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_d_wu(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3153 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_d_wu((v4u64)_1, (v8u32)_2,`.
  **L3153 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_d_wu((v4u64)_1, (v8u32)_2,` 从当前函数返回。
- **L3154 EN**: Executes a call or declaration centered on `statement`.
  **L3154 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3155 EN**: Closes the current lexical scope or compound statement.
  **L3155 CN**: 结束当前词法作用域或复合语句块。
- **L3156 EN**: Blank line separating nearby declarations or logic blocks.
  **L3156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3157 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3157 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3158 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3158 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3159 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_w_hu(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3159 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_w_hu(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3160 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_w_hu((v8u32)_1, (v16u16)_2,`.
  **L3160 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_w_hu((v8u32)_1, (v16u16)_2,` 从当前函数返回。
- **L3161 EN**: Executes a call or declaration centered on `statement`.
  **L3161 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3162 EN**: Closes the current lexical scope or compound statement.
  **L3162 CN**: 结束当前词法作用域或复合语句块。
- **L3163 EN**: Blank line separating nearby declarations or logic blocks.
  **L3163 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3164 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3164 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3165 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3165 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3166 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_h_bu(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3166 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_h_bu(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3167 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_h_bu((v16u16)_1, (v32u8)_2,`.
  **L3167 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_h_bu((v16u16)_1, (v32u8)_2,` 从当前函数返回。
- **L3168 EN**: Executes a call or declaration centered on `statement`.
  **L3168 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 3169-3192

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_q_d(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_q_d((v4i64)_1, (v4i64)_2, (v4i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_d_w(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_d_w((v4i64)_1, (v8i32)_2, (v8i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_w_h(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_w_h((v8i32)_1, (v16i16)_2,
                                               (v16i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_h_b(__m256i _1, __m256i _2, __m256i _3) {
````
- **L3169 EN**: Closes the current lexical scope or compound statement.
  **L3169 CN**: 结束当前词法作用域或复合语句块。
- **L3170 EN**: Blank line separating nearby declarations or logic blocks.
  **L3170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3171 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3171 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3172 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3172 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3173 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_q_d(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3173 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_q_d(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3174 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_q_d((v4i64)_1, (v4i64)_2, (v4i64)_3)`.
  **L3174 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_q_d((v4i64)_1, (v4i64)_2, (v4i64)_3)` 从当前函数返回。
- **L3175 EN**: Closes the current lexical scope or compound statement.
  **L3175 CN**: 结束当前词法作用域或复合语句块。
- **L3176 EN**: Blank line separating nearby declarations or logic blocks.
  **L3176 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3177 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3177 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3178 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3178 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3179 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_d_w(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3179 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_d_w(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3180 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_d_w((v4i64)_1, (v8i32)_2, (v8i32)_3)`.
  **L3180 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_d_w((v4i64)_1, (v8i32)_2, (v8i32)_3)` 从当前函数返回。
- **L3181 EN**: Closes the current lexical scope or compound statement.
  **L3181 CN**: 结束当前词法作用域或复合语句块。
- **L3182 EN**: Blank line separating nearby declarations or logic blocks.
  **L3182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3183 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3183 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3184 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3184 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3185 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_w_h(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3185 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_w_h(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3186 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_w_h((v8i32)_1, (v16i16)_2,`.
  **L3186 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_w_h((v8i32)_1, (v16i16)_2,` 从当前函数返回。
- **L3187 EN**: Executes a call or declaration centered on `statement`.
  **L3187 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3188 EN**: Closes the current lexical scope or compound statement.
  **L3188 CN**: 结束当前词法作用域或复合语句块。
- **L3189 EN**: Blank line separating nearby declarations or logic blocks.
  **L3189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3190 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3190 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3191 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3191 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3192 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_h_b(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3192 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_h_b(__m256i _1, __m256i _2, __m256i _3) {`。

### Lines 3193-3216

````c
  return (__m256i)__builtin_lasx_xvmaddwod_h_b((v16i16)_1, (v32i8)_2,
                                               (v32i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_q_du(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_q_du((v4u64)_1, (v4u64)_2,
                                                (v4u64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_d_wu(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_d_wu((v4u64)_1, (v8u32)_2,
                                                (v8u32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_w_hu(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_w_hu((v8u32)_1, (v16u16)_2,
                                                (v16u16)_3);
}
````
- **L3193 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_h_b((v16i16)_1, (v32i8)_2,`.
  **L3193 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_h_b((v16i16)_1, (v32i8)_2,` 从当前函数返回。
- **L3194 EN**: Executes a call or declaration centered on `statement`.
  **L3194 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3195 EN**: Closes the current lexical scope or compound statement.
  **L3195 CN**: 结束当前词法作用域或复合语句块。
- **L3196 EN**: Blank line separating nearby declarations or logic blocks.
  **L3196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3197 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3197 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3198 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3198 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3199 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_q_du(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3199 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_q_du(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3200 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_q_du((v4u64)_1, (v4u64)_2,`.
  **L3200 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_q_du((v4u64)_1, (v4u64)_2,` 从当前函数返回。
- **L3201 EN**: Executes a call or declaration centered on `statement`.
  **L3201 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3202 EN**: Closes the current lexical scope or compound statement.
  **L3202 CN**: 结束当前词法作用域或复合语句块。
- **L3203 EN**: Blank line separating nearby declarations or logic blocks.
  **L3203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3204 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3204 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3205 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3205 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3206 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_d_wu(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3206 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_d_wu(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3207 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_d_wu((v4u64)_1, (v8u32)_2,`.
  **L3207 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_d_wu((v4u64)_1, (v8u32)_2,` 从当前函数返回。
- **L3208 EN**: Executes a call or declaration centered on `statement`.
  **L3208 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3209 EN**: Closes the current lexical scope or compound statement.
  **L3209 CN**: 结束当前词法作用域或复合语句块。
- **L3210 EN**: Blank line separating nearby declarations or logic blocks.
  **L3210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3211 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3211 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3212 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3212 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3213 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_w_hu(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3213 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_w_hu(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3214 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_w_hu((v8u32)_1, (v16u16)_2,`.
  **L3214 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_w_hu((v8u32)_1, (v16u16)_2,` 从当前函数返回。
- **L3215 EN**: Executes a call or declaration centered on `statement`.
  **L3215 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3216 EN**: Closes the current lexical scope or compound statement.
  **L3216 CN**: 结束当前词法作用域或复合语句块。

### Lines 3217-3240

````c

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_h_bu(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_h_bu((v16u16)_1, (v32u8)_2,
                                                (v32u8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_q_du_d(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_q_du_d((v4i64)_1, (v4u64)_2,
                                                  (v4i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_d_wu_w(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_d_wu_w((v4i64)_1, (v8u32)_2,
                                                  (v8i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
````
- **L3217 EN**: Blank line separating nearby declarations or logic blocks.
  **L3217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3218 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3218 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3219 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3219 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3220 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_h_bu(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3220 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_h_bu(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3221 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_h_bu((v16u16)_1, (v32u8)_2,`.
  **L3221 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_h_bu((v16u16)_1, (v32u8)_2,` 从当前函数返回。
- **L3222 EN**: Executes a call or declaration centered on `statement`.
  **L3222 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3223 EN**: Closes the current lexical scope or compound statement.
  **L3223 CN**: 结束当前词法作用域或复合语句块。
- **L3224 EN**: Blank line separating nearby declarations or logic blocks.
  **L3224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3225 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3225 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3226 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3226 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_q_du_d(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_q_du_d(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3228 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_q_du_d((v4i64)_1, (v4u64)_2,`.
  **L3228 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_q_du_d((v4i64)_1, (v4u64)_2,` 从当前函数返回。
- **L3229 EN**: Executes a call or declaration centered on `statement`.
  **L3229 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3230 EN**: Closes the current lexical scope or compound statement.
  **L3230 CN**: 结束当前词法作用域或复合语句块。
- **L3231 EN**: Blank line separating nearby declarations or logic blocks.
  **L3231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3232 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3232 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3233 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3233 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3234 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_d_wu_w(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3234 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_d_wu_w(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3235 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_d_wu_w((v4i64)_1, (v8u32)_2,`.
  **L3235 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_d_wu_w((v4i64)_1, (v8u32)_2,` 从当前函数返回。
- **L3236 EN**: Executes a call or declaration centered on `statement`.
  **L3236 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3237 EN**: Closes the current lexical scope or compound statement.
  **L3237 CN**: 结束当前词法作用域或复合语句块。
- **L3238 EN**: Blank line separating nearby declarations or logic blocks.
  **L3238 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3239 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3239 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3240 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3240 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。

### Lines 3241-3264

````c
    __lasx_xvmaddwev_w_hu_h(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_w_hu_h((v8i32)_1, (v16u16)_2,
                                                  (v16i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwev_h_bu_b(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwev_h_bu_b((v16i16)_1, (v32u8)_2,
                                                  (v32i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_q_du_d(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_q_du_d((v4i64)_1, (v4u64)_2,
                                                  (v4i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_d_wu_w(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_d_wu_w((v4i64)_1, (v8u32)_2,
                                                  (v8i32)_3);
````
- **L3241 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_w_hu_h(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3241 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_w_hu_h(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3242 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_w_hu_h((v8i32)_1, (v16u16)_2,`.
  **L3242 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_w_hu_h((v8i32)_1, (v16u16)_2,` 从当前函数返回。
- **L3243 EN**: Executes a call or declaration centered on `statement`.
  **L3243 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3244 EN**: Closes the current lexical scope or compound statement.
  **L3244 CN**: 结束当前词法作用域或复合语句块。
- **L3245 EN**: Blank line separating nearby declarations or logic blocks.
  **L3245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3246 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3246 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3247 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3247 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3248 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwev_h_bu_b(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3248 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwev_h_bu_b(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3249 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwev_h_bu_b((v16i16)_1, (v32u8)_2,`.
  **L3249 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwev_h_bu_b((v16i16)_1, (v32u8)_2,` 从当前函数返回。
- **L3250 EN**: Executes a call or declaration centered on `statement`.
  **L3250 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3251 EN**: Closes the current lexical scope or compound statement.
  **L3251 CN**: 结束当前词法作用域或复合语句块。
- **L3252 EN**: Blank line separating nearby declarations or logic blocks.
  **L3252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3253 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3253 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3254 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3254 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3255 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_q_du_d(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3255 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_q_du_d(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3256 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_q_du_d((v4i64)_1, (v4u64)_2,`.
  **L3256 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_q_du_d((v4i64)_1, (v4u64)_2,` 从当前函数返回。
- **L3257 EN**: Executes a call or declaration centered on `statement`.
  **L3257 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3258 EN**: Closes the current lexical scope or compound statement.
  **L3258 CN**: 结束当前词法作用域或复合语句块。
- **L3259 EN**: Blank line separating nearby declarations or logic blocks.
  **L3259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3260 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3260 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3261 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3261 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3262 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_d_wu_w(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3262 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_d_wu_w(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3263 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_d_wu_w((v4i64)_1, (v8u32)_2,`.
  **L3263 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_d_wu_w((v4i64)_1, (v8u32)_2,` 从当前函数返回。
- **L3264 EN**: Executes a call or declaration centered on `statement`.
  **L3264 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 3265-3288

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_w_hu_h(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_w_hu_h((v8i32)_1, (v16u16)_2,
                                                  (v16i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmaddwod_h_bu_b(__m256i _1, __m256i _2, __m256i _3) {
  return (__m256i)__builtin_lasx_xvmaddwod_h_bu_b((v16i16)_1, (v32u8)_2,
                                                  (v32i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvrotr_b(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvrotr_b((v32i8)_1, (v32i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
````
- **L3265 EN**: Closes the current lexical scope or compound statement.
  **L3265 CN**: 结束当前词法作用域或复合语句块。
- **L3266 EN**: Blank line separating nearby declarations or logic blocks.
  **L3266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3267 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3267 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3268 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3268 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3269 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_w_hu_h(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3269 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_w_hu_h(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3270 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_w_hu_h((v8i32)_1, (v16u16)_2,`.
  **L3270 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_w_hu_h((v8i32)_1, (v16u16)_2,` 从当前函数返回。
- **L3271 EN**: Executes a call or declaration centered on `statement`.
  **L3271 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3272 EN**: Closes the current lexical scope or compound statement.
  **L3272 CN**: 结束当前词法作用域或复合语句块。
- **L3273 EN**: Blank line separating nearby declarations or logic blocks.
  **L3273 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3274 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3274 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3275 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3275 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3276 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmaddwod_h_bu_b(__m256i _1, __m256i _2, __m256i _3) {`.
  **L3276 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmaddwod_h_bu_b(__m256i _1, __m256i _2, __m256i _3) {`。
- **L3277 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmaddwod_h_bu_b((v16i16)_1, (v32u8)_2,`.
  **L3277 CN**: 以 `(__m256i)__builtin_lasx_xvmaddwod_h_bu_b((v16i16)_1, (v32u8)_2,` 从当前函数返回。
- **L3278 EN**: Executes a call or declaration centered on `statement`.
  **L3278 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3279 EN**: Closes the current lexical scope or compound statement.
  **L3279 CN**: 结束当前词法作用域或复合语句块。
- **L3280 EN**: Blank line separating nearby declarations or logic blocks.
  **L3280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3281 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3281 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3282 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3282 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3283 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvrotr_b(__m256i _1, __m256i _2) {`.
  **L3283 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvrotr_b(__m256i _1, __m256i _2) {`。
- **L3284 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvrotr_b((v32i8)_1, (v32i8)_2)`.
  **L3284 CN**: 以 `(__m256i)__builtin_lasx_xvrotr_b((v32i8)_1, (v32i8)_2)` 从当前函数返回。
- **L3285 EN**: Closes the current lexical scope or compound statement.
  **L3285 CN**: 结束当前词法作用域或复合语句块。
- **L3286 EN**: Blank line separating nearby declarations or logic blocks.
  **L3286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3287 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3287 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3288 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3288 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。

### Lines 3289-3312

````c
    __lasx_xvrotr_h(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvrotr_h((v16i16)_1, (v16i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvrotr_w(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvrotr_w((v8i32)_1, (v8i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvrotr_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvrotr_d((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvadd_q(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvadd_q((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
````
- **L3289 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvrotr_h(__m256i _1, __m256i _2) {`.
  **L3289 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvrotr_h(__m256i _1, __m256i _2) {`。
- **L3290 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvrotr_h((v16i16)_1, (v16i16)_2)`.
  **L3290 CN**: 以 `(__m256i)__builtin_lasx_xvrotr_h((v16i16)_1, (v16i16)_2)` 从当前函数返回。
- **L3291 EN**: Closes the current lexical scope or compound statement.
  **L3291 CN**: 结束当前词法作用域或复合语句块。
- **L3292 EN**: Blank line separating nearby declarations or logic blocks.
  **L3292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3293 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3293 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3294 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3294 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3295 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvrotr_w(__m256i _1, __m256i _2) {`.
  **L3295 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvrotr_w(__m256i _1, __m256i _2) {`。
- **L3296 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvrotr_w((v8i32)_1, (v8i32)_2)`.
  **L3296 CN**: 以 `(__m256i)__builtin_lasx_xvrotr_w((v8i32)_1, (v8i32)_2)` 从当前函数返回。
- **L3297 EN**: Closes the current lexical scope or compound statement.
  **L3297 CN**: 结束当前词法作用域或复合语句块。
- **L3298 EN**: Blank line separating nearby declarations or logic blocks.
  **L3298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3299 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3299 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3300 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3300 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3301 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvrotr_d(__m256i _1, __m256i _2) {`.
  **L3301 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvrotr_d(__m256i _1, __m256i _2) {`。
- **L3302 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvrotr_d((v4i64)_1, (v4i64)_2)`.
  **L3302 CN**: 以 `(__m256i)__builtin_lasx_xvrotr_d((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L3303 EN**: Closes the current lexical scope or compound statement.
  **L3303 CN**: 结束当前词法作用域或复合语句块。
- **L3304 EN**: Blank line separating nearby declarations or logic blocks.
  **L3304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3305 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3305 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3306 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3306 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3307 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvadd_q(__m256i _1, __m256i _2) {`.
  **L3307 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvadd_q(__m256i _1, __m256i _2) {`。
- **L3308 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvadd_q((v4i64)_1, (v4i64)_2)`.
  **L3308 CN**: 以 `(__m256i)__builtin_lasx_xvadd_q((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L3309 EN**: Closes the current lexical scope or compound statement.
  **L3309 CN**: 结束当前词法作用域或复合语句块。
- **L3310 EN**: Blank line separating nearby declarations or logic blocks.
  **L3310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3311 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3311 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3312 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3312 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。

### Lines 3313-3336

````c
    __lasx_xvsub_q(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvsub_q((v4i64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwev_q_du_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwev_q_du_d((v4u64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvaddwod_q_du_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvaddwod_q_du_d((v4u64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmulwev_q_du_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwev_q_du_d((v4u64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
````
- **L3313 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvsub_q(__m256i _1, __m256i _2) {`.
  **L3313 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvsub_q(__m256i _1, __m256i _2) {`。
- **L3314 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvsub_q((v4i64)_1, (v4i64)_2)`.
  **L3314 CN**: 以 `(__m256i)__builtin_lasx_xvsub_q((v4i64)_1, (v4i64)_2)` 从当前函数返回。
- **L3315 EN**: Closes the current lexical scope or compound statement.
  **L3315 CN**: 结束当前词法作用域或复合语句块。
- **L3316 EN**: Blank line separating nearby declarations or logic blocks.
  **L3316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3317 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3317 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3318 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3318 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3319 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwev_q_du_d(__m256i _1, __m256i _2) {`.
  **L3319 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwev_q_du_d(__m256i _1, __m256i _2) {`。
- **L3320 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwev_q_du_d((v4u64)_1, (v4i64)_2)`.
  **L3320 CN**: 以 `(__m256i)__builtin_lasx_xvaddwev_q_du_d((v4u64)_1, (v4i64)_2)` 从当前函数返回。
- **L3321 EN**: Closes the current lexical scope or compound statement.
  **L3321 CN**: 结束当前词法作用域或复合语句块。
- **L3322 EN**: Blank line separating nearby declarations or logic blocks.
  **L3322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3323 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3323 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3324 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3324 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3325 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvaddwod_q_du_d(__m256i _1, __m256i _2) {`.
  **L3325 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvaddwod_q_du_d(__m256i _1, __m256i _2) {`。
- **L3326 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvaddwod_q_du_d((v4u64)_1, (v4i64)_2)`.
  **L3326 CN**: 以 `(__m256i)__builtin_lasx_xvaddwod_q_du_d((v4u64)_1, (v4i64)_2)` 从当前函数返回。
- **L3327 EN**: Closes the current lexical scope or compound statement.
  **L3327 CN**: 结束当前词法作用域或复合语句块。
- **L3328 EN**: Blank line separating nearby declarations or logic blocks.
  **L3328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3329 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3329 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3330 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3330 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3331 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwev_q_du_d(__m256i _1, __m256i _2) {`.
  **L3331 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwev_q_du_d(__m256i _1, __m256i _2) {`。
- **L3332 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwev_q_du_d((v4u64)_1, (v4i64)_2)`.
  **L3332 CN**: 以 `(__m256i)__builtin_lasx_xvmulwev_q_du_d((v4u64)_1, (v4i64)_2)` 从当前函数返回。
- **L3333 EN**: Closes the current lexical scope or compound statement.
  **L3333 CN**: 结束当前词法作用域或复合语句块。
- **L3334 EN**: Blank line separating nearby declarations or logic blocks.
  **L3334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3335 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3335 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3336 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3336 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。

### Lines 3337-3360

````c
    __lasx_xvmulwod_q_du_d(__m256i _1, __m256i _2) {
  return (__m256i)__builtin_lasx_xvmulwod_q_du_d((v4u64)_1, (v4i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmskgez_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvmskgez_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvmsknz_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvmsknz_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvexth_h_b(__m256i _1) {
  return (__m256i)__builtin_lasx_xvexth_h_b((v32i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
````
- **L3337 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmulwod_q_du_d(__m256i _1, __m256i _2) {`.
  **L3337 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmulwod_q_du_d(__m256i _1, __m256i _2) {`。
- **L3338 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmulwod_q_du_d((v4u64)_1, (v4i64)_2)`.
  **L3338 CN**: 以 `(__m256i)__builtin_lasx_xvmulwod_q_du_d((v4u64)_1, (v4i64)_2)` 从当前函数返回。
- **L3339 EN**: Closes the current lexical scope or compound statement.
  **L3339 CN**: 结束当前词法作用域或复合语句块。
- **L3340 EN**: Blank line separating nearby declarations or logic blocks.
  **L3340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3341 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3341 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3342 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3342 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3343 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmskgez_b(__m256i _1) {`.
  **L3343 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmskgez_b(__m256i _1) {`。
- **L3344 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmskgez_b((v32i8)_1)`.
  **L3344 CN**: 以 `(__m256i)__builtin_lasx_xvmskgez_b((v32i8)_1)` 从当前函数返回。
- **L3345 EN**: Closes the current lexical scope or compound statement.
  **L3345 CN**: 结束当前词法作用域或复合语句块。
- **L3346 EN**: Blank line separating nearby declarations or logic blocks.
  **L3346 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3347 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3347 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3348 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3348 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3349 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvmsknz_b(__m256i _1) {`.
  **L3349 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvmsknz_b(__m256i _1) {`。
- **L3350 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvmsknz_b((v32i8)_1)`.
  **L3350 CN**: 以 `(__m256i)__builtin_lasx_xvmsknz_b((v32i8)_1)` 从当前函数返回。
- **L3351 EN**: Closes the current lexical scope or compound statement.
  **L3351 CN**: 结束当前词法作用域或复合语句块。
- **L3352 EN**: Blank line separating nearby declarations or logic blocks.
  **L3352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3353 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3353 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3354 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3354 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3355 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvexth_h_b(__m256i _1) {`.
  **L3355 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvexth_h_b(__m256i _1) {`。
- **L3356 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvexth_h_b((v32i8)_1)`.
  **L3356 CN**: 以 `(__m256i)__builtin_lasx_xvexth_h_b((v32i8)_1)` 从当前函数返回。
- **L3357 EN**: Closes the current lexical scope or compound statement.
  **L3357 CN**: 结束当前词法作用域或复合语句块。
- **L3358 EN**: Blank line separating nearby declarations or logic blocks.
  **L3358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3359 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3359 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3360 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3360 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。

### Lines 3361-3384

````c
    __lasx_xvexth_w_h(__m256i _1) {
  return (__m256i)__builtin_lasx_xvexth_w_h((v16i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvexth_d_w(__m256i _1) {
  return (__m256i)__builtin_lasx_xvexth_d_w((v8i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvexth_q_d(__m256i _1) {
  return (__m256i)__builtin_lasx_xvexth_q_d((v4i64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvexth_hu_bu(__m256i _1) {
  return (__m256i)__builtin_lasx_xvexth_hu_bu((v32u8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
````
- **L3361 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvexth_w_h(__m256i _1) {`.
  **L3361 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvexth_w_h(__m256i _1) {`。
- **L3362 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvexth_w_h((v16i16)_1)`.
  **L3362 CN**: 以 `(__m256i)__builtin_lasx_xvexth_w_h((v16i16)_1)` 从当前函数返回。
- **L3363 EN**: Closes the current lexical scope or compound statement.
  **L3363 CN**: 结束当前词法作用域或复合语句块。
- **L3364 EN**: Blank line separating nearby declarations or logic blocks.
  **L3364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3365 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3365 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3366 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3366 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3367 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvexth_d_w(__m256i _1) {`.
  **L3367 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvexth_d_w(__m256i _1) {`。
- **L3368 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvexth_d_w((v8i32)_1)`.
  **L3368 CN**: 以 `(__m256i)__builtin_lasx_xvexth_d_w((v8i32)_1)` 从当前函数返回。
- **L3369 EN**: Closes the current lexical scope or compound statement.
  **L3369 CN**: 结束当前词法作用域或复合语句块。
- **L3370 EN**: Blank line separating nearby declarations or logic blocks.
  **L3370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3371 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3371 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3372 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3372 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3373 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvexth_q_d(__m256i _1) {`.
  **L3373 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvexth_q_d(__m256i _1) {`。
- **L3374 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvexth_q_d((v4i64)_1)`.
  **L3374 CN**: 以 `(__m256i)__builtin_lasx_xvexth_q_d((v4i64)_1)` 从当前函数返回。
- **L3375 EN**: Closes the current lexical scope or compound statement.
  **L3375 CN**: 结束当前词法作用域或复合语句块。
- **L3376 EN**: Blank line separating nearby declarations or logic blocks.
  **L3376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3377 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3377 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3378 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3378 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvexth_hu_bu(__m256i _1) {`.
  **L3379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvexth_hu_bu(__m256i _1) {`。
- **L3380 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvexth_hu_bu((v32u8)_1)`.
  **L3380 CN**: 以 `(__m256i)__builtin_lasx_xvexth_hu_bu((v32u8)_1)` 从当前函数返回。
- **L3381 EN**: Closes the current lexical scope or compound statement.
  **L3381 CN**: 结束当前词法作用域或复合语句块。
- **L3382 EN**: Blank line separating nearby declarations or logic blocks.
  **L3382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3383 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3383 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3384 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3384 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。

### Lines 3385-3408

````c
    __lasx_xvexth_wu_hu(__m256i _1) {
  return (__m256i)__builtin_lasx_xvexth_wu_hu((v16u16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvexth_du_wu(__m256i _1) {
  return (__m256i)__builtin_lasx_xvexth_du_wu((v8u32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvexth_qu_du(__m256i _1) {
  return (__m256i)__builtin_lasx_xvexth_qu_du((v4u64)_1);
}

#define __lasx_xvrotri_b(/*__m256i*/ _1, /*ui3*/ _2)                           \
  ((__m256i)__builtin_lasx_xvrotri_b((v32i8)(_1), (_2)))

#define __lasx_xvrotri_h(/*__m256i*/ _1, /*ui4*/ _2)                           \
  ((__m256i)__builtin_lasx_xvrotri_h((v16i16)(_1), (_2)))

#define __lasx_xvrotri_w(/*__m256i*/ _1, /*ui5*/ _2)                           \
  ((__m256i)__builtin_lasx_xvrotri_w((v8i32)(_1), (_2)))
````
- **L3385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvexth_wu_hu(__m256i _1) {`.
  **L3385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvexth_wu_hu(__m256i _1) {`。
- **L3386 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvexth_wu_hu((v16u16)_1)`.
  **L3386 CN**: 以 `(__m256i)__builtin_lasx_xvexth_wu_hu((v16u16)_1)` 从当前函数返回。
- **L3387 EN**: Closes the current lexical scope or compound statement.
  **L3387 CN**: 结束当前词法作用域或复合语句块。
- **L3388 EN**: Blank line separating nearby declarations or logic blocks.
  **L3388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3389 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3389 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3390 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3390 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvexth_du_wu(__m256i _1) {`.
  **L3391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvexth_du_wu(__m256i _1) {`。
- **L3392 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvexth_du_wu((v8u32)_1)`.
  **L3392 CN**: 以 `(__m256i)__builtin_lasx_xvexth_du_wu((v8u32)_1)` 从当前函数返回。
- **L3393 EN**: Closes the current lexical scope or compound statement.
  **L3393 CN**: 结束当前词法作用域或复合语句块。
- **L3394 EN**: Blank line separating nearby declarations or logic blocks.
  **L3394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3395 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3395 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3396 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3396 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3397 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvexth_qu_du(__m256i _1) {`.
  **L3397 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvexth_qu_du(__m256i _1) {`。
- **L3398 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvexth_qu_du((v4u64)_1)`.
  **L3398 CN**: 以 `(__m256i)__builtin_lasx_xvexth_qu_du((v4u64)_1)` 从当前函数返回。
- **L3399 EN**: Closes the current lexical scope or compound statement.
  **L3399 CN**: 结束当前词法作用域或复合语句块。
- **L3400 EN**: Blank line separating nearby declarations or logic blocks.
  **L3400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3401 EN**: Defines macro `__lasx_xvrotri_b(/*__m256i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3401 CN**: 定义宏 `__lasx_xvrotri_b(/*__m256i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L3402 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvrotri_b`.
  **L3402 CN**: 继续与可调用符号 `__builtin_lasx_xvrotri_b` 相关的逻辑。
- **L3403 EN**: Blank line separating nearby declarations or logic blocks.
  **L3403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3404 EN**: Defines macro `__lasx_xvrotri_h(/*__m256i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3404 CN**: 定义宏 `__lasx_xvrotri_h(/*__m256i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L3405 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvrotri_h`.
  **L3405 CN**: 继续与可调用符号 `__builtin_lasx_xvrotri_h` 相关的逻辑。
- **L3406 EN**: Blank line separating nearby declarations or logic blocks.
  **L3406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3407 EN**: Defines macro `__lasx_xvrotri_w(/*__m256i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3407 CN**: 定义宏 `__lasx_xvrotri_w(/*__m256i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L3408 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvrotri_w`.
  **L3408 CN**: 继续与可调用符号 `__builtin_lasx_xvrotri_w` 相关的逻辑。

### Lines 3409-3432

````c

#define __lasx_xvrotri_d(/*__m256i*/ _1, /*ui6*/ _2)                           \
  ((__m256i)__builtin_lasx_xvrotri_d((v4i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvextl_q_d(__m256i _1) {
  return (__m256i)__builtin_lasx_xvextl_q_d((v4i64)_1);
}

#define __lasx_xvsrlni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)         \
  ((__m256i)__builtin_lasx_xvsrlni_b_h((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvsrlni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)         \
  ((__m256i)__builtin_lasx_xvsrlni_h_w((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvsrlni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)         \
  ((__m256i)__builtin_lasx_xvsrlni_w_d((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvsrlni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)         \
  ((__m256i)__builtin_lasx_xvsrlni_d_q((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvsrlrni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)        \
  ((__m256i)__builtin_lasx_xvsrlrni_b_h((v32i8)(_1), (v32i8)(_2), (_3)))
````
- **L3409 EN**: Blank line separating nearby declarations or logic blocks.
  **L3409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3410 EN**: Defines macro `__lasx_xvrotri_d(/*__m256i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3410 CN**: 定义宏 `__lasx_xvrotri_d(/*__m256i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L3411 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvrotri_d`.
  **L3411 CN**: 继续与可调用符号 `__builtin_lasx_xvrotri_d` 相关的逻辑。
- **L3412 EN**: Blank line separating nearby declarations or logic blocks.
  **L3412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3413 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3413 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3414 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3414 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3415 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvextl_q_d(__m256i _1) {`.
  **L3415 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvextl_q_d(__m256i _1) {`。
- **L3416 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvextl_q_d((v4i64)_1)`.
  **L3416 CN**: 以 `(__m256i)__builtin_lasx_xvextl_q_d((v4i64)_1)` 从当前函数返回。
- **L3417 EN**: Closes the current lexical scope or compound statement.
  **L3417 CN**: 结束当前词法作用域或复合语句块。
- **L3418 EN**: Blank line separating nearby declarations or logic blocks.
  **L3418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3419 EN**: Defines macro `__lasx_xvsrlni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3419 CN**: 定义宏 `__lasx_xvsrlni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3420 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlni_b_h`.
  **L3420 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlni_b_h` 相关的逻辑。
- **L3421 EN**: Blank line separating nearby declarations or logic blocks.
  **L3421 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3422 EN**: Defines macro `__lasx_xvsrlni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3422 CN**: 定义宏 `__lasx_xvsrlni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3423 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlni_h_w`.
  **L3423 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlni_h_w` 相关的逻辑。
- **L3424 EN**: Blank line separating nearby declarations or logic blocks.
  **L3424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3425 EN**: Defines macro `__lasx_xvsrlni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3425 CN**: 定义宏 `__lasx_xvsrlni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3426 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlni_w_d`.
  **L3426 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlni_w_d` 相关的逻辑。
- **L3427 EN**: Blank line separating nearby declarations or logic blocks.
  **L3427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3428 EN**: Defines macro `__lasx_xvsrlni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3428 CN**: 定义宏 `__lasx_xvsrlni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3429 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlni_d_q`.
  **L3429 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlni_d_q` 相关的逻辑。
- **L3430 EN**: Blank line separating nearby declarations or logic blocks.
  **L3430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3431 EN**: Defines macro `__lasx_xvsrlrni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3431 CN**: 定义宏 `__lasx_xvsrlrni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3432 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlrni_b_h`.
  **L3432 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlrni_b_h` 相关的逻辑。

### Lines 3433-3456

````c

#define __lasx_xvsrlrni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)        \
  ((__m256i)__builtin_lasx_xvsrlrni_h_w((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvsrlrni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)        \
  ((__m256i)__builtin_lasx_xvsrlrni_w_d((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvsrlrni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)        \
  ((__m256i)__builtin_lasx_xvsrlrni_d_q((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvssrlni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)        \
  ((__m256i)__builtin_lasx_xvssrlni_b_h((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvssrlni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)        \
  ((__m256i)__builtin_lasx_xvssrlni_h_w((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvssrlni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)        \
  ((__m256i)__builtin_lasx_xvssrlni_w_d((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvssrlni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)        \
  ((__m256i)__builtin_lasx_xvssrlni_d_q((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvssrlni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrlni_bu_h((v32u8)(_1), (v32i8)(_2), (_3)))
````
- **L3433 EN**: Blank line separating nearby declarations or logic blocks.
  **L3433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3434 EN**: Defines macro `__lasx_xvsrlrni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3434 CN**: 定义宏 `__lasx_xvsrlrni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3435 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlrni_h_w`.
  **L3435 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlrni_h_w` 相关的逻辑。
- **L3436 EN**: Blank line separating nearby declarations or logic blocks.
  **L3436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3437 EN**: Defines macro `__lasx_xvsrlrni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3437 CN**: 定义宏 `__lasx_xvsrlrni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3438 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlrni_w_d`.
  **L3438 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlrni_w_d` 相关的逻辑。
- **L3439 EN**: Blank line separating nearby declarations or logic blocks.
  **L3439 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3440 EN**: Defines macro `__lasx_xvsrlrni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3440 CN**: 定义宏 `__lasx_xvsrlrni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3441 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrlrni_d_q`.
  **L3441 CN**: 继续与可调用符号 `__builtin_lasx_xvsrlrni_d_q` 相关的逻辑。
- **L3442 EN**: Blank line separating nearby declarations or logic blocks.
  **L3442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3443 EN**: Defines macro `__lasx_xvssrlni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3443 CN**: 定义宏 `__lasx_xvssrlni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3444 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlni_b_h`.
  **L3444 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlni_b_h` 相关的逻辑。
- **L3445 EN**: Blank line separating nearby declarations or logic blocks.
  **L3445 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3446 EN**: Defines macro `__lasx_xvssrlni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3446 CN**: 定义宏 `__lasx_xvssrlni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3447 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlni_h_w`.
  **L3447 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlni_h_w` 相关的逻辑。
- **L3448 EN**: Blank line separating nearby declarations or logic blocks.
  **L3448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3449 EN**: Defines macro `__lasx_xvssrlni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3449 CN**: 定义宏 `__lasx_xvssrlni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3450 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlni_w_d`.
  **L3450 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlni_w_d` 相关的逻辑。
- **L3451 EN**: Blank line separating nearby declarations or logic blocks.
  **L3451 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3452 EN**: Defines macro `__lasx_xvssrlni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3452 CN**: 定义宏 `__lasx_xvssrlni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3453 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlni_d_q`.
  **L3453 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlni_d_q` 相关的逻辑。
- **L3454 EN**: Blank line separating nearby declarations or logic blocks.
  **L3454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3455 EN**: Defines macro `__lasx_xvssrlni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3455 CN**: 定义宏 `__lasx_xvssrlni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3456 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlni_bu_h`.
  **L3456 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlni_bu_h` 相关的逻辑。

### Lines 3457-3480

````c

#define __lasx_xvssrlni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrlni_hu_w((v16u16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvssrlni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrlni_wu_d((v8u32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvssrlni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrlni_du_q((v4u64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvssrlrni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrlrni_b_h((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvssrlrni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrlrni_h_w((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvssrlrni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrlrni_w_d((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvssrlrni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrlrni_d_q((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvssrlrni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)      \
  ((__m256i)__builtin_lasx_xvssrlrni_bu_h((v32u8)(_1), (v32i8)(_2), (_3)))
````
- **L3457 EN**: Blank line separating nearby declarations or logic blocks.
  **L3457 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3458 EN**: Defines macro `__lasx_xvssrlni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3458 CN**: 定义宏 `__lasx_xvssrlni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3459 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlni_hu_w`.
  **L3459 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlni_hu_w` 相关的逻辑。
- **L3460 EN**: Blank line separating nearby declarations or logic blocks.
  **L3460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3461 EN**: Defines macro `__lasx_xvssrlni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3461 CN**: 定义宏 `__lasx_xvssrlni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3462 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlni_wu_d`.
  **L3462 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlni_wu_d` 相关的逻辑。
- **L3463 EN**: Blank line separating nearby declarations or logic blocks.
  **L3463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3464 EN**: Defines macro `__lasx_xvssrlni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3464 CN**: 定义宏 `__lasx_xvssrlni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3465 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlni_du_q`.
  **L3465 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlni_du_q` 相关的逻辑。
- **L3466 EN**: Blank line separating nearby declarations or logic blocks.
  **L3466 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3467 EN**: Defines macro `__lasx_xvssrlrni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3467 CN**: 定义宏 `__lasx_xvssrlrni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3468 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlrni_b_h`.
  **L3468 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlrni_b_h` 相关的逻辑。
- **L3469 EN**: Blank line separating nearby declarations or logic blocks.
  **L3469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3470 EN**: Defines macro `__lasx_xvssrlrni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3470 CN**: 定义宏 `__lasx_xvssrlrni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3471 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlrni_h_w`.
  **L3471 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlrni_h_w` 相关的逻辑。
- **L3472 EN**: Blank line separating nearby declarations or logic blocks.
  **L3472 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3473 EN**: Defines macro `__lasx_xvssrlrni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3473 CN**: 定义宏 `__lasx_xvssrlrni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3474 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlrni_w_d`.
  **L3474 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlrni_w_d` 相关的逻辑。
- **L3475 EN**: Blank line separating nearby declarations or logic blocks.
  **L3475 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3476 EN**: Defines macro `__lasx_xvssrlrni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3476 CN**: 定义宏 `__lasx_xvssrlrni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3477 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlrni_d_q`.
  **L3477 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlrni_d_q` 相关的逻辑。
- **L3478 EN**: Blank line separating nearby declarations or logic blocks.
  **L3478 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3479 EN**: Defines macro `__lasx_xvssrlrni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3479 CN**: 定义宏 `__lasx_xvssrlrni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3480 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlrni_bu_h`.
  **L3480 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlrni_bu_h` 相关的逻辑。

### Lines 3481-3504

````c

#define __lasx_xvssrlrni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)      \
  ((__m256i)__builtin_lasx_xvssrlrni_hu_w((v16u16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvssrlrni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)      \
  ((__m256i)__builtin_lasx_xvssrlrni_wu_d((v8u32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvssrlrni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)      \
  ((__m256i)__builtin_lasx_xvssrlrni_du_q((v4u64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvsrani_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)         \
  ((__m256i)__builtin_lasx_xvsrani_b_h((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvsrani_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)         \
  ((__m256i)__builtin_lasx_xvsrani_h_w((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvsrani_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)         \
  ((__m256i)__builtin_lasx_xvsrani_w_d((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvsrani_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)         \
  ((__m256i)__builtin_lasx_xvsrani_d_q((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvsrarni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)        \
  ((__m256i)__builtin_lasx_xvsrarni_b_h((v32i8)(_1), (v32i8)(_2), (_3)))
````
- **L3481 EN**: Blank line separating nearby declarations or logic blocks.
  **L3481 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3482 EN**: Defines macro `__lasx_xvssrlrni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3482 CN**: 定义宏 `__lasx_xvssrlrni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3483 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlrni_hu_w`.
  **L3483 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlrni_hu_w` 相关的逻辑。
- **L3484 EN**: Blank line separating nearby declarations or logic blocks.
  **L3484 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3485 EN**: Defines macro `__lasx_xvssrlrni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3485 CN**: 定义宏 `__lasx_xvssrlrni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3486 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlrni_wu_d`.
  **L3486 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlrni_wu_d` 相关的逻辑。
- **L3487 EN**: Blank line separating nearby declarations or logic blocks.
  **L3487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3488 EN**: Defines macro `__lasx_xvssrlrni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3488 CN**: 定义宏 `__lasx_xvssrlrni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3489 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrlrni_du_q`.
  **L3489 CN**: 继续与可调用符号 `__builtin_lasx_xvssrlrni_du_q` 相关的逻辑。
- **L3490 EN**: Blank line separating nearby declarations or logic blocks.
  **L3490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3491 EN**: Defines macro `__lasx_xvsrani_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3491 CN**: 定义宏 `__lasx_xvsrani_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3492 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrani_b_h`.
  **L3492 CN**: 继续与可调用符号 `__builtin_lasx_xvsrani_b_h` 相关的逻辑。
- **L3493 EN**: Blank line separating nearby declarations or logic blocks.
  **L3493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3494 EN**: Defines macro `__lasx_xvsrani_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3494 CN**: 定义宏 `__lasx_xvsrani_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3495 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrani_h_w`.
  **L3495 CN**: 继续与可调用符号 `__builtin_lasx_xvsrani_h_w` 相关的逻辑。
- **L3496 EN**: Blank line separating nearby declarations or logic blocks.
  **L3496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3497 EN**: Defines macro `__lasx_xvsrani_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3497 CN**: 定义宏 `__lasx_xvsrani_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3498 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrani_w_d`.
  **L3498 CN**: 继续与可调用符号 `__builtin_lasx_xvsrani_w_d` 相关的逻辑。
- **L3499 EN**: Blank line separating nearby declarations or logic blocks.
  **L3499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3500 EN**: Defines macro `__lasx_xvsrani_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3500 CN**: 定义宏 `__lasx_xvsrani_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3501 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrani_d_q`.
  **L3501 CN**: 继续与可调用符号 `__builtin_lasx_xvsrani_d_q` 相关的逻辑。
- **L3502 EN**: Blank line separating nearby declarations or logic blocks.
  **L3502 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3503 EN**: Defines macro `__lasx_xvsrarni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3503 CN**: 定义宏 `__lasx_xvsrarni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3504 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrarni_b_h`.
  **L3504 CN**: 继续与可调用符号 `__builtin_lasx_xvsrarni_b_h` 相关的逻辑。

### Lines 3505-3528

````c

#define __lasx_xvsrarni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)        \
  ((__m256i)__builtin_lasx_xvsrarni_h_w((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvsrarni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)        \
  ((__m256i)__builtin_lasx_xvsrarni_w_d((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvsrarni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)        \
  ((__m256i)__builtin_lasx_xvsrarni_d_q((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvssrani_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)        \
  ((__m256i)__builtin_lasx_xvssrani_b_h((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvssrani_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)        \
  ((__m256i)__builtin_lasx_xvssrani_h_w((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvssrani_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)        \
  ((__m256i)__builtin_lasx_xvssrani_w_d((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvssrani_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)        \
  ((__m256i)__builtin_lasx_xvssrani_d_q((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvssrani_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrani_bu_h((v32u8)(_1), (v32i8)(_2), (_3)))
````
- **L3505 EN**: Blank line separating nearby declarations or logic blocks.
  **L3505 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3506 EN**: Defines macro `__lasx_xvsrarni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3506 CN**: 定义宏 `__lasx_xvsrarni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3507 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrarni_h_w`.
  **L3507 CN**: 继续与可调用符号 `__builtin_lasx_xvsrarni_h_w` 相关的逻辑。
- **L3508 EN**: Blank line separating nearby declarations or logic blocks.
  **L3508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3509 EN**: Defines macro `__lasx_xvsrarni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3509 CN**: 定义宏 `__lasx_xvsrarni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3510 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrarni_w_d`.
  **L3510 CN**: 继续与可调用符号 `__builtin_lasx_xvsrarni_w_d` 相关的逻辑。
- **L3511 EN**: Blank line separating nearby declarations or logic blocks.
  **L3511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3512 EN**: Defines macro `__lasx_xvsrarni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3512 CN**: 定义宏 `__lasx_xvsrarni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3513 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvsrarni_d_q`.
  **L3513 CN**: 继续与可调用符号 `__builtin_lasx_xvsrarni_d_q` 相关的逻辑。
- **L3514 EN**: Blank line separating nearby declarations or logic blocks.
  **L3514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3515 EN**: Defines macro `__lasx_xvssrani_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3515 CN**: 定义宏 `__lasx_xvssrani_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3516 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrani_b_h`.
  **L3516 CN**: 继续与可调用符号 `__builtin_lasx_xvssrani_b_h` 相关的逻辑。
- **L3517 EN**: Blank line separating nearby declarations or logic blocks.
  **L3517 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3518 EN**: Defines macro `__lasx_xvssrani_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3518 CN**: 定义宏 `__lasx_xvssrani_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3519 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrani_h_w`.
  **L3519 CN**: 继续与可调用符号 `__builtin_lasx_xvssrani_h_w` 相关的逻辑。
- **L3520 EN**: Blank line separating nearby declarations or logic blocks.
  **L3520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3521 EN**: Defines macro `__lasx_xvssrani_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3521 CN**: 定义宏 `__lasx_xvssrani_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3522 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrani_w_d`.
  **L3522 CN**: 继续与可调用符号 `__builtin_lasx_xvssrani_w_d` 相关的逻辑。
- **L3523 EN**: Blank line separating nearby declarations or logic blocks.
  **L3523 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3524 EN**: Defines macro `__lasx_xvssrani_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3524 CN**: 定义宏 `__lasx_xvssrani_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3525 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrani_d_q`.
  **L3525 CN**: 继续与可调用符号 `__builtin_lasx_xvssrani_d_q` 相关的逻辑。
- **L3526 EN**: Blank line separating nearby declarations or logic blocks.
  **L3526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3527 EN**: Defines macro `__lasx_xvssrani_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3527 CN**: 定义宏 `__lasx_xvssrani_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3528 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrani_bu_h`.
  **L3528 CN**: 继续与可调用符号 `__builtin_lasx_xvssrani_bu_h` 相关的逻辑。

### Lines 3529-3552

````c

#define __lasx_xvssrani_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrani_hu_w((v16u16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvssrani_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrani_wu_d((v8u32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvssrani_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrani_du_q((v4u64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvssrarni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrarni_b_h((v32i8)(_1), (v32i8)(_2), (_3)))

#define __lasx_xvssrarni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrarni_h_w((v16i16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvssrarni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrarni_w_d((v8i32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvssrarni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)       \
  ((__m256i)__builtin_lasx_xvssrarni_d_q((v4i64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xvssrarni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)      \
  ((__m256i)__builtin_lasx_xvssrarni_bu_h((v32u8)(_1), (v32i8)(_2), (_3)))
````
- **L3529 EN**: Blank line separating nearby declarations or logic blocks.
  **L3529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3530 EN**: Defines macro `__lasx_xvssrani_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3530 CN**: 定义宏 `__lasx_xvssrani_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3531 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrani_hu_w`.
  **L3531 CN**: 继续与可调用符号 `__builtin_lasx_xvssrani_hu_w` 相关的逻辑。
- **L3532 EN**: Blank line separating nearby declarations or logic blocks.
  **L3532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3533 EN**: Defines macro `__lasx_xvssrani_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3533 CN**: 定义宏 `__lasx_xvssrani_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3534 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrani_wu_d`.
  **L3534 CN**: 继续与可调用符号 `__builtin_lasx_xvssrani_wu_d` 相关的逻辑。
- **L3535 EN**: Blank line separating nearby declarations or logic blocks.
  **L3535 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3536 EN**: Defines macro `__lasx_xvssrani_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3536 CN**: 定义宏 `__lasx_xvssrani_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3537 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrani_du_q`.
  **L3537 CN**: 继续与可调用符号 `__builtin_lasx_xvssrani_du_q` 相关的逻辑。
- **L3538 EN**: Blank line separating nearby declarations or logic blocks.
  **L3538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3539 EN**: Defines macro `__lasx_xvssrarni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3539 CN**: 定义宏 `__lasx_xvssrarni_b_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3540 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrarni_b_h`.
  **L3540 CN**: 继续与可调用符号 `__builtin_lasx_xvssrarni_b_h` 相关的逻辑。
- **L3541 EN**: Blank line separating nearby declarations or logic blocks.
  **L3541 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3542 EN**: Defines macro `__lasx_xvssrarni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3542 CN**: 定义宏 `__lasx_xvssrarni_h_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3543 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrarni_h_w`.
  **L3543 CN**: 继续与可调用符号 `__builtin_lasx_xvssrarni_h_w` 相关的逻辑。
- **L3544 EN**: Blank line separating nearby declarations or logic blocks.
  **L3544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3545 EN**: Defines macro `__lasx_xvssrarni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3545 CN**: 定义宏 `__lasx_xvssrarni_w_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3546 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrarni_w_d`.
  **L3546 CN**: 继续与可调用符号 `__builtin_lasx_xvssrarni_w_d` 相关的逻辑。
- **L3547 EN**: Blank line separating nearby declarations or logic blocks.
  **L3547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3548 EN**: Defines macro `__lasx_xvssrarni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3548 CN**: 定义宏 `__lasx_xvssrarni_d_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3549 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrarni_d_q`.
  **L3549 CN**: 继续与可调用符号 `__builtin_lasx_xvssrarni_d_q` 相关的逻辑。
- **L3550 EN**: Blank line separating nearby declarations or logic blocks.
  **L3550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3551 EN**: Defines macro `__lasx_xvssrarni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3551 CN**: 定义宏 `__lasx_xvssrarni_bu_h(/*__m256i*/ _1, /*__m256i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3552 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrarni_bu_h`.
  **L3552 CN**: 继续与可调用符号 `__builtin_lasx_xvssrarni_bu_h` 相关的逻辑。

### Lines 3553-3576

````c

#define __lasx_xvssrarni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)      \
  ((__m256i)__builtin_lasx_xvssrarni_hu_w((v16u16)(_1), (v16i16)(_2), (_3)))

#define __lasx_xvssrarni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)      \
  ((__m256i)__builtin_lasx_xvssrarni_wu_d((v8u32)(_1), (v8i32)(_2), (_3)))

#define __lasx_xvssrarni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)      \
  ((__m256i)__builtin_lasx_xvssrarni_du_q((v4u64)(_1), (v4i64)(_2), (_3)))

#define __lasx_xbnz_b(/*__m256i*/ _1) ((int)__builtin_lasx_xbnz_b((v32u8)(_1)))

#define __lasx_xbnz_d(/*__m256i*/ _1) ((int)__builtin_lasx_xbnz_d((v4u64)(_1)))

#define __lasx_xbnz_h(/*__m256i*/ _1) ((int)__builtin_lasx_xbnz_h((v16u16)(_1)))

#define __lasx_xbnz_v(/*__m256i*/ _1) ((int)__builtin_lasx_xbnz_v((v32u8)(_1)))

#define __lasx_xbnz_w(/*__m256i*/ _1) ((int)__builtin_lasx_xbnz_w((v8u32)(_1)))

#define __lasx_xbz_b(/*__m256i*/ _1) ((int)__builtin_lasx_xbz_b((v32u8)(_1)))

#define __lasx_xbz_d(/*__m256i*/ _1) ((int)__builtin_lasx_xbz_d((v4u64)(_1)))

````
- **L3553 EN**: Blank line separating nearby declarations or logic blocks.
  **L3553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3554 EN**: Defines macro `__lasx_xvssrarni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3554 CN**: 定义宏 `__lasx_xvssrarni_hu_w(/*__m256i*/ _1, /*__m256i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3555 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrarni_hu_w`.
  **L3555 CN**: 继续与可调用符号 `__builtin_lasx_xvssrarni_hu_w` 相关的逻辑。
- **L3556 EN**: Blank line separating nearby declarations or logic blocks.
  **L3556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3557 EN**: Defines macro `__lasx_xvssrarni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3557 CN**: 定义宏 `__lasx_xvssrarni_wu_d(/*__m256i*/ _1, /*__m256i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3558 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrarni_wu_d`.
  **L3558 CN**: 继续与可调用符号 `__builtin_lasx_xvssrarni_wu_d` 相关的逻辑。
- **L3559 EN**: Blank line separating nearby declarations or logic blocks.
  **L3559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3560 EN**: Defines macro `__lasx_xvssrarni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3560 CN**: 定义宏 `__lasx_xvssrarni_du_q(/*__m256i*/ _1, /*__m256i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3561 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvssrarni_du_q`.
  **L3561 CN**: 继续与可调用符号 `__builtin_lasx_xvssrarni_du_q` 相关的逻辑。
- **L3562 EN**: Blank line separating nearby declarations or logic blocks.
  **L3562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3563 EN**: Defines macro `__lasx_xbnz_b(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3563 CN**: 定义宏 `__lasx_xbnz_b(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3564 EN**: Blank line separating nearby declarations or logic blocks.
  **L3564 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3565 EN**: Defines macro `__lasx_xbnz_d(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3565 CN**: 定义宏 `__lasx_xbnz_d(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3566 EN**: Blank line separating nearby declarations or logic blocks.
  **L3566 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3567 EN**: Defines macro `__lasx_xbnz_h(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3567 CN**: 定义宏 `__lasx_xbnz_h(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3568 EN**: Blank line separating nearby declarations or logic blocks.
  **L3568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3569 EN**: Defines macro `__lasx_xbnz_v(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3569 CN**: 定义宏 `__lasx_xbnz_v(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3570 EN**: Blank line separating nearby declarations or logic blocks.
  **L3570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3571 EN**: Defines macro `__lasx_xbnz_w(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3571 CN**: 定义宏 `__lasx_xbnz_w(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3572 EN**: Blank line separating nearby declarations or logic blocks.
  **L3572 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3573 EN**: Defines macro `__lasx_xbz_b(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3573 CN**: 定义宏 `__lasx_xbz_b(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3574 EN**: Blank line separating nearby declarations or logic blocks.
  **L3574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3575 EN**: Defines macro `__lasx_xbz_d(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3575 CN**: 定义宏 `__lasx_xbz_d(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3576 EN**: Blank line separating nearby declarations or logic blocks.
  **L3576 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3577-3600

````c
#define __lasx_xbz_h(/*__m256i*/ _1) ((int)__builtin_lasx_xbz_h((v16u16)(_1)))

#define __lasx_xbz_v(/*__m256i*/ _1) ((int)__builtin_lasx_xbz_v((v32u8)(_1)))

#define __lasx_xbz_w(/*__m256i*/ _1) ((int)__builtin_lasx_xbz_w((v8u32)(_1)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_caf_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_caf_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_caf_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_caf_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_ceq_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_ceq_d((v4f64)_1, (v4f64)_2);
}

````
- **L3577 EN**: Defines macro `__lasx_xbz_h(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3577 CN**: 定义宏 `__lasx_xbz_h(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3578 EN**: Blank line separating nearby declarations or logic blocks.
  **L3578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3579 EN**: Defines macro `__lasx_xbz_v(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3579 CN**: 定义宏 `__lasx_xbz_v(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3580 EN**: Blank line separating nearby declarations or logic blocks.
  **L3580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3581 EN**: Defines macro `__lasx_xbz_w(/*__m256i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3581 CN**: 定义宏 `__lasx_xbz_w(/*__m256i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3582 EN**: Blank line separating nearby declarations or logic blocks.
  **L3582 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3583 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3583 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3584 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3584 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3585 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_caf_d(__m256d _1, __m256d _2) {`.
  **L3585 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_caf_d(__m256d _1, __m256d _2) {`。
- **L3586 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_caf_d((v4f64)_1, (v4f64)_2)`.
  **L3586 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_caf_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3587 EN**: Closes the current lexical scope or compound statement.
  **L3587 CN**: 结束当前词法作用域或复合语句块。
- **L3588 EN**: Blank line separating nearby declarations or logic blocks.
  **L3588 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3589 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3589 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3590 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3590 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3591 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_caf_s(__m256 _1, __m256 _2) {`.
  **L3591 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_caf_s(__m256 _1, __m256 _2) {`。
- **L3592 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_caf_s((v8f32)_1, (v8f32)_2)`.
  **L3592 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_caf_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3593 EN**: Closes the current lexical scope or compound statement.
  **L3593 CN**: 结束当前词法作用域或复合语句块。
- **L3594 EN**: Blank line separating nearby declarations or logic blocks.
  **L3594 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3595 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3595 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3596 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3596 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3597 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_ceq_d(__m256d _1, __m256d _2) {`.
  **L3597 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_ceq_d(__m256d _1, __m256d _2) {`。
- **L3598 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_ceq_d((v4f64)_1, (v4f64)_2)`.
  **L3598 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_ceq_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3599 EN**: Closes the current lexical scope or compound statement.
  **L3599 CN**: 结束当前词法作用域或复合语句块。
- **L3600 EN**: Blank line separating nearby declarations or logic blocks.
  **L3600 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3601-3624

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_ceq_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_ceq_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cle_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cle_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cle_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cle_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_clt_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_clt_d((v4f64)_1, (v4f64)_2);
}

````
- **L3601 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3601 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3602 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3602 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3603 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_ceq_s(__m256 _1, __m256 _2) {`.
  **L3603 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_ceq_s(__m256 _1, __m256 _2) {`。
- **L3604 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_ceq_s((v8f32)_1, (v8f32)_2)`.
  **L3604 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_ceq_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3605 EN**: Closes the current lexical scope or compound statement.
  **L3605 CN**: 结束当前词法作用域或复合语句块。
- **L3606 EN**: Blank line separating nearby declarations or logic blocks.
  **L3606 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3607 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3607 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3608 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3608 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3609 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cle_d(__m256d _1, __m256d _2) {`.
  **L3609 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cle_d(__m256d _1, __m256d _2) {`。
- **L3610 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cle_d((v4f64)_1, (v4f64)_2)`.
  **L3610 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cle_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3611 EN**: Closes the current lexical scope or compound statement.
  **L3611 CN**: 结束当前词法作用域或复合语句块。
- **L3612 EN**: Blank line separating nearby declarations or logic blocks.
  **L3612 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3613 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3613 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3614 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3614 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3615 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cle_s(__m256 _1, __m256 _2) {`.
  **L3615 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cle_s(__m256 _1, __m256 _2) {`。
- **L3616 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cle_s((v8f32)_1, (v8f32)_2)`.
  **L3616 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cle_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3617 EN**: Closes the current lexical scope or compound statement.
  **L3617 CN**: 结束当前词法作用域或复合语句块。
- **L3618 EN**: Blank line separating nearby declarations or logic blocks.
  **L3618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3619 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3619 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3620 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3620 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3621 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_clt_d(__m256d _1, __m256d _2) {`.
  **L3621 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_clt_d(__m256d _1, __m256d _2) {`。
- **L3622 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_clt_d((v4f64)_1, (v4f64)_2)`.
  **L3622 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_clt_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3623 EN**: Closes the current lexical scope or compound statement.
  **L3623 CN**: 结束当前词法作用域或复合语句块。
- **L3624 EN**: Blank line separating nearby declarations or logic blocks.
  **L3624 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3625-3648

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_clt_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_clt_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cne_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cne_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cne_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cne_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cor_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cor_d((v4f64)_1, (v4f64)_2);
}

````
- **L3625 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3625 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3626 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3626 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3627 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_clt_s(__m256 _1, __m256 _2) {`.
  **L3627 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_clt_s(__m256 _1, __m256 _2) {`。
- **L3628 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_clt_s((v8f32)_1, (v8f32)_2)`.
  **L3628 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_clt_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3629 EN**: Closes the current lexical scope or compound statement.
  **L3629 CN**: 结束当前词法作用域或复合语句块。
- **L3630 EN**: Blank line separating nearby declarations or logic blocks.
  **L3630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3631 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3631 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3632 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3632 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3633 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cne_d(__m256d _1, __m256d _2) {`.
  **L3633 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cne_d(__m256d _1, __m256d _2) {`。
- **L3634 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cne_d((v4f64)_1, (v4f64)_2)`.
  **L3634 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cne_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3635 EN**: Closes the current lexical scope or compound statement.
  **L3635 CN**: 结束当前词法作用域或复合语句块。
- **L3636 EN**: Blank line separating nearby declarations or logic blocks.
  **L3636 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3637 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3637 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3638 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3638 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3639 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cne_s(__m256 _1, __m256 _2) {`.
  **L3639 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cne_s(__m256 _1, __m256 _2) {`。
- **L3640 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cne_s((v8f32)_1, (v8f32)_2)`.
  **L3640 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cne_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3641 EN**: Closes the current lexical scope or compound statement.
  **L3641 CN**: 结束当前词法作用域或复合语句块。
- **L3642 EN**: Blank line separating nearby declarations or logic blocks.
  **L3642 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3643 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3643 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3644 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3644 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3645 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cor_d(__m256d _1, __m256d _2) {`.
  **L3645 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cor_d(__m256d _1, __m256d _2) {`。
- **L3646 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cor_d((v4f64)_1, (v4f64)_2)`.
  **L3646 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cor_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3647 EN**: Closes the current lexical scope or compound statement.
  **L3647 CN**: 结束当前词法作用域或复合语句块。
- **L3648 EN**: Blank line separating nearby declarations or logic blocks.
  **L3648 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3649-3672

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cor_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cor_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cueq_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cueq_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cueq_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cueq_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cule_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cule_d((v4f64)_1, (v4f64)_2);
}

````
- **L3649 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3649 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3650 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3650 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3651 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cor_s(__m256 _1, __m256 _2) {`.
  **L3651 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cor_s(__m256 _1, __m256 _2) {`。
- **L3652 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cor_s((v8f32)_1, (v8f32)_2)`.
  **L3652 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cor_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3653 EN**: Closes the current lexical scope or compound statement.
  **L3653 CN**: 结束当前词法作用域或复合语句块。
- **L3654 EN**: Blank line separating nearby declarations or logic blocks.
  **L3654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3655 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3655 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3656 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3656 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3657 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cueq_d(__m256d _1, __m256d _2) {`.
  **L3657 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cueq_d(__m256d _1, __m256d _2) {`。
- **L3658 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cueq_d((v4f64)_1, (v4f64)_2)`.
  **L3658 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cueq_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3659 EN**: Closes the current lexical scope or compound statement.
  **L3659 CN**: 结束当前词法作用域或复合语句块。
- **L3660 EN**: Blank line separating nearby declarations or logic blocks.
  **L3660 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3661 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3661 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3662 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3662 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3663 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cueq_s(__m256 _1, __m256 _2) {`.
  **L3663 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cueq_s(__m256 _1, __m256 _2) {`。
- **L3664 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cueq_s((v8f32)_1, (v8f32)_2)`.
  **L3664 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cueq_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3665 EN**: Closes the current lexical scope or compound statement.
  **L3665 CN**: 结束当前词法作用域或复合语句块。
- **L3666 EN**: Blank line separating nearby declarations or logic blocks.
  **L3666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3667 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3667 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3668 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3668 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3669 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cule_d(__m256d _1, __m256d _2) {`.
  **L3669 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cule_d(__m256d _1, __m256d _2) {`。
- **L3670 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cule_d((v4f64)_1, (v4f64)_2)`.
  **L3670 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cule_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3671 EN**: Closes the current lexical scope or compound statement.
  **L3671 CN**: 结束当前词法作用域或复合语句块。
- **L3672 EN**: Blank line separating nearby declarations or logic blocks.
  **L3672 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3673-3696

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cule_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cule_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cult_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cult_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cult_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cult_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cun_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cun_d((v4f64)_1, (v4f64)_2);
}

````
- **L3673 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3673 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3674 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3674 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3675 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cule_s(__m256 _1, __m256 _2) {`.
  **L3675 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cule_s(__m256 _1, __m256 _2) {`。
- **L3676 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cule_s((v8f32)_1, (v8f32)_2)`.
  **L3676 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cule_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3677 EN**: Closes the current lexical scope or compound statement.
  **L3677 CN**: 结束当前词法作用域或复合语句块。
- **L3678 EN**: Blank line separating nearby declarations or logic blocks.
  **L3678 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3679 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3679 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3680 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3680 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3681 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cult_d(__m256d _1, __m256d _2) {`.
  **L3681 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cult_d(__m256d _1, __m256d _2) {`。
- **L3682 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cult_d((v4f64)_1, (v4f64)_2)`.
  **L3682 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cult_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3683 EN**: Closes the current lexical scope or compound statement.
  **L3683 CN**: 结束当前词法作用域或复合语句块。
- **L3684 EN**: Blank line separating nearby declarations or logic blocks.
  **L3684 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3685 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3685 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3686 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3686 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3687 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cult_s(__m256 _1, __m256 _2) {`.
  **L3687 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cult_s(__m256 _1, __m256 _2) {`。
- **L3688 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cult_s((v8f32)_1, (v8f32)_2)`.
  **L3688 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cult_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3689 EN**: Closes the current lexical scope or compound statement.
  **L3689 CN**: 结束当前词法作用域或复合语句块。
- **L3690 EN**: Blank line separating nearby declarations or logic blocks.
  **L3690 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3691 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3691 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3692 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3692 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3693 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cun_d(__m256d _1, __m256d _2) {`.
  **L3693 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cun_d(__m256d _1, __m256d _2) {`。
- **L3694 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cun_d((v4f64)_1, (v4f64)_2)`.
  **L3694 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cun_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3695 EN**: Closes the current lexical scope or compound statement.
  **L3695 CN**: 结束当前词法作用域或复合语句块。
- **L3696 EN**: Blank line separating nearby declarations or logic blocks.
  **L3696 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3697-3720

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cune_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cune_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cune_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cune_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_cun_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_cun_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_saf_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_saf_d((v4f64)_1, (v4f64)_2);
}

````
- **L3697 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3697 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3698 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3698 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3699 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cune_d(__m256d _1, __m256d _2) {`.
  **L3699 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cune_d(__m256d _1, __m256d _2) {`。
- **L3700 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cune_d((v4f64)_1, (v4f64)_2)`.
  **L3700 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cune_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3701 EN**: Closes the current lexical scope or compound statement.
  **L3701 CN**: 结束当前词法作用域或复合语句块。
- **L3702 EN**: Blank line separating nearby declarations or logic blocks.
  **L3702 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3703 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3703 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3704 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3704 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3705 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cune_s(__m256 _1, __m256 _2) {`.
  **L3705 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cune_s(__m256 _1, __m256 _2) {`。
- **L3706 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cune_s((v8f32)_1, (v8f32)_2)`.
  **L3706 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cune_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3707 EN**: Closes the current lexical scope or compound statement.
  **L3707 CN**: 结束当前词法作用域或复合语句块。
- **L3708 EN**: Blank line separating nearby declarations or logic blocks.
  **L3708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3709 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3709 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3710 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3710 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3711 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_cun_s(__m256 _1, __m256 _2) {`.
  **L3711 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_cun_s(__m256 _1, __m256 _2) {`。
- **L3712 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_cun_s((v8f32)_1, (v8f32)_2)`.
  **L3712 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_cun_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3713 EN**: Closes the current lexical scope or compound statement.
  **L3713 CN**: 结束当前词法作用域或复合语句块。
- **L3714 EN**: Blank line separating nearby declarations or logic blocks.
  **L3714 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3715 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3715 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3716 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3716 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3717 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_saf_d(__m256d _1, __m256d _2) {`.
  **L3717 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_saf_d(__m256d _1, __m256d _2) {`。
- **L3718 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_saf_d((v4f64)_1, (v4f64)_2)`.
  **L3718 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_saf_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3719 EN**: Closes the current lexical scope or compound statement.
  **L3719 CN**: 结束当前词法作用域或复合语句块。
- **L3720 EN**: Blank line separating nearby declarations or logic blocks.
  **L3720 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3721-3744

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_saf_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_saf_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_seq_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_seq_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_seq_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_seq_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sle_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sle_d((v4f64)_1, (v4f64)_2);
}

````
- **L3721 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3721 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3722 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3722 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3723 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_saf_s(__m256 _1, __m256 _2) {`.
  **L3723 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_saf_s(__m256 _1, __m256 _2) {`。
- **L3724 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_saf_s((v8f32)_1, (v8f32)_2)`.
  **L3724 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_saf_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3725 EN**: Closes the current lexical scope or compound statement.
  **L3725 CN**: 结束当前词法作用域或复合语句块。
- **L3726 EN**: Blank line separating nearby declarations or logic blocks.
  **L3726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3727 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3727 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3728 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3728 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3729 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_seq_d(__m256d _1, __m256d _2) {`.
  **L3729 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_seq_d(__m256d _1, __m256d _2) {`。
- **L3730 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_seq_d((v4f64)_1, (v4f64)_2)`.
  **L3730 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_seq_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3731 EN**: Closes the current lexical scope or compound statement.
  **L3731 CN**: 结束当前词法作用域或复合语句块。
- **L3732 EN**: Blank line separating nearby declarations or logic blocks.
  **L3732 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3733 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3733 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3734 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3734 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3735 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_seq_s(__m256 _1, __m256 _2) {`.
  **L3735 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_seq_s(__m256 _1, __m256 _2) {`。
- **L3736 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_seq_s((v8f32)_1, (v8f32)_2)`.
  **L3736 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_seq_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3737 EN**: Closes the current lexical scope or compound statement.
  **L3737 CN**: 结束当前词法作用域或复合语句块。
- **L3738 EN**: Blank line separating nearby declarations or logic blocks.
  **L3738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3739 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3739 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3740 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3740 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3741 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sle_d(__m256d _1, __m256d _2) {`.
  **L3741 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sle_d(__m256d _1, __m256d _2) {`。
- **L3742 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sle_d((v4f64)_1, (v4f64)_2)`.
  **L3742 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sle_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3743 EN**: Closes the current lexical scope or compound statement.
  **L3743 CN**: 结束当前词法作用域或复合语句块。
- **L3744 EN**: Blank line separating nearby declarations or logic blocks.
  **L3744 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3745-3768

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sle_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sle_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_slt_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_slt_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_slt_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_slt_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sne_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sne_d((v4f64)_1, (v4f64)_2);
}

````
- **L3745 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3745 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3746 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3746 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3747 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sle_s(__m256 _1, __m256 _2) {`.
  **L3747 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sle_s(__m256 _1, __m256 _2) {`。
- **L3748 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sle_s((v8f32)_1, (v8f32)_2)`.
  **L3748 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sle_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3749 EN**: Closes the current lexical scope or compound statement.
  **L3749 CN**: 结束当前词法作用域或复合语句块。
- **L3750 EN**: Blank line separating nearby declarations or logic blocks.
  **L3750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3751 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3751 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3752 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3752 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3753 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_slt_d(__m256d _1, __m256d _2) {`.
  **L3753 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_slt_d(__m256d _1, __m256d _2) {`。
- **L3754 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_slt_d((v4f64)_1, (v4f64)_2)`.
  **L3754 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_slt_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3755 EN**: Closes the current lexical scope or compound statement.
  **L3755 CN**: 结束当前词法作用域或复合语句块。
- **L3756 EN**: Blank line separating nearby declarations or logic blocks.
  **L3756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3757 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3757 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3758 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3758 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3759 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_slt_s(__m256 _1, __m256 _2) {`.
  **L3759 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_slt_s(__m256 _1, __m256 _2) {`。
- **L3760 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_slt_s((v8f32)_1, (v8f32)_2)`.
  **L3760 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_slt_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3761 EN**: Closes the current lexical scope or compound statement.
  **L3761 CN**: 结束当前词法作用域或复合语句块。
- **L3762 EN**: Blank line separating nearby declarations or logic blocks.
  **L3762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3763 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3763 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3764 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3764 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3765 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sne_d(__m256d _1, __m256d _2) {`.
  **L3765 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sne_d(__m256d _1, __m256d _2) {`。
- **L3766 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sne_d((v4f64)_1, (v4f64)_2)`.
  **L3766 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sne_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3767 EN**: Closes the current lexical scope or compound statement.
  **L3767 CN**: 结束当前词法作用域或复合语句块。
- **L3768 EN**: Blank line separating nearby declarations or logic blocks.
  **L3768 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3769-3792

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sne_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sne_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sor_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sor_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sor_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sor_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sueq_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sueq_d((v4f64)_1, (v4f64)_2);
}

````
- **L3769 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3769 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3770 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3770 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3771 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sne_s(__m256 _1, __m256 _2) {`.
  **L3771 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sne_s(__m256 _1, __m256 _2) {`。
- **L3772 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sne_s((v8f32)_1, (v8f32)_2)`.
  **L3772 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sne_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3773 EN**: Closes the current lexical scope or compound statement.
  **L3773 CN**: 结束当前词法作用域或复合语句块。
- **L3774 EN**: Blank line separating nearby declarations or logic blocks.
  **L3774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3775 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3775 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3776 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3776 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3777 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sor_d(__m256d _1, __m256d _2) {`.
  **L3777 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sor_d(__m256d _1, __m256d _2) {`。
- **L3778 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sor_d((v4f64)_1, (v4f64)_2)`.
  **L3778 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sor_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3779 EN**: Closes the current lexical scope or compound statement.
  **L3779 CN**: 结束当前词法作用域或复合语句块。
- **L3780 EN**: Blank line separating nearby declarations or logic blocks.
  **L3780 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3781 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3781 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3782 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3782 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3783 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sor_s(__m256 _1, __m256 _2) {`.
  **L3783 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sor_s(__m256 _1, __m256 _2) {`。
- **L3784 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sor_s((v8f32)_1, (v8f32)_2)`.
  **L3784 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sor_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3785 EN**: Closes the current lexical scope or compound statement.
  **L3785 CN**: 结束当前词法作用域或复合语句块。
- **L3786 EN**: Blank line separating nearby declarations or logic blocks.
  **L3786 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3787 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3787 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3788 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3788 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3789 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sueq_d(__m256d _1, __m256d _2) {`.
  **L3789 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sueq_d(__m256d _1, __m256d _2) {`。
- **L3790 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sueq_d((v4f64)_1, (v4f64)_2)`.
  **L3790 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sueq_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3791 EN**: Closes the current lexical scope or compound statement.
  **L3791 CN**: 结束当前词法作用域或复合语句块。
- **L3792 EN**: Blank line separating nearby declarations or logic blocks.
  **L3792 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3793-3816

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sueq_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sueq_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sule_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sule_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sule_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sule_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sult_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sult_d((v4f64)_1, (v4f64)_2);
}

````
- **L3793 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3793 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3794 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3794 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3795 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sueq_s(__m256 _1, __m256 _2) {`.
  **L3795 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sueq_s(__m256 _1, __m256 _2) {`。
- **L3796 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sueq_s((v8f32)_1, (v8f32)_2)`.
  **L3796 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sueq_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3797 EN**: Closes the current lexical scope or compound statement.
  **L3797 CN**: 结束当前词法作用域或复合语句块。
- **L3798 EN**: Blank line separating nearby declarations or logic blocks.
  **L3798 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3799 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3799 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3800 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3800 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3801 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sule_d(__m256d _1, __m256d _2) {`.
  **L3801 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sule_d(__m256d _1, __m256d _2) {`。
- **L3802 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sule_d((v4f64)_1, (v4f64)_2)`.
  **L3802 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sule_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3803 EN**: Closes the current lexical scope or compound statement.
  **L3803 CN**: 结束当前词法作用域或复合语句块。
- **L3804 EN**: Blank line separating nearby declarations or logic blocks.
  **L3804 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3805 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3805 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3806 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3806 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3807 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sule_s(__m256 _1, __m256 _2) {`.
  **L3807 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sule_s(__m256 _1, __m256 _2) {`。
- **L3808 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sule_s((v8f32)_1, (v8f32)_2)`.
  **L3808 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sule_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3809 EN**: Closes the current lexical scope or compound statement.
  **L3809 CN**: 结束当前词法作用域或复合语句块。
- **L3810 EN**: Blank line separating nearby declarations or logic blocks.
  **L3810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3811 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3811 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3812 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3812 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3813 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sult_d(__m256d _1, __m256d _2) {`.
  **L3813 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sult_d(__m256d _1, __m256d _2) {`。
- **L3814 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sult_d((v4f64)_1, (v4f64)_2)`.
  **L3814 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sult_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3815 EN**: Closes the current lexical scope or compound statement.
  **L3815 CN**: 结束当前词法作用域或复合语句块。
- **L3816 EN**: Blank line separating nearby declarations or logic blocks.
  **L3816 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3817-3840

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sult_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sult_s((v8f32)_1, (v8f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sun_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sun_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sune_d(__m256d _1, __m256d _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sune_d((v4f64)_1, (v4f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sune_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sune_s((v8f32)_1, (v8f32)_2);
}

````
- **L3817 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3817 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3818 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3818 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3819 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sult_s(__m256 _1, __m256 _2) {`.
  **L3819 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sult_s(__m256 _1, __m256 _2) {`。
- **L3820 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sult_s((v8f32)_1, (v8f32)_2)`.
  **L3820 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sult_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3821 EN**: Closes the current lexical scope or compound statement.
  **L3821 CN**: 结束当前词法作用域或复合语句块。
- **L3822 EN**: Blank line separating nearby declarations or logic blocks.
  **L3822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3823 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3823 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3824 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3824 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3825 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sun_d(__m256d _1, __m256d _2) {`.
  **L3825 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sun_d(__m256d _1, __m256d _2) {`。
- **L3826 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sun_d((v4f64)_1, (v4f64)_2)`.
  **L3826 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sun_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3827 EN**: Closes the current lexical scope or compound statement.
  **L3827 CN**: 结束当前词法作用域或复合语句块。
- **L3828 EN**: Blank line separating nearby declarations or logic blocks.
  **L3828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3829 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3829 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3830 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3830 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3831 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sune_d(__m256d _1, __m256d _2) {`.
  **L3831 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sune_d(__m256d _1, __m256d _2) {`。
- **L3832 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sune_d((v4f64)_1, (v4f64)_2)`.
  **L3832 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sune_d((v4f64)_1, (v4f64)_2)` 从当前函数返回。
- **L3833 EN**: Closes the current lexical scope or compound statement.
  **L3833 CN**: 结束当前词法作用域或复合语句块。
- **L3834 EN**: Blank line separating nearby declarations or logic blocks.
  **L3834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3835 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3835 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3836 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3836 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3837 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sune_s(__m256 _1, __m256 _2) {`.
  **L3837 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sune_s(__m256 _1, __m256 _2) {`。
- **L3838 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sune_s((v8f32)_1, (v8f32)_2)`.
  **L3838 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sune_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3839 EN**: Closes the current lexical scope or compound statement.
  **L3839 CN**: 结束当前词法作用域或复合语句块。
- **L3840 EN**: Blank line separating nearby declarations or logic blocks.
  **L3840 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3841-3864

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_xvfcmp_sun_s(__m256 _1, __m256 _2) {
  return (__m256i)__builtin_lasx_xvfcmp_sun_s((v8f32)_1, (v8f32)_2);
}

#if defined(__loongarch_frecipe)
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrecipe_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrecipe_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrecipe_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrecipe_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_xvfrsqrte_s(__m256 _1) {
  return (__m256)__builtin_lasx_xvfrsqrte_s((v8f32)_1);
}
````
- **L3841 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3841 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3842 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3842 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3843 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfcmp_sun_s(__m256 _1, __m256 _2) {`.
  **L3843 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfcmp_sun_s(__m256 _1, __m256 _2) {`。
- **L3844 EN**: Returns from the current function with `(__m256i)__builtin_lasx_xvfcmp_sun_s((v8f32)_1, (v8f32)_2)`.
  **L3844 CN**: 以 `(__m256i)__builtin_lasx_xvfcmp_sun_s((v8f32)_1, (v8f32)_2)` 从当前函数返回。
- **L3845 EN**: Closes the current lexical scope or compound statement.
  **L3845 CN**: 结束当前词法作用域或复合语句块。
- **L3846 EN**: Blank line separating nearby declarations or logic blocks.
  **L3846 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3847 EN**: Starts a preprocessor conditional block: `#if defined(__loongarch_frecipe)`.
  **L3847 CN**: 开始一个预处理条件块：`#if defined(__loongarch_frecipe)`。
- **L3848 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3848 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3849 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L3849 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L3850 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrecipe_s(__m256 _1) {`.
  **L3850 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrecipe_s(__m256 _1) {`。
- **L3851 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrecipe_s((v8f32)_1)`.
  **L3851 CN**: 以 `(__m256)__builtin_lasx_xvfrecipe_s((v8f32)_1)` 从当前函数返回。
- **L3852 EN**: Closes the current lexical scope or compound statement.
  **L3852 CN**: 结束当前词法作用域或复合语句块。
- **L3853 EN**: Blank line separating nearby declarations or logic blocks.
  **L3853 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3854 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3854 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3855 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L3855 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L3856 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrecipe_d(__m256d _1) {`.
  **L3856 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrecipe_d(__m256d _1) {`。
- **L3857 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrecipe_d((v4f64)_1)`.
  **L3857 CN**: 以 `(__m256d)__builtin_lasx_xvfrecipe_d((v4f64)_1)` 从当前函数返回。
- **L3858 EN**: Closes the current lexical scope or compound statement.
  **L3858 CN**: 结束当前词法作用域或复合语句块。
- **L3859 EN**: Blank line separating nearby declarations or logic blocks.
  **L3859 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3860 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3860 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3861 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L3861 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L3862 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrsqrte_s(__m256 _1) {`.
  **L3862 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrsqrte_s(__m256 _1) {`。
- **L3863 EN**: Returns from the current function with `(__m256)__builtin_lasx_xvfrsqrte_s((v8f32)_1)`.
  **L3863 CN**: 以 `(__m256)__builtin_lasx_xvfrsqrte_s((v8f32)_1)` 从当前函数返回。
- **L3864 EN**: Closes the current lexical scope or compound statement.
  **L3864 CN**: 结束当前词法作用域或复合语句块。

### Lines 3865-3888

````c

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_xvfrsqrte_d(__m256d _1) {
  return (__m256d)__builtin_lasx_xvfrsqrte_d((v4f64)_1);
}
#endif

#define __lasx_xvpickve_d_f(/*__m256d*/ _1, /*ui2*/ _2)                        \
  ((__m256d)__builtin_lasx_xvpickve_d_f((v4f64)(_1), (_2)))

#define __lasx_xvpickve_w_f(/*__m256*/ _1, /*ui3*/ _2)                         \
  ((__m256)__builtin_lasx_xvpickve_w_f((v8f32)(_1), (_2)))

#define __lasx_xvrepli_b(/*si10*/ _1) ((__m256i)__builtin_lasx_xvrepli_b((_1)))

#define __lasx_xvrepli_d(/*si10*/ _1) ((__m256i)__builtin_lasx_xvrepli_d((_1)))

#define __lasx_xvrepli_h(/*si10*/ _1) ((__m256i)__builtin_lasx_xvrepli_h((_1)))

#define __lasx_xvrepli_w(/*si10*/ _1) ((__m256i)__builtin_lasx_xvrepli_w((_1)))

#if defined(__loongarch_asx_sx_conv)

````
- **L3865 EN**: Blank line separating nearby declarations or logic blocks.
  **L3865 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3866 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3866 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3867 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L3867 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L3868 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_xvfrsqrte_d(__m256d _1) {`.
  **L3868 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_xvfrsqrte_d(__m256d _1) {`。
- **L3869 EN**: Returns from the current function with `(__m256d)__builtin_lasx_xvfrsqrte_d((v4f64)_1)`.
  **L3869 CN**: 以 `(__m256d)__builtin_lasx_xvfrsqrte_d((v4f64)_1)` 从当前函数返回。
- **L3870 EN**: Closes the current lexical scope or compound statement.
  **L3870 CN**: 结束当前词法作用域或复合语句块。
- **L3871 EN**: Closes the current preprocessor conditional block.
  **L3871 CN**: 结束当前预处理条件块。
- **L3872 EN**: Blank line separating nearby declarations or logic blocks.
  **L3872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3873 EN**: Defines macro `__lasx_xvpickve_d_f(/*__m256d*/ _1, /*ui2*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3873 CN**: 定义宏 `__lasx_xvpickve_d_f(/*__m256d*/ _1, /*ui2*/ _2)`，用于条件编译、简写或 API 生成。
- **L3874 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpickve_d_f`.
  **L3874 CN**: 继续与可调用符号 `__builtin_lasx_xvpickve_d_f` 相关的逻辑。
- **L3875 EN**: Blank line separating nearby declarations or logic blocks.
  **L3875 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3876 EN**: Defines macro `__lasx_xvpickve_w_f(/*__m256*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3876 CN**: 定义宏 `__lasx_xvpickve_w_f(/*__m256*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L3877 EN**: Continues logic associated with callable symbol `__builtin_lasx_xvpickve_w_f`.
  **L3877 CN**: 继续与可调用符号 `__builtin_lasx_xvpickve_w_f` 相关的逻辑。
- **L3878 EN**: Blank line separating nearby declarations or logic blocks.
  **L3878 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3879 EN**: Defines macro `__lasx_xvrepli_b(/*si10*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3879 CN**: 定义宏 `__lasx_xvrepli_b(/*si10*/ _1)`，用于条件编译、简写或 API 生成。
- **L3880 EN**: Blank line separating nearby declarations or logic blocks.
  **L3880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3881 EN**: Defines macro `__lasx_xvrepli_d(/*si10*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3881 CN**: 定义宏 `__lasx_xvrepli_d(/*si10*/ _1)`，用于条件编译、简写或 API 生成。
- **L3882 EN**: Blank line separating nearby declarations or logic blocks.
  **L3882 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3883 EN**: Defines macro `__lasx_xvrepli_h(/*si10*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3883 CN**: 定义宏 `__lasx_xvrepli_h(/*si10*/ _1)`，用于条件编译、简写或 API 生成。
- **L3884 EN**: Blank line separating nearby declarations or logic blocks.
  **L3884 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3885 EN**: Defines macro `__lasx_xvrepli_w(/*si10*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3885 CN**: 定义宏 `__lasx_xvrepli_w(/*si10*/ _1)`，用于条件编译、简写或 API 生成。
- **L3886 EN**: Blank line separating nearby declarations or logic blocks.
  **L3886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3887 EN**: Starts a preprocessor conditional block: `#if defined(__loongarch_asx_sx_conv)`.
  **L3887 CN**: 开始一个预处理条件块：`#if defined(__loongarch_asx_sx_conv)`。
- **L3888 EN**: Blank line separating nearby declarations or logic blocks.
  **L3888 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3889-3912

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__,
                   __artificial__)) __m256 __lasx_cast_128_s(__m128 _1) {
  return (__m256)__builtin_lasx_cast_128_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_cast_128_d(__m128d _1) {
  return (__m256d)__builtin_lasx_cast_128_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_cast_128(__m128i _1) {
  return (__m256i)__builtin_lasx_cast_128((v2i64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_concat_128_s(__m128 _1, __m128 _2) {
  return (__m256)__builtin_lasx_concat_128_s((v4f32)_1, (v4f32)_2);
}

````
- **L3889 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3889 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3890 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__,`.
  **L3890 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__,`。
- **L3891 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__artificial__)) __m256 __lasx_cast_128_s(__m128 _1) {`.
  **L3891 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__artificial__)) __m256 __lasx_cast_128_s(__m128 _1) {`。
- **L3892 EN**: Returns from the current function with `(__m256)__builtin_lasx_cast_128_s((v4f32)_1)`.
  **L3892 CN**: 以 `(__m256)__builtin_lasx_cast_128_s((v4f32)_1)` 从当前函数返回。
- **L3893 EN**: Closes the current lexical scope or compound statement.
  **L3893 CN**: 结束当前词法作用域或复合语句块。
- **L3894 EN**: Blank line separating nearby declarations or logic blocks.
  **L3894 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3895 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3895 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3896 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L3896 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L3897 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_cast_128_d(__m128d _1) {`.
  **L3897 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_cast_128_d(__m128d _1) {`。
- **L3898 EN**: Returns from the current function with `(__m256d)__builtin_lasx_cast_128_d((v2f64)_1)`.
  **L3898 CN**: 以 `(__m256d)__builtin_lasx_cast_128_d((v2f64)_1)` 从当前函数返回。
- **L3899 EN**: Closes the current lexical scope or compound statement.
  **L3899 CN**: 结束当前词法作用域或复合语句块。
- **L3900 EN**: Blank line separating nearby declarations or logic blocks.
  **L3900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3901 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3901 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3902 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3902 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3903 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_cast_128(__m128i _1) {`.
  **L3903 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_cast_128(__m128i _1) {`。
- **L3904 EN**: Returns from the current function with `(__m256i)__builtin_lasx_cast_128((v2i64)_1)`.
  **L3904 CN**: 以 `(__m256i)__builtin_lasx_cast_128((v2i64)_1)` 从当前函数返回。
- **L3905 EN**: Closes the current lexical scope or compound statement.
  **L3905 CN**: 结束当前词法作用域或复合语句块。
- **L3906 EN**: Blank line separating nearby declarations or logic blocks.
  **L3906 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3907 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3907 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3908 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L3908 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L3909 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_concat_128_s(__m128 _1, __m128 _2) {`.
  **L3909 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_concat_128_s(__m128 _1, __m128 _2) {`。
- **L3910 EN**: Returns from the current function with `(__m256)__builtin_lasx_concat_128_s((v4f32)_1, (v4f32)_2)`.
  **L3910 CN**: 以 `(__m256)__builtin_lasx_concat_128_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3911 EN**: Closes the current lexical scope or compound statement.
  **L3911 CN**: 结束当前词法作用域或复合语句块。
- **L3912 EN**: Blank line separating nearby declarations or logic blocks.
  **L3912 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3913-3936

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_concat_128_d(__m128d _1, __m128d _2) {
  return (__m256d)__builtin_lasx_concat_128_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_concat_128(__m128i _1, __m128i _2) {
  return (__m256i)__builtin_lasx_concat_128((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lasx_extract_128_lo_s(__m256 _1) {
  return (__m128)__builtin_lasx_extract_128_lo_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lasx_extract_128_lo_d(__m256d _1) {
  return (__m128d)__builtin_lasx_extract_128_lo_d((v4f64)_1);
}

````
- **L3913 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3913 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3914 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L3914 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L3915 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_concat_128_d(__m128d _1, __m128d _2) {`.
  **L3915 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_concat_128_d(__m128d _1, __m128d _2) {`。
- **L3916 EN**: Returns from the current function with `(__m256d)__builtin_lasx_concat_128_d((v2f64)_1, (v2f64)_2)`.
  **L3916 CN**: 以 `(__m256d)__builtin_lasx_concat_128_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3917 EN**: Closes the current lexical scope or compound statement.
  **L3917 CN**: 结束当前词法作用域或复合语句块。
- **L3918 EN**: Blank line separating nearby declarations or logic blocks.
  **L3918 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3919 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3919 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3920 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3920 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3921 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_concat_128(__m128i _1, __m128i _2) {`.
  **L3921 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_concat_128(__m128i _1, __m128i _2) {`。
- **L3922 EN**: Returns from the current function with `(__m256i)__builtin_lasx_concat_128((v2i64)_1, (v2i64)_2)`.
  **L3922 CN**: 以 `(__m256i)__builtin_lasx_concat_128((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L3923 EN**: Closes the current lexical scope or compound statement.
  **L3923 CN**: 结束当前词法作用域或复合语句块。
- **L3924 EN**: Blank line separating nearby declarations or logic blocks.
  **L3924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3925 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3925 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3926 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L3926 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L3927 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_extract_128_lo_s(__m256 _1) {`.
  **L3927 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_extract_128_lo_s(__m256 _1) {`。
- **L3928 EN**: Returns from the current function with `(__m128)__builtin_lasx_extract_128_lo_s((v8f32)_1)`.
  **L3928 CN**: 以 `(__m128)__builtin_lasx_extract_128_lo_s((v8f32)_1)` 从当前函数返回。
- **L3929 EN**: Closes the current lexical scope or compound statement.
  **L3929 CN**: 结束当前词法作用域或复合语句块。
- **L3930 EN**: Blank line separating nearby declarations or logic blocks.
  **L3930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3931 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3931 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3932 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L3932 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L3933 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_extract_128_lo_d(__m256d _1) {`.
  **L3933 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_extract_128_lo_d(__m256d _1) {`。
- **L3934 EN**: Returns from the current function with `(__m128d)__builtin_lasx_extract_128_lo_d((v4f64)_1)`.
  **L3934 CN**: 以 `(__m128d)__builtin_lasx_extract_128_lo_d((v4f64)_1)` 从当前函数返回。
- **L3935 EN**: Closes the current lexical scope or compound statement.
  **L3935 CN**: 结束当前词法作用域或复合语句块。
- **L3936 EN**: Blank line separating nearby declarations or logic blocks.
  **L3936 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3937-3960

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lasx_extract_128_lo(__m256i _1) {
  return (__m128i)__builtin_lasx_extract_128_lo((v4i64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lasx_extract_128_hi_s(__m256 _1) {
  return (__m128)__builtin_lasx_extract_128_hi_s((v8f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lasx_extract_128_hi_d(__m256d _1) {
  return (__m128d)__builtin_lasx_extract_128_hi_d((v4f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lasx_extract_128_hi(__m256i _1) {
  return (__m128i)__builtin_lasx_extract_128_hi((v4i64)_1);
}

````
- **L3937 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3937 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3938 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3938 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3939 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_extract_128_lo(__m256i _1) {`.
  **L3939 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_extract_128_lo(__m256i _1) {`。
- **L3940 EN**: Returns from the current function with `(__m128i)__builtin_lasx_extract_128_lo((v4i64)_1)`.
  **L3940 CN**: 以 `(__m128i)__builtin_lasx_extract_128_lo((v4i64)_1)` 从当前函数返回。
- **L3941 EN**: Closes the current lexical scope or compound statement.
  **L3941 CN**: 结束当前词法作用域或复合语句块。
- **L3942 EN**: Blank line separating nearby declarations or logic blocks.
  **L3942 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3943 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3943 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3944 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L3944 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L3945 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_extract_128_hi_s(__m256 _1) {`.
  **L3945 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_extract_128_hi_s(__m256 _1) {`。
- **L3946 EN**: Returns from the current function with `(__m128)__builtin_lasx_extract_128_hi_s((v8f32)_1)`.
  **L3946 CN**: 以 `(__m128)__builtin_lasx_extract_128_hi_s((v8f32)_1)` 从当前函数返回。
- **L3947 EN**: Closes the current lexical scope or compound statement.
  **L3947 CN**: 结束当前词法作用域或复合语句块。
- **L3948 EN**: Blank line separating nearby declarations or logic blocks.
  **L3948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3949 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3949 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3950 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L3950 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L3951 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_extract_128_hi_d(__m256d _1) {`.
  **L3951 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_extract_128_hi_d(__m256d _1) {`。
- **L3952 EN**: Returns from the current function with `(__m128d)__builtin_lasx_extract_128_hi_d((v4f64)_1)`.
  **L3952 CN**: 以 `(__m128d)__builtin_lasx_extract_128_hi_d((v4f64)_1)` 从当前函数返回。
- **L3953 EN**: Closes the current lexical scope or compound statement.
  **L3953 CN**: 结束当前词法作用域或复合语句块。
- **L3954 EN**: Blank line separating nearby declarations or logic blocks.
  **L3954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3955 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3955 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3956 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3956 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3957 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_extract_128_hi(__m256i _1) {`.
  **L3957 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_extract_128_hi(__m256i _1) {`。
- **L3958 EN**: Returns from the current function with `(__m128i)__builtin_lasx_extract_128_hi((v4i64)_1)`.
  **L3958 CN**: 以 `(__m128i)__builtin_lasx_extract_128_hi((v4i64)_1)` 从当前函数返回。
- **L3959 EN**: Closes the current lexical scope or compound statement.
  **L3959 CN**: 结束当前词法作用域或复合语句块。
- **L3960 EN**: Blank line separating nearby declarations or logic blocks.
  **L3960 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3961-3984

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_insert_128_lo_s(__m256 _1, __m128 _2) {
  return (__m256)__builtin_lasx_insert_128_lo_s((v8f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_insert_128_lo_d(__m256d _1, __m128d _2) {
  return (__m256d)__builtin_lasx_insert_128_lo_d((v4f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_insert_128_lo(__m256i _1, __m128i _2) {
  return (__m256i)__builtin_lasx_insert_128_lo((v4i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256
    __lasx_insert_128_hi_s(__m256 _1, __m128 _2) {
  return (__m256)__builtin_lasx_insert_128_hi_s((v8f32)_1, (v4f32)_2);
}

````
- **L3961 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3961 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3962 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L3962 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L3963 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_insert_128_lo_s(__m256 _1, __m128 _2) {`.
  **L3963 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_insert_128_lo_s(__m256 _1, __m128 _2) {`。
- **L3964 EN**: Returns from the current function with `(__m256)__builtin_lasx_insert_128_lo_s((v8f32)_1, (v4f32)_2)`.
  **L3964 CN**: 以 `(__m256)__builtin_lasx_insert_128_lo_s((v8f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3965 EN**: Closes the current lexical scope or compound statement.
  **L3965 CN**: 结束当前词法作用域或复合语句块。
- **L3966 EN**: Blank line separating nearby declarations or logic blocks.
  **L3966 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3967 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3967 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3968 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L3968 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L3969 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_insert_128_lo_d(__m256d _1, __m128d _2) {`.
  **L3969 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_insert_128_lo_d(__m256d _1, __m128d _2) {`。
- **L3970 EN**: Returns from the current function with `(__m256d)__builtin_lasx_insert_128_lo_d((v4f64)_1, (v2f64)_2)`.
  **L3970 CN**: 以 `(__m256d)__builtin_lasx_insert_128_lo_d((v4f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3971 EN**: Closes the current lexical scope or compound statement.
  **L3971 CN**: 结束当前词法作用域或复合语句块。
- **L3972 EN**: Blank line separating nearby declarations or logic blocks.
  **L3972 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3973 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3973 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3974 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3974 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3975 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_insert_128_lo(__m256i _1, __m128i _2) {`.
  **L3975 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_insert_128_lo(__m256i _1, __m128i _2) {`。
- **L3976 EN**: Returns from the current function with `(__m256i)__builtin_lasx_insert_128_lo((v4i64)_1, (v2i64)_2)`.
  **L3976 CN**: 以 `(__m256i)__builtin_lasx_insert_128_lo((v4i64)_1, (v2i64)_2)` 从当前函数返回。
- **L3977 EN**: Closes the current lexical scope or compound statement.
  **L3977 CN**: 结束当前词法作用域或复合语句块。
- **L3978 EN**: Blank line separating nearby declarations or logic blocks.
  **L3978 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3979 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3979 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3980 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`.
  **L3980 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256`。
- **L3981 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_insert_128_hi_s(__m256 _1, __m128 _2) {`.
  **L3981 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_insert_128_hi_s(__m256 _1, __m128 _2) {`。
- **L3982 EN**: Returns from the current function with `(__m256)__builtin_lasx_insert_128_hi_s((v8f32)_1, (v4f32)_2)`.
  **L3982 CN**: 以 `(__m256)__builtin_lasx_insert_128_hi_s((v8f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3983 EN**: Closes the current lexical scope or compound statement.
  **L3983 CN**: 结束当前词法作用域或复合语句块。
- **L3984 EN**: Blank line separating nearby declarations or logic blocks.
  **L3984 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3985-3999

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d
    __lasx_insert_128_hi_d(__m256d _1, __m128d _2) {
  return (__m256d)__builtin_lasx_insert_128_hi_d((v4f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i
    __lasx_insert_128_hi(__m256i _1, __m128i _2) {
  return (__m256i)__builtin_lasx_insert_128_hi((v4i64)_1, (v2i64)_2);
}

#endif /* defined(__loongarch_asx_sx_conv).  */
#endif /* defined(__loongarch_asx).  */
#endif /* _LOONGSON_ASXINTRIN_H.  */
````
- **L3985 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3985 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3986 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`.
  **L3986 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256d`。
- **L3987 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_insert_128_hi_d(__m256d _1, __m128d _2) {`.
  **L3987 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_insert_128_hi_d(__m256d _1, __m128d _2) {`。
- **L3988 EN**: Returns from the current function with `(__m256d)__builtin_lasx_insert_128_hi_d((v4f64)_1, (v2f64)_2)`.
  **L3988 CN**: 以 `(__m256d)__builtin_lasx_insert_128_hi_d((v4f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3989 EN**: Closes the current lexical scope or compound statement.
  **L3989 CN**: 结束当前词法作用域或复合语句块。
- **L3990 EN**: Blank line separating nearby declarations or logic blocks.
  **L3990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3991 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3991 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3992 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`.
  **L3992 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m256i`。
- **L3993 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lasx_insert_128_hi(__m256i _1, __m128i _2) {`.
  **L3993 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lasx_insert_128_hi(__m256i _1, __m128i _2) {`。
- **L3994 EN**: Returns from the current function with `(__m256i)__builtin_lasx_insert_128_hi((v4i64)_1, (v2i64)_2)`.
  **L3994 CN**: 以 `(__m256i)__builtin_lasx_insert_128_hi((v4i64)_1, (v2i64)_2)` 从当前函数返回。
- **L3995 EN**: Closes the current lexical scope or compound statement.
  **L3995 CN**: 结束当前词法作用域或复合语句块。
- **L3996 EN**: Blank line separating nearby declarations or logic blocks.
  **L3996 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3997 EN**: Closes the current preprocessor conditional block.
  **L3997 CN**: 结束当前预处理条件块。
- **L3998 EN**: Closes the current preprocessor conditional block.
  **L3998 CN**: 结束当前预处理条件块。
- **L3999 EN**: Closes the current preprocessor conditional block.
  **L3999 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **LoongArch intrinsics / LoongArch intrinsic 接口**
- **Header composition through includes / 通过 include 组合头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `lsxintrin.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `_LOONGSON_ASXINTRIN_H`, `__loongarch_asx`, `__loongarch_frecipe`, `__loongarch_asx_sx_conv`
- **External builtins / 外部 builtin**: `__builtin_lasx_xvsll_b`, `__builtin_lasx_xvsll_h`, `__builtin_lasx_xvsll_w`, `__builtin_lasx_xvsll_d`, `__builtin_lasx_xvslli_b`, `__builtin_lasx_xvslli_h`, `__builtin_lasx_xvslli_w`, `__builtin_lasx_xvslli_d`, `__builtin_lasx_xvsra_b`, `__builtin_lasx_xvsra_h`, `__builtin_lasx_xvsra_w`, `__builtin_lasx_xvsra_d`
