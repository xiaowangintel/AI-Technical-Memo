# lsxintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/lsxintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: LoongArch LSX intrinsics.
- **Purpose (CN)**: 提供 LoongArch LSX intrinsic 接口。
- **Line Count / 行数**: 3752

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===------------- lsxintrin.h - LoongArch LSX intrinsics ------------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef _LOONGSON_SXINTRIN_H
#define _LOONGSON_SXINTRIN_H 1

#if defined(__loongarch_sx)
typedef signed char v16i8 __attribute__((vector_size(16), aligned(16)));
typedef signed char v16i8_b __attribute__((vector_size(16), aligned(1)));
typedef unsigned char v16u8 __attribute__((vector_size(16), aligned(16)));
typedef unsigned char v16u8_b __attribute__((vector_size(16), aligned(1)));
typedef short v8i16 __attribute__((vector_size(16), aligned(16)));
typedef short v8i16_h __attribute__((vector_size(16), aligned(2)));
typedef unsigned short v8u16 __attribute__((vector_size(16), aligned(16)));
typedef unsigned short v8u16_h __attribute__((vector_size(16), aligned(2)));
typedef int v4i32 __attribute__((vector_size(16), aligned(16)));
typedef int v4i32_w __attribute__((vector_size(16), aligned(4)));
typedef unsigned int v4u32 __attribute__((vector_size(16), aligned(16)));
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
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef _LOONGSON_SXINTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef _LOONGSON_SXINTRIN_H`。
- **L11 EN**: Defines macro `_LOONGSON_SXINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `_LOONGSON_SXINTRIN_H`，用于条件编译、简写或 API 生成。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#if defined(__loongarch_sx)`.
  **L13 CN**: 开始一个预处理条件块：`#if defined(__loongarch_sx)`。
- **L14 EN**: Introduces an alias or helper declaration: `typedef signed char v16i8 __attribute__((vector_size(16), aligned(16)));`.
  **L14 CN**: 引入一条别名或辅助声明：`typedef signed char v16i8 __attribute__((vector_size(16), aligned(16)));`。
- **L15 EN**: Introduces an alias or helper declaration: `typedef signed char v16i8_b __attribute__((vector_size(16), aligned(1)));`.
  **L15 CN**: 引入一条别名或辅助声明：`typedef signed char v16i8_b __attribute__((vector_size(16), aligned(1)));`。
- **L16 EN**: Introduces an alias or helper declaration: `typedef unsigned char v16u8 __attribute__((vector_size(16), aligned(16)));`.
  **L16 CN**: 引入一条别名或辅助声明：`typedef unsigned char v16u8 __attribute__((vector_size(16), aligned(16)));`。
- **L17 EN**: Introduces an alias or helper declaration: `typedef unsigned char v16u8_b __attribute__((vector_size(16), aligned(1)));`.
  **L17 CN**: 引入一条别名或辅助声明：`typedef unsigned char v16u8_b __attribute__((vector_size(16), aligned(1)));`。
- **L18 EN**: Introduces an alias or helper declaration: `typedef short v8i16 __attribute__((vector_size(16), aligned(16)));`.
  **L18 CN**: 引入一条别名或辅助声明：`typedef short v8i16 __attribute__((vector_size(16), aligned(16)));`。
- **L19 EN**: Introduces an alias or helper declaration: `typedef short v8i16_h __attribute__((vector_size(16), aligned(2)));`.
  **L19 CN**: 引入一条别名或辅助声明：`typedef short v8i16_h __attribute__((vector_size(16), aligned(2)));`。
- **L20 EN**: Introduces an alias or helper declaration: `typedef unsigned short v8u16 __attribute__((vector_size(16), aligned(16)));`.
  **L20 CN**: 引入一条别名或辅助声明：`typedef unsigned short v8u16 __attribute__((vector_size(16), aligned(16)));`。
- **L21 EN**: Introduces an alias or helper declaration: `typedef unsigned short v8u16_h __attribute__((vector_size(16), aligned(2)));`.
  **L21 CN**: 引入一条别名或辅助声明：`typedef unsigned short v8u16_h __attribute__((vector_size(16), aligned(2)));`。
- **L22 EN**: Introduces an alias or helper declaration: `typedef int v4i32 __attribute__((vector_size(16), aligned(16)));`.
  **L22 CN**: 引入一条别名或辅助声明：`typedef int v4i32 __attribute__((vector_size(16), aligned(16)));`。
- **L23 EN**: Introduces an alias or helper declaration: `typedef int v4i32_w __attribute__((vector_size(16), aligned(4)));`.
  **L23 CN**: 引入一条别名或辅助声明：`typedef int v4i32_w __attribute__((vector_size(16), aligned(4)));`。
- **L24 EN**: Introduces an alias or helper declaration: `typedef unsigned int v4u32 __attribute__((vector_size(16), aligned(16)));`.
  **L24 CN**: 引入一条别名或辅助声明：`typedef unsigned int v4u32 __attribute__((vector_size(16), aligned(16)));`。

### Lines 25-48

````c
typedef unsigned int v4u32_w __attribute__((vector_size(16), aligned(4)));
typedef long long v2i64 __attribute__((vector_size(16), aligned(16)));
typedef long long v2i64_d __attribute__((vector_size(16), aligned(8)));
typedef unsigned long long v2u64 __attribute__((vector_size(16), aligned(16)));
typedef unsigned long long v2u64_d __attribute__((vector_size(16), aligned(8)));
typedef float v4f32 __attribute__((vector_size(16), aligned(16)));
typedef float v4f32_w __attribute__((vector_size(16), aligned(4)));
typedef double v2f64 __attribute__((vector_size(16), aligned(16)));
typedef double v2f64_d __attribute__((vector_size(16), aligned(8)));

typedef long long __m128i __attribute__((__vector_size__(16), __may_alias__));
typedef float __m128 __attribute__((__vector_size__(16), __may_alias__));
typedef double __m128d __attribute__((__vector_size__(16), __may_alias__));

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsll_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsll_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsll_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsll_h((v8i16)_1, (v8i16)_2);
````
- **L25 EN**: Introduces an alias or helper declaration: `typedef unsigned int v4u32_w __attribute__((vector_size(16), aligned(4)));`.
  **L25 CN**: 引入一条别名或辅助声明：`typedef unsigned int v4u32_w __attribute__((vector_size(16), aligned(4)));`。
- **L26 EN**: Introduces an alias or helper declaration: `typedef long long v2i64 __attribute__((vector_size(16), aligned(16)));`.
  **L26 CN**: 引入一条别名或辅助声明：`typedef long long v2i64 __attribute__((vector_size(16), aligned(16)));`。
- **L27 EN**: Introduces an alias or helper declaration: `typedef long long v2i64_d __attribute__((vector_size(16), aligned(8)));`.
  **L27 CN**: 引入一条别名或辅助声明：`typedef long long v2i64_d __attribute__((vector_size(16), aligned(8)));`。
- **L28 EN**: Introduces an alias or helper declaration: `typedef unsigned long long v2u64 __attribute__((vector_size(16), aligned(16)));`.
  **L28 CN**: 引入一条别名或辅助声明：`typedef unsigned long long v2u64 __attribute__((vector_size(16), aligned(16)));`。
- **L29 EN**: Introduces an alias or helper declaration: `typedef unsigned long long v2u64_d __attribute__((vector_size(16), aligned(8)));`.
  **L29 CN**: 引入一条别名或辅助声明：`typedef unsigned long long v2u64_d __attribute__((vector_size(16), aligned(8)));`。
- **L30 EN**: Introduces an alias or helper declaration: `typedef float v4f32 __attribute__((vector_size(16), aligned(16)));`.
  **L30 CN**: 引入一条别名或辅助声明：`typedef float v4f32 __attribute__((vector_size(16), aligned(16)));`。
- **L31 EN**: Introduces an alias or helper declaration: `typedef float v4f32_w __attribute__((vector_size(16), aligned(4)));`.
  **L31 CN**: 引入一条别名或辅助声明：`typedef float v4f32_w __attribute__((vector_size(16), aligned(4)));`。
- **L32 EN**: Introduces an alias or helper declaration: `typedef double v2f64 __attribute__((vector_size(16), aligned(16)));`.
  **L32 CN**: 引入一条别名或辅助声明：`typedef double v2f64 __attribute__((vector_size(16), aligned(16)));`。
- **L33 EN**: Introduces an alias or helper declaration: `typedef double v2f64_d __attribute__((vector_size(16), aligned(8)));`.
  **L33 CN**: 引入一条别名或辅助声明：`typedef double v2f64_d __attribute__((vector_size(16), aligned(8)));`。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Introduces an alias or helper declaration: `typedef long long __m128i __attribute__((__vector_size__(16), __may_alias__));`.
  **L35 CN**: 引入一条别名或辅助声明：`typedef long long __m128i __attribute__((__vector_size__(16), __may_alias__));`。
- **L36 EN**: Introduces an alias or helper declaration: `typedef float __m128 __attribute__((__vector_size__(16), __may_alias__));`.
  **L36 CN**: 引入一条别名或辅助声明：`typedef float __m128 __attribute__((__vector_size__(16), __may_alias__));`。
- **L37 EN**: Introduces an alias or helper declaration: `typedef double __m128d __attribute__((__vector_size__(16), __may_alias__));`.
  **L37 CN**: 引入一条别名或辅助声明：`typedef double __m128d __attribute__((__vector_size__(16), __may_alias__));`。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L39 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L39 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L40 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L40 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L41 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsll_b(__m128i _1, __m128i _2) {`.
  **L41 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsll_b(__m128i _1, __m128i _2) {`。
- **L42 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsll_b((v16i8)_1, (v16i8)_2)`.
  **L42 CN**: 以 `(__m128i)__builtin_lsx_vsll_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L43 EN**: Closes the current lexical scope or compound statement.
  **L43 CN**: 结束当前词法作用域或复合语句块。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L45 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L45 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L46 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L46 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L47 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsll_h(__m128i _1, __m128i _2) {`.
  **L47 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsll_h(__m128i _1, __m128i _2) {`。
- **L48 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsll_h((v8i16)_1, (v8i16)_2)`.
  **L48 CN**: 以 `(__m128i)__builtin_lsx_vsll_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 49-72

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsll_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsll_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsll_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsll_d((v2i64)_1, (v2i64)_2);
}

#define __lsx_vslli_b(/*__m128i*/ _1, /*ui3*/ _2)                              \
  ((__m128i)__builtin_lsx_vslli_b((v16i8)(_1), (_2)))

#define __lsx_vslli_h(/*__m128i*/ _1, /*ui4*/ _2)                              \
  ((__m128i)__builtin_lsx_vslli_h((v8i16)(_1), (_2)))

#define __lsx_vslli_w(/*__m128i*/ _1, /*ui5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslli_w((v4i32)(_1), (_2)))

#define __lsx_vslli_d(/*__m128i*/ _1, /*ui6*/ _2)                              \
````
- **L49 EN**: Closes the current lexical scope or compound statement.
  **L49 CN**: 结束当前词法作用域或复合语句块。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L51 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L51 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L52 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L52 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L53 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsll_w(__m128i _1, __m128i _2) {`.
  **L53 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsll_w(__m128i _1, __m128i _2) {`。
- **L54 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsll_w((v4i32)_1, (v4i32)_2)`.
  **L54 CN**: 以 `(__m128i)__builtin_lsx_vsll_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L57 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L57 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L58 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L58 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L59 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsll_d(__m128i _1, __m128i _2) {`.
  **L59 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsll_d(__m128i _1, __m128i _2) {`。
- **L60 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsll_d((v2i64)_1, (v2i64)_2)`.
  **L60 CN**: 以 `(__m128i)__builtin_lsx_vsll_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Defines macro `__lsx_vslli_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L63 CN**: 定义宏 `__lsx_vslli_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L64 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslli_b`.
  **L64 CN**: 继续与可调用符号 `__builtin_lsx_vslli_b` 相关的逻辑。
- **L65 EN**: Blank line separating nearby declarations or logic blocks.
  **L65 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L66 EN**: Defines macro `__lsx_vslli_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L66 CN**: 定义宏 `__lsx_vslli_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L67 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslli_h`.
  **L67 CN**: 继续与可调用符号 `__builtin_lsx_vslli_h` 相关的逻辑。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L69 EN**: Defines macro `__lsx_vslli_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L69 CN**: 定义宏 `__lsx_vslli_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L70 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslli_w`.
  **L70 CN**: 继续与可调用符号 `__builtin_lsx_vslli_w` 相关的逻辑。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Defines macro `__lsx_vslli_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L72 CN**: 定义宏 `__lsx_vslli_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 73-96

````c
  ((__m128i)__builtin_lsx_vslli_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsra_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsra_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsra_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsra_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsra_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsra_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsra_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsra_d((v2i64)_1, (v2i64)_2);
````
- **L73 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslli_d`.
  **L73 CN**: 继续与可调用符号 `__builtin_lsx_vslli_d` 相关的逻辑。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L75 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L75 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L76 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L76 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L77 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsra_b(__m128i _1, __m128i _2) {`.
  **L77 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsra_b(__m128i _1, __m128i _2) {`。
- **L78 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsra_b((v16i8)_1, (v16i8)_2)`.
  **L78 CN**: 以 `(__m128i)__builtin_lsx_vsra_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L79 EN**: Closes the current lexical scope or compound statement.
  **L79 CN**: 结束当前词法作用域或复合语句块。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L81 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L81 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L82 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L82 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L83 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsra_h(__m128i _1, __m128i _2) {`.
  **L83 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsra_h(__m128i _1, __m128i _2) {`。
- **L84 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsra_h((v8i16)_1, (v8i16)_2)`.
  **L84 CN**: 以 `(__m128i)__builtin_lsx_vsra_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L85 EN**: Closes the current lexical scope or compound statement.
  **L85 CN**: 结束当前词法作用域或复合语句块。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L87 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L87 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L88 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L88 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L89 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsra_w(__m128i _1, __m128i _2) {`.
  **L89 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsra_w(__m128i _1, __m128i _2) {`。
- **L90 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsra_w((v4i32)_1, (v4i32)_2)`.
  **L90 CN**: 以 `(__m128i)__builtin_lsx_vsra_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L93 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L93 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L94 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L94 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L95 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsra_d(__m128i _1, __m128i _2) {`.
  **L95 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsra_d(__m128i _1, __m128i _2) {`。
- **L96 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsra_d((v2i64)_1, (v2i64)_2)`.
  **L96 CN**: 以 `(__m128i)__builtin_lsx_vsra_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 97-120

````c
}

#define __lsx_vsrai_b(/*__m128i*/ _1, /*ui3*/ _2)                              \
  ((__m128i)__builtin_lsx_vsrai_b((v16i8)(_1), (_2)))

#define __lsx_vsrai_h(/*__m128i*/ _1, /*ui4*/ _2)                              \
  ((__m128i)__builtin_lsx_vsrai_h((v8i16)(_1), (_2)))

#define __lsx_vsrai_w(/*__m128i*/ _1, /*ui5*/ _2)                              \
  ((__m128i)__builtin_lsx_vsrai_w((v4i32)(_1), (_2)))

#define __lsx_vsrai_d(/*__m128i*/ _1, /*ui6*/ _2)                              \
  ((__m128i)__builtin_lsx_vsrai_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrar_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrar_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrar_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrar_h((v8i16)_1, (v8i16)_2);
````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L99 EN**: Defines macro `__lsx_vsrai_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L99 CN**: 定义宏 `__lsx_vsrai_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L100 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrai_b`.
  **L100 CN**: 继续与可调用符号 `__builtin_lsx_vsrai_b` 相关的逻辑。
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L102 EN**: Defines macro `__lsx_vsrai_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L102 CN**: 定义宏 `__lsx_vsrai_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L103 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrai_h`.
  **L103 CN**: 继续与可调用符号 `__builtin_lsx_vsrai_h` 相关的逻辑。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Defines macro `__lsx_vsrai_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L105 CN**: 定义宏 `__lsx_vsrai_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L106 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrai_w`.
  **L106 CN**: 继续与可调用符号 `__builtin_lsx_vsrai_w` 相关的逻辑。
- **L107 EN**: Blank line separating nearby declarations or logic blocks.
  **L107 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L108 EN**: Defines macro `__lsx_vsrai_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L108 CN**: 定义宏 `__lsx_vsrai_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L109 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrai_d`.
  **L109 CN**: 继续与可调用符号 `__builtin_lsx_vsrai_d` 相关的逻辑。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L111 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L111 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L112 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L112 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L113 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrar_b(__m128i _1, __m128i _2) {`.
  **L113 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrar_b(__m128i _1, __m128i _2) {`。
- **L114 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrar_b((v16i8)_1, (v16i8)_2)`.
  **L114 CN**: 以 `(__m128i)__builtin_lsx_vsrar_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L115 EN**: Closes the current lexical scope or compound statement.
  **L115 CN**: 结束当前词法作用域或复合语句块。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L117 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L117 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L118 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L118 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L119 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrar_h(__m128i _1, __m128i _2) {`.
  **L119 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrar_h(__m128i _1, __m128i _2) {`。
- **L120 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrar_h((v8i16)_1, (v8i16)_2)`.
  **L120 CN**: 以 `(__m128i)__builtin_lsx_vsrar_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 121-144

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrar_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrar_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrar_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrar_d((v2i64)_1, (v2i64)_2);
}

#define __lsx_vsrari_b(/*__m128i*/ _1, /*ui3*/ _2)                             \
  ((__m128i)__builtin_lsx_vsrari_b((v16i8)(_1), (_2)))

#define __lsx_vsrari_h(/*__m128i*/ _1, /*ui4*/ _2)                             \
  ((__m128i)__builtin_lsx_vsrari_h((v8i16)(_1), (_2)))

#define __lsx_vsrari_w(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vsrari_w((v4i32)(_1), (_2)))

#define __lsx_vsrari_d(/*__m128i*/ _1, /*ui6*/ _2)                             \
````
- **L121 EN**: Closes the current lexical scope or compound statement.
  **L121 CN**: 结束当前词法作用域或复合语句块。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L123 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L123 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L124 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L124 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrar_w(__m128i _1, __m128i _2) {`.
  **L125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrar_w(__m128i _1, __m128i _2) {`。
- **L126 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrar_w((v4i32)_1, (v4i32)_2)`.
  **L126 CN**: 以 `(__m128i)__builtin_lsx_vsrar_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L127 EN**: Closes the current lexical scope or compound statement.
  **L127 CN**: 结束当前词法作用域或复合语句块。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L129 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L129 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L130 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L130 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L131 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrar_d(__m128i _1, __m128i _2) {`.
  **L131 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrar_d(__m128i _1, __m128i _2) {`。
- **L132 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrar_d((v2i64)_1, (v2i64)_2)`.
  **L132 CN**: 以 `(__m128i)__builtin_lsx_vsrar_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Blank line separating nearby declarations or logic blocks.
  **L134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L135 EN**: Defines macro `__lsx_vsrari_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L135 CN**: 定义宏 `__lsx_vsrari_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L136 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrari_b`.
  **L136 CN**: 继续与可调用符号 `__builtin_lsx_vsrari_b` 相关的逻辑。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Defines macro `__lsx_vsrari_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L138 CN**: 定义宏 `__lsx_vsrari_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L139 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrari_h`.
  **L139 CN**: 继续与可调用符号 `__builtin_lsx_vsrari_h` 相关的逻辑。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L141 EN**: Defines macro `__lsx_vsrari_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L141 CN**: 定义宏 `__lsx_vsrari_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L142 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrari_w`.
  **L142 CN**: 继续与可调用符号 `__builtin_lsx_vsrari_w` 相关的逻辑。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L144 EN**: Defines macro `__lsx_vsrari_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L144 CN**: 定义宏 `__lsx_vsrari_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 145-168

````c
  ((__m128i)__builtin_lsx_vsrari_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrl_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrl_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrl_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrl_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrl_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrl_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrl_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrl_d((v2i64)_1, (v2i64)_2);
````
- **L145 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrari_d`.
  **L145 CN**: 继续与可调用符号 `__builtin_lsx_vsrari_d` 相关的逻辑。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L147 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L147 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L148 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L148 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrl_b(__m128i _1, __m128i _2) {`.
  **L149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrl_b(__m128i _1, __m128i _2) {`。
- **L150 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrl_b((v16i8)_1, (v16i8)_2)`.
  **L150 CN**: 以 `(__m128i)__builtin_lsx_vsrl_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L151 EN**: Closes the current lexical scope or compound statement.
  **L151 CN**: 结束当前词法作用域或复合语句块。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L153 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L153 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L154 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L154 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L155 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrl_h(__m128i _1, __m128i _2) {`.
  **L155 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrl_h(__m128i _1, __m128i _2) {`。
- **L156 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrl_h((v8i16)_1, (v8i16)_2)`.
  **L156 CN**: 以 `(__m128i)__builtin_lsx_vsrl_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L157 EN**: Closes the current lexical scope or compound statement.
  **L157 CN**: 结束当前词法作用域或复合语句块。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L159 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L159 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L160 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L160 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L161 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrl_w(__m128i _1, __m128i _2) {`.
  **L161 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrl_w(__m128i _1, __m128i _2) {`。
- **L162 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrl_w((v4i32)_1, (v4i32)_2)`.
  **L162 CN**: 以 `(__m128i)__builtin_lsx_vsrl_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L165 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L166 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L166 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L167 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrl_d(__m128i _1, __m128i _2) {`.
  **L167 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrl_d(__m128i _1, __m128i _2) {`。
- **L168 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrl_d((v2i64)_1, (v2i64)_2)`.
  **L168 CN**: 以 `(__m128i)__builtin_lsx_vsrl_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 169-192

````c
}

#define __lsx_vsrli_b(/*__m128i*/ _1, /*ui3*/ _2)                              \
  ((__m128i)__builtin_lsx_vsrli_b((v16i8)(_1), (_2)))

#define __lsx_vsrli_h(/*__m128i*/ _1, /*ui4*/ _2)                              \
  ((__m128i)__builtin_lsx_vsrli_h((v8i16)(_1), (_2)))

#define __lsx_vsrli_w(/*__m128i*/ _1, /*ui5*/ _2)                              \
  ((__m128i)__builtin_lsx_vsrli_w((v4i32)(_1), (_2)))

#define __lsx_vsrli_d(/*__m128i*/ _1, /*ui6*/ _2)                              \
  ((__m128i)__builtin_lsx_vsrli_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrlr_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrlr_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrlr_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrlr_h((v8i16)_1, (v8i16)_2);
````
- **L169 EN**: Closes the current lexical scope or compound statement.
  **L169 CN**: 结束当前词法作用域或复合语句块。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L171 EN**: Defines macro `__lsx_vsrli_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L171 CN**: 定义宏 `__lsx_vsrli_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L172 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrli_b`.
  **L172 CN**: 继续与可调用符号 `__builtin_lsx_vsrli_b` 相关的逻辑。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L174 EN**: Defines macro `__lsx_vsrli_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L174 CN**: 定义宏 `__lsx_vsrli_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L175 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrli_h`.
  **L175 CN**: 继续与可调用符号 `__builtin_lsx_vsrli_h` 相关的逻辑。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L177 EN**: Defines macro `__lsx_vsrli_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L177 CN**: 定义宏 `__lsx_vsrli_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L178 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrli_w`.
  **L178 CN**: 继续与可调用符号 `__builtin_lsx_vsrli_w` 相关的逻辑。
- **L179 EN**: Blank line separating nearby declarations or logic blocks.
  **L179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L180 EN**: Defines macro `__lsx_vsrli_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L180 CN**: 定义宏 `__lsx_vsrli_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L181 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrli_d`.
  **L181 CN**: 继续与可调用符号 `__builtin_lsx_vsrli_d` 相关的逻辑。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L183 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L183 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L184 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L184 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L185 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrlr_b(__m128i _1, __m128i _2) {`.
  **L185 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrlr_b(__m128i _1, __m128i _2) {`。
- **L186 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrlr_b((v16i8)_1, (v16i8)_2)`.
  **L186 CN**: 以 `(__m128i)__builtin_lsx_vsrlr_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L187 EN**: Closes the current lexical scope or compound statement.
  **L187 CN**: 结束当前词法作用域或复合语句块。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L189 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L189 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L190 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L190 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L191 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrlr_h(__m128i _1, __m128i _2) {`.
  **L191 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrlr_h(__m128i _1, __m128i _2) {`。
- **L192 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrlr_h((v8i16)_1, (v8i16)_2)`.
  **L192 CN**: 以 `(__m128i)__builtin_lsx_vsrlr_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 193-216

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrlr_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrlr_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrlr_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrlr_d((v2i64)_1, (v2i64)_2);
}

#define __lsx_vsrlri_b(/*__m128i*/ _1, /*ui3*/ _2)                             \
  ((__m128i)__builtin_lsx_vsrlri_b((v16i8)(_1), (_2)))

#define __lsx_vsrlri_h(/*__m128i*/ _1, /*ui4*/ _2)                             \
  ((__m128i)__builtin_lsx_vsrlri_h((v8i16)(_1), (_2)))

#define __lsx_vsrlri_w(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vsrlri_w((v4i32)(_1), (_2)))

#define __lsx_vsrlri_d(/*__m128i*/ _1, /*ui6*/ _2)                             \
````
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L195 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L195 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L196 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L196 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L197 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrlr_w(__m128i _1, __m128i _2) {`.
  **L197 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrlr_w(__m128i _1, __m128i _2) {`。
- **L198 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrlr_w((v4i32)_1, (v4i32)_2)`.
  **L198 CN**: 以 `(__m128i)__builtin_lsx_vsrlr_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L201 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L201 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L202 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L202 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L203 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrlr_d(__m128i _1, __m128i _2) {`.
  **L203 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrlr_d(__m128i _1, __m128i _2) {`。
- **L204 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrlr_d((v2i64)_1, (v2i64)_2)`.
  **L204 CN**: 以 `(__m128i)__builtin_lsx_vsrlr_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L205 EN**: Closes the current lexical scope or compound statement.
  **L205 CN**: 结束当前词法作用域或复合语句块。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L207 EN**: Defines macro `__lsx_vsrlri_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L207 CN**: 定义宏 `__lsx_vsrlri_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L208 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlri_b`.
  **L208 CN**: 继续与可调用符号 `__builtin_lsx_vsrlri_b` 相关的逻辑。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L210 EN**: Defines macro `__lsx_vsrlri_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L210 CN**: 定义宏 `__lsx_vsrlri_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L211 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlri_h`.
  **L211 CN**: 继续与可调用符号 `__builtin_lsx_vsrlri_h` 相关的逻辑。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L213 EN**: Defines macro `__lsx_vsrlri_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L213 CN**: 定义宏 `__lsx_vsrlri_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L214 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlri_w`.
  **L214 CN**: 继续与可调用符号 `__builtin_lsx_vsrlri_w` 相关的逻辑。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L216 EN**: Defines macro `__lsx_vsrlri_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L216 CN**: 定义宏 `__lsx_vsrlri_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 217-240

````c
  ((__m128i)__builtin_lsx_vsrlri_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitclr_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitclr_b((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitclr_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitclr_h((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitclr_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitclr_w((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitclr_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitclr_d((v2u64)_1, (v2u64)_2);
````
- **L217 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlri_d`.
  **L217 CN**: 继续与可调用符号 `__builtin_lsx_vsrlri_d` 相关的逻辑。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L219 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L219 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L220 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L220 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L221 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitclr_b(__m128i _1, __m128i _2) {`.
  **L221 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitclr_b(__m128i _1, __m128i _2) {`。
- **L222 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitclr_b((v16u8)_1, (v16u8)_2)`.
  **L222 CN**: 以 `(__m128i)__builtin_lsx_vbitclr_b((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L223 EN**: Closes the current lexical scope or compound statement.
  **L223 CN**: 结束当前词法作用域或复合语句块。
- **L224 EN**: Blank line separating nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L225 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L225 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L226 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L226 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitclr_h(__m128i _1, __m128i _2) {`.
  **L227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitclr_h(__m128i _1, __m128i _2) {`。
- **L228 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitclr_h((v8u16)_1, (v8u16)_2)`.
  **L228 CN**: 以 `(__m128i)__builtin_lsx_vbitclr_h((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L229 EN**: Closes the current lexical scope or compound statement.
  **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L231 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L231 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L232 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L232 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L233 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitclr_w(__m128i _1, __m128i _2) {`.
  **L233 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitclr_w(__m128i _1, __m128i _2) {`。
- **L234 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitclr_w((v4u32)_1, (v4u32)_2)`.
  **L234 CN**: 以 `(__m128i)__builtin_lsx_vbitclr_w((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L235 EN**: Closes the current lexical scope or compound statement.
  **L235 CN**: 结束当前词法作用域或复合语句块。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L237 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L237 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L238 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L238 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L239 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitclr_d(__m128i _1, __m128i _2) {`.
  **L239 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitclr_d(__m128i _1, __m128i _2) {`。
- **L240 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitclr_d((v2u64)_1, (v2u64)_2)`.
  **L240 CN**: 以 `(__m128i)__builtin_lsx_vbitclr_d((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 241-264

````c
}

#define __lsx_vbitclri_b(/*__m128i*/ _1, /*ui3*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitclri_b((v16u8)(_1), (_2)))

#define __lsx_vbitclri_h(/*__m128i*/ _1, /*ui4*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitclri_h((v8u16)(_1), (_2)))

#define __lsx_vbitclri_w(/*__m128i*/ _1, /*ui5*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitclri_w((v4u32)(_1), (_2)))

#define __lsx_vbitclri_d(/*__m128i*/ _1, /*ui6*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitclri_d((v2u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitset_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitset_b((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitset_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitset_h((v8u16)_1, (v8u16)_2);
````
- **L241 EN**: Closes the current lexical scope or compound statement.
  **L241 CN**: 结束当前词法作用域或复合语句块。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L243 EN**: Defines macro `__lsx_vbitclri_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L243 CN**: 定义宏 `__lsx_vbitclri_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L244 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitclri_b`.
  **L244 CN**: 继续与可调用符号 `__builtin_lsx_vbitclri_b` 相关的逻辑。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L246 EN**: Defines macro `__lsx_vbitclri_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L246 CN**: 定义宏 `__lsx_vbitclri_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L247 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitclri_h`.
  **L247 CN**: 继续与可调用符号 `__builtin_lsx_vbitclri_h` 相关的逻辑。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L249 EN**: Defines macro `__lsx_vbitclri_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L249 CN**: 定义宏 `__lsx_vbitclri_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L250 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitclri_w`.
  **L250 CN**: 继续与可调用符号 `__builtin_lsx_vbitclri_w` 相关的逻辑。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L252 EN**: Defines macro `__lsx_vbitclri_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L252 CN**: 定义宏 `__lsx_vbitclri_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L253 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitclri_d`.
  **L253 CN**: 继续与可调用符号 `__builtin_lsx_vbitclri_d` 相关的逻辑。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L255 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L255 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L256 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L256 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L257 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitset_b(__m128i _1, __m128i _2) {`.
  **L257 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitset_b(__m128i _1, __m128i _2) {`。
- **L258 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitset_b((v16u8)_1, (v16u8)_2)`.
  **L258 CN**: 以 `(__m128i)__builtin_lsx_vbitset_b((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L259 EN**: Closes the current lexical scope or compound statement.
  **L259 CN**: 结束当前词法作用域或复合语句块。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L261 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L261 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L262 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L262 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L263 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitset_h(__m128i _1, __m128i _2) {`.
  **L263 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitset_h(__m128i _1, __m128i _2) {`。
- **L264 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitset_h((v8u16)_1, (v8u16)_2)`.
  **L264 CN**: 以 `(__m128i)__builtin_lsx_vbitset_h((v8u16)_1, (v8u16)_2)` 从当前函数返回。

### Lines 265-288

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitset_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitset_w((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitset_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitset_d((v2u64)_1, (v2u64)_2);
}

#define __lsx_vbitseti_b(/*__m128i*/ _1, /*ui3*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitseti_b((v16u8)(_1), (_2)))

#define __lsx_vbitseti_h(/*__m128i*/ _1, /*ui4*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitseti_h((v8u16)(_1), (_2)))

#define __lsx_vbitseti_w(/*__m128i*/ _1, /*ui5*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitseti_w((v4u32)(_1), (_2)))

#define __lsx_vbitseti_d(/*__m128i*/ _1, /*ui6*/ _2)                           \
````
- **L265 EN**: Closes the current lexical scope or compound statement.
  **L265 CN**: 结束当前词法作用域或复合语句块。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L267 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L267 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L268 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L268 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L269 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitset_w(__m128i _1, __m128i _2) {`.
  **L269 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitset_w(__m128i _1, __m128i _2) {`。
- **L270 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitset_w((v4u32)_1, (v4u32)_2)`.
  **L270 CN**: 以 `(__m128i)__builtin_lsx_vbitset_w((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L273 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L273 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L274 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L274 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L275 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitset_d(__m128i _1, __m128i _2) {`.
  **L275 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitset_d(__m128i _1, __m128i _2) {`。
- **L276 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitset_d((v2u64)_1, (v2u64)_2)`.
  **L276 CN**: 以 `(__m128i)__builtin_lsx_vbitset_d((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L277 EN**: Closes the current lexical scope or compound statement.
  **L277 CN**: 结束当前词法作用域或复合语句块。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L279 EN**: Defines macro `__lsx_vbitseti_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L279 CN**: 定义宏 `__lsx_vbitseti_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L280 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitseti_b`.
  **L280 CN**: 继续与可调用符号 `__builtin_lsx_vbitseti_b` 相关的逻辑。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L282 EN**: Defines macro `__lsx_vbitseti_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L282 CN**: 定义宏 `__lsx_vbitseti_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L283 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitseti_h`.
  **L283 CN**: 继续与可调用符号 `__builtin_lsx_vbitseti_h` 相关的逻辑。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L285 EN**: Defines macro `__lsx_vbitseti_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L285 CN**: 定义宏 `__lsx_vbitseti_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L286 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitseti_w`.
  **L286 CN**: 继续与可调用符号 `__builtin_lsx_vbitseti_w` 相关的逻辑。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L288 EN**: Defines macro `__lsx_vbitseti_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L288 CN**: 定义宏 `__lsx_vbitseti_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 289-312

````c
  ((__m128i)__builtin_lsx_vbitseti_d((v2u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitrev_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitrev_b((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitrev_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitrev_h((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitrev_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitrev_w((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitrev_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vbitrev_d((v2u64)_1, (v2u64)_2);
````
- **L289 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitseti_d`.
  **L289 CN**: 继续与可调用符号 `__builtin_lsx_vbitseti_d` 相关的逻辑。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L291 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L292 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L292 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L293 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitrev_b(__m128i _1, __m128i _2) {`.
  **L293 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitrev_b(__m128i _1, __m128i _2) {`。
- **L294 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitrev_b((v16u8)_1, (v16u8)_2)`.
  **L294 CN**: 以 `(__m128i)__builtin_lsx_vbitrev_b((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L295 EN**: Closes the current lexical scope or compound statement.
  **L295 CN**: 结束当前词法作用域或复合语句块。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L297 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L298 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L298 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L299 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitrev_h(__m128i _1, __m128i _2) {`.
  **L299 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitrev_h(__m128i _1, __m128i _2) {`。
- **L300 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitrev_h((v8u16)_1, (v8u16)_2)`.
  **L300 CN**: 以 `(__m128i)__builtin_lsx_vbitrev_h((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L301 EN**: Closes the current lexical scope or compound statement.
  **L301 CN**: 结束当前词法作用域或复合语句块。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L303 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L303 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L304 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L304 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L305 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitrev_w(__m128i _1, __m128i _2) {`.
  **L305 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitrev_w(__m128i _1, __m128i _2) {`。
- **L306 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitrev_w((v4u32)_1, (v4u32)_2)`.
  **L306 CN**: 以 `(__m128i)__builtin_lsx_vbitrev_w((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L307 EN**: Closes the current lexical scope or compound statement.
  **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L309 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L309 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L310 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L310 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L311 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitrev_d(__m128i _1, __m128i _2) {`.
  **L311 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitrev_d(__m128i _1, __m128i _2) {`。
- **L312 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitrev_d((v2u64)_1, (v2u64)_2)`.
  **L312 CN**: 以 `(__m128i)__builtin_lsx_vbitrev_d((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 313-336

````c
}

#define __lsx_vbitrevi_b(/*__m128i*/ _1, /*ui3*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitrevi_b((v16u8)(_1), (_2)))

#define __lsx_vbitrevi_h(/*__m128i*/ _1, /*ui4*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitrevi_h((v8u16)(_1), (_2)))

#define __lsx_vbitrevi_w(/*__m128i*/ _1, /*ui5*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitrevi_w((v4u32)(_1), (_2)))

#define __lsx_vbitrevi_d(/*__m128i*/ _1, /*ui6*/ _2)                           \
  ((__m128i)__builtin_lsx_vbitrevi_d((v2u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadd_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadd_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadd_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadd_h((v8i16)_1, (v8i16)_2);
````
- **L313 EN**: Closes the current lexical scope or compound statement.
  **L313 CN**: 结束当前词法作用域或复合语句块。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Defines macro `__lsx_vbitrevi_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L315 CN**: 定义宏 `__lsx_vbitrevi_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L316 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitrevi_b`.
  **L316 CN**: 继续与可调用符号 `__builtin_lsx_vbitrevi_b` 相关的逻辑。
- **L317 EN**: Blank line separating nearby declarations or logic blocks.
  **L317 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L318 EN**: Defines macro `__lsx_vbitrevi_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L318 CN**: 定义宏 `__lsx_vbitrevi_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L319 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitrevi_h`.
  **L319 CN**: 继续与可调用符号 `__builtin_lsx_vbitrevi_h` 相关的逻辑。
- **L320 EN**: Blank line separating nearby declarations or logic blocks.
  **L320 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L321 EN**: Defines macro `__lsx_vbitrevi_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L321 CN**: 定义宏 `__lsx_vbitrevi_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L322 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitrevi_w`.
  **L322 CN**: 继续与可调用符号 `__builtin_lsx_vbitrevi_w` 相关的逻辑。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L324 EN**: Defines macro `__lsx_vbitrevi_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L324 CN**: 定义宏 `__lsx_vbitrevi_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L325 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitrevi_d`.
  **L325 CN**: 继续与可调用符号 `__builtin_lsx_vbitrevi_d` 相关的逻辑。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L327 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L328 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L328 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L329 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadd_b(__m128i _1, __m128i _2) {`.
  **L329 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadd_b(__m128i _1, __m128i _2) {`。
- **L330 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadd_b((v16i8)_1, (v16i8)_2)`.
  **L330 CN**: 以 `(__m128i)__builtin_lsx_vadd_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L333 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L334 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L334 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L335 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadd_h(__m128i _1, __m128i _2) {`.
  **L335 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadd_h(__m128i _1, __m128i _2) {`。
- **L336 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadd_h((v8i16)_1, (v8i16)_2)`.
  **L336 CN**: 以 `(__m128i)__builtin_lsx_vadd_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 337-360

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadd_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadd_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadd_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadd_d((v2i64)_1, (v2i64)_2);
}

#define __lsx_vaddi_bu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vaddi_bu((v16i8)(_1), (_2)))

#define __lsx_vaddi_hu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vaddi_hu((v8i16)(_1), (_2)))

#define __lsx_vaddi_wu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vaddi_wu((v4i32)(_1), (_2)))

#define __lsx_vaddi_du(/*__m128i*/ _1, /*ui5*/ _2)                             \
````
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L339 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L340 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L340 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L341 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadd_w(__m128i _1, __m128i _2) {`.
  **L341 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadd_w(__m128i _1, __m128i _2) {`。
- **L342 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadd_w((v4i32)_1, (v4i32)_2)`.
  **L342 CN**: 以 `(__m128i)__builtin_lsx_vadd_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L343 EN**: Closes the current lexical scope or compound statement.
  **L343 CN**: 结束当前词法作用域或复合语句块。
- **L344 EN**: Blank line separating nearby declarations or logic blocks.
  **L344 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L345 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L345 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L346 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L346 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L347 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadd_d(__m128i _1, __m128i _2) {`.
  **L347 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadd_d(__m128i _1, __m128i _2) {`。
- **L348 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadd_d((v2i64)_1, (v2i64)_2)`.
  **L348 CN**: 以 `(__m128i)__builtin_lsx_vadd_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L349 EN**: Closes the current lexical scope or compound statement.
  **L349 CN**: 结束当前词法作用域或复合语句块。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L351 EN**: Defines macro `__lsx_vaddi_bu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L351 CN**: 定义宏 `__lsx_vaddi_bu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L352 EN**: Continues logic associated with callable symbol `__builtin_lsx_vaddi_bu`.
  **L352 CN**: 继续与可调用符号 `__builtin_lsx_vaddi_bu` 相关的逻辑。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L354 EN**: Defines macro `__lsx_vaddi_hu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L354 CN**: 定义宏 `__lsx_vaddi_hu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L355 EN**: Continues logic associated with callable symbol `__builtin_lsx_vaddi_hu`.
  **L355 CN**: 继续与可调用符号 `__builtin_lsx_vaddi_hu` 相关的逻辑。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L357 EN**: Defines macro `__lsx_vaddi_wu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L357 CN**: 定义宏 `__lsx_vaddi_wu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L358 EN**: Continues logic associated with callable symbol `__builtin_lsx_vaddi_wu`.
  **L358 CN**: 继续与可调用符号 `__builtin_lsx_vaddi_wu` 相关的逻辑。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L360 EN**: Defines macro `__lsx_vaddi_du(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L360 CN**: 定义宏 `__lsx_vaddi_du(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 361-384

````c
  ((__m128i)__builtin_lsx_vaddi_du((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsub_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsub_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsub_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsub_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsub_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsub_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsub_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsub_d((v2i64)_1, (v2i64)_2);
````
- **L361 EN**: Continues logic associated with callable symbol `__builtin_lsx_vaddi_du`.
  **L361 CN**: 继续与可调用符号 `__builtin_lsx_vaddi_du` 相关的逻辑。
- **L362 EN**: Blank line separating nearby declarations or logic blocks.
  **L362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L363 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L363 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L364 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L364 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L365 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsub_b(__m128i _1, __m128i _2) {`.
  **L365 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsub_b(__m128i _1, __m128i _2) {`。
- **L366 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsub_b((v16i8)_1, (v16i8)_2)`.
  **L366 CN**: 以 `(__m128i)__builtin_lsx_vsub_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L367 EN**: Closes the current lexical scope or compound statement.
  **L367 CN**: 结束当前词法作用域或复合语句块。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L369 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L369 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L370 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L370 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L371 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsub_h(__m128i _1, __m128i _2) {`.
  **L371 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsub_h(__m128i _1, __m128i _2) {`。
- **L372 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsub_h((v8i16)_1, (v8i16)_2)`.
  **L372 CN**: 以 `(__m128i)__builtin_lsx_vsub_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L375 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L375 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L376 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L376 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L377 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsub_w(__m128i _1, __m128i _2) {`.
  **L377 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsub_w(__m128i _1, __m128i _2) {`。
- **L378 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsub_w((v4i32)_1, (v4i32)_2)`.
  **L378 CN**: 以 `(__m128i)__builtin_lsx_vsub_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L379 EN**: Closes the current lexical scope or compound statement.
  **L379 CN**: 结束当前词法作用域或复合语句块。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L381 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L381 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L382 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L382 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L383 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsub_d(__m128i _1, __m128i _2) {`.
  **L383 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsub_d(__m128i _1, __m128i _2) {`。
- **L384 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsub_d((v2i64)_1, (v2i64)_2)`.
  **L384 CN**: 以 `(__m128i)__builtin_lsx_vsub_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 385-408

````c
}

#define __lsx_vsubi_bu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vsubi_bu((v16i8)(_1), (_2)))

#define __lsx_vsubi_hu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vsubi_hu((v8i16)(_1), (_2)))

#define __lsx_vsubi_wu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vsubi_wu((v4i32)(_1), (_2)))

#define __lsx_vsubi_du(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vsubi_du((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmax_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmax_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmax_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmax_h((v8i16)_1, (v8i16)_2);
````
- **L385 EN**: Closes the current lexical scope or compound statement.
  **L385 CN**: 结束当前词法作用域或复合语句块。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L387 EN**: Defines macro `__lsx_vsubi_bu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L387 CN**: 定义宏 `__lsx_vsubi_bu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L388 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsubi_bu`.
  **L388 CN**: 继续与可调用符号 `__builtin_lsx_vsubi_bu` 相关的逻辑。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L390 EN**: Defines macro `__lsx_vsubi_hu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L390 CN**: 定义宏 `__lsx_vsubi_hu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L391 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsubi_hu`.
  **L391 CN**: 继续与可调用符号 `__builtin_lsx_vsubi_hu` 相关的逻辑。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L393 EN**: Defines macro `__lsx_vsubi_wu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L393 CN**: 定义宏 `__lsx_vsubi_wu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L394 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsubi_wu`.
  **L394 CN**: 继续与可调用符号 `__builtin_lsx_vsubi_wu` 相关的逻辑。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Defines macro `__lsx_vsubi_du(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L396 CN**: 定义宏 `__lsx_vsubi_du(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L397 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsubi_du`.
  **L397 CN**: 继续与可调用符号 `__builtin_lsx_vsubi_du` 相关的逻辑。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L399 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L399 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L400 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L400 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L401 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmax_b(__m128i _1, __m128i _2) {`.
  **L401 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmax_b(__m128i _1, __m128i _2) {`。
- **L402 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmax_b((v16i8)_1, (v16i8)_2)`.
  **L402 CN**: 以 `(__m128i)__builtin_lsx_vmax_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L403 EN**: Closes the current lexical scope or compound statement.
  **L403 CN**: 结束当前词法作用域或复合语句块。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L405 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L405 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L406 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L406 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L407 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmax_h(__m128i _1, __m128i _2) {`.
  **L407 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmax_h(__m128i _1, __m128i _2) {`。
- **L408 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmax_h((v8i16)_1, (v8i16)_2)`.
  **L408 CN**: 以 `(__m128i)__builtin_lsx_vmax_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 409-432

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmax_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmax_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmax_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmax_d((v2i64)_1, (v2i64)_2);
}

#define __lsx_vmaxi_b(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vmaxi_b((v16i8)(_1), (_2)))

#define __lsx_vmaxi_h(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vmaxi_h((v8i16)(_1), (_2)))

#define __lsx_vmaxi_w(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vmaxi_w((v4i32)(_1), (_2)))

#define __lsx_vmaxi_d(/*__m128i*/ _1, /*si5*/ _2)                              \
````
- **L409 EN**: Closes the current lexical scope or compound statement.
  **L409 CN**: 结束当前词法作用域或复合语句块。
- **L410 EN**: Blank line separating nearby declarations or logic blocks.
  **L410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L411 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L411 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L412 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L412 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L413 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmax_w(__m128i _1, __m128i _2) {`.
  **L413 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmax_w(__m128i _1, __m128i _2) {`。
- **L414 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmax_w((v4i32)_1, (v4i32)_2)`.
  **L414 CN**: 以 `(__m128i)__builtin_lsx_vmax_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L415 EN**: Closes the current lexical scope or compound statement.
  **L415 CN**: 结束当前词法作用域或复合语句块。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L417 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L417 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L418 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L418 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L419 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmax_d(__m128i _1, __m128i _2) {`.
  **L419 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmax_d(__m128i _1, __m128i _2) {`。
- **L420 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmax_d((v2i64)_1, (v2i64)_2)`.
  **L420 CN**: 以 `(__m128i)__builtin_lsx_vmax_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L421 EN**: Closes the current lexical scope or compound statement.
  **L421 CN**: 结束当前词法作用域或复合语句块。
- **L422 EN**: Blank line separating nearby declarations or logic blocks.
  **L422 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L423 EN**: Defines macro `__lsx_vmaxi_b(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L423 CN**: 定义宏 `__lsx_vmaxi_b(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L424 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmaxi_b`.
  **L424 CN**: 继续与可调用符号 `__builtin_lsx_vmaxi_b` 相关的逻辑。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L426 EN**: Defines macro `__lsx_vmaxi_h(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L426 CN**: 定义宏 `__lsx_vmaxi_h(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L427 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmaxi_h`.
  **L427 CN**: 继续与可调用符号 `__builtin_lsx_vmaxi_h` 相关的逻辑。
- **L428 EN**: Blank line separating nearby declarations or logic blocks.
  **L428 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L429 EN**: Defines macro `__lsx_vmaxi_w(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L429 CN**: 定义宏 `__lsx_vmaxi_w(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L430 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmaxi_w`.
  **L430 CN**: 继续与可调用符号 `__builtin_lsx_vmaxi_w` 相关的逻辑。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L432 EN**: Defines macro `__lsx_vmaxi_d(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L432 CN**: 定义宏 `__lsx_vmaxi_d(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 433-456

````c
  ((__m128i)__builtin_lsx_vmaxi_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmax_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmax_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmax_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmax_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmax_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmax_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmax_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmax_du((v2u64)_1, (v2u64)_2);
````
- **L433 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmaxi_d`.
  **L433 CN**: 继续与可调用符号 `__builtin_lsx_vmaxi_d` 相关的逻辑。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L435 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L435 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L436 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L436 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L437 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmax_bu(__m128i _1, __m128i _2) {`.
  **L437 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmax_bu(__m128i _1, __m128i _2) {`。
- **L438 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmax_bu((v16u8)_1, (v16u8)_2)`.
  **L438 CN**: 以 `(__m128i)__builtin_lsx_vmax_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L439 EN**: Closes the current lexical scope or compound statement.
  **L439 CN**: 结束当前词法作用域或复合语句块。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L441 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L441 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L442 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L442 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L443 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmax_hu(__m128i _1, __m128i _2) {`.
  **L443 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmax_hu(__m128i _1, __m128i _2) {`。
- **L444 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmax_hu((v8u16)_1, (v8u16)_2)`.
  **L444 CN**: 以 `(__m128i)__builtin_lsx_vmax_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L445 EN**: Closes the current lexical scope or compound statement.
  **L445 CN**: 结束当前词法作用域或复合语句块。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L447 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L447 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L448 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L448 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L449 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmax_wu(__m128i _1, __m128i _2) {`.
  **L449 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmax_wu(__m128i _1, __m128i _2) {`。
- **L450 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmax_wu((v4u32)_1, (v4u32)_2)`.
  **L450 CN**: 以 `(__m128i)__builtin_lsx_vmax_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L451 EN**: Closes the current lexical scope or compound statement.
  **L451 CN**: 结束当前词法作用域或复合语句块。
- **L452 EN**: Blank line separating nearby declarations or logic blocks.
  **L452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L453 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L453 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L454 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L454 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L455 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmax_du(__m128i _1, __m128i _2) {`.
  **L455 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmax_du(__m128i _1, __m128i _2) {`。
- **L456 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmax_du((v2u64)_1, (v2u64)_2)`.
  **L456 CN**: 以 `(__m128i)__builtin_lsx_vmax_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 457-480

````c
}

#define __lsx_vmaxi_bu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vmaxi_bu((v16u8)(_1), (_2)))

#define __lsx_vmaxi_hu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vmaxi_hu((v8u16)(_1), (_2)))

#define __lsx_vmaxi_wu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vmaxi_wu((v4u32)(_1), (_2)))

#define __lsx_vmaxi_du(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vmaxi_du((v2u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmin_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmin_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmin_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmin_h((v8i16)_1, (v8i16)_2);
````
- **L457 EN**: Closes the current lexical scope or compound statement.
  **L457 CN**: 结束当前词法作用域或复合语句块。
- **L458 EN**: Blank line separating nearby declarations or logic blocks.
  **L458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L459 EN**: Defines macro `__lsx_vmaxi_bu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L459 CN**: 定义宏 `__lsx_vmaxi_bu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L460 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmaxi_bu`.
  **L460 CN**: 继续与可调用符号 `__builtin_lsx_vmaxi_bu` 相关的逻辑。
- **L461 EN**: Blank line separating nearby declarations or logic blocks.
  **L461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L462 EN**: Defines macro `__lsx_vmaxi_hu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L462 CN**: 定义宏 `__lsx_vmaxi_hu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L463 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmaxi_hu`.
  **L463 CN**: 继续与可调用符号 `__builtin_lsx_vmaxi_hu` 相关的逻辑。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L465 EN**: Defines macro `__lsx_vmaxi_wu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L465 CN**: 定义宏 `__lsx_vmaxi_wu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L466 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmaxi_wu`.
  **L466 CN**: 继续与可调用符号 `__builtin_lsx_vmaxi_wu` 相关的逻辑。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Defines macro `__lsx_vmaxi_du(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L468 CN**: 定义宏 `__lsx_vmaxi_du(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L469 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmaxi_du`.
  **L469 CN**: 继续与可调用符号 `__builtin_lsx_vmaxi_du` 相关的逻辑。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L471 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L472 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L472 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L473 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmin_b(__m128i _1, __m128i _2) {`.
  **L473 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmin_b(__m128i _1, __m128i _2) {`。
- **L474 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmin_b((v16i8)_1, (v16i8)_2)`.
  **L474 CN**: 以 `(__m128i)__builtin_lsx_vmin_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L475 EN**: Closes the current lexical scope or compound statement.
  **L475 CN**: 结束当前词法作用域或复合语句块。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L477 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L477 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L478 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L478 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L479 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmin_h(__m128i _1, __m128i _2) {`.
  **L479 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmin_h(__m128i _1, __m128i _2) {`。
- **L480 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmin_h((v8i16)_1, (v8i16)_2)`.
  **L480 CN**: 以 `(__m128i)__builtin_lsx_vmin_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 481-504

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmin_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmin_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmin_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmin_d((v2i64)_1, (v2i64)_2);
}

#define __lsx_vmini_b(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vmini_b((v16i8)(_1), (_2)))

#define __lsx_vmini_h(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vmini_h((v8i16)(_1), (_2)))

#define __lsx_vmini_w(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vmini_w((v4i32)(_1), (_2)))

#define __lsx_vmini_d(/*__m128i*/ _1, /*si5*/ _2)                              \
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L483 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L483 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L484 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L484 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L485 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmin_w(__m128i _1, __m128i _2) {`.
  **L485 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmin_w(__m128i _1, __m128i _2) {`。
- **L486 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmin_w((v4i32)_1, (v4i32)_2)`.
  **L486 CN**: 以 `(__m128i)__builtin_lsx_vmin_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L489 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L489 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L490 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L490 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L491 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmin_d(__m128i _1, __m128i _2) {`.
  **L491 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmin_d(__m128i _1, __m128i _2) {`。
- **L492 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmin_d((v2i64)_1, (v2i64)_2)`.
  **L492 CN**: 以 `(__m128i)__builtin_lsx_vmin_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L493 EN**: Closes the current lexical scope or compound statement.
  **L493 CN**: 结束当前词法作用域或复合语句块。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L495 EN**: Defines macro `__lsx_vmini_b(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L495 CN**: 定义宏 `__lsx_vmini_b(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L496 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmini_b`.
  **L496 CN**: 继续与可调用符号 `__builtin_lsx_vmini_b` 相关的逻辑。
- **L497 EN**: Blank line separating nearby declarations or logic blocks.
  **L497 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L498 EN**: Defines macro `__lsx_vmini_h(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L498 CN**: 定义宏 `__lsx_vmini_h(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L499 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmini_h`.
  **L499 CN**: 继续与可调用符号 `__builtin_lsx_vmini_h` 相关的逻辑。
- **L500 EN**: Blank line separating nearby declarations or logic blocks.
  **L500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L501 EN**: Defines macro `__lsx_vmini_w(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L501 CN**: 定义宏 `__lsx_vmini_w(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L502 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmini_w`.
  **L502 CN**: 继续与可调用符号 `__builtin_lsx_vmini_w` 相关的逻辑。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L504 EN**: Defines macro `__lsx_vmini_d(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L504 CN**: 定义宏 `__lsx_vmini_d(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 505-528

````c
  ((__m128i)__builtin_lsx_vmini_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmin_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmin_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmin_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmin_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmin_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmin_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmin_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmin_du((v2u64)_1, (v2u64)_2);
````
- **L505 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmini_d`.
  **L505 CN**: 继续与可调用符号 `__builtin_lsx_vmini_d` 相关的逻辑。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L507 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L507 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L508 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L508 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L509 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmin_bu(__m128i _1, __m128i _2) {`.
  **L509 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmin_bu(__m128i _1, __m128i _2) {`。
- **L510 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmin_bu((v16u8)_1, (v16u8)_2)`.
  **L510 CN**: 以 `(__m128i)__builtin_lsx_vmin_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L513 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L513 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L514 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L514 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L515 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmin_hu(__m128i _1, __m128i _2) {`.
  **L515 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmin_hu(__m128i _1, __m128i _2) {`。
- **L516 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmin_hu((v8u16)_1, (v8u16)_2)`.
  **L516 CN**: 以 `(__m128i)__builtin_lsx_vmin_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L517 EN**: Closes the current lexical scope or compound statement.
  **L517 CN**: 结束当前词法作用域或复合语句块。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L519 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L520 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L520 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L521 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmin_wu(__m128i _1, __m128i _2) {`.
  **L521 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmin_wu(__m128i _1, __m128i _2) {`。
- **L522 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmin_wu((v4u32)_1, (v4u32)_2)`.
  **L522 CN**: 以 `(__m128i)__builtin_lsx_vmin_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L523 EN**: Closes the current lexical scope or compound statement.
  **L523 CN**: 结束当前词法作用域或复合语句块。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L525 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L525 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L526 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L526 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L527 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmin_du(__m128i _1, __m128i _2) {`.
  **L527 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmin_du(__m128i _1, __m128i _2) {`。
- **L528 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmin_du((v2u64)_1, (v2u64)_2)`.
  **L528 CN**: 以 `(__m128i)__builtin_lsx_vmin_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 529-552

````c
}

#define __lsx_vmini_bu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vmini_bu((v16u8)(_1), (_2)))

#define __lsx_vmini_hu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vmini_hu((v8u16)(_1), (_2)))

#define __lsx_vmini_wu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vmini_wu((v4u32)(_1), (_2)))

#define __lsx_vmini_du(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vmini_du((v2u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vseq_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vseq_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vseq_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vseq_h((v8i16)_1, (v8i16)_2);
````
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L531 EN**: Defines macro `__lsx_vmini_bu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L531 CN**: 定义宏 `__lsx_vmini_bu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L532 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmini_bu`.
  **L532 CN**: 继续与可调用符号 `__builtin_lsx_vmini_bu` 相关的逻辑。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L534 EN**: Defines macro `__lsx_vmini_hu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L534 CN**: 定义宏 `__lsx_vmini_hu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L535 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmini_hu`.
  **L535 CN**: 继续与可调用符号 `__builtin_lsx_vmini_hu` 相关的逻辑。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Defines macro `__lsx_vmini_wu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L537 CN**: 定义宏 `__lsx_vmini_wu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L538 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmini_wu`.
  **L538 CN**: 继续与可调用符号 `__builtin_lsx_vmini_wu` 相关的逻辑。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L540 EN**: Defines macro `__lsx_vmini_du(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L540 CN**: 定义宏 `__lsx_vmini_du(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L541 EN**: Continues logic associated with callable symbol `__builtin_lsx_vmini_du`.
  **L541 CN**: 继续与可调用符号 `__builtin_lsx_vmini_du` 相关的逻辑。
- **L542 EN**: Blank line separating nearby declarations or logic blocks.
  **L542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L543 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L543 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L544 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L544 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L545 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vseq_b(__m128i _1, __m128i _2) {`.
  **L545 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vseq_b(__m128i _1, __m128i _2) {`。
- **L546 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vseq_b((v16i8)_1, (v16i8)_2)`.
  **L546 CN**: 以 `(__m128i)__builtin_lsx_vseq_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Blank line separating nearby declarations or logic blocks.
  **L548 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L549 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L549 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L550 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L550 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L551 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vseq_h(__m128i _1, __m128i _2) {`.
  **L551 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vseq_h(__m128i _1, __m128i _2) {`。
- **L552 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vseq_h((v8i16)_1, (v8i16)_2)`.
  **L552 CN**: 以 `(__m128i)__builtin_lsx_vseq_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 553-576

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vseq_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vseq_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vseq_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vseq_d((v2i64)_1, (v2i64)_2);
}

#define __lsx_vseqi_b(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vseqi_b((v16i8)(_1), (_2)))

#define __lsx_vseqi_h(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vseqi_h((v8i16)(_1), (_2)))

#define __lsx_vseqi_w(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vseqi_w((v4i32)(_1), (_2)))

#define __lsx_vseqi_d(/*__m128i*/ _1, /*si5*/ _2)                              \
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L555 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L556 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L556 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L557 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vseq_w(__m128i _1, __m128i _2) {`.
  **L557 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vseq_w(__m128i _1, __m128i _2) {`。
- **L558 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vseq_w((v4i32)_1, (v4i32)_2)`.
  **L558 CN**: 以 `(__m128i)__builtin_lsx_vseq_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L559 EN**: Closes the current lexical scope or compound statement.
  **L559 CN**: 结束当前词法作用域或复合语句块。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L561 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L561 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L562 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L562 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L563 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vseq_d(__m128i _1, __m128i _2) {`.
  **L563 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vseq_d(__m128i _1, __m128i _2) {`。
- **L564 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vseq_d((v2i64)_1, (v2i64)_2)`.
  **L564 CN**: 以 `(__m128i)__builtin_lsx_vseq_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L565 EN**: Closes the current lexical scope or compound statement.
  **L565 CN**: 结束当前词法作用域或复合语句块。
- **L566 EN**: Blank line separating nearby declarations or logic blocks.
  **L566 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L567 EN**: Defines macro `__lsx_vseqi_b(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L567 CN**: 定义宏 `__lsx_vseqi_b(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L568 EN**: Continues logic associated with callable symbol `__builtin_lsx_vseqi_b`.
  **L568 CN**: 继续与可调用符号 `__builtin_lsx_vseqi_b` 相关的逻辑。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L570 EN**: Defines macro `__lsx_vseqi_h(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L570 CN**: 定义宏 `__lsx_vseqi_h(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L571 EN**: Continues logic associated with callable symbol `__builtin_lsx_vseqi_h`.
  **L571 CN**: 继续与可调用符号 `__builtin_lsx_vseqi_h` 相关的逻辑。
- **L572 EN**: Blank line separating nearby declarations or logic blocks.
  **L572 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L573 EN**: Defines macro `__lsx_vseqi_w(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L573 CN**: 定义宏 `__lsx_vseqi_w(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L574 EN**: Continues logic associated with callable symbol `__builtin_lsx_vseqi_w`.
  **L574 CN**: 继续与可调用符号 `__builtin_lsx_vseqi_w` 相关的逻辑。
- **L575 EN**: Blank line separating nearby declarations or logic blocks.
  **L575 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L576 EN**: Defines macro `__lsx_vseqi_d(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L576 CN**: 定义宏 `__lsx_vseqi_d(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 577-600

````c
  ((__m128i)__builtin_lsx_vseqi_d((v2i64)(_1), (_2)))

#define __lsx_vslti_b(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslti_b((v16i8)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vslt_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vslt_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vslt_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vslt_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vslt_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vslt_w((v4i32)_1, (v4i32)_2);
}

extern __inline
````
- **L577 EN**: Continues logic associated with callable symbol `__builtin_lsx_vseqi_d`.
  **L577 CN**: 继续与可调用符号 `__builtin_lsx_vseqi_d` 相关的逻辑。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L579 EN**: Defines macro `__lsx_vslti_b(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L579 CN**: 定义宏 `__lsx_vslti_b(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L580 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslti_b`.
  **L580 CN**: 继续与可调用符号 `__builtin_lsx_vslti_b` 相关的逻辑。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L582 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L583 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L583 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L584 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vslt_b(__m128i _1, __m128i _2) {`.
  **L584 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vslt_b(__m128i _1, __m128i _2) {`。
- **L585 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vslt_b((v16i8)_1, (v16i8)_2)`.
  **L585 CN**: 以 `(__m128i)__builtin_lsx_vslt_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L586 EN**: Closes the current lexical scope or compound statement.
  **L586 CN**: 结束当前词法作用域或复合语句块。
- **L587 EN**: Blank line separating nearby declarations or logic blocks.
  **L587 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L588 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L588 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L589 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L589 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L590 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vslt_h(__m128i _1, __m128i _2) {`.
  **L590 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vslt_h(__m128i _1, __m128i _2) {`。
- **L591 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vslt_h((v8i16)_1, (v8i16)_2)`.
  **L591 CN**: 以 `(__m128i)__builtin_lsx_vslt_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L592 EN**: Closes the current lexical scope or compound statement.
  **L592 CN**: 结束当前词法作用域或复合语句块。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L594 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L595 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L595 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L596 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vslt_w(__m128i _1, __m128i _2) {`.
  **L596 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vslt_w(__m128i _1, __m128i _2) {`。
- **L597 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vslt_w((v4i32)_1, (v4i32)_2)`.
  **L597 CN**: 以 `(__m128i)__builtin_lsx_vslt_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L598 EN**: Closes the current lexical scope or compound statement.
  **L598 CN**: 结束当前词法作用域或复合语句块。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L600 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L600 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 601-624

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vslt_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vslt_d((v2i64)_1, (v2i64)_2);
}

#define __lsx_vslti_h(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslti_h((v8i16)(_1), (_2)))

#define __lsx_vslti_w(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslti_w((v4i32)(_1), (_2)))

#define __lsx_vslti_d(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslti_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vslt_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vslt_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vslt_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vslt_hu((v8u16)_1, (v8u16)_2);
````
- **L601 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L601 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L602 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vslt_d(__m128i _1, __m128i _2) {`.
  **L602 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vslt_d(__m128i _1, __m128i _2) {`。
- **L603 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vslt_d((v2i64)_1, (v2i64)_2)`.
  **L603 CN**: 以 `(__m128i)__builtin_lsx_vslt_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L604 EN**: Closes the current lexical scope or compound statement.
  **L604 CN**: 结束当前词法作用域或复合语句块。
- **L605 EN**: Blank line separating nearby declarations or logic blocks.
  **L605 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L606 EN**: Defines macro `__lsx_vslti_h(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L606 CN**: 定义宏 `__lsx_vslti_h(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L607 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslti_h`.
  **L607 CN**: 继续与可调用符号 `__builtin_lsx_vslti_h` 相关的逻辑。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Defines macro `__lsx_vslti_w(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L609 CN**: 定义宏 `__lsx_vslti_w(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L610 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslti_w`.
  **L610 CN**: 继续与可调用符号 `__builtin_lsx_vslti_w` 相关的逻辑。
- **L611 EN**: Blank line separating nearby declarations or logic blocks.
  **L611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L612 EN**: Defines macro `__lsx_vslti_d(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L612 CN**: 定义宏 `__lsx_vslti_d(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L613 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslti_d`.
  **L613 CN**: 继续与可调用符号 `__builtin_lsx_vslti_d` 相关的逻辑。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L615 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L615 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L616 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L616 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L617 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vslt_bu(__m128i _1, __m128i _2) {`.
  **L617 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vslt_bu(__m128i _1, __m128i _2) {`。
- **L618 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vslt_bu((v16u8)_1, (v16u8)_2)`.
  **L618 CN**: 以 `(__m128i)__builtin_lsx_vslt_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L619 EN**: Closes the current lexical scope or compound statement.
  **L619 CN**: 结束当前词法作用域或复合语句块。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L621 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L622 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L622 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L623 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vslt_hu(__m128i _1, __m128i _2) {`.
  **L623 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vslt_hu(__m128i _1, __m128i _2) {`。
- **L624 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vslt_hu((v8u16)_1, (v8u16)_2)`.
  **L624 CN**: 以 `(__m128i)__builtin_lsx_vslt_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。

### Lines 625-648

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vslt_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vslt_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vslt_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vslt_du((v2u64)_1, (v2u64)_2);
}

#define __lsx_vslti_bu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vslti_bu((v16u8)(_1), (_2)))

#define __lsx_vslti_hu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vslti_hu((v8u16)(_1), (_2)))

#define __lsx_vslti_wu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vslti_wu((v4u32)(_1), (_2)))

#define __lsx_vslti_du(/*__m128i*/ _1, /*ui5*/ _2)                             \
````
- **L625 EN**: Closes the current lexical scope or compound statement.
  **L625 CN**: 结束当前词法作用域或复合语句块。
- **L626 EN**: Blank line separating nearby declarations or logic blocks.
  **L626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L627 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L627 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L628 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L628 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L629 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vslt_wu(__m128i _1, __m128i _2) {`.
  **L629 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vslt_wu(__m128i _1, __m128i _2) {`。
- **L630 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vslt_wu((v4u32)_1, (v4u32)_2)`.
  **L630 CN**: 以 `(__m128i)__builtin_lsx_vslt_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L633 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L634 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L634 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L635 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vslt_du(__m128i _1, __m128i _2) {`.
  **L635 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vslt_du(__m128i _1, __m128i _2) {`。
- **L636 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vslt_du((v2u64)_1, (v2u64)_2)`.
  **L636 CN**: 以 `(__m128i)__builtin_lsx_vslt_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L639 EN**: Defines macro `__lsx_vslti_bu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L639 CN**: 定义宏 `__lsx_vslti_bu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L640 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslti_bu`.
  **L640 CN**: 继续与可调用符号 `__builtin_lsx_vslti_bu` 相关的逻辑。
- **L641 EN**: Blank line separating nearby declarations or logic blocks.
  **L641 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L642 EN**: Defines macro `__lsx_vslti_hu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L642 CN**: 定义宏 `__lsx_vslti_hu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L643 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslti_hu`.
  **L643 CN**: 继续与可调用符号 `__builtin_lsx_vslti_hu` 相关的逻辑。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L645 EN**: Defines macro `__lsx_vslti_wu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L645 CN**: 定义宏 `__lsx_vslti_wu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L646 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslti_wu`.
  **L646 CN**: 继续与可调用符号 `__builtin_lsx_vslti_wu` 相关的逻辑。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L648 EN**: Defines macro `__lsx_vslti_du(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L648 CN**: 定义宏 `__lsx_vslti_du(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 649-672

````c
  ((__m128i)__builtin_lsx_vslti_du((v2u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsle_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsle_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsle_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsle_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsle_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsle_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsle_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsle_d((v2i64)_1, (v2i64)_2);
````
- **L649 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslti_du`.
  **L649 CN**: 继续与可调用符号 `__builtin_lsx_vslti_du` 相关的逻辑。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L651 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L651 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L652 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L652 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L653 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsle_b(__m128i _1, __m128i _2) {`.
  **L653 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsle_b(__m128i _1, __m128i _2) {`。
- **L654 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsle_b((v16i8)_1, (v16i8)_2)`.
  **L654 CN**: 以 `(__m128i)__builtin_lsx_vsle_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L655 EN**: Closes the current lexical scope or compound statement.
  **L655 CN**: 结束当前词法作用域或复合语句块。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L657 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L658 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L658 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L659 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsle_h(__m128i _1, __m128i _2) {`.
  **L659 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsle_h(__m128i _1, __m128i _2) {`。
- **L660 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsle_h((v8i16)_1, (v8i16)_2)`.
  **L660 CN**: 以 `(__m128i)__builtin_lsx_vsle_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Blank line separating nearby declarations or logic blocks.
  **L662 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L663 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L663 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L664 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L664 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L665 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsle_w(__m128i _1, __m128i _2) {`.
  **L665 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsle_w(__m128i _1, __m128i _2) {`。
- **L666 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsle_w((v4i32)_1, (v4i32)_2)`.
  **L666 CN**: 以 `(__m128i)__builtin_lsx_vsle_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L669 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L670 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L670 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L671 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsle_d(__m128i _1, __m128i _2) {`.
  **L671 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsle_d(__m128i _1, __m128i _2) {`。
- **L672 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsle_d((v2i64)_1, (v2i64)_2)`.
  **L672 CN**: 以 `(__m128i)__builtin_lsx_vsle_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 673-696

````c
}

#define __lsx_vslei_b(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslei_b((v16i8)(_1), (_2)))

#define __lsx_vslei_h(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslei_h((v8i16)(_1), (_2)))

#define __lsx_vslei_w(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslei_w((v4i32)(_1), (_2)))

#define __lsx_vslei_d(/*__m128i*/ _1, /*si5*/ _2)                              \
  ((__m128i)__builtin_lsx_vslei_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsle_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsle_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsle_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsle_hu((v8u16)_1, (v8u16)_2);
````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L675 EN**: Defines macro `__lsx_vslei_b(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L675 CN**: 定义宏 `__lsx_vslei_b(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L676 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslei_b`.
  **L676 CN**: 继续与可调用符号 `__builtin_lsx_vslei_b` 相关的逻辑。
- **L677 EN**: Blank line separating nearby declarations or logic blocks.
  **L677 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L678 EN**: Defines macro `__lsx_vslei_h(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L678 CN**: 定义宏 `__lsx_vslei_h(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L679 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslei_h`.
  **L679 CN**: 继续与可调用符号 `__builtin_lsx_vslei_h` 相关的逻辑。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L681 EN**: Defines macro `__lsx_vslei_w(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L681 CN**: 定义宏 `__lsx_vslei_w(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L682 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslei_w`.
  **L682 CN**: 继续与可调用符号 `__builtin_lsx_vslei_w` 相关的逻辑。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L684 EN**: Defines macro `__lsx_vslei_d(/*__m128i*/ _1, /*si5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L684 CN**: 定义宏 `__lsx_vslei_d(/*__m128i*/ _1, /*si5*/ _2)`，用于条件编译、简写或 API 生成。
- **L685 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslei_d`.
  **L685 CN**: 继续与可调用符号 `__builtin_lsx_vslei_d` 相关的逻辑。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L687 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L687 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L688 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L688 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L689 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsle_bu(__m128i _1, __m128i _2) {`.
  **L689 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsle_bu(__m128i _1, __m128i _2) {`。
- **L690 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsle_bu((v16u8)_1, (v16u8)_2)`.
  **L690 CN**: 以 `(__m128i)__builtin_lsx_vsle_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L691 EN**: Closes the current lexical scope or compound statement.
  **L691 CN**: 结束当前词法作用域或复合语句块。
- **L692 EN**: Blank line separating nearby declarations or logic blocks.
  **L692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L693 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L693 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L694 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L694 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L695 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsle_hu(__m128i _1, __m128i _2) {`.
  **L695 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsle_hu(__m128i _1, __m128i _2) {`。
- **L696 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsle_hu((v8u16)_1, (v8u16)_2)`.
  **L696 CN**: 以 `(__m128i)__builtin_lsx_vsle_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。

### Lines 697-720

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsle_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsle_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsle_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsle_du((v2u64)_1, (v2u64)_2);
}

#define __lsx_vslei_bu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vslei_bu((v16u8)(_1), (_2)))

#define __lsx_vslei_hu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vslei_hu((v8u16)(_1), (_2)))

#define __lsx_vslei_wu(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vslei_wu((v4u32)(_1), (_2)))

#define __lsx_vslei_du(/*__m128i*/ _1, /*ui5*/ _2)                             \
````
- **L697 EN**: Closes the current lexical scope or compound statement.
  **L697 CN**: 结束当前词法作用域或复合语句块。
- **L698 EN**: Blank line separating nearby declarations or logic blocks.
  **L698 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L699 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L699 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L700 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L700 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L701 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsle_wu(__m128i _1, __m128i _2) {`.
  **L701 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsle_wu(__m128i _1, __m128i _2) {`。
- **L702 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsle_wu((v4u32)_1, (v4u32)_2)`.
  **L702 CN**: 以 `(__m128i)__builtin_lsx_vsle_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L703 EN**: Closes the current lexical scope or compound statement.
  **L703 CN**: 结束当前词法作用域或复合语句块。
- **L704 EN**: Blank line separating nearby declarations or logic blocks.
  **L704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L705 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L705 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L706 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L706 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L707 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsle_du(__m128i _1, __m128i _2) {`.
  **L707 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsle_du(__m128i _1, __m128i _2) {`。
- **L708 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsle_du((v2u64)_1, (v2u64)_2)`.
  **L708 CN**: 以 `(__m128i)__builtin_lsx_vsle_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L709 EN**: Closes the current lexical scope or compound statement.
  **L709 CN**: 结束当前词法作用域或复合语句块。
- **L710 EN**: Blank line separating nearby declarations or logic blocks.
  **L710 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L711 EN**: Defines macro `__lsx_vslei_bu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L711 CN**: 定义宏 `__lsx_vslei_bu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L712 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslei_bu`.
  **L712 CN**: 继续与可调用符号 `__builtin_lsx_vslei_bu` 相关的逻辑。
- **L713 EN**: Blank line separating nearby declarations or logic blocks.
  **L713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L714 EN**: Defines macro `__lsx_vslei_hu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L714 CN**: 定义宏 `__lsx_vslei_hu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L715 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslei_hu`.
  **L715 CN**: 继续与可调用符号 `__builtin_lsx_vslei_hu` 相关的逻辑。
- **L716 EN**: Blank line separating nearby declarations or logic blocks.
  **L716 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L717 EN**: Defines macro `__lsx_vslei_wu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L717 CN**: 定义宏 `__lsx_vslei_wu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L718 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslei_wu`.
  **L718 CN**: 继续与可调用符号 `__builtin_lsx_vslei_wu` 相关的逻辑。
- **L719 EN**: Blank line separating nearby declarations or logic blocks.
  **L719 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L720 EN**: Defines macro `__lsx_vslei_du(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L720 CN**: 定义宏 `__lsx_vslei_du(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 721-744

````c
  ((__m128i)__builtin_lsx_vslei_du((v2u64)(_1), (_2)))

#define __lsx_vsat_b(/*__m128i*/ _1, /*ui3*/ _2)                               \
  ((__m128i)__builtin_lsx_vsat_b((v16i8)(_1), (_2)))

#define __lsx_vsat_h(/*__m128i*/ _1, /*ui4*/ _2)                               \
  ((__m128i)__builtin_lsx_vsat_h((v8i16)(_1), (_2)))

#define __lsx_vsat_w(/*__m128i*/ _1, /*ui5*/ _2)                               \
  ((__m128i)__builtin_lsx_vsat_w((v4i32)(_1), (_2)))

#define __lsx_vsat_d(/*__m128i*/ _1, /*ui6*/ _2)                               \
  ((__m128i)__builtin_lsx_vsat_d((v2i64)(_1), (_2)))

#define __lsx_vsat_bu(/*__m128i*/ _1, /*ui3*/ _2)                              \
  ((__m128i)__builtin_lsx_vsat_bu((v16u8)(_1), (_2)))

#define __lsx_vsat_hu(/*__m128i*/ _1, /*ui4*/ _2)                              \
  ((__m128i)__builtin_lsx_vsat_hu((v8u16)(_1), (_2)))

#define __lsx_vsat_wu(/*__m128i*/ _1, /*ui5*/ _2)                              \
  ((__m128i)__builtin_lsx_vsat_wu((v4u32)(_1), (_2)))

#define __lsx_vsat_du(/*__m128i*/ _1, /*ui6*/ _2)                              \
````
- **L721 EN**: Continues logic associated with callable symbol `__builtin_lsx_vslei_du`.
  **L721 CN**: 继续与可调用符号 `__builtin_lsx_vslei_du` 相关的逻辑。
- **L722 EN**: Blank line separating nearby declarations or logic blocks.
  **L722 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L723 EN**: Defines macro `__lsx_vsat_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L723 CN**: 定义宏 `__lsx_vsat_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L724 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsat_b`.
  **L724 CN**: 继续与可调用符号 `__builtin_lsx_vsat_b` 相关的逻辑。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L726 EN**: Defines macro `__lsx_vsat_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L726 CN**: 定义宏 `__lsx_vsat_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L727 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsat_h`.
  **L727 CN**: 继续与可调用符号 `__builtin_lsx_vsat_h` 相关的逻辑。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L729 EN**: Defines macro `__lsx_vsat_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L729 CN**: 定义宏 `__lsx_vsat_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L730 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsat_w`.
  **L730 CN**: 继续与可调用符号 `__builtin_lsx_vsat_w` 相关的逻辑。
- **L731 EN**: Blank line separating nearby declarations or logic blocks.
  **L731 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L732 EN**: Defines macro `__lsx_vsat_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L732 CN**: 定义宏 `__lsx_vsat_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L733 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsat_d`.
  **L733 CN**: 继续与可调用符号 `__builtin_lsx_vsat_d` 相关的逻辑。
- **L734 EN**: Blank line separating nearby declarations or logic blocks.
  **L734 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L735 EN**: Defines macro `__lsx_vsat_bu(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L735 CN**: 定义宏 `__lsx_vsat_bu(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L736 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsat_bu`.
  **L736 CN**: 继续与可调用符号 `__builtin_lsx_vsat_bu` 相关的逻辑。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L738 EN**: Defines macro `__lsx_vsat_hu(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L738 CN**: 定义宏 `__lsx_vsat_hu(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L739 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsat_hu`.
  **L739 CN**: 继续与可调用符号 `__builtin_lsx_vsat_hu` 相关的逻辑。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Defines macro `__lsx_vsat_wu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L741 CN**: 定义宏 `__lsx_vsat_wu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L742 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsat_wu`.
  **L742 CN**: 继续与可调用符号 `__builtin_lsx_vsat_wu` 相关的逻辑。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L744 EN**: Defines macro `__lsx_vsat_du(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L744 CN**: 定义宏 `__lsx_vsat_du(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 745-768

````c
  ((__m128i)__builtin_lsx_vsat_du((v2u64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadda_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadda_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadda_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadda_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadda_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadda_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadda_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadda_d((v2i64)_1, (v2i64)_2);
````
- **L745 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsat_du`.
  **L745 CN**: 继续与可调用符号 `__builtin_lsx_vsat_du` 相关的逻辑。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L747 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L747 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L748 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L748 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L749 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadda_b(__m128i _1, __m128i _2) {`.
  **L749 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadda_b(__m128i _1, __m128i _2) {`。
- **L750 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadda_b((v16i8)_1, (v16i8)_2)`.
  **L750 CN**: 以 `(__m128i)__builtin_lsx_vadda_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L751 EN**: Closes the current lexical scope or compound statement.
  **L751 CN**: 结束当前词法作用域或复合语句块。
- **L752 EN**: Blank line separating nearby declarations or logic blocks.
  **L752 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L753 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L753 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L754 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L754 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L755 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadda_h(__m128i _1, __m128i _2) {`.
  **L755 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadda_h(__m128i _1, __m128i _2) {`。
- **L756 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadda_h((v8i16)_1, (v8i16)_2)`.
  **L756 CN**: 以 `(__m128i)__builtin_lsx_vadda_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L757 EN**: Closes the current lexical scope or compound statement.
  **L757 CN**: 结束当前词法作用域或复合语句块。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L759 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L759 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L760 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L760 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L761 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadda_w(__m128i _1, __m128i _2) {`.
  **L761 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadda_w(__m128i _1, __m128i _2) {`。
- **L762 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadda_w((v4i32)_1, (v4i32)_2)`.
  **L762 CN**: 以 `(__m128i)__builtin_lsx_vadda_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L763 EN**: Closes the current lexical scope or compound statement.
  **L763 CN**: 结束当前词法作用域或复合语句块。
- **L764 EN**: Blank line separating nearby declarations or logic blocks.
  **L764 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L765 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L765 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L766 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L766 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L767 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadda_d(__m128i _1, __m128i _2) {`.
  **L767 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadda_d(__m128i _1, __m128i _2) {`。
- **L768 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadda_d((v2i64)_1, (v2i64)_2)`.
  **L768 CN**: 以 `(__m128i)__builtin_lsx_vadda_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 769-792

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsadd_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsadd_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsadd_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsadd_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsadd_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsadd_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsadd_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsadd_d((v2i64)_1, (v2i64)_2);
````
- **L769 EN**: Closes the current lexical scope or compound statement.
  **L769 CN**: 结束当前词法作用域或复合语句块。
- **L770 EN**: Blank line separating nearby declarations or logic blocks.
  **L770 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L771 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L771 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L772 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L772 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L773 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsadd_b(__m128i _1, __m128i _2) {`.
  **L773 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsadd_b(__m128i _1, __m128i _2) {`。
- **L774 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsadd_b((v16i8)_1, (v16i8)_2)`.
  **L774 CN**: 以 `(__m128i)__builtin_lsx_vsadd_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L775 EN**: Closes the current lexical scope or compound statement.
  **L775 CN**: 结束当前词法作用域或复合语句块。
- **L776 EN**: Blank line separating nearby declarations or logic blocks.
  **L776 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L777 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L777 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L778 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L778 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L779 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsadd_h(__m128i _1, __m128i _2) {`.
  **L779 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsadd_h(__m128i _1, __m128i _2) {`。
- **L780 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsadd_h((v8i16)_1, (v8i16)_2)`.
  **L780 CN**: 以 `(__m128i)__builtin_lsx_vsadd_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L781 EN**: Closes the current lexical scope or compound statement.
  **L781 CN**: 结束当前词法作用域或复合语句块。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L783 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L783 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L784 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L784 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L785 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsadd_w(__m128i _1, __m128i _2) {`.
  **L785 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsadd_w(__m128i _1, __m128i _2) {`。
- **L786 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsadd_w((v4i32)_1, (v4i32)_2)`.
  **L786 CN**: 以 `(__m128i)__builtin_lsx_vsadd_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L787 EN**: Closes the current lexical scope or compound statement.
  **L787 CN**: 结束当前词法作用域或复合语句块。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L789 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L789 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L790 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L790 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L791 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsadd_d(__m128i _1, __m128i _2) {`.
  **L791 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsadd_d(__m128i _1, __m128i _2) {`。
- **L792 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsadd_d((v2i64)_1, (v2i64)_2)`.
  **L792 CN**: 以 `(__m128i)__builtin_lsx_vsadd_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 793-816

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsadd_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsadd_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsadd_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsadd_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsadd_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsadd_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsadd_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsadd_du((v2u64)_1, (v2u64)_2);
````
- **L793 EN**: Closes the current lexical scope or compound statement.
  **L793 CN**: 结束当前词法作用域或复合语句块。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L795 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L795 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L796 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L796 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L797 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsadd_bu(__m128i _1, __m128i _2) {`.
  **L797 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsadd_bu(__m128i _1, __m128i _2) {`。
- **L798 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsadd_bu((v16u8)_1, (v16u8)_2)`.
  **L798 CN**: 以 `(__m128i)__builtin_lsx_vsadd_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L799 EN**: Closes the current lexical scope or compound statement.
  **L799 CN**: 结束当前词法作用域或复合语句块。
- **L800 EN**: Blank line separating nearby declarations or logic blocks.
  **L800 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L801 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L801 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L802 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L802 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L803 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsadd_hu(__m128i _1, __m128i _2) {`.
  **L803 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsadd_hu(__m128i _1, __m128i _2) {`。
- **L804 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsadd_hu((v8u16)_1, (v8u16)_2)`.
  **L804 CN**: 以 `(__m128i)__builtin_lsx_vsadd_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L807 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L807 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L808 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L808 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L809 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsadd_wu(__m128i _1, __m128i _2) {`.
  **L809 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsadd_wu(__m128i _1, __m128i _2) {`。
- **L810 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsadd_wu((v4u32)_1, (v4u32)_2)`.
  **L810 CN**: 以 `(__m128i)__builtin_lsx_vsadd_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L811 EN**: Closes the current lexical scope or compound statement.
  **L811 CN**: 结束当前词法作用域或复合语句块。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L813 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L814 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L814 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L815 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsadd_du(__m128i _1, __m128i _2) {`.
  **L815 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsadd_du(__m128i _1, __m128i _2) {`。
- **L816 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsadd_du((v2u64)_1, (v2u64)_2)`.
  **L816 CN**: 以 `(__m128i)__builtin_lsx_vsadd_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 817-840

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavg_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavg_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavg_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavg_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavg_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavg_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavg_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavg_d((v2i64)_1, (v2i64)_2);
````
- **L817 EN**: Closes the current lexical scope or compound statement.
  **L817 CN**: 结束当前词法作用域或复合语句块。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L819 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L819 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L820 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L820 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L821 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavg_b(__m128i _1, __m128i _2) {`.
  **L821 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavg_b(__m128i _1, __m128i _2) {`。
- **L822 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavg_b((v16i8)_1, (v16i8)_2)`.
  **L822 CN**: 以 `(__m128i)__builtin_lsx_vavg_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L823 EN**: Closes the current lexical scope or compound statement.
  **L823 CN**: 结束当前词法作用域或复合语句块。
- **L824 EN**: Blank line separating nearby declarations or logic blocks.
  **L824 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L825 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L825 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L826 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L826 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L827 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavg_h(__m128i _1, __m128i _2) {`.
  **L827 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavg_h(__m128i _1, __m128i _2) {`。
- **L828 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavg_h((v8i16)_1, (v8i16)_2)`.
  **L828 CN**: 以 `(__m128i)__builtin_lsx_vavg_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L829 EN**: Closes the current lexical scope or compound statement.
  **L829 CN**: 结束当前词法作用域或复合语句块。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L831 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L831 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L832 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L832 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L833 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavg_w(__m128i _1, __m128i _2) {`.
  **L833 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavg_w(__m128i _1, __m128i _2) {`。
- **L834 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavg_w((v4i32)_1, (v4i32)_2)`.
  **L834 CN**: 以 `(__m128i)__builtin_lsx_vavg_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L835 EN**: Closes the current lexical scope or compound statement.
  **L835 CN**: 结束当前词法作用域或复合语句块。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L837 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L837 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L838 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L838 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L839 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavg_d(__m128i _1, __m128i _2) {`.
  **L839 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavg_d(__m128i _1, __m128i _2) {`。
- **L840 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavg_d((v2i64)_1, (v2i64)_2)`.
  **L840 CN**: 以 `(__m128i)__builtin_lsx_vavg_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 841-864

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavg_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavg_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavg_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavg_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavg_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavg_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavg_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavg_du((v2u64)_1, (v2u64)_2);
````
- **L841 EN**: Closes the current lexical scope or compound statement.
  **L841 CN**: 结束当前词法作用域或复合语句块。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L843 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L843 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L844 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L844 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L845 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavg_bu(__m128i _1, __m128i _2) {`.
  **L845 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavg_bu(__m128i _1, __m128i _2) {`。
- **L846 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavg_bu((v16u8)_1, (v16u8)_2)`.
  **L846 CN**: 以 `(__m128i)__builtin_lsx_vavg_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L849 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L850 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L850 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L851 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavg_hu(__m128i _1, __m128i _2) {`.
  **L851 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavg_hu(__m128i _1, __m128i _2) {`。
- **L852 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavg_hu((v8u16)_1, (v8u16)_2)`.
  **L852 CN**: 以 `(__m128i)__builtin_lsx_vavg_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L853 EN**: Closes the current lexical scope or compound statement.
  **L853 CN**: 结束当前词法作用域或复合语句块。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L855 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L855 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L856 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L856 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L857 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavg_wu(__m128i _1, __m128i _2) {`.
  **L857 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavg_wu(__m128i _1, __m128i _2) {`。
- **L858 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavg_wu((v4u32)_1, (v4u32)_2)`.
  **L858 CN**: 以 `(__m128i)__builtin_lsx_vavg_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L859 EN**: Closes the current lexical scope or compound statement.
  **L859 CN**: 结束当前词法作用域或复合语句块。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L861 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L861 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L862 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L862 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L863 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavg_du(__m128i _1, __m128i _2) {`.
  **L863 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavg_du(__m128i _1, __m128i _2) {`。
- **L864 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavg_du((v2u64)_1, (v2u64)_2)`.
  **L864 CN**: 以 `(__m128i)__builtin_lsx_vavg_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 865-888

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavgr_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavgr_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavgr_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavgr_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavgr_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavgr_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavgr_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavgr_d((v2i64)_1, (v2i64)_2);
````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L867 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L868 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L868 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L869 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavgr_b(__m128i _1, __m128i _2) {`.
  **L869 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavgr_b(__m128i _1, __m128i _2) {`。
- **L870 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavgr_b((v16i8)_1, (v16i8)_2)`.
  **L870 CN**: 以 `(__m128i)__builtin_lsx_vavgr_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L871 EN**: Closes the current lexical scope or compound statement.
  **L871 CN**: 结束当前词法作用域或复合语句块。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L873 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L873 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L874 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L874 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L875 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavgr_h(__m128i _1, __m128i _2) {`.
  **L875 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavgr_h(__m128i _1, __m128i _2) {`。
- **L876 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavgr_h((v8i16)_1, (v8i16)_2)`.
  **L876 CN**: 以 `(__m128i)__builtin_lsx_vavgr_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L877 EN**: Closes the current lexical scope or compound statement.
  **L877 CN**: 结束当前词法作用域或复合语句块。
- **L878 EN**: Blank line separating nearby declarations or logic blocks.
  **L878 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L879 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L879 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L880 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L880 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L881 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavgr_w(__m128i _1, __m128i _2) {`.
  **L881 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavgr_w(__m128i _1, __m128i _2) {`。
- **L882 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavgr_w((v4i32)_1, (v4i32)_2)`.
  **L882 CN**: 以 `(__m128i)__builtin_lsx_vavgr_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L883 EN**: Closes the current lexical scope or compound statement.
  **L883 CN**: 结束当前词法作用域或复合语句块。
- **L884 EN**: Blank line separating nearby declarations or logic blocks.
  **L884 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L885 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L885 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L886 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L886 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L887 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavgr_d(__m128i _1, __m128i _2) {`.
  **L887 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavgr_d(__m128i _1, __m128i _2) {`。
- **L888 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavgr_d((v2i64)_1, (v2i64)_2)`.
  **L888 CN**: 以 `(__m128i)__builtin_lsx_vavgr_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 889-912

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavgr_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavgr_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavgr_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavgr_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavgr_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavgr_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vavgr_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vavgr_du((v2u64)_1, (v2u64)_2);
````
- **L889 EN**: Closes the current lexical scope or compound statement.
  **L889 CN**: 结束当前词法作用域或复合语句块。
- **L890 EN**: Blank line separating nearby declarations or logic blocks.
  **L890 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L891 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L891 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L892 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L892 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L893 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavgr_bu(__m128i _1, __m128i _2) {`.
  **L893 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavgr_bu(__m128i _1, __m128i _2) {`。
- **L894 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavgr_bu((v16u8)_1, (v16u8)_2)`.
  **L894 CN**: 以 `(__m128i)__builtin_lsx_vavgr_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L895 EN**: Closes the current lexical scope or compound statement.
  **L895 CN**: 结束当前词法作用域或复合语句块。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L897 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L897 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L898 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L898 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L899 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavgr_hu(__m128i _1, __m128i _2) {`.
  **L899 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavgr_hu(__m128i _1, __m128i _2) {`。
- **L900 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavgr_hu((v8u16)_1, (v8u16)_2)`.
  **L900 CN**: 以 `(__m128i)__builtin_lsx_vavgr_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L901 EN**: Closes the current lexical scope or compound statement.
  **L901 CN**: 结束当前词法作用域或复合语句块。
- **L902 EN**: Blank line separating nearby declarations or logic blocks.
  **L902 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L903 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L903 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L904 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L904 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L905 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavgr_wu(__m128i _1, __m128i _2) {`.
  **L905 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavgr_wu(__m128i _1, __m128i _2) {`。
- **L906 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavgr_wu((v4u32)_1, (v4u32)_2)`.
  **L906 CN**: 以 `(__m128i)__builtin_lsx_vavgr_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L907 EN**: Closes the current lexical scope or compound statement.
  **L907 CN**: 结束当前词法作用域或复合语句块。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L909 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L909 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L910 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L910 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L911 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vavgr_du(__m128i _1, __m128i _2) {`.
  **L911 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vavgr_du(__m128i _1, __m128i _2) {`。
- **L912 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vavgr_du((v2u64)_1, (v2u64)_2)`.
  **L912 CN**: 以 `(__m128i)__builtin_lsx_vavgr_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 913-936

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssub_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssub_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssub_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssub_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssub_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssub_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssub_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssub_d((v2i64)_1, (v2i64)_2);
````
- **L913 EN**: Closes the current lexical scope or compound statement.
  **L913 CN**: 结束当前词法作用域或复合语句块。
- **L914 EN**: Blank line separating nearby declarations or logic blocks.
  **L914 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L915 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L915 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L916 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L916 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L917 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssub_b(__m128i _1, __m128i _2) {`.
  **L917 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssub_b(__m128i _1, __m128i _2) {`。
- **L918 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssub_b((v16i8)_1, (v16i8)_2)`.
  **L918 CN**: 以 `(__m128i)__builtin_lsx_vssub_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L919 EN**: Closes the current lexical scope or compound statement.
  **L919 CN**: 结束当前词法作用域或复合语句块。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L921 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L921 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L922 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L922 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L923 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssub_h(__m128i _1, __m128i _2) {`.
  **L923 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssub_h(__m128i _1, __m128i _2) {`。
- **L924 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssub_h((v8i16)_1, (v8i16)_2)`.
  **L924 CN**: 以 `(__m128i)__builtin_lsx_vssub_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L925 EN**: Closes the current lexical scope or compound statement.
  **L925 CN**: 结束当前词法作用域或复合语句块。
- **L926 EN**: Blank line separating nearby declarations or logic blocks.
  **L926 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L927 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L927 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L928 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L928 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L929 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssub_w(__m128i _1, __m128i _2) {`.
  **L929 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssub_w(__m128i _1, __m128i _2) {`。
- **L930 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssub_w((v4i32)_1, (v4i32)_2)`.
  **L930 CN**: 以 `(__m128i)__builtin_lsx_vssub_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L931 EN**: Closes the current lexical scope or compound statement.
  **L931 CN**: 结束当前词法作用域或复合语句块。
- **L932 EN**: Blank line separating nearby declarations or logic blocks.
  **L932 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L933 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L933 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L934 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L934 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L935 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssub_d(__m128i _1, __m128i _2) {`.
  **L935 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssub_d(__m128i _1, __m128i _2) {`。
- **L936 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssub_d((v2i64)_1, (v2i64)_2)`.
  **L936 CN**: 以 `(__m128i)__builtin_lsx_vssub_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 937-960

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssub_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssub_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssub_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssub_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssub_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssub_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssub_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssub_du((v2u64)_1, (v2u64)_2);
````
- **L937 EN**: Closes the current lexical scope or compound statement.
  **L937 CN**: 结束当前词法作用域或复合语句块。
- **L938 EN**: Blank line separating nearby declarations or logic blocks.
  **L938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L939 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L939 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L940 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L940 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L941 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssub_bu(__m128i _1, __m128i _2) {`.
  **L941 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssub_bu(__m128i _1, __m128i _2) {`。
- **L942 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssub_bu((v16u8)_1, (v16u8)_2)`.
  **L942 CN**: 以 `(__m128i)__builtin_lsx_vssub_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L943 EN**: Closes the current lexical scope or compound statement.
  **L943 CN**: 结束当前词法作用域或复合语句块。
- **L944 EN**: Blank line separating nearby declarations or logic blocks.
  **L944 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L945 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L945 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L946 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L946 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L947 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssub_hu(__m128i _1, __m128i _2) {`.
  **L947 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssub_hu(__m128i _1, __m128i _2) {`。
- **L948 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssub_hu((v8u16)_1, (v8u16)_2)`.
  **L948 CN**: 以 `(__m128i)__builtin_lsx_vssub_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Blank line separating nearby declarations or logic blocks.
  **L950 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L951 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L951 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L952 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L952 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L953 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssub_wu(__m128i _1, __m128i _2) {`.
  **L953 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssub_wu(__m128i _1, __m128i _2) {`。
- **L954 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssub_wu((v4u32)_1, (v4u32)_2)`.
  **L954 CN**: 以 `(__m128i)__builtin_lsx_vssub_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L955 EN**: Closes the current lexical scope or compound statement.
  **L955 CN**: 结束当前词法作用域或复合语句块。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L957 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L957 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L958 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L958 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L959 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssub_du(__m128i _1, __m128i _2) {`.
  **L959 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssub_du(__m128i _1, __m128i _2) {`。
- **L960 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssub_du((v2u64)_1, (v2u64)_2)`.
  **L960 CN**: 以 `(__m128i)__builtin_lsx_vssub_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 961-984

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vabsd_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vabsd_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vabsd_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vabsd_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vabsd_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vabsd_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vabsd_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vabsd_d((v2i64)_1, (v2i64)_2);
````
- **L961 EN**: Closes the current lexical scope or compound statement.
  **L961 CN**: 结束当前词法作用域或复合语句块。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L963 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L963 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L964 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L964 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L965 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vabsd_b(__m128i _1, __m128i _2) {`.
  **L965 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vabsd_b(__m128i _1, __m128i _2) {`。
- **L966 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vabsd_b((v16i8)_1, (v16i8)_2)`.
  **L966 CN**: 以 `(__m128i)__builtin_lsx_vabsd_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L967 EN**: Closes the current lexical scope or compound statement.
  **L967 CN**: 结束当前词法作用域或复合语句块。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L969 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L970 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L970 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L971 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vabsd_h(__m128i _1, __m128i _2) {`.
  **L971 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vabsd_h(__m128i _1, __m128i _2) {`。
- **L972 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vabsd_h((v8i16)_1, (v8i16)_2)`.
  **L972 CN**: 以 `(__m128i)__builtin_lsx_vabsd_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L973 EN**: Closes the current lexical scope or compound statement.
  **L973 CN**: 结束当前词法作用域或复合语句块。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L975 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L976 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L976 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L977 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vabsd_w(__m128i _1, __m128i _2) {`.
  **L977 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vabsd_w(__m128i _1, __m128i _2) {`。
- **L978 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vabsd_w((v4i32)_1, (v4i32)_2)`.
  **L978 CN**: 以 `(__m128i)__builtin_lsx_vabsd_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L979 EN**: Closes the current lexical scope or compound statement.
  **L979 CN**: 结束当前词法作用域或复合语句块。
- **L980 EN**: Blank line separating nearby declarations or logic blocks.
  **L980 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L981 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L981 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L982 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L982 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L983 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vabsd_d(__m128i _1, __m128i _2) {`.
  **L983 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vabsd_d(__m128i _1, __m128i _2) {`。
- **L984 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vabsd_d((v2i64)_1, (v2i64)_2)`.
  **L984 CN**: 以 `(__m128i)__builtin_lsx_vabsd_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 985-1008

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vabsd_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vabsd_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vabsd_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vabsd_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vabsd_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vabsd_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vabsd_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vabsd_du((v2u64)_1, (v2u64)_2);
````
- **L985 EN**: Closes the current lexical scope or compound statement.
  **L985 CN**: 结束当前词法作用域或复合语句块。
- **L986 EN**: Blank line separating nearby declarations or logic blocks.
  **L986 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L987 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L987 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L988 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L988 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L989 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vabsd_bu(__m128i _1, __m128i _2) {`.
  **L989 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vabsd_bu(__m128i _1, __m128i _2) {`。
- **L990 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vabsd_bu((v16u8)_1, (v16u8)_2)`.
  **L990 CN**: 以 `(__m128i)__builtin_lsx_vabsd_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L991 EN**: Closes the current lexical scope or compound statement.
  **L991 CN**: 结束当前词法作用域或复合语句块。
- **L992 EN**: Blank line separating nearby declarations or logic blocks.
  **L992 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L993 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L993 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L994 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L994 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L995 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vabsd_hu(__m128i _1, __m128i _2) {`.
  **L995 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vabsd_hu(__m128i _1, __m128i _2) {`。
- **L996 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vabsd_hu((v8u16)_1, (v8u16)_2)`.
  **L996 CN**: 以 `(__m128i)__builtin_lsx_vabsd_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L997 EN**: Closes the current lexical scope or compound statement.
  **L997 CN**: 结束当前词法作用域或复合语句块。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L999 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L999 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1000 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1000 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1001 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vabsd_wu(__m128i _1, __m128i _2) {`.
  **L1001 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vabsd_wu(__m128i _1, __m128i _2) {`。
- **L1002 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vabsd_wu((v4u32)_1, (v4u32)_2)`.
  **L1002 CN**: 以 `(__m128i)__builtin_lsx_vabsd_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L1003 EN**: Closes the current lexical scope or compound statement.
  **L1003 CN**: 结束当前词法作用域或复合语句块。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1005 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1005 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1006 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1006 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1007 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vabsd_du(__m128i _1, __m128i _2) {`.
  **L1007 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vabsd_du(__m128i _1, __m128i _2) {`。
- **L1008 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vabsd_du((v2u64)_1, (v2u64)_2)`.
  **L1008 CN**: 以 `(__m128i)__builtin_lsx_vabsd_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 1009-1032

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmul_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmul_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmul_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmul_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmul_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmul_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmul_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmul_d((v2i64)_1, (v2i64)_2);
````
- **L1009 EN**: Closes the current lexical scope or compound statement.
  **L1009 CN**: 结束当前词法作用域或复合语句块。
- **L1010 EN**: Blank line separating nearby declarations or logic blocks.
  **L1010 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1011 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1011 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1012 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1012 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1013 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmul_b(__m128i _1, __m128i _2) {`.
  **L1013 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmul_b(__m128i _1, __m128i _2) {`。
- **L1014 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmul_b((v16i8)_1, (v16i8)_2)`.
  **L1014 CN**: 以 `(__m128i)__builtin_lsx_vmul_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1015 EN**: Closes the current lexical scope or compound statement.
  **L1015 CN**: 结束当前词法作用域或复合语句块。
- **L1016 EN**: Blank line separating nearby declarations or logic blocks.
  **L1016 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1017 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1017 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1018 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1018 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1019 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmul_h(__m128i _1, __m128i _2) {`.
  **L1019 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmul_h(__m128i _1, __m128i _2) {`。
- **L1020 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmul_h((v8i16)_1, (v8i16)_2)`.
  **L1020 CN**: 以 `(__m128i)__builtin_lsx_vmul_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L1021 EN**: Closes the current lexical scope or compound statement.
  **L1021 CN**: 结束当前词法作用域或复合语句块。
- **L1022 EN**: Blank line separating nearby declarations or logic blocks.
  **L1022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1023 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1023 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1024 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1024 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1025 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmul_w(__m128i _1, __m128i _2) {`.
  **L1025 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmul_w(__m128i _1, __m128i _2) {`。
- **L1026 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmul_w((v4i32)_1, (v4i32)_2)`.
  **L1026 CN**: 以 `(__m128i)__builtin_lsx_vmul_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1027 EN**: Closes the current lexical scope or compound statement.
  **L1027 CN**: 结束当前词法作用域或复合语句块。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1029 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1029 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1030 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1030 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1031 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmul_d(__m128i _1, __m128i _2) {`.
  **L1031 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmul_d(__m128i _1, __m128i _2) {`。
- **L1032 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmul_d((v2i64)_1, (v2i64)_2)`.
  **L1032 CN**: 以 `(__m128i)__builtin_lsx_vmul_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 1033-1056

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmadd_b(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmadd_b((v16i8)_1, (v16i8)_2, (v16i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmadd_h(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmadd_h((v8i16)_1, (v8i16)_2, (v8i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmadd_w(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmadd_w((v4i32)_1, (v4i32)_2, (v4i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmadd_d(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmadd_d((v2i64)_1, (v2i64)_2, (v2i64)_3);
````
- **L1033 EN**: Closes the current lexical scope or compound statement.
  **L1033 CN**: 结束当前词法作用域或复合语句块。
- **L1034 EN**: Blank line separating nearby declarations or logic blocks.
  **L1034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1035 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1035 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1036 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1036 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1037 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmadd_b(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1037 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmadd_b(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1038 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmadd_b((v16i8)_1, (v16i8)_2, (v16i8)_3)`.
  **L1038 CN**: 以 `(__m128i)__builtin_lsx_vmadd_b((v16i8)_1, (v16i8)_2, (v16i8)_3)` 从当前函数返回。
- **L1039 EN**: Closes the current lexical scope or compound statement.
  **L1039 CN**: 结束当前词法作用域或复合语句块。
- **L1040 EN**: Blank line separating nearby declarations or logic blocks.
  **L1040 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1041 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1041 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1042 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1042 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1043 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmadd_h(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1043 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmadd_h(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1044 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmadd_h((v8i16)_1, (v8i16)_2, (v8i16)_3)`.
  **L1044 CN**: 以 `(__m128i)__builtin_lsx_vmadd_h((v8i16)_1, (v8i16)_2, (v8i16)_3)` 从当前函数返回。
- **L1045 EN**: Closes the current lexical scope or compound statement.
  **L1045 CN**: 结束当前词法作用域或复合语句块。
- **L1046 EN**: Blank line separating nearby declarations or logic blocks.
  **L1046 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1047 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1047 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1048 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1048 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1049 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmadd_w(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1049 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmadd_w(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1050 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmadd_w((v4i32)_1, (v4i32)_2, (v4i32)_3)`.
  **L1050 CN**: 以 `(__m128i)__builtin_lsx_vmadd_w((v4i32)_1, (v4i32)_2, (v4i32)_3)` 从当前函数返回。
- **L1051 EN**: Closes the current lexical scope or compound statement.
  **L1051 CN**: 结束当前词法作用域或复合语句块。
- **L1052 EN**: Blank line separating nearby declarations or logic blocks.
  **L1052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1053 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1053 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1054 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1054 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1055 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmadd_d(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1055 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmadd_d(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1056 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmadd_d((v2i64)_1, (v2i64)_2, (v2i64)_3)`.
  **L1056 CN**: 以 `(__m128i)__builtin_lsx_vmadd_d((v2i64)_1, (v2i64)_2, (v2i64)_3)` 从当前函数返回。

### Lines 1057-1080

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmsub_b(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmsub_b((v16i8)_1, (v16i8)_2, (v16i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmsub_h(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmsub_h((v8i16)_1, (v8i16)_2, (v8i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmsub_w(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmsub_w((v4i32)_1, (v4i32)_2, (v4i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmsub_d(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmsub_d((v2i64)_1, (v2i64)_2, (v2i64)_3);
````
- **L1057 EN**: Closes the current lexical scope or compound statement.
  **L1057 CN**: 结束当前词法作用域或复合语句块。
- **L1058 EN**: Blank line separating nearby declarations or logic blocks.
  **L1058 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1059 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1059 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1060 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1060 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1061 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmsub_b(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1061 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmsub_b(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1062 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmsub_b((v16i8)_1, (v16i8)_2, (v16i8)_3)`.
  **L1062 CN**: 以 `(__m128i)__builtin_lsx_vmsub_b((v16i8)_1, (v16i8)_2, (v16i8)_3)` 从当前函数返回。
- **L1063 EN**: Closes the current lexical scope or compound statement.
  **L1063 CN**: 结束当前词法作用域或复合语句块。
- **L1064 EN**: Blank line separating nearby declarations or logic blocks.
  **L1064 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1065 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1065 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1066 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1066 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1067 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmsub_h(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1067 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmsub_h(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1068 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmsub_h((v8i16)_1, (v8i16)_2, (v8i16)_3)`.
  **L1068 CN**: 以 `(__m128i)__builtin_lsx_vmsub_h((v8i16)_1, (v8i16)_2, (v8i16)_3)` 从当前函数返回。
- **L1069 EN**: Closes the current lexical scope or compound statement.
  **L1069 CN**: 结束当前词法作用域或复合语句块。
- **L1070 EN**: Blank line separating nearby declarations or logic blocks.
  **L1070 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1071 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1071 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1072 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1072 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1073 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmsub_w(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1073 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmsub_w(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1074 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmsub_w((v4i32)_1, (v4i32)_2, (v4i32)_3)`.
  **L1074 CN**: 以 `(__m128i)__builtin_lsx_vmsub_w((v4i32)_1, (v4i32)_2, (v4i32)_3)` 从当前函数返回。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Blank line separating nearby declarations or logic blocks.
  **L1076 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1077 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1077 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1078 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1078 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1079 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmsub_d(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1079 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmsub_d(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1080 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmsub_d((v2i64)_1, (v2i64)_2, (v2i64)_3)`.
  **L1080 CN**: 以 `(__m128i)__builtin_lsx_vmsub_d((v2i64)_1, (v2i64)_2, (v2i64)_3)` 从当前函数返回。

### Lines 1081-1104

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vdiv_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vdiv_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vdiv_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vdiv_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vdiv_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vdiv_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vdiv_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vdiv_d((v2i64)_1, (v2i64)_2);
````
- **L1081 EN**: Closes the current lexical scope or compound statement.
  **L1081 CN**: 结束当前词法作用域或复合语句块。
- **L1082 EN**: Blank line separating nearby declarations or logic blocks.
  **L1082 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1083 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1083 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1084 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1084 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1085 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vdiv_b(__m128i _1, __m128i _2) {`.
  **L1085 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vdiv_b(__m128i _1, __m128i _2) {`。
- **L1086 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vdiv_b((v16i8)_1, (v16i8)_2)`.
  **L1086 CN**: 以 `(__m128i)__builtin_lsx_vdiv_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1087 EN**: Closes the current lexical scope or compound statement.
  **L1087 CN**: 结束当前词法作用域或复合语句块。
- **L1088 EN**: Blank line separating nearby declarations or logic blocks.
  **L1088 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1089 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1089 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1090 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1090 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1091 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vdiv_h(__m128i _1, __m128i _2) {`.
  **L1091 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vdiv_h(__m128i _1, __m128i _2) {`。
- **L1092 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vdiv_h((v8i16)_1, (v8i16)_2)`.
  **L1092 CN**: 以 `(__m128i)__builtin_lsx_vdiv_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L1093 EN**: Closes the current lexical scope or compound statement.
  **L1093 CN**: 结束当前词法作用域或复合语句块。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1095 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1095 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1096 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1096 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1097 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vdiv_w(__m128i _1, __m128i _2) {`.
  **L1097 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vdiv_w(__m128i _1, __m128i _2) {`。
- **L1098 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vdiv_w((v4i32)_1, (v4i32)_2)`.
  **L1098 CN**: 以 `(__m128i)__builtin_lsx_vdiv_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1099 EN**: Closes the current lexical scope or compound statement.
  **L1099 CN**: 结束当前词法作用域或复合语句块。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1101 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1101 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1102 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1102 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1103 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vdiv_d(__m128i _1, __m128i _2) {`.
  **L1103 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vdiv_d(__m128i _1, __m128i _2) {`。
- **L1104 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vdiv_d((v2i64)_1, (v2i64)_2)`.
  **L1104 CN**: 以 `(__m128i)__builtin_lsx_vdiv_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 1105-1128

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vdiv_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vdiv_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vdiv_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vdiv_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vdiv_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vdiv_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vdiv_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vdiv_du((v2u64)_1, (v2u64)_2);
````
- **L1105 EN**: Closes the current lexical scope or compound statement.
  **L1105 CN**: 结束当前词法作用域或复合语句块。
- **L1106 EN**: Blank line separating nearby declarations or logic blocks.
  **L1106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1107 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1107 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1108 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1108 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1109 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vdiv_bu(__m128i _1, __m128i _2) {`.
  **L1109 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vdiv_bu(__m128i _1, __m128i _2) {`。
- **L1110 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vdiv_bu((v16u8)_1, (v16u8)_2)`.
  **L1110 CN**: 以 `(__m128i)__builtin_lsx_vdiv_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L1111 EN**: Closes the current lexical scope or compound statement.
  **L1111 CN**: 结束当前词法作用域或复合语句块。
- **L1112 EN**: Blank line separating nearby declarations or logic blocks.
  **L1112 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1113 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1113 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1114 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1114 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1115 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vdiv_hu(__m128i _1, __m128i _2) {`.
  **L1115 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vdiv_hu(__m128i _1, __m128i _2) {`。
- **L1116 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vdiv_hu((v8u16)_1, (v8u16)_2)`.
  **L1116 CN**: 以 `(__m128i)__builtin_lsx_vdiv_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L1117 EN**: Closes the current lexical scope or compound statement.
  **L1117 CN**: 结束当前词法作用域或复合语句块。
- **L1118 EN**: Blank line separating nearby declarations or logic blocks.
  **L1118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1119 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1119 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1120 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1120 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1121 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vdiv_wu(__m128i _1, __m128i _2) {`.
  **L1121 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vdiv_wu(__m128i _1, __m128i _2) {`。
- **L1122 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vdiv_wu((v4u32)_1, (v4u32)_2)`.
  **L1122 CN**: 以 `(__m128i)__builtin_lsx_vdiv_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L1123 EN**: Closes the current lexical scope or compound statement.
  **L1123 CN**: 结束当前词法作用域或复合语句块。
- **L1124 EN**: Blank line separating nearby declarations or logic blocks.
  **L1124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1125 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1125 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1126 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1126 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1127 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vdiv_du(__m128i _1, __m128i _2) {`.
  **L1127 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vdiv_du(__m128i _1, __m128i _2) {`。
- **L1128 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vdiv_du((v2u64)_1, (v2u64)_2)`.
  **L1128 CN**: 以 `(__m128i)__builtin_lsx_vdiv_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 1129-1152

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhaddw_h_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhaddw_h_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhaddw_w_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhaddw_w_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhaddw_d_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhaddw_d_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhaddw_hu_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhaddw_hu_bu((v16u8)_1, (v16u8)_2);
````
- **L1129 EN**: Closes the current lexical scope or compound statement.
  **L1129 CN**: 结束当前词法作用域或复合语句块。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1131 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1131 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1132 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1132 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1133 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhaddw_h_b(__m128i _1, __m128i _2) {`.
  **L1133 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhaddw_h_b(__m128i _1, __m128i _2) {`。
- **L1134 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhaddw_h_b((v16i8)_1, (v16i8)_2)`.
  **L1134 CN**: 以 `(__m128i)__builtin_lsx_vhaddw_h_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1135 EN**: Closes the current lexical scope or compound statement.
  **L1135 CN**: 结束当前词法作用域或复合语句块。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1137 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1137 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1138 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1138 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1139 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhaddw_w_h(__m128i _1, __m128i _2) {`.
  **L1139 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhaddw_w_h(__m128i _1, __m128i _2) {`。
- **L1140 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhaddw_w_h((v8i16)_1, (v8i16)_2)`.
  **L1140 CN**: 以 `(__m128i)__builtin_lsx_vhaddw_w_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L1141 EN**: Closes the current lexical scope or compound statement.
  **L1141 CN**: 结束当前词法作用域或复合语句块。
- **L1142 EN**: Blank line separating nearby declarations or logic blocks.
  **L1142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1143 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1143 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1144 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1144 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1145 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhaddw_d_w(__m128i _1, __m128i _2) {`.
  **L1145 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhaddw_d_w(__m128i _1, __m128i _2) {`。
- **L1146 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhaddw_d_w((v4i32)_1, (v4i32)_2)`.
  **L1146 CN**: 以 `(__m128i)__builtin_lsx_vhaddw_d_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1147 EN**: Closes the current lexical scope or compound statement.
  **L1147 CN**: 结束当前词法作用域或复合语句块。
- **L1148 EN**: Blank line separating nearby declarations or logic blocks.
  **L1148 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1149 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1149 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1150 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1150 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1151 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhaddw_hu_bu(__m128i _1, __m128i _2) {`.
  **L1151 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhaddw_hu_bu(__m128i _1, __m128i _2) {`。
- **L1152 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhaddw_hu_bu((v16u8)_1, (v16u8)_2)`.
  **L1152 CN**: 以 `(__m128i)__builtin_lsx_vhaddw_hu_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。

### Lines 1153-1176

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhaddw_wu_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhaddw_wu_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhaddw_du_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhaddw_du_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhsubw_h_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhsubw_h_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhsubw_w_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhsubw_w_h((v8i16)_1, (v8i16)_2);
````
- **L1153 EN**: Closes the current lexical scope or compound statement.
  **L1153 CN**: 结束当前词法作用域或复合语句块。
- **L1154 EN**: Blank line separating nearby declarations or logic blocks.
  **L1154 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1155 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1155 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1156 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1156 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1157 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhaddw_wu_hu(__m128i _1, __m128i _2) {`.
  **L1157 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhaddw_wu_hu(__m128i _1, __m128i _2) {`。
- **L1158 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhaddw_wu_hu((v8u16)_1, (v8u16)_2)`.
  **L1158 CN**: 以 `(__m128i)__builtin_lsx_vhaddw_wu_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Blank line separating nearby declarations or logic blocks.
  **L1160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1161 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1161 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1162 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1162 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhaddw_du_wu(__m128i _1, __m128i _2) {`.
  **L1163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhaddw_du_wu(__m128i _1, __m128i _2) {`。
- **L1164 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhaddw_du_wu((v4u32)_1, (v4u32)_2)`.
  **L1164 CN**: 以 `(__m128i)__builtin_lsx_vhaddw_du_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1167 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1167 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1168 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1168 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1169 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhsubw_h_b(__m128i _1, __m128i _2) {`.
  **L1169 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhsubw_h_b(__m128i _1, __m128i _2) {`。
- **L1170 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhsubw_h_b((v16i8)_1, (v16i8)_2)`.
  **L1170 CN**: 以 `(__m128i)__builtin_lsx_vhsubw_h_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1171 EN**: Closes the current lexical scope or compound statement.
  **L1171 CN**: 结束当前词法作用域或复合语句块。
- **L1172 EN**: Blank line separating nearby declarations or logic blocks.
  **L1172 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1173 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1173 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1174 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1174 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1175 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhsubw_w_h(__m128i _1, __m128i _2) {`.
  **L1175 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhsubw_w_h(__m128i _1, __m128i _2) {`。
- **L1176 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhsubw_w_h((v8i16)_1, (v8i16)_2)`.
  **L1176 CN**: 以 `(__m128i)__builtin_lsx_vhsubw_w_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 1177-1200

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhsubw_d_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhsubw_d_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhsubw_hu_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhsubw_hu_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhsubw_wu_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhsubw_wu_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhsubw_du_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhsubw_du_wu((v4u32)_1, (v4u32)_2);
````
- **L1177 EN**: Closes the current lexical scope or compound statement.
  **L1177 CN**: 结束当前词法作用域或复合语句块。
- **L1178 EN**: Blank line separating nearby declarations or logic blocks.
  **L1178 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1179 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1179 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1180 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1180 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1181 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhsubw_d_w(__m128i _1, __m128i _2) {`.
  **L1181 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhsubw_d_w(__m128i _1, __m128i _2) {`。
- **L1182 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhsubw_d_w((v4i32)_1, (v4i32)_2)`.
  **L1182 CN**: 以 `(__m128i)__builtin_lsx_vhsubw_d_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1183 EN**: Closes the current lexical scope or compound statement.
  **L1183 CN**: 结束当前词法作用域或复合语句块。
- **L1184 EN**: Blank line separating nearby declarations or logic blocks.
  **L1184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1185 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1185 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1186 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1186 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1187 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhsubw_hu_bu(__m128i _1, __m128i _2) {`.
  **L1187 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhsubw_hu_bu(__m128i _1, __m128i _2) {`。
- **L1188 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhsubw_hu_bu((v16u8)_1, (v16u8)_2)`.
  **L1188 CN**: 以 `(__m128i)__builtin_lsx_vhsubw_hu_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L1189 EN**: Closes the current lexical scope or compound statement.
  **L1189 CN**: 结束当前词法作用域或复合语句块。
- **L1190 EN**: Blank line separating nearby declarations or logic blocks.
  **L1190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1191 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1191 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1192 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1192 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1193 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhsubw_wu_hu(__m128i _1, __m128i _2) {`.
  **L1193 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhsubw_wu_hu(__m128i _1, __m128i _2) {`。
- **L1194 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhsubw_wu_hu((v8u16)_1, (v8u16)_2)`.
  **L1194 CN**: 以 `(__m128i)__builtin_lsx_vhsubw_wu_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L1195 EN**: Closes the current lexical scope or compound statement.
  **L1195 CN**: 结束当前词法作用域或复合语句块。
- **L1196 EN**: Blank line separating nearby declarations or logic blocks.
  **L1196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1197 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1197 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1198 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1198 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1199 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhsubw_du_wu(__m128i _1, __m128i _2) {`.
  **L1199 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhsubw_du_wu(__m128i _1, __m128i _2) {`。
- **L1200 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhsubw_du_wu((v4u32)_1, (v4u32)_2)`.
  **L1200 CN**: 以 `(__m128i)__builtin_lsx_vhsubw_du_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。

### Lines 1201-1224

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmod_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmod_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmod_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmod_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmod_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmod_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmod_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmod_d((v2i64)_1, (v2i64)_2);
````
- **L1201 EN**: Closes the current lexical scope or compound statement.
  **L1201 CN**: 结束当前词法作用域或复合语句块。
- **L1202 EN**: Blank line separating nearby declarations or logic blocks.
  **L1202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1203 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1203 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1204 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1204 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1205 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmod_b(__m128i _1, __m128i _2) {`.
  **L1205 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmod_b(__m128i _1, __m128i _2) {`。
- **L1206 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmod_b((v16i8)_1, (v16i8)_2)`.
  **L1206 CN**: 以 `(__m128i)__builtin_lsx_vmod_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1209 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1209 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1210 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1210 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1211 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmod_h(__m128i _1, __m128i _2) {`.
  **L1211 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmod_h(__m128i _1, __m128i _2) {`。
- **L1212 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmod_h((v8i16)_1, (v8i16)_2)`.
  **L1212 CN**: 以 `(__m128i)__builtin_lsx_vmod_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L1213 EN**: Closes the current lexical scope or compound statement.
  **L1213 CN**: 结束当前词法作用域或复合语句块。
- **L1214 EN**: Blank line separating nearby declarations or logic blocks.
  **L1214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1215 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1215 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1216 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1216 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1217 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmod_w(__m128i _1, __m128i _2) {`.
  **L1217 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmod_w(__m128i _1, __m128i _2) {`。
- **L1218 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmod_w((v4i32)_1, (v4i32)_2)`.
  **L1218 CN**: 以 `(__m128i)__builtin_lsx_vmod_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1219 EN**: Closes the current lexical scope or compound statement.
  **L1219 CN**: 结束当前词法作用域或复合语句块。
- **L1220 EN**: Blank line separating nearby declarations or logic blocks.
  **L1220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1221 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1221 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1222 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1222 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1223 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmod_d(__m128i _1, __m128i _2) {`.
  **L1223 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmod_d(__m128i _1, __m128i _2) {`。
- **L1224 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmod_d((v2i64)_1, (v2i64)_2)`.
  **L1224 CN**: 以 `(__m128i)__builtin_lsx_vmod_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 1225-1248

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmod_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmod_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmod_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmod_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmod_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmod_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmod_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmod_du((v2u64)_1, (v2u64)_2);
````
- **L1225 EN**: Closes the current lexical scope or compound statement.
  **L1225 CN**: 结束当前词法作用域或复合语句块。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1227 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1227 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1228 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1228 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1229 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmod_bu(__m128i _1, __m128i _2) {`.
  **L1229 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmod_bu(__m128i _1, __m128i _2) {`。
- **L1230 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmod_bu((v16u8)_1, (v16u8)_2)`.
  **L1230 CN**: 以 `(__m128i)__builtin_lsx_vmod_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L1231 EN**: Closes the current lexical scope or compound statement.
  **L1231 CN**: 结束当前词法作用域或复合语句块。
- **L1232 EN**: Blank line separating nearby declarations or logic blocks.
  **L1232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1233 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1233 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1234 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1234 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1235 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmod_hu(__m128i _1, __m128i _2) {`.
  **L1235 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmod_hu(__m128i _1, __m128i _2) {`。
- **L1236 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmod_hu((v8u16)_1, (v8u16)_2)`.
  **L1236 CN**: 以 `(__m128i)__builtin_lsx_vmod_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Blank line separating nearby declarations or logic blocks.
  **L1238 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1239 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1239 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1240 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1240 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1241 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmod_wu(__m128i _1, __m128i _2) {`.
  **L1241 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmod_wu(__m128i _1, __m128i _2) {`。
- **L1242 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmod_wu((v4u32)_1, (v4u32)_2)`.
  **L1242 CN**: 以 `(__m128i)__builtin_lsx_vmod_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L1243 EN**: Closes the current lexical scope or compound statement.
  **L1243 CN**: 结束当前词法作用域或复合语句块。
- **L1244 EN**: Blank line separating nearby declarations or logic blocks.
  **L1244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1245 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1245 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1246 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1246 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1247 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmod_du(__m128i _1, __m128i _2) {`.
  **L1247 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmod_du(__m128i _1, __m128i _2) {`。
- **L1248 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmod_du((v2u64)_1, (v2u64)_2)`.
  **L1248 CN**: 以 `(__m128i)__builtin_lsx_vmod_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 1249-1272

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vreplve_b(__m128i _1, int _2) {
  return (__m128i)__builtin_lsx_vreplve_b((v16i8)_1, (int)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vreplve_h(__m128i _1, int _2) {
  return (__m128i)__builtin_lsx_vreplve_h((v8i16)_1, (int)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vreplve_w(__m128i _1, int _2) {
  return (__m128i)__builtin_lsx_vreplve_w((v4i32)_1, (int)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vreplve_d(__m128i _1, int _2) {
  return (__m128i)__builtin_lsx_vreplve_d((v2i64)_1, (int)_2);
````
- **L1249 EN**: Closes the current lexical scope or compound statement.
  **L1249 CN**: 结束当前词法作用域或复合语句块。
- **L1250 EN**: Blank line separating nearby declarations or logic blocks.
  **L1250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1251 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1251 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1252 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1252 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1253 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vreplve_b(__m128i _1, int _2) {`.
  **L1253 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vreplve_b(__m128i _1, int _2) {`。
- **L1254 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vreplve_b((v16i8)_1, (int)_2)`.
  **L1254 CN**: 以 `(__m128i)__builtin_lsx_vreplve_b((v16i8)_1, (int)_2)` 从当前函数返回。
- **L1255 EN**: Closes the current lexical scope or compound statement.
  **L1255 CN**: 结束当前词法作用域或复合语句块。
- **L1256 EN**: Blank line separating nearby declarations or logic blocks.
  **L1256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1257 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1257 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1258 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1258 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1259 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vreplve_h(__m128i _1, int _2) {`.
  **L1259 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vreplve_h(__m128i _1, int _2) {`。
- **L1260 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vreplve_h((v8i16)_1, (int)_2)`.
  **L1260 CN**: 以 `(__m128i)__builtin_lsx_vreplve_h((v8i16)_1, (int)_2)` 从当前函数返回。
- **L1261 EN**: Closes the current lexical scope or compound statement.
  **L1261 CN**: 结束当前词法作用域或复合语句块。
- **L1262 EN**: Blank line separating nearby declarations or logic blocks.
  **L1262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1263 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1263 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1264 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1264 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1265 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vreplve_w(__m128i _1, int _2) {`.
  **L1265 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vreplve_w(__m128i _1, int _2) {`。
- **L1266 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vreplve_w((v4i32)_1, (int)_2)`.
  **L1266 CN**: 以 `(__m128i)__builtin_lsx_vreplve_w((v4i32)_1, (int)_2)` 从当前函数返回。
- **L1267 EN**: Closes the current lexical scope or compound statement.
  **L1267 CN**: 结束当前词法作用域或复合语句块。
- **L1268 EN**: Blank line separating nearby declarations or logic blocks.
  **L1268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1269 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1269 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1270 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1270 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1271 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vreplve_d(__m128i _1, int _2) {`.
  **L1271 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vreplve_d(__m128i _1, int _2) {`。
- **L1272 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vreplve_d((v2i64)_1, (int)_2)`.
  **L1272 CN**: 以 `(__m128i)__builtin_lsx_vreplve_d((v2i64)_1, (int)_2)` 从当前函数返回。

### Lines 1273-1296

````c
}

#define __lsx_vreplvei_b(/*__m128i*/ _1, /*ui4*/ _2)                           \
  ((__m128i)__builtin_lsx_vreplvei_b((v16i8)(_1), (_2)))

#define __lsx_vreplvei_h(/*__m128i*/ _1, /*ui3*/ _2)                           \
  ((__m128i)__builtin_lsx_vreplvei_h((v8i16)(_1), (_2)))

#define __lsx_vreplvei_w(/*__m128i*/ _1, /*ui2*/ _2)                           \
  ((__m128i)__builtin_lsx_vreplvei_w((v4i32)(_1), (_2)))

#define __lsx_vreplvei_d(/*__m128i*/ _1, /*ui1*/ _2)                           \
  ((__m128i)__builtin_lsx_vreplvei_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpickev_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpickev_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpickev_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpickev_h((v8i16)_1, (v8i16)_2);
````
- **L1273 EN**: Closes the current lexical scope or compound statement.
  **L1273 CN**: 结束当前词法作用域或复合语句块。
- **L1274 EN**: Blank line separating nearby declarations or logic blocks.
  **L1274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1275 EN**: Defines macro `__lsx_vreplvei_b(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1275 CN**: 定义宏 `__lsx_vreplvei_b(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L1276 EN**: Continues logic associated with callable symbol `__builtin_lsx_vreplvei_b`.
  **L1276 CN**: 继续与可调用符号 `__builtin_lsx_vreplvei_b` 相关的逻辑。
- **L1277 EN**: Blank line separating nearby declarations or logic blocks.
  **L1277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1278 EN**: Defines macro `__lsx_vreplvei_h(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1278 CN**: 定义宏 `__lsx_vreplvei_h(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L1279 EN**: Continues logic associated with callable symbol `__builtin_lsx_vreplvei_h`.
  **L1279 CN**: 继续与可调用符号 `__builtin_lsx_vreplvei_h` 相关的逻辑。
- **L1280 EN**: Blank line separating nearby declarations or logic blocks.
  **L1280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1281 EN**: Defines macro `__lsx_vreplvei_w(/*__m128i*/ _1, /*ui2*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1281 CN**: 定义宏 `__lsx_vreplvei_w(/*__m128i*/ _1, /*ui2*/ _2)`，用于条件编译、简写或 API 生成。
- **L1282 EN**: Continues logic associated with callable symbol `__builtin_lsx_vreplvei_w`.
  **L1282 CN**: 继续与可调用符号 `__builtin_lsx_vreplvei_w` 相关的逻辑。
- **L1283 EN**: Blank line separating nearby declarations or logic blocks.
  **L1283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1284 EN**: Defines macro `__lsx_vreplvei_d(/*__m128i*/ _1, /*ui1*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1284 CN**: 定义宏 `__lsx_vreplvei_d(/*__m128i*/ _1, /*ui1*/ _2)`，用于条件编译、简写或 API 生成。
- **L1285 EN**: Continues logic associated with callable symbol `__builtin_lsx_vreplvei_d`.
  **L1285 CN**: 继续与可调用符号 `__builtin_lsx_vreplvei_d` 相关的逻辑。
- **L1286 EN**: Blank line separating nearby declarations or logic blocks.
  **L1286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1287 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1287 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1288 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1288 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1289 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpickev_b(__m128i _1, __m128i _2) {`.
  **L1289 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpickev_b(__m128i _1, __m128i _2) {`。
- **L1290 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpickev_b((v16i8)_1, (v16i8)_2)`.
  **L1290 CN**: 以 `(__m128i)__builtin_lsx_vpickev_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1291 EN**: Closes the current lexical scope or compound statement.
  **L1291 CN**: 结束当前词法作用域或复合语句块。
- **L1292 EN**: Blank line separating nearby declarations or logic blocks.
  **L1292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1293 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1293 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1294 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1294 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1295 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpickev_h(__m128i _1, __m128i _2) {`.
  **L1295 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpickev_h(__m128i _1, __m128i _2) {`。
- **L1296 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpickev_h((v8i16)_1, (v8i16)_2)`.
  **L1296 CN**: 以 `(__m128i)__builtin_lsx_vpickev_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 1297-1320

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpickev_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpickev_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpickev_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpickev_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpickod_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpickod_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpickod_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpickod_h((v8i16)_1, (v8i16)_2);
````
- **L1297 EN**: Closes the current lexical scope or compound statement.
  **L1297 CN**: 结束当前词法作用域或复合语句块。
- **L1298 EN**: Blank line separating nearby declarations or logic blocks.
  **L1298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1299 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1299 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1300 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1300 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1301 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpickev_w(__m128i _1, __m128i _2) {`.
  **L1301 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpickev_w(__m128i _1, __m128i _2) {`。
- **L1302 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpickev_w((v4i32)_1, (v4i32)_2)`.
  **L1302 CN**: 以 `(__m128i)__builtin_lsx_vpickev_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1303 EN**: Closes the current lexical scope or compound statement.
  **L1303 CN**: 结束当前词法作用域或复合语句块。
- **L1304 EN**: Blank line separating nearby declarations or logic blocks.
  **L1304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1305 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1305 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1306 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1306 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1307 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpickev_d(__m128i _1, __m128i _2) {`.
  **L1307 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpickev_d(__m128i _1, __m128i _2) {`。
- **L1308 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpickev_d((v2i64)_1, (v2i64)_2)`.
  **L1308 CN**: 以 `(__m128i)__builtin_lsx_vpickev_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L1309 EN**: Closes the current lexical scope or compound statement.
  **L1309 CN**: 结束当前词法作用域或复合语句块。
- **L1310 EN**: Blank line separating nearby declarations or logic blocks.
  **L1310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1311 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1311 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1312 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1312 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1313 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpickod_b(__m128i _1, __m128i _2) {`.
  **L1313 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpickod_b(__m128i _1, __m128i _2) {`。
- **L1314 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpickod_b((v16i8)_1, (v16i8)_2)`.
  **L1314 CN**: 以 `(__m128i)__builtin_lsx_vpickod_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1315 EN**: Closes the current lexical scope or compound statement.
  **L1315 CN**: 结束当前词法作用域或复合语句块。
- **L1316 EN**: Blank line separating nearby declarations or logic blocks.
  **L1316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1317 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1317 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1318 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1318 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1319 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpickod_h(__m128i _1, __m128i _2) {`.
  **L1319 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpickod_h(__m128i _1, __m128i _2) {`。
- **L1320 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpickod_h((v8i16)_1, (v8i16)_2)`.
  **L1320 CN**: 以 `(__m128i)__builtin_lsx_vpickod_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 1321-1344

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpickod_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpickod_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpickod_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpickod_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vilvh_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vilvh_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vilvh_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vilvh_h((v8i16)_1, (v8i16)_2);
````
- **L1321 EN**: Closes the current lexical scope or compound statement.
  **L1321 CN**: 结束当前词法作用域或复合语句块。
- **L1322 EN**: Blank line separating nearby declarations or logic blocks.
  **L1322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1323 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1323 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1324 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1324 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1325 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpickod_w(__m128i _1, __m128i _2) {`.
  **L1325 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpickod_w(__m128i _1, __m128i _2) {`。
- **L1326 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpickod_w((v4i32)_1, (v4i32)_2)`.
  **L1326 CN**: 以 `(__m128i)__builtin_lsx_vpickod_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1327 EN**: Closes the current lexical scope or compound statement.
  **L1327 CN**: 结束当前词法作用域或复合语句块。
- **L1328 EN**: Blank line separating nearby declarations or logic blocks.
  **L1328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1329 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1329 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1330 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1330 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1331 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpickod_d(__m128i _1, __m128i _2) {`.
  **L1331 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpickod_d(__m128i _1, __m128i _2) {`。
- **L1332 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpickod_d((v2i64)_1, (v2i64)_2)`.
  **L1332 CN**: 以 `(__m128i)__builtin_lsx_vpickod_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L1333 EN**: Closes the current lexical scope or compound statement.
  **L1333 CN**: 结束当前词法作用域或复合语句块。
- **L1334 EN**: Blank line separating nearby declarations or logic blocks.
  **L1334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1335 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1335 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1336 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1336 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1337 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vilvh_b(__m128i _1, __m128i _2) {`.
  **L1337 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vilvh_b(__m128i _1, __m128i _2) {`。
- **L1338 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vilvh_b((v16i8)_1, (v16i8)_2)`.
  **L1338 CN**: 以 `(__m128i)__builtin_lsx_vilvh_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1339 EN**: Closes the current lexical scope or compound statement.
  **L1339 CN**: 结束当前词法作用域或复合语句块。
- **L1340 EN**: Blank line separating nearby declarations or logic blocks.
  **L1340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1341 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1341 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1342 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1342 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1343 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vilvh_h(__m128i _1, __m128i _2) {`.
  **L1343 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vilvh_h(__m128i _1, __m128i _2) {`。
- **L1344 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vilvh_h((v8i16)_1, (v8i16)_2)`.
  **L1344 CN**: 以 `(__m128i)__builtin_lsx_vilvh_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 1345-1368

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vilvh_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vilvh_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vilvh_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vilvh_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vilvl_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vilvl_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vilvl_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vilvl_h((v8i16)_1, (v8i16)_2);
````
- **L1345 EN**: Closes the current lexical scope or compound statement.
  **L1345 CN**: 结束当前词法作用域或复合语句块。
- **L1346 EN**: Blank line separating nearby declarations or logic blocks.
  **L1346 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1347 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1347 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1348 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1348 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1349 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vilvh_w(__m128i _1, __m128i _2) {`.
  **L1349 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vilvh_w(__m128i _1, __m128i _2) {`。
- **L1350 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vilvh_w((v4i32)_1, (v4i32)_2)`.
  **L1350 CN**: 以 `(__m128i)__builtin_lsx_vilvh_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1351 EN**: Closes the current lexical scope or compound statement.
  **L1351 CN**: 结束当前词法作用域或复合语句块。
- **L1352 EN**: Blank line separating nearby declarations or logic blocks.
  **L1352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1353 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1353 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1354 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1354 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1355 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vilvh_d(__m128i _1, __m128i _2) {`.
  **L1355 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vilvh_d(__m128i _1, __m128i _2) {`。
- **L1356 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vilvh_d((v2i64)_1, (v2i64)_2)`.
  **L1356 CN**: 以 `(__m128i)__builtin_lsx_vilvh_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L1357 EN**: Closes the current lexical scope or compound statement.
  **L1357 CN**: 结束当前词法作用域或复合语句块。
- **L1358 EN**: Blank line separating nearby declarations or logic blocks.
  **L1358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1359 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1359 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1360 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1360 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1361 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vilvl_b(__m128i _1, __m128i _2) {`.
  **L1361 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vilvl_b(__m128i _1, __m128i _2) {`。
- **L1362 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vilvl_b((v16i8)_1, (v16i8)_2)`.
  **L1362 CN**: 以 `(__m128i)__builtin_lsx_vilvl_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1363 EN**: Closes the current lexical scope or compound statement.
  **L1363 CN**: 结束当前词法作用域或复合语句块。
- **L1364 EN**: Blank line separating nearby declarations or logic blocks.
  **L1364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1365 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1365 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1366 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1366 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1367 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vilvl_h(__m128i _1, __m128i _2) {`.
  **L1367 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vilvl_h(__m128i _1, __m128i _2) {`。
- **L1368 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vilvl_h((v8i16)_1, (v8i16)_2)`.
  **L1368 CN**: 以 `(__m128i)__builtin_lsx_vilvl_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 1369-1392

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vilvl_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vilvl_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vilvl_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vilvl_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpackev_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpackev_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpackev_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpackev_h((v8i16)_1, (v8i16)_2);
````
- **L1369 EN**: Closes the current lexical scope or compound statement.
  **L1369 CN**: 结束当前词法作用域或复合语句块。
- **L1370 EN**: Blank line separating nearby declarations or logic blocks.
  **L1370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1371 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1371 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1372 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1372 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1373 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vilvl_w(__m128i _1, __m128i _2) {`.
  **L1373 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vilvl_w(__m128i _1, __m128i _2) {`。
- **L1374 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vilvl_w((v4i32)_1, (v4i32)_2)`.
  **L1374 CN**: 以 `(__m128i)__builtin_lsx_vilvl_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1375 EN**: Closes the current lexical scope or compound statement.
  **L1375 CN**: 结束当前词法作用域或复合语句块。
- **L1376 EN**: Blank line separating nearby declarations or logic blocks.
  **L1376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1377 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1377 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1378 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1378 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vilvl_d(__m128i _1, __m128i _2) {`.
  **L1379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vilvl_d(__m128i _1, __m128i _2) {`。
- **L1380 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vilvl_d((v2i64)_1, (v2i64)_2)`.
  **L1380 CN**: 以 `(__m128i)__builtin_lsx_vilvl_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L1381 EN**: Closes the current lexical scope or compound statement.
  **L1381 CN**: 结束当前词法作用域或复合语句块。
- **L1382 EN**: Blank line separating nearby declarations or logic blocks.
  **L1382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1383 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1383 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1384 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1384 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpackev_b(__m128i _1, __m128i _2) {`.
  **L1385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpackev_b(__m128i _1, __m128i _2) {`。
- **L1386 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpackev_b((v16i8)_1, (v16i8)_2)`.
  **L1386 CN**: 以 `(__m128i)__builtin_lsx_vpackev_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1387 EN**: Closes the current lexical scope or compound statement.
  **L1387 CN**: 结束当前词法作用域或复合语句块。
- **L1388 EN**: Blank line separating nearby declarations or logic blocks.
  **L1388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1389 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1389 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1390 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1390 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpackev_h(__m128i _1, __m128i _2) {`.
  **L1391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpackev_h(__m128i _1, __m128i _2) {`。
- **L1392 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpackev_h((v8i16)_1, (v8i16)_2)`.
  **L1392 CN**: 以 `(__m128i)__builtin_lsx_vpackev_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 1393-1416

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpackev_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpackev_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpackev_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpackev_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpackod_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpackod_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpackod_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpackod_h((v8i16)_1, (v8i16)_2);
````
- **L1393 EN**: Closes the current lexical scope or compound statement.
  **L1393 CN**: 结束当前词法作用域或复合语句块。
- **L1394 EN**: Blank line separating nearby declarations or logic blocks.
  **L1394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1395 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1395 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1396 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1396 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1397 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpackev_w(__m128i _1, __m128i _2) {`.
  **L1397 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpackev_w(__m128i _1, __m128i _2) {`。
- **L1398 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpackev_w((v4i32)_1, (v4i32)_2)`.
  **L1398 CN**: 以 `(__m128i)__builtin_lsx_vpackev_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1399 EN**: Closes the current lexical scope or compound statement.
  **L1399 CN**: 结束当前词法作用域或复合语句块。
- **L1400 EN**: Blank line separating nearby declarations or logic blocks.
  **L1400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1401 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1401 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1402 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1402 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1403 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpackev_d(__m128i _1, __m128i _2) {`.
  **L1403 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpackev_d(__m128i _1, __m128i _2) {`。
- **L1404 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpackev_d((v2i64)_1, (v2i64)_2)`.
  **L1404 CN**: 以 `(__m128i)__builtin_lsx_vpackev_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L1405 EN**: Closes the current lexical scope or compound statement.
  **L1405 CN**: 结束当前词法作用域或复合语句块。
- **L1406 EN**: Blank line separating nearby declarations or logic blocks.
  **L1406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1407 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1407 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1408 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1408 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1409 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpackod_b(__m128i _1, __m128i _2) {`.
  **L1409 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpackod_b(__m128i _1, __m128i _2) {`。
- **L1410 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpackod_b((v16i8)_1, (v16i8)_2)`.
  **L1410 CN**: 以 `(__m128i)__builtin_lsx_vpackod_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L1411 EN**: Closes the current lexical scope or compound statement.
  **L1411 CN**: 结束当前词法作用域或复合语句块。
- **L1412 EN**: Blank line separating nearby declarations or logic blocks.
  **L1412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1413 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1413 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1414 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1414 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1415 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpackod_h(__m128i _1, __m128i _2) {`.
  **L1415 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpackod_h(__m128i _1, __m128i _2) {`。
- **L1416 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpackod_h((v8i16)_1, (v8i16)_2)`.
  **L1416 CN**: 以 `(__m128i)__builtin_lsx_vpackod_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 1417-1440

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpackod_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpackod_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpackod_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vpackod_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vshuf_h(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vshuf_h((v8i16)_1, (v8i16)_2, (v8i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vshuf_w(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vshuf_w((v4i32)_1, (v4i32)_2, (v4i32)_3);
````
- **L1417 EN**: Closes the current lexical scope or compound statement.
  **L1417 CN**: 结束当前词法作用域或复合语句块。
- **L1418 EN**: Blank line separating nearby declarations or logic blocks.
  **L1418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1419 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1419 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1420 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1420 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1421 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpackod_w(__m128i _1, __m128i _2) {`.
  **L1421 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpackod_w(__m128i _1, __m128i _2) {`。
- **L1422 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpackod_w((v4i32)_1, (v4i32)_2)`.
  **L1422 CN**: 以 `(__m128i)__builtin_lsx_vpackod_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1423 EN**: Closes the current lexical scope or compound statement.
  **L1423 CN**: 结束当前词法作用域或复合语句块。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1425 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1425 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1426 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1426 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1427 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpackod_d(__m128i _1, __m128i _2) {`.
  **L1427 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpackod_d(__m128i _1, __m128i _2) {`。
- **L1428 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpackod_d((v2i64)_1, (v2i64)_2)`.
  **L1428 CN**: 以 `(__m128i)__builtin_lsx_vpackod_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L1429 EN**: Closes the current lexical scope or compound statement.
  **L1429 CN**: 结束当前词法作用域或复合语句块。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1431 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1431 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1432 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1432 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1433 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vshuf_h(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1433 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vshuf_h(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1434 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vshuf_h((v8i16)_1, (v8i16)_2, (v8i16)_3)`.
  **L1434 CN**: 以 `(__m128i)__builtin_lsx_vshuf_h((v8i16)_1, (v8i16)_2, (v8i16)_3)` 从当前函数返回。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Blank line separating nearby declarations or logic blocks.
  **L1436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1437 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1437 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1438 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1438 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1439 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vshuf_w(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1439 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vshuf_w(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1440 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vshuf_w((v4i32)_1, (v4i32)_2, (v4i32)_3)`.
  **L1440 CN**: 以 `(__m128i)__builtin_lsx_vshuf_w((v4i32)_1, (v4i32)_2, (v4i32)_3)` 从当前函数返回。

### Lines 1441-1464

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vshuf_d(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vshuf_d((v2i64)_1, (v2i64)_2, (v2i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vand_v(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vand_v((v16u8)_1, (v16u8)_2);
}

#define __lsx_vandi_b(/*__m128i*/ _1, /*ui8*/ _2)                              \
  ((__m128i)__builtin_lsx_vandi_b((v16u8)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vor_v(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vor_v((v16u8)_1, (v16u8)_2);
}

#define __lsx_vori_b(/*__m128i*/ _1, /*ui8*/ _2)                               \
````
- **L1441 EN**: Closes the current lexical scope or compound statement.
  **L1441 CN**: 结束当前词法作用域或复合语句块。
- **L1442 EN**: Blank line separating nearby declarations or logic blocks.
  **L1442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1443 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1443 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1444 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1444 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1445 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vshuf_d(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1445 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vshuf_d(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1446 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vshuf_d((v2i64)_1, (v2i64)_2, (v2i64)_3)`.
  **L1446 CN**: 以 `(__m128i)__builtin_lsx_vshuf_d((v2i64)_1, (v2i64)_2, (v2i64)_3)` 从当前函数返回。
- **L1447 EN**: Closes the current lexical scope or compound statement.
  **L1447 CN**: 结束当前词法作用域或复合语句块。
- **L1448 EN**: Blank line separating nearby declarations or logic blocks.
  **L1448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1449 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1449 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1450 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1450 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1451 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vand_v(__m128i _1, __m128i _2) {`.
  **L1451 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vand_v(__m128i _1, __m128i _2) {`。
- **L1452 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vand_v((v16u8)_1, (v16u8)_2)`.
  **L1452 CN**: 以 `(__m128i)__builtin_lsx_vand_v((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L1453 EN**: Closes the current lexical scope or compound statement.
  **L1453 CN**: 结束当前词法作用域或复合语句块。
- **L1454 EN**: Blank line separating nearby declarations or logic blocks.
  **L1454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1455 EN**: Defines macro `__lsx_vandi_b(/*__m128i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1455 CN**: 定义宏 `__lsx_vandi_b(/*__m128i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1456 EN**: Continues logic associated with callable symbol `__builtin_lsx_vandi_b`.
  **L1456 CN**: 继续与可调用符号 `__builtin_lsx_vandi_b` 相关的逻辑。
- **L1457 EN**: Blank line separating nearby declarations or logic blocks.
  **L1457 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1458 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1458 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1459 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1459 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1460 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vor_v(__m128i _1, __m128i _2) {`.
  **L1460 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vor_v(__m128i _1, __m128i _2) {`。
- **L1461 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vor_v((v16u8)_1, (v16u8)_2)`.
  **L1461 CN**: 以 `(__m128i)__builtin_lsx_vor_v((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L1462 EN**: Closes the current lexical scope or compound statement.
  **L1462 CN**: 结束当前词法作用域或复合语句块。
- **L1463 EN**: Blank line separating nearby declarations or logic blocks.
  **L1463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1464 EN**: Defines macro `__lsx_vori_b(/*__m128i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1464 CN**: 定义宏 `__lsx_vori_b(/*__m128i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 1465-1488

````c
  ((__m128i)__builtin_lsx_vori_b((v16u8)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vnor_v(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vnor_v((v16u8)_1, (v16u8)_2);
}

#define __lsx_vnori_b(/*__m128i*/ _1, /*ui8*/ _2)                              \
  ((__m128i)__builtin_lsx_vnori_b((v16u8)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vxor_v(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vxor_v((v16u8)_1, (v16u8)_2);
}

#define __lsx_vxori_b(/*__m128i*/ _1, /*ui8*/ _2)                              \
  ((__m128i)__builtin_lsx_vxori_b((v16u8)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vbitsel_v(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vbitsel_v((v16u8)_1, (v16u8)_2, (v16u8)_3);
````
- **L1465 EN**: Continues logic associated with callable symbol `__builtin_lsx_vori_b`.
  **L1465 CN**: 继续与可调用符号 `__builtin_lsx_vori_b` 相关的逻辑。
- **L1466 EN**: Blank line separating nearby declarations or logic blocks.
  **L1466 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1467 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1467 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1468 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1468 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1469 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vnor_v(__m128i _1, __m128i _2) {`.
  **L1469 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vnor_v(__m128i _1, __m128i _2) {`。
- **L1470 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vnor_v((v16u8)_1, (v16u8)_2)`.
  **L1470 CN**: 以 `(__m128i)__builtin_lsx_vnor_v((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L1471 EN**: Closes the current lexical scope or compound statement.
  **L1471 CN**: 结束当前词法作用域或复合语句块。
- **L1472 EN**: Blank line separating nearby declarations or logic blocks.
  **L1472 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1473 EN**: Defines macro `__lsx_vnori_b(/*__m128i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1473 CN**: 定义宏 `__lsx_vnori_b(/*__m128i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1474 EN**: Continues logic associated with callable symbol `__builtin_lsx_vnori_b`.
  **L1474 CN**: 继续与可调用符号 `__builtin_lsx_vnori_b` 相关的逻辑。
- **L1475 EN**: Blank line separating nearby declarations or logic blocks.
  **L1475 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1476 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1476 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1477 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1477 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1478 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vxor_v(__m128i _1, __m128i _2) {`.
  **L1478 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vxor_v(__m128i _1, __m128i _2) {`。
- **L1479 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vxor_v((v16u8)_1, (v16u8)_2)`.
  **L1479 CN**: 以 `(__m128i)__builtin_lsx_vxor_v((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L1480 EN**: Closes the current lexical scope or compound statement.
  **L1480 CN**: 结束当前词法作用域或复合语句块。
- **L1481 EN**: Blank line separating nearby declarations or logic blocks.
  **L1481 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1482 EN**: Defines macro `__lsx_vxori_b(/*__m128i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1482 CN**: 定义宏 `__lsx_vxori_b(/*__m128i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1483 EN**: Continues logic associated with callable symbol `__builtin_lsx_vxori_b`.
  **L1483 CN**: 继续与可调用符号 `__builtin_lsx_vxori_b` 相关的逻辑。
- **L1484 EN**: Blank line separating nearby declarations or logic blocks.
  **L1484 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1485 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1485 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1486 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1486 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1487 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vbitsel_v(__m128i _1, __m128i _2, __m128i _3) {`.
  **L1487 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vbitsel_v(__m128i _1, __m128i _2, __m128i _3) {`。
- **L1488 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vbitsel_v((v16u8)_1, (v16u8)_2, (v16u8)_3)`.
  **L1488 CN**: 以 `(__m128i)__builtin_lsx_vbitsel_v((v16u8)_1, (v16u8)_2, (v16u8)_3)` 从当前函数返回。

### Lines 1489-1512

````c
}

#define __lsx_vbitseli_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)           \
  ((__m128i)__builtin_lsx_vbitseli_b((v16u8)(_1), (v16u8)(_2), (_3)))

#define __lsx_vshuf4i_b(/*__m128i*/ _1, /*ui8*/ _2)                            \
  ((__m128i)__builtin_lsx_vshuf4i_b((v16i8)(_1), (_2)))

#define __lsx_vshuf4i_h(/*__m128i*/ _1, /*ui8*/ _2)                            \
  ((__m128i)__builtin_lsx_vshuf4i_h((v8i16)(_1), (_2)))

#define __lsx_vshuf4i_w(/*__m128i*/ _1, /*ui8*/ _2)                            \
  ((__m128i)__builtin_lsx_vshuf4i_w((v4i32)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vreplgr2vr_b(int _1) {
  return (__m128i)__builtin_lsx_vreplgr2vr_b((int)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vreplgr2vr_h(int _1) {
  return (__m128i)__builtin_lsx_vreplgr2vr_h((int)_1);
````
- **L1489 EN**: Closes the current lexical scope or compound statement.
  **L1489 CN**: 结束当前词法作用域或复合语句块。
- **L1490 EN**: Blank line separating nearby declarations or logic blocks.
  **L1490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1491 EN**: Defines macro `__lsx_vbitseli_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L1491 CN**: 定义宏 `__lsx_vbitseli_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L1492 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbitseli_b`.
  **L1492 CN**: 继续与可调用符号 `__builtin_lsx_vbitseli_b` 相关的逻辑。
- **L1493 EN**: Blank line separating nearby declarations or logic blocks.
  **L1493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1494 EN**: Defines macro `__lsx_vshuf4i_b(/*__m128i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1494 CN**: 定义宏 `__lsx_vshuf4i_b(/*__m128i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1495 EN**: Continues logic associated with callable symbol `__builtin_lsx_vshuf4i_b`.
  **L1495 CN**: 继续与可调用符号 `__builtin_lsx_vshuf4i_b` 相关的逻辑。
- **L1496 EN**: Blank line separating nearby declarations or logic blocks.
  **L1496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1497 EN**: Defines macro `__lsx_vshuf4i_h(/*__m128i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1497 CN**: 定义宏 `__lsx_vshuf4i_h(/*__m128i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1498 EN**: Continues logic associated with callable symbol `__builtin_lsx_vshuf4i_h`.
  **L1498 CN**: 继续与可调用符号 `__builtin_lsx_vshuf4i_h` 相关的逻辑。
- **L1499 EN**: Blank line separating nearby declarations or logic blocks.
  **L1499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1500 EN**: Defines macro `__lsx_vshuf4i_w(/*__m128i*/ _1, /*ui8*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1500 CN**: 定义宏 `__lsx_vshuf4i_w(/*__m128i*/ _1, /*ui8*/ _2)`，用于条件编译、简写或 API 生成。
- **L1501 EN**: Continues logic associated with callable symbol `__builtin_lsx_vshuf4i_w`.
  **L1501 CN**: 继续与可调用符号 `__builtin_lsx_vshuf4i_w` 相关的逻辑。
- **L1502 EN**: Blank line separating nearby declarations or logic blocks.
  **L1502 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1503 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1503 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1504 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1504 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1505 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vreplgr2vr_b(int _1) {`.
  **L1505 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vreplgr2vr_b(int _1) {`。
- **L1506 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vreplgr2vr_b((int)_1)`.
  **L1506 CN**: 以 `(__m128i)__builtin_lsx_vreplgr2vr_b((int)_1)` 从当前函数返回。
- **L1507 EN**: Closes the current lexical scope or compound statement.
  **L1507 CN**: 结束当前词法作用域或复合语句块。
- **L1508 EN**: Blank line separating nearby declarations or logic blocks.
  **L1508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1509 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1509 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1510 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1510 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1511 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vreplgr2vr_h(int _1) {`.
  **L1511 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vreplgr2vr_h(int _1) {`。
- **L1512 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vreplgr2vr_h((int)_1)`.
  **L1512 CN**: 以 `(__m128i)__builtin_lsx_vreplgr2vr_h((int)_1)` 从当前函数返回。

### Lines 1513-1536

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vreplgr2vr_w(int _1) {
  return (__m128i)__builtin_lsx_vreplgr2vr_w((int)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vreplgr2vr_d(long int _1) {
  return (__m128i)__builtin_lsx_vreplgr2vr_d((long int)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpcnt_b(__m128i _1) {
  return (__m128i)__builtin_lsx_vpcnt_b((v16i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpcnt_h(__m128i _1) {
  return (__m128i)__builtin_lsx_vpcnt_h((v8i16)_1);
````
- **L1513 EN**: Closes the current lexical scope or compound statement.
  **L1513 CN**: 结束当前词法作用域或复合语句块。
- **L1514 EN**: Blank line separating nearby declarations or logic blocks.
  **L1514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1515 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1515 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1516 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1516 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1517 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vreplgr2vr_w(int _1) {`.
  **L1517 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vreplgr2vr_w(int _1) {`。
- **L1518 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vreplgr2vr_w((int)_1)`.
  **L1518 CN**: 以 `(__m128i)__builtin_lsx_vreplgr2vr_w((int)_1)` 从当前函数返回。
- **L1519 EN**: Closes the current lexical scope or compound statement.
  **L1519 CN**: 结束当前词法作用域或复合语句块。
- **L1520 EN**: Blank line separating nearby declarations or logic blocks.
  **L1520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1521 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1521 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1522 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1522 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1523 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vreplgr2vr_d(long int _1) {`.
  **L1523 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vreplgr2vr_d(long int _1) {`。
- **L1524 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vreplgr2vr_d((long int)_1)`.
  **L1524 CN**: 以 `(__m128i)__builtin_lsx_vreplgr2vr_d((long int)_1)` 从当前函数返回。
- **L1525 EN**: Closes the current lexical scope or compound statement.
  **L1525 CN**: 结束当前词法作用域或复合语句块。
- **L1526 EN**: Blank line separating nearby declarations or logic blocks.
  **L1526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1527 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1527 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1528 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1528 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1529 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpcnt_b(__m128i _1) {`.
  **L1529 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpcnt_b(__m128i _1) {`。
- **L1530 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpcnt_b((v16i8)_1)`.
  **L1530 CN**: 以 `(__m128i)__builtin_lsx_vpcnt_b((v16i8)_1)` 从当前函数返回。
- **L1531 EN**: Closes the current lexical scope or compound statement.
  **L1531 CN**: 结束当前词法作用域或复合语句块。
- **L1532 EN**: Blank line separating nearby declarations or logic blocks.
  **L1532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1533 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1533 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1534 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1534 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1535 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpcnt_h(__m128i _1) {`.
  **L1535 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpcnt_h(__m128i _1) {`。
- **L1536 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpcnt_h((v8i16)_1)`.
  **L1536 CN**: 以 `(__m128i)__builtin_lsx_vpcnt_h((v8i16)_1)` 从当前函数返回。

### Lines 1537-1560

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpcnt_w(__m128i _1) {
  return (__m128i)__builtin_lsx_vpcnt_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vpcnt_d(__m128i _1) {
  return (__m128i)__builtin_lsx_vpcnt_d((v2i64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vclo_b(__m128i _1) {
  return (__m128i)__builtin_lsx_vclo_b((v16i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vclo_h(__m128i _1) {
  return (__m128i)__builtin_lsx_vclo_h((v8i16)_1);
````
- **L1537 EN**: Closes the current lexical scope or compound statement.
  **L1537 CN**: 结束当前词法作用域或复合语句块。
- **L1538 EN**: Blank line separating nearby declarations or logic blocks.
  **L1538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1539 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1539 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1540 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1540 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1541 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpcnt_w(__m128i _1) {`.
  **L1541 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpcnt_w(__m128i _1) {`。
- **L1542 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpcnt_w((v4i32)_1)`.
  **L1542 CN**: 以 `(__m128i)__builtin_lsx_vpcnt_w((v4i32)_1)` 从当前函数返回。
- **L1543 EN**: Closes the current lexical scope or compound statement.
  **L1543 CN**: 结束当前词法作用域或复合语句块。
- **L1544 EN**: Blank line separating nearby declarations or logic blocks.
  **L1544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1545 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1545 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1546 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1546 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1547 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vpcnt_d(__m128i _1) {`.
  **L1547 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vpcnt_d(__m128i _1) {`。
- **L1548 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vpcnt_d((v2i64)_1)`.
  **L1548 CN**: 以 `(__m128i)__builtin_lsx_vpcnt_d((v2i64)_1)` 从当前函数返回。
- **L1549 EN**: Closes the current lexical scope or compound statement.
  **L1549 CN**: 结束当前词法作用域或复合语句块。
- **L1550 EN**: Blank line separating nearby declarations or logic blocks.
  **L1550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1551 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1551 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1552 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1552 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1553 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vclo_b(__m128i _1) {`.
  **L1553 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vclo_b(__m128i _1) {`。
- **L1554 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vclo_b((v16i8)_1)`.
  **L1554 CN**: 以 `(__m128i)__builtin_lsx_vclo_b((v16i8)_1)` 从当前函数返回。
- **L1555 EN**: Closes the current lexical scope or compound statement.
  **L1555 CN**: 结束当前词法作用域或复合语句块。
- **L1556 EN**: Blank line separating nearby declarations or logic blocks.
  **L1556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1557 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1557 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1558 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1558 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1559 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vclo_h(__m128i _1) {`.
  **L1559 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vclo_h(__m128i _1) {`。
- **L1560 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vclo_h((v8i16)_1)`.
  **L1560 CN**: 以 `(__m128i)__builtin_lsx_vclo_h((v8i16)_1)` 从当前函数返回。

### Lines 1561-1584

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vclo_w(__m128i _1) {
  return (__m128i)__builtin_lsx_vclo_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vclo_d(__m128i _1) {
  return (__m128i)__builtin_lsx_vclo_d((v2i64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vclz_b(__m128i _1) {
  return (__m128i)__builtin_lsx_vclz_b((v16i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vclz_h(__m128i _1) {
  return (__m128i)__builtin_lsx_vclz_h((v8i16)_1);
````
- **L1561 EN**: Closes the current lexical scope or compound statement.
  **L1561 CN**: 结束当前词法作用域或复合语句块。
- **L1562 EN**: Blank line separating nearby declarations or logic blocks.
  **L1562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1563 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1563 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1564 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1564 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1565 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vclo_w(__m128i _1) {`.
  **L1565 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vclo_w(__m128i _1) {`。
- **L1566 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vclo_w((v4i32)_1)`.
  **L1566 CN**: 以 `(__m128i)__builtin_lsx_vclo_w((v4i32)_1)` 从当前函数返回。
- **L1567 EN**: Closes the current lexical scope or compound statement.
  **L1567 CN**: 结束当前词法作用域或复合语句块。
- **L1568 EN**: Blank line separating nearby declarations or logic blocks.
  **L1568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1569 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1569 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1570 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1570 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1571 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vclo_d(__m128i _1) {`.
  **L1571 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vclo_d(__m128i _1) {`。
- **L1572 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vclo_d((v2i64)_1)`.
  **L1572 CN**: 以 `(__m128i)__builtin_lsx_vclo_d((v2i64)_1)` 从当前函数返回。
- **L1573 EN**: Closes the current lexical scope or compound statement.
  **L1573 CN**: 结束当前词法作用域或复合语句块。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1575 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1575 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1576 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1576 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1577 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vclz_b(__m128i _1) {`.
  **L1577 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vclz_b(__m128i _1) {`。
- **L1578 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vclz_b((v16i8)_1)`.
  **L1578 CN**: 以 `(__m128i)__builtin_lsx_vclz_b((v16i8)_1)` 从当前函数返回。
- **L1579 EN**: Closes the current lexical scope or compound statement.
  **L1579 CN**: 结束当前词法作用域或复合语句块。
- **L1580 EN**: Blank line separating nearby declarations or logic blocks.
  **L1580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1581 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1581 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1582 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1582 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1583 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vclz_h(__m128i _1) {`.
  **L1583 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vclz_h(__m128i _1) {`。
- **L1584 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vclz_h((v8i16)_1)`.
  **L1584 CN**: 以 `(__m128i)__builtin_lsx_vclz_h((v8i16)_1)` 从当前函数返回。

### Lines 1585-1608

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vclz_w(__m128i _1) {
  return (__m128i)__builtin_lsx_vclz_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vclz_d(__m128i _1) {
  return (__m128i)__builtin_lsx_vclz_d((v2i64)_1);
}

#define __lsx_vpickve2gr_b(/*__m128i*/ _1, /*ui4*/ _2)                         \
  ((int)__builtin_lsx_vpickve2gr_b((v16i8)(_1), (_2)))

#define __lsx_vpickve2gr_h(/*__m128i*/ _1, /*ui3*/ _2)                         \
  ((int)__builtin_lsx_vpickve2gr_h((v8i16)(_1), (_2)))

#define __lsx_vpickve2gr_w(/*__m128i*/ _1, /*ui2*/ _2)                         \
  ((int)__builtin_lsx_vpickve2gr_w((v4i32)(_1), (_2)))

#define __lsx_vpickve2gr_d(/*__m128i*/ _1, /*ui1*/ _2)                         \
````
- **L1585 EN**: Closes the current lexical scope or compound statement.
  **L1585 CN**: 结束当前词法作用域或复合语句块。
- **L1586 EN**: Blank line separating nearby declarations or logic blocks.
  **L1586 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1587 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1587 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1588 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1588 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1589 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vclz_w(__m128i _1) {`.
  **L1589 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vclz_w(__m128i _1) {`。
- **L1590 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vclz_w((v4i32)_1)`.
  **L1590 CN**: 以 `(__m128i)__builtin_lsx_vclz_w((v4i32)_1)` 从当前函数返回。
- **L1591 EN**: Closes the current lexical scope or compound statement.
  **L1591 CN**: 结束当前词法作用域或复合语句块。
- **L1592 EN**: Blank line separating nearby declarations or logic blocks.
  **L1592 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1593 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1593 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1594 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1594 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1595 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vclz_d(__m128i _1) {`.
  **L1595 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vclz_d(__m128i _1) {`。
- **L1596 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vclz_d((v2i64)_1)`.
  **L1596 CN**: 以 `(__m128i)__builtin_lsx_vclz_d((v2i64)_1)` 从当前函数返回。
- **L1597 EN**: Closes the current lexical scope or compound statement.
  **L1597 CN**: 结束当前词法作用域或复合语句块。
- **L1598 EN**: Blank line separating nearby declarations or logic blocks.
  **L1598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1599 EN**: Defines macro `__lsx_vpickve2gr_b(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1599 CN**: 定义宏 `__lsx_vpickve2gr_b(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L1600 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpickve2gr_b`.
  **L1600 CN**: 继续与可调用符号 `__builtin_lsx_vpickve2gr_b` 相关的逻辑。
- **L1601 EN**: Blank line separating nearby declarations or logic blocks.
  **L1601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1602 EN**: Defines macro `__lsx_vpickve2gr_h(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1602 CN**: 定义宏 `__lsx_vpickve2gr_h(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L1603 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpickve2gr_h`.
  **L1603 CN**: 继续与可调用符号 `__builtin_lsx_vpickve2gr_h` 相关的逻辑。
- **L1604 EN**: Blank line separating nearby declarations or logic blocks.
  **L1604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1605 EN**: Defines macro `__lsx_vpickve2gr_w(/*__m128i*/ _1, /*ui2*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1605 CN**: 定义宏 `__lsx_vpickve2gr_w(/*__m128i*/ _1, /*ui2*/ _2)`，用于条件编译、简写或 API 生成。
- **L1606 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpickve2gr_w`.
  **L1606 CN**: 继续与可调用符号 `__builtin_lsx_vpickve2gr_w` 相关的逻辑。
- **L1607 EN**: Blank line separating nearby declarations or logic blocks.
  **L1607 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1608 EN**: Defines macro `__lsx_vpickve2gr_d(/*__m128i*/ _1, /*ui1*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1608 CN**: 定义宏 `__lsx_vpickve2gr_d(/*__m128i*/ _1, /*ui1*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 1609-1632

````c
  ((long int)__builtin_lsx_vpickve2gr_d((v2i64)(_1), (_2)))

#define __lsx_vpickve2gr_bu(/*__m128i*/ _1, /*ui4*/ _2)                        \
  ((unsigned int)__builtin_lsx_vpickve2gr_bu((v16i8)(_1), (_2)))

#define __lsx_vpickve2gr_hu(/*__m128i*/ _1, /*ui3*/ _2)                        \
  ((unsigned int)__builtin_lsx_vpickve2gr_hu((v8i16)(_1), (_2)))

#define __lsx_vpickve2gr_wu(/*__m128i*/ _1, /*ui2*/ _2)                        \
  ((unsigned int)__builtin_lsx_vpickve2gr_wu((v4i32)(_1), (_2)))

#define __lsx_vpickve2gr_du(/*__m128i*/ _1, /*ui1*/ _2)                        \
  ((unsigned long int)__builtin_lsx_vpickve2gr_du((v2i64)(_1), (_2)))

#define __lsx_vinsgr2vr_b(/*__m128i*/ _1, /*int*/ _2, /*ui4*/ _3)              \
  ((__m128i)__builtin_lsx_vinsgr2vr_b((v16i8)(_1), (int)(_2), (_3)))

#define __lsx_vinsgr2vr_h(/*__m128i*/ _1, /*int*/ _2, /*ui3*/ _3)              \
  ((__m128i)__builtin_lsx_vinsgr2vr_h((v8i16)(_1), (int)(_2), (_3)))

#define __lsx_vinsgr2vr_w(/*__m128i*/ _1, /*int*/ _2, /*ui2*/ _3)              \
  ((__m128i)__builtin_lsx_vinsgr2vr_w((v4i32)(_1), (int)(_2), (_3)))

#define __lsx_vinsgr2vr_d(/*__m128i*/ _1, /*long int*/ _2, /*ui1*/ _3)         \
````
- **L1609 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpickve2gr_d`.
  **L1609 CN**: 继续与可调用符号 `__builtin_lsx_vpickve2gr_d` 相关的逻辑。
- **L1610 EN**: Blank line separating nearby declarations or logic blocks.
  **L1610 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1611 EN**: Defines macro `__lsx_vpickve2gr_bu(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1611 CN**: 定义宏 `__lsx_vpickve2gr_bu(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L1612 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpickve2gr_bu`.
  **L1612 CN**: 继续与可调用符号 `__builtin_lsx_vpickve2gr_bu` 相关的逻辑。
- **L1613 EN**: Blank line separating nearby declarations or logic blocks.
  **L1613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1614 EN**: Defines macro `__lsx_vpickve2gr_hu(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1614 CN**: 定义宏 `__lsx_vpickve2gr_hu(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L1615 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpickve2gr_hu`.
  **L1615 CN**: 继续与可调用符号 `__builtin_lsx_vpickve2gr_hu` 相关的逻辑。
- **L1616 EN**: Blank line separating nearby declarations or logic blocks.
  **L1616 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1617 EN**: Defines macro `__lsx_vpickve2gr_wu(/*__m128i*/ _1, /*ui2*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1617 CN**: 定义宏 `__lsx_vpickve2gr_wu(/*__m128i*/ _1, /*ui2*/ _2)`，用于条件编译、简写或 API 生成。
- **L1618 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpickve2gr_wu`.
  **L1618 CN**: 继续与可调用符号 `__builtin_lsx_vpickve2gr_wu` 相关的逻辑。
- **L1619 EN**: Blank line separating nearby declarations or logic blocks.
  **L1619 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1620 EN**: Defines macro `__lsx_vpickve2gr_du(/*__m128i*/ _1, /*ui1*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1620 CN**: 定义宏 `__lsx_vpickve2gr_du(/*__m128i*/ _1, /*ui1*/ _2)`，用于条件编译、简写或 API 生成。
- **L1621 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpickve2gr_du`.
  **L1621 CN**: 继续与可调用符号 `__builtin_lsx_vpickve2gr_du` 相关的逻辑。
- **L1622 EN**: Blank line separating nearby declarations or logic blocks.
  **L1622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1623 EN**: Defines macro `__lsx_vinsgr2vr_b(/*__m128i*/ _1, /*int*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L1623 CN**: 定义宏 `__lsx_vinsgr2vr_b(/*__m128i*/ _1, /*int*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L1624 EN**: Continues logic associated with callable symbol `__builtin_lsx_vinsgr2vr_b`.
  **L1624 CN**: 继续与可调用符号 `__builtin_lsx_vinsgr2vr_b` 相关的逻辑。
- **L1625 EN**: Blank line separating nearby declarations or logic blocks.
  **L1625 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1626 EN**: Defines macro `__lsx_vinsgr2vr_h(/*__m128i*/ _1, /*int*/ _2, /*ui3*/ _3)` for conditional compilation, shorthand, or API generation.
  **L1626 CN**: 定义宏 `__lsx_vinsgr2vr_h(/*__m128i*/ _1, /*int*/ _2, /*ui3*/ _3)`，用于条件编译、简写或 API 生成。
- **L1627 EN**: Continues logic associated with callable symbol `__builtin_lsx_vinsgr2vr_h`.
  **L1627 CN**: 继续与可调用符号 `__builtin_lsx_vinsgr2vr_h` 相关的逻辑。
- **L1628 EN**: Blank line separating nearby declarations or logic blocks.
  **L1628 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1629 EN**: Defines macro `__lsx_vinsgr2vr_w(/*__m128i*/ _1, /*int*/ _2, /*ui2*/ _3)` for conditional compilation, shorthand, or API generation.
  **L1629 CN**: 定义宏 `__lsx_vinsgr2vr_w(/*__m128i*/ _1, /*int*/ _2, /*ui2*/ _3)`，用于条件编译、简写或 API 生成。
- **L1630 EN**: Continues logic associated with callable symbol `__builtin_lsx_vinsgr2vr_w`.
  **L1630 CN**: 继续与可调用符号 `__builtin_lsx_vinsgr2vr_w` 相关的逻辑。
- **L1631 EN**: Blank line separating nearby declarations or logic blocks.
  **L1631 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1632 EN**: Defines macro `__lsx_vinsgr2vr_d(/*__m128i*/ _1, /*long int*/ _2, /*ui1*/ _3)` for conditional compilation, shorthand, or API generation.
  **L1632 CN**: 定义宏 `__lsx_vinsgr2vr_d(/*__m128i*/ _1, /*long int*/ _2, /*ui1*/ _3)`，用于条件编译、简写或 API 生成。

### Lines 1633-1656

````c
  ((__m128i)__builtin_lsx_vinsgr2vr_d((v2i64)(_1), (long int)(_2), (_3)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfadd_s(__m128 _1, __m128 _2) {
  return (__m128)__builtin_lsx_vfadd_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfadd_d(__m128d _1, __m128d _2) {
  return (__m128d)__builtin_lsx_vfadd_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfsub_s(__m128 _1, __m128 _2) {
  return (__m128)__builtin_lsx_vfsub_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfsub_d(__m128d _1, __m128d _2) {
  return (__m128d)__builtin_lsx_vfsub_d((v2f64)_1, (v2f64)_2);
````
- **L1633 EN**: Continues logic associated with callable symbol `__builtin_lsx_vinsgr2vr_d`.
  **L1633 CN**: 继续与可调用符号 `__builtin_lsx_vinsgr2vr_d` 相关的逻辑。
- **L1634 EN**: Blank line separating nearby declarations or logic blocks.
  **L1634 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1635 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1635 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1636 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1636 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1637 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfadd_s(__m128 _1, __m128 _2) {`.
  **L1637 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfadd_s(__m128 _1, __m128 _2) {`。
- **L1638 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfadd_s((v4f32)_1, (v4f32)_2)`.
  **L1638 CN**: 以 `(__m128)__builtin_lsx_vfadd_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1639 EN**: Closes the current lexical scope or compound statement.
  **L1639 CN**: 结束当前词法作用域或复合语句块。
- **L1640 EN**: Blank line separating nearby declarations or logic blocks.
  **L1640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1641 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1641 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1642 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1642 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1643 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfadd_d(__m128d _1, __m128d _2) {`.
  **L1643 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfadd_d(__m128d _1, __m128d _2) {`。
- **L1644 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfadd_d((v2f64)_1, (v2f64)_2)`.
  **L1644 CN**: 以 `(__m128d)__builtin_lsx_vfadd_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L1645 EN**: Closes the current lexical scope or compound statement.
  **L1645 CN**: 结束当前词法作用域或复合语句块。
- **L1646 EN**: Blank line separating nearby declarations or logic blocks.
  **L1646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1647 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1647 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1648 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1648 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1649 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfsub_s(__m128 _1, __m128 _2) {`.
  **L1649 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfsub_s(__m128 _1, __m128 _2) {`。
- **L1650 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfsub_s((v4f32)_1, (v4f32)_2)`.
  **L1650 CN**: 以 `(__m128)__builtin_lsx_vfsub_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1651 EN**: Closes the current lexical scope or compound statement.
  **L1651 CN**: 结束当前词法作用域或复合语句块。
- **L1652 EN**: Blank line separating nearby declarations or logic blocks.
  **L1652 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1653 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1653 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1654 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1654 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1655 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfsub_d(__m128d _1, __m128d _2) {`.
  **L1655 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfsub_d(__m128d _1, __m128d _2) {`。
- **L1656 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfsub_d((v2f64)_1, (v2f64)_2)`.
  **L1656 CN**: 以 `(__m128d)__builtin_lsx_vfsub_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 1657-1680

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfmul_s(__m128 _1, __m128 _2) {
  return (__m128)__builtin_lsx_vfmul_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfmul_d(__m128d _1, __m128d _2) {
  return (__m128d)__builtin_lsx_vfmul_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfdiv_s(__m128 _1, __m128 _2) {
  return (__m128)__builtin_lsx_vfdiv_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfdiv_d(__m128d _1, __m128d _2) {
  return (__m128d)__builtin_lsx_vfdiv_d((v2f64)_1, (v2f64)_2);
````
- **L1657 EN**: Closes the current lexical scope or compound statement.
  **L1657 CN**: 结束当前词法作用域或复合语句块。
- **L1658 EN**: Blank line separating nearby declarations or logic blocks.
  **L1658 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1659 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1659 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1660 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1660 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1661 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmul_s(__m128 _1, __m128 _2) {`.
  **L1661 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmul_s(__m128 _1, __m128 _2) {`。
- **L1662 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfmul_s((v4f32)_1, (v4f32)_2)`.
  **L1662 CN**: 以 `(__m128)__builtin_lsx_vfmul_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1663 EN**: Closes the current lexical scope or compound statement.
  **L1663 CN**: 结束当前词法作用域或复合语句块。
- **L1664 EN**: Blank line separating nearby declarations or logic blocks.
  **L1664 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1665 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1665 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1666 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1666 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1667 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmul_d(__m128d _1, __m128d _2) {`.
  **L1667 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmul_d(__m128d _1, __m128d _2) {`。
- **L1668 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfmul_d((v2f64)_1, (v2f64)_2)`.
  **L1668 CN**: 以 `(__m128d)__builtin_lsx_vfmul_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L1669 EN**: Closes the current lexical scope or compound statement.
  **L1669 CN**: 结束当前词法作用域或复合语句块。
- **L1670 EN**: Blank line separating nearby declarations or logic blocks.
  **L1670 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1671 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1671 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1672 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1672 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1673 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfdiv_s(__m128 _1, __m128 _2) {`.
  **L1673 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfdiv_s(__m128 _1, __m128 _2) {`。
- **L1674 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfdiv_s((v4f32)_1, (v4f32)_2)`.
  **L1674 CN**: 以 `(__m128)__builtin_lsx_vfdiv_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1675 EN**: Closes the current lexical scope or compound statement.
  **L1675 CN**: 结束当前词法作用域或复合语句块。
- **L1676 EN**: Blank line separating nearby declarations or logic blocks.
  **L1676 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1677 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1677 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1678 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1678 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1679 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfdiv_d(__m128d _1, __m128d _2) {`.
  **L1679 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfdiv_d(__m128d _1, __m128d _2) {`。
- **L1680 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfdiv_d((v2f64)_1, (v2f64)_2)`.
  **L1680 CN**: 以 `(__m128d)__builtin_lsx_vfdiv_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 1681-1704

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcvt_h_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcvt_h_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfcvt_s_d(__m128d _1, __m128d _2) {
  return (__m128)__builtin_lsx_vfcvt_s_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfmin_s(__m128 _1, __m128 _2) {
  return (__m128)__builtin_lsx_vfmin_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfmin_d(__m128d _1, __m128d _2) {
  return (__m128d)__builtin_lsx_vfmin_d((v2f64)_1, (v2f64)_2);
````
- **L1681 EN**: Closes the current lexical scope or compound statement.
  **L1681 CN**: 结束当前词法作用域或复合语句块。
- **L1682 EN**: Blank line separating nearby declarations or logic blocks.
  **L1682 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1683 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1683 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1684 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1684 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1685 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcvt_h_s(__m128 _1, __m128 _2) {`.
  **L1685 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcvt_h_s(__m128 _1, __m128 _2) {`。
- **L1686 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcvt_h_s((v4f32)_1, (v4f32)_2)`.
  **L1686 CN**: 以 `(__m128i)__builtin_lsx_vfcvt_h_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1687 EN**: Closes the current lexical scope or compound statement.
  **L1687 CN**: 结束当前词法作用域或复合语句块。
- **L1688 EN**: Blank line separating nearby declarations or logic blocks.
  **L1688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1689 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1689 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1690 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1690 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1691 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcvt_s_d(__m128d _1, __m128d _2) {`.
  **L1691 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcvt_s_d(__m128d _1, __m128d _2) {`。
- **L1692 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfcvt_s_d((v2f64)_1, (v2f64)_2)`.
  **L1692 CN**: 以 `(__m128)__builtin_lsx_vfcvt_s_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L1693 EN**: Closes the current lexical scope or compound statement.
  **L1693 CN**: 结束当前词法作用域或复合语句块。
- **L1694 EN**: Blank line separating nearby declarations or logic blocks.
  **L1694 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1695 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1695 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1696 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1696 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1697 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmin_s(__m128 _1, __m128 _2) {`.
  **L1697 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmin_s(__m128 _1, __m128 _2) {`。
- **L1698 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfmin_s((v4f32)_1, (v4f32)_2)`.
  **L1698 CN**: 以 `(__m128)__builtin_lsx_vfmin_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1699 EN**: Closes the current lexical scope or compound statement.
  **L1699 CN**: 结束当前词法作用域或复合语句块。
- **L1700 EN**: Blank line separating nearby declarations or logic blocks.
  **L1700 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1701 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1701 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1702 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1702 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1703 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmin_d(__m128d _1, __m128d _2) {`.
  **L1703 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmin_d(__m128d _1, __m128d _2) {`。
- **L1704 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfmin_d((v2f64)_1, (v2f64)_2)`.
  **L1704 CN**: 以 `(__m128d)__builtin_lsx_vfmin_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 1705-1728

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfmina_s(__m128 _1, __m128 _2) {
  return (__m128)__builtin_lsx_vfmina_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfmina_d(__m128d _1, __m128d _2) {
  return (__m128d)__builtin_lsx_vfmina_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfmax_s(__m128 _1, __m128 _2) {
  return (__m128)__builtin_lsx_vfmax_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfmax_d(__m128d _1, __m128d _2) {
  return (__m128d)__builtin_lsx_vfmax_d((v2f64)_1, (v2f64)_2);
````
- **L1705 EN**: Closes the current lexical scope or compound statement.
  **L1705 CN**: 结束当前词法作用域或复合语句块。
- **L1706 EN**: Blank line separating nearby declarations or logic blocks.
  **L1706 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1707 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1707 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1708 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1708 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1709 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmina_s(__m128 _1, __m128 _2) {`.
  **L1709 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmina_s(__m128 _1, __m128 _2) {`。
- **L1710 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfmina_s((v4f32)_1, (v4f32)_2)`.
  **L1710 CN**: 以 `(__m128)__builtin_lsx_vfmina_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1711 EN**: Closes the current lexical scope or compound statement.
  **L1711 CN**: 结束当前词法作用域或复合语句块。
- **L1712 EN**: Blank line separating nearby declarations or logic blocks.
  **L1712 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1713 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1713 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1714 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1714 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1715 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmina_d(__m128d _1, __m128d _2) {`.
  **L1715 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmina_d(__m128d _1, __m128d _2) {`。
- **L1716 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfmina_d((v2f64)_1, (v2f64)_2)`.
  **L1716 CN**: 以 `(__m128d)__builtin_lsx_vfmina_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L1717 EN**: Closes the current lexical scope or compound statement.
  **L1717 CN**: 结束当前词法作用域或复合语句块。
- **L1718 EN**: Blank line separating nearby declarations or logic blocks.
  **L1718 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1719 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1719 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1720 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1720 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1721 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmax_s(__m128 _1, __m128 _2) {`.
  **L1721 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmax_s(__m128 _1, __m128 _2) {`。
- **L1722 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfmax_s((v4f32)_1, (v4f32)_2)`.
  **L1722 CN**: 以 `(__m128)__builtin_lsx_vfmax_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1723 EN**: Closes the current lexical scope or compound statement.
  **L1723 CN**: 结束当前词法作用域或复合语句块。
- **L1724 EN**: Blank line separating nearby declarations or logic blocks.
  **L1724 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1725 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1725 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1726 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1726 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1727 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmax_d(__m128d _1, __m128d _2) {`.
  **L1727 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmax_d(__m128d _1, __m128d _2) {`。
- **L1728 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfmax_d((v2f64)_1, (v2f64)_2)`.
  **L1728 CN**: 以 `(__m128d)__builtin_lsx_vfmax_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 1729-1752

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfmaxa_s(__m128 _1, __m128 _2) {
  return (__m128)__builtin_lsx_vfmaxa_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfmaxa_d(__m128d _1, __m128d _2) {
  return (__m128d)__builtin_lsx_vfmaxa_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfclass_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vfclass_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfclass_d(__m128d _1) {
  return (__m128i)__builtin_lsx_vfclass_d((v2f64)_1);
````
- **L1729 EN**: Closes the current lexical scope or compound statement.
  **L1729 CN**: 结束当前词法作用域或复合语句块。
- **L1730 EN**: Blank line separating nearby declarations or logic blocks.
  **L1730 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1731 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1731 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1732 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1732 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1733 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmaxa_s(__m128 _1, __m128 _2) {`.
  **L1733 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmaxa_s(__m128 _1, __m128 _2) {`。
- **L1734 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfmaxa_s((v4f32)_1, (v4f32)_2)`.
  **L1734 CN**: 以 `(__m128)__builtin_lsx_vfmaxa_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L1735 EN**: Closes the current lexical scope or compound statement.
  **L1735 CN**: 结束当前词法作用域或复合语句块。
- **L1736 EN**: Blank line separating nearby declarations or logic blocks.
  **L1736 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1737 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1737 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1738 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1738 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1739 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmaxa_d(__m128d _1, __m128d _2) {`.
  **L1739 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmaxa_d(__m128d _1, __m128d _2) {`。
- **L1740 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfmaxa_d((v2f64)_1, (v2f64)_2)`.
  **L1740 CN**: 以 `(__m128d)__builtin_lsx_vfmaxa_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L1741 EN**: Closes the current lexical scope or compound statement.
  **L1741 CN**: 结束当前词法作用域或复合语句块。
- **L1742 EN**: Blank line separating nearby declarations or logic blocks.
  **L1742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1743 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1743 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1744 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1744 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1745 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfclass_s(__m128 _1) {`.
  **L1745 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfclass_s(__m128 _1) {`。
- **L1746 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfclass_s((v4f32)_1)`.
  **L1746 CN**: 以 `(__m128i)__builtin_lsx_vfclass_s((v4f32)_1)` 从当前函数返回。
- **L1747 EN**: Closes the current lexical scope or compound statement.
  **L1747 CN**: 结束当前词法作用域或复合语句块。
- **L1748 EN**: Blank line separating nearby declarations or logic blocks.
  **L1748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1749 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1749 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1750 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1750 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1751 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfclass_d(__m128d _1) {`.
  **L1751 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfclass_d(__m128d _1) {`。
- **L1752 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfclass_d((v2f64)_1)`.
  **L1752 CN**: 以 `(__m128i)__builtin_lsx_vfclass_d((v2f64)_1)` 从当前函数返回。

### Lines 1753-1776

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfsqrt_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfsqrt_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfsqrt_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfsqrt_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrecip_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfrecip_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrecip_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrecip_d((v2f64)_1);
````
- **L1753 EN**: Closes the current lexical scope or compound statement.
  **L1753 CN**: 结束当前词法作用域或复合语句块。
- **L1754 EN**: Blank line separating nearby declarations or logic blocks.
  **L1754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1755 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1755 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1756 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1756 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1757 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfsqrt_s(__m128 _1) {`.
  **L1757 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfsqrt_s(__m128 _1) {`。
- **L1758 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfsqrt_s((v4f32)_1)`.
  **L1758 CN**: 以 `(__m128)__builtin_lsx_vfsqrt_s((v4f32)_1)` 从当前函数返回。
- **L1759 EN**: Closes the current lexical scope or compound statement.
  **L1759 CN**: 结束当前词法作用域或复合语句块。
- **L1760 EN**: Blank line separating nearby declarations or logic blocks.
  **L1760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1761 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1761 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1762 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1762 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1763 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfsqrt_d(__m128d _1) {`.
  **L1763 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfsqrt_d(__m128d _1) {`。
- **L1764 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfsqrt_d((v2f64)_1)`.
  **L1764 CN**: 以 `(__m128d)__builtin_lsx_vfsqrt_d((v2f64)_1)` 从当前函数返回。
- **L1765 EN**: Closes the current lexical scope or compound statement.
  **L1765 CN**: 结束当前词法作用域或复合语句块。
- **L1766 EN**: Blank line separating nearby declarations or logic blocks.
  **L1766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1767 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1767 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1768 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1768 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1769 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrecip_s(__m128 _1) {`.
  **L1769 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrecip_s(__m128 _1) {`。
- **L1770 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrecip_s((v4f32)_1)`.
  **L1770 CN**: 以 `(__m128)__builtin_lsx_vfrecip_s((v4f32)_1)` 从当前函数返回。
- **L1771 EN**: Closes the current lexical scope or compound statement.
  **L1771 CN**: 结束当前词法作用域或复合语句块。
- **L1772 EN**: Blank line separating nearby declarations or logic blocks.
  **L1772 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1773 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1773 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1774 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1774 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1775 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrecip_d(__m128d _1) {`.
  **L1775 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrecip_d(__m128d _1) {`。
- **L1776 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrecip_d((v2f64)_1)`.
  **L1776 CN**: 以 `(__m128d)__builtin_lsx_vfrecip_d((v2f64)_1)` 从当前函数返回。

### Lines 1777-1800

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrint_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfrint_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrint_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrint_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrsqrt_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfrsqrt_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrsqrt_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrsqrt_d((v2f64)_1);
````
- **L1777 EN**: Closes the current lexical scope or compound statement.
  **L1777 CN**: 结束当前词法作用域或复合语句块。
- **L1778 EN**: Blank line separating nearby declarations or logic blocks.
  **L1778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1779 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1779 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1780 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1780 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1781 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrint_s(__m128 _1) {`.
  **L1781 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrint_s(__m128 _1) {`。
- **L1782 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrint_s((v4f32)_1)`.
  **L1782 CN**: 以 `(__m128)__builtin_lsx_vfrint_s((v4f32)_1)` 从当前函数返回。
- **L1783 EN**: Closes the current lexical scope or compound statement.
  **L1783 CN**: 结束当前词法作用域或复合语句块。
- **L1784 EN**: Blank line separating nearby declarations or logic blocks.
  **L1784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1785 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1785 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1786 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1786 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1787 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrint_d(__m128d _1) {`.
  **L1787 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrint_d(__m128d _1) {`。
- **L1788 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrint_d((v2f64)_1)`.
  **L1788 CN**: 以 `(__m128d)__builtin_lsx_vfrint_d((v2f64)_1)` 从当前函数返回。
- **L1789 EN**: Closes the current lexical scope or compound statement.
  **L1789 CN**: 结束当前词法作用域或复合语句块。
- **L1790 EN**: Blank line separating nearby declarations or logic blocks.
  **L1790 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1791 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1791 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1792 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1792 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1793 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrsqrt_s(__m128 _1) {`.
  **L1793 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrsqrt_s(__m128 _1) {`。
- **L1794 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrsqrt_s((v4f32)_1)`.
  **L1794 CN**: 以 `(__m128)__builtin_lsx_vfrsqrt_s((v4f32)_1)` 从当前函数返回。
- **L1795 EN**: Closes the current lexical scope or compound statement.
  **L1795 CN**: 结束当前词法作用域或复合语句块。
- **L1796 EN**: Blank line separating nearby declarations or logic blocks.
  **L1796 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1797 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1797 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1798 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1798 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1799 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrsqrt_d(__m128d _1) {`.
  **L1799 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrsqrt_d(__m128d _1) {`。
- **L1800 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrsqrt_d((v2f64)_1)`.
  **L1800 CN**: 以 `(__m128d)__builtin_lsx_vfrsqrt_d((v2f64)_1)` 从当前函数返回。

### Lines 1801-1824

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vflogb_s(__m128 _1) {
  return (__m128)__builtin_lsx_vflogb_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vflogb_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vflogb_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfcvth_s_h(__m128i _1) {
  return (__m128)__builtin_lsx_vfcvth_s_h((v8i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfcvth_d_s(__m128 _1) {
  return (__m128d)__builtin_lsx_vfcvth_d_s((v4f32)_1);
````
- **L1801 EN**: Closes the current lexical scope or compound statement.
  **L1801 CN**: 结束当前词法作用域或复合语句块。
- **L1802 EN**: Blank line separating nearby declarations or logic blocks.
  **L1802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1803 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1803 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1804 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1804 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1805 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vflogb_s(__m128 _1) {`.
  **L1805 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vflogb_s(__m128 _1) {`。
- **L1806 EN**: Returns from the current function with `(__m128)__builtin_lsx_vflogb_s((v4f32)_1)`.
  **L1806 CN**: 以 `(__m128)__builtin_lsx_vflogb_s((v4f32)_1)` 从当前函数返回。
- **L1807 EN**: Closes the current lexical scope or compound statement.
  **L1807 CN**: 结束当前词法作用域或复合语句块。
- **L1808 EN**: Blank line separating nearby declarations or logic blocks.
  **L1808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1809 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1809 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1810 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1810 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1811 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vflogb_d(__m128d _1) {`.
  **L1811 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vflogb_d(__m128d _1) {`。
- **L1812 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vflogb_d((v2f64)_1)`.
  **L1812 CN**: 以 `(__m128d)__builtin_lsx_vflogb_d((v2f64)_1)` 从当前函数返回。
- **L1813 EN**: Closes the current lexical scope or compound statement.
  **L1813 CN**: 结束当前词法作用域或复合语句块。
- **L1814 EN**: Blank line separating nearby declarations or logic blocks.
  **L1814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1815 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1815 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1816 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1816 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1817 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcvth_s_h(__m128i _1) {`.
  **L1817 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcvth_s_h(__m128i _1) {`。
- **L1818 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfcvth_s_h((v8i16)_1)`.
  **L1818 CN**: 以 `(__m128)__builtin_lsx_vfcvth_s_h((v8i16)_1)` 从当前函数返回。
- **L1819 EN**: Closes the current lexical scope or compound statement.
  **L1819 CN**: 结束当前词法作用域或复合语句块。
- **L1820 EN**: Blank line separating nearby declarations or logic blocks.
  **L1820 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1821 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1821 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1822 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1822 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1823 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcvth_d_s(__m128 _1) {`.
  **L1823 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcvth_d_s(__m128 _1) {`。
- **L1824 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfcvth_d_s((v4f32)_1)`.
  **L1824 CN**: 以 `(__m128d)__builtin_lsx_vfcvth_d_s((v4f32)_1)` 从当前函数返回。

### Lines 1825-1848

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfcvtl_s_h(__m128i _1) {
  return (__m128)__builtin_lsx_vfcvtl_s_h((v8i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfcvtl_d_s(__m128 _1) {
  return (__m128d)__builtin_lsx_vfcvtl_d_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftint_w_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftint_w_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftint_l_d(__m128d _1) {
  return (__m128i)__builtin_lsx_vftint_l_d((v2f64)_1);
````
- **L1825 EN**: Closes the current lexical scope or compound statement.
  **L1825 CN**: 结束当前词法作用域或复合语句块。
- **L1826 EN**: Blank line separating nearby declarations or logic blocks.
  **L1826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1827 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1827 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1828 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1828 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1829 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcvtl_s_h(__m128i _1) {`.
  **L1829 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcvtl_s_h(__m128i _1) {`。
- **L1830 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfcvtl_s_h((v8i16)_1)`.
  **L1830 CN**: 以 `(__m128)__builtin_lsx_vfcvtl_s_h((v8i16)_1)` 从当前函数返回。
- **L1831 EN**: Closes the current lexical scope or compound statement.
  **L1831 CN**: 结束当前词法作用域或复合语句块。
- **L1832 EN**: Blank line separating nearby declarations or logic blocks.
  **L1832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1833 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1833 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1834 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1834 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1835 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcvtl_d_s(__m128 _1) {`.
  **L1835 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcvtl_d_s(__m128 _1) {`。
- **L1836 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfcvtl_d_s((v4f32)_1)`.
  **L1836 CN**: 以 `(__m128d)__builtin_lsx_vfcvtl_d_s((v4f32)_1)` 从当前函数返回。
- **L1837 EN**: Closes the current lexical scope or compound statement.
  **L1837 CN**: 结束当前词法作用域或复合语句块。
- **L1838 EN**: Blank line separating nearby declarations or logic blocks.
  **L1838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1839 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1839 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1840 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1840 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1841 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftint_w_s(__m128 _1) {`.
  **L1841 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftint_w_s(__m128 _1) {`。
- **L1842 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftint_w_s((v4f32)_1)`.
  **L1842 CN**: 以 `(__m128i)__builtin_lsx_vftint_w_s((v4f32)_1)` 从当前函数返回。
- **L1843 EN**: Closes the current lexical scope or compound statement.
  **L1843 CN**: 结束当前词法作用域或复合语句块。
- **L1844 EN**: Blank line separating nearby declarations or logic blocks.
  **L1844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1845 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1845 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1846 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1846 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1847 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftint_l_d(__m128d _1) {`.
  **L1847 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftint_l_d(__m128d _1) {`。
- **L1848 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftint_l_d((v2f64)_1)`.
  **L1848 CN**: 以 `(__m128i)__builtin_lsx_vftint_l_d((v2f64)_1)` 从当前函数返回。

### Lines 1849-1872

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftint_wu_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftint_wu_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftint_lu_d(__m128d _1) {
  return (__m128i)__builtin_lsx_vftint_lu_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrz_w_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrz_w_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrz_l_d(__m128d _1) {
  return (__m128i)__builtin_lsx_vftintrz_l_d((v2f64)_1);
````
- **L1849 EN**: Closes the current lexical scope or compound statement.
  **L1849 CN**: 结束当前词法作用域或复合语句块。
- **L1850 EN**: Blank line separating nearby declarations or logic blocks.
  **L1850 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1851 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1851 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1852 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1852 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1853 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftint_wu_s(__m128 _1) {`.
  **L1853 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftint_wu_s(__m128 _1) {`。
- **L1854 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftint_wu_s((v4f32)_1)`.
  **L1854 CN**: 以 `(__m128i)__builtin_lsx_vftint_wu_s((v4f32)_1)` 从当前函数返回。
- **L1855 EN**: Closes the current lexical scope or compound statement.
  **L1855 CN**: 结束当前词法作用域或复合语句块。
- **L1856 EN**: Blank line separating nearby declarations or logic blocks.
  **L1856 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1857 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1857 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1858 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1858 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1859 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftint_lu_d(__m128d _1) {`.
  **L1859 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftint_lu_d(__m128d _1) {`。
- **L1860 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftint_lu_d((v2f64)_1)`.
  **L1860 CN**: 以 `(__m128i)__builtin_lsx_vftint_lu_d((v2f64)_1)` 从当前函数返回。
- **L1861 EN**: Closes the current lexical scope or compound statement.
  **L1861 CN**: 结束当前词法作用域或复合语句块。
- **L1862 EN**: Blank line separating nearby declarations or logic blocks.
  **L1862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1863 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1863 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1864 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1864 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1865 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrz_w_s(__m128 _1) {`.
  **L1865 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrz_w_s(__m128 _1) {`。
- **L1866 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrz_w_s((v4f32)_1)`.
  **L1866 CN**: 以 `(__m128i)__builtin_lsx_vftintrz_w_s((v4f32)_1)` 从当前函数返回。
- **L1867 EN**: Closes the current lexical scope or compound statement.
  **L1867 CN**: 结束当前词法作用域或复合语句块。
- **L1868 EN**: Blank line separating nearby declarations or logic blocks.
  **L1868 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1869 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1869 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1870 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1870 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1871 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrz_l_d(__m128d _1) {`.
  **L1871 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrz_l_d(__m128d _1) {`。
- **L1872 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrz_l_d((v2f64)_1)`.
  **L1872 CN**: 以 `(__m128i)__builtin_lsx_vftintrz_l_d((v2f64)_1)` 从当前函数返回。

### Lines 1873-1896

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrz_wu_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrz_wu_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrz_lu_d(__m128d _1) {
  return (__m128i)__builtin_lsx_vftintrz_lu_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vffint_s_w(__m128i _1) {
  return (__m128)__builtin_lsx_vffint_s_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vffint_d_l(__m128i _1) {
  return (__m128d)__builtin_lsx_vffint_d_l((v2i64)_1);
````
- **L1873 EN**: Closes the current lexical scope or compound statement.
  **L1873 CN**: 结束当前词法作用域或复合语句块。
- **L1874 EN**: Blank line separating nearby declarations or logic blocks.
  **L1874 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1875 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1875 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1876 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1876 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1877 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrz_wu_s(__m128 _1) {`.
  **L1877 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrz_wu_s(__m128 _1) {`。
- **L1878 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrz_wu_s((v4f32)_1)`.
  **L1878 CN**: 以 `(__m128i)__builtin_lsx_vftintrz_wu_s((v4f32)_1)` 从当前函数返回。
- **L1879 EN**: Closes the current lexical scope or compound statement.
  **L1879 CN**: 结束当前词法作用域或复合语句块。
- **L1880 EN**: Blank line separating nearby declarations or logic blocks.
  **L1880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1881 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1881 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1882 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1882 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1883 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrz_lu_d(__m128d _1) {`.
  **L1883 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrz_lu_d(__m128d _1) {`。
- **L1884 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrz_lu_d((v2f64)_1)`.
  **L1884 CN**: 以 `(__m128i)__builtin_lsx_vftintrz_lu_d((v2f64)_1)` 从当前函数返回。
- **L1885 EN**: Closes the current lexical scope or compound statement.
  **L1885 CN**: 结束当前词法作用域或复合语句块。
- **L1886 EN**: Blank line separating nearby declarations or logic blocks.
  **L1886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1887 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1887 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1888 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1888 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1889 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vffint_s_w(__m128i _1) {`.
  **L1889 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vffint_s_w(__m128i _1) {`。
- **L1890 EN**: Returns from the current function with `(__m128)__builtin_lsx_vffint_s_w((v4i32)_1)`.
  **L1890 CN**: 以 `(__m128)__builtin_lsx_vffint_s_w((v4i32)_1)` 从当前函数返回。
- **L1891 EN**: Closes the current lexical scope or compound statement.
  **L1891 CN**: 结束当前词法作用域或复合语句块。
- **L1892 EN**: Blank line separating nearby declarations or logic blocks.
  **L1892 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1893 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1893 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1894 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1894 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1895 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vffint_d_l(__m128i _1) {`.
  **L1895 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vffint_d_l(__m128i _1) {`。
- **L1896 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vffint_d_l((v2i64)_1)`.
  **L1896 CN**: 以 `(__m128d)__builtin_lsx_vffint_d_l((v2i64)_1)` 从当前函数返回。

### Lines 1897-1920

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vffint_s_wu(__m128i _1) {
  return (__m128)__builtin_lsx_vffint_s_wu((v4u32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vffint_d_lu(__m128i _1) {
  return (__m128d)__builtin_lsx_vffint_d_lu((v2u64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vandn_v(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vandn_v((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vneg_b(__m128i _1) {
  return (__m128i)__builtin_lsx_vneg_b((v16i8)_1);
````
- **L1897 EN**: Closes the current lexical scope or compound statement.
  **L1897 CN**: 结束当前词法作用域或复合语句块。
- **L1898 EN**: Blank line separating nearby declarations or logic blocks.
  **L1898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1899 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1899 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1900 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L1900 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L1901 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vffint_s_wu(__m128i _1) {`.
  **L1901 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vffint_s_wu(__m128i _1) {`。
- **L1902 EN**: Returns from the current function with `(__m128)__builtin_lsx_vffint_s_wu((v4u32)_1)`.
  **L1902 CN**: 以 `(__m128)__builtin_lsx_vffint_s_wu((v4u32)_1)` 从当前函数返回。
- **L1903 EN**: Closes the current lexical scope or compound statement.
  **L1903 CN**: 结束当前词法作用域或复合语句块。
- **L1904 EN**: Blank line separating nearby declarations or logic blocks.
  **L1904 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1905 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1905 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1906 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L1906 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L1907 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vffint_d_lu(__m128i _1) {`.
  **L1907 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vffint_d_lu(__m128i _1) {`。
- **L1908 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vffint_d_lu((v2u64)_1)`.
  **L1908 CN**: 以 `(__m128d)__builtin_lsx_vffint_d_lu((v2u64)_1)` 从当前函数返回。
- **L1909 EN**: Closes the current lexical scope or compound statement.
  **L1909 CN**: 结束当前词法作用域或复合语句块。
- **L1910 EN**: Blank line separating nearby declarations or logic blocks.
  **L1910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1911 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1911 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1912 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1912 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1913 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vandn_v(__m128i _1, __m128i _2) {`.
  **L1913 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vandn_v(__m128i _1, __m128i _2) {`。
- **L1914 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vandn_v((v16u8)_1, (v16u8)_2)`.
  **L1914 CN**: 以 `(__m128i)__builtin_lsx_vandn_v((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L1915 EN**: Closes the current lexical scope or compound statement.
  **L1915 CN**: 结束当前词法作用域或复合语句块。
- **L1916 EN**: Blank line separating nearby declarations or logic blocks.
  **L1916 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1917 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1917 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1918 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1918 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1919 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vneg_b(__m128i _1) {`.
  **L1919 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vneg_b(__m128i _1) {`。
- **L1920 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vneg_b((v16i8)_1)`.
  **L1920 CN**: 以 `(__m128i)__builtin_lsx_vneg_b((v16i8)_1)` 从当前函数返回。

### Lines 1921-1944

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vneg_h(__m128i _1) {
  return (__m128i)__builtin_lsx_vneg_h((v8i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vneg_w(__m128i _1) {
  return (__m128i)__builtin_lsx_vneg_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vneg_d(__m128i _1) {
  return (__m128i)__builtin_lsx_vneg_d((v2i64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmuh_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmuh_b((v16i8)_1, (v16i8)_2);
````
- **L1921 EN**: Closes the current lexical scope or compound statement.
  **L1921 CN**: 结束当前词法作用域或复合语句块。
- **L1922 EN**: Blank line separating nearby declarations or logic blocks.
  **L1922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1923 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1923 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1924 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1924 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1925 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vneg_h(__m128i _1) {`.
  **L1925 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vneg_h(__m128i _1) {`。
- **L1926 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vneg_h((v8i16)_1)`.
  **L1926 CN**: 以 `(__m128i)__builtin_lsx_vneg_h((v8i16)_1)` 从当前函数返回。
- **L1927 EN**: Closes the current lexical scope or compound statement.
  **L1927 CN**: 结束当前词法作用域或复合语句块。
- **L1928 EN**: Blank line separating nearby declarations or logic blocks.
  **L1928 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1929 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1929 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1930 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1930 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1931 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vneg_w(__m128i _1) {`.
  **L1931 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vneg_w(__m128i _1) {`。
- **L1932 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vneg_w((v4i32)_1)`.
  **L1932 CN**: 以 `(__m128i)__builtin_lsx_vneg_w((v4i32)_1)` 从当前函数返回。
- **L1933 EN**: Closes the current lexical scope or compound statement.
  **L1933 CN**: 结束当前词法作用域或复合语句块。
- **L1934 EN**: Blank line separating nearby declarations or logic blocks.
  **L1934 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1935 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1935 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1936 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1936 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1937 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vneg_d(__m128i _1) {`.
  **L1937 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vneg_d(__m128i _1) {`。
- **L1938 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vneg_d((v2i64)_1)`.
  **L1938 CN**: 以 `(__m128i)__builtin_lsx_vneg_d((v2i64)_1)` 从当前函数返回。
- **L1939 EN**: Closes the current lexical scope or compound statement.
  **L1939 CN**: 结束当前词法作用域或复合语句块。
- **L1940 EN**: Blank line separating nearby declarations or logic blocks.
  **L1940 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1941 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1941 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1942 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1942 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1943 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmuh_b(__m128i _1, __m128i _2) {`.
  **L1943 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmuh_b(__m128i _1, __m128i _2) {`。
- **L1944 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmuh_b((v16i8)_1, (v16i8)_2)`.
  **L1944 CN**: 以 `(__m128i)__builtin_lsx_vmuh_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。

### Lines 1945-1968

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmuh_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmuh_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmuh_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmuh_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmuh_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmuh_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmuh_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmuh_bu((v16u8)_1, (v16u8)_2);
````
- **L1945 EN**: Closes the current lexical scope or compound statement.
  **L1945 CN**: 结束当前词法作用域或复合语句块。
- **L1946 EN**: Blank line separating nearby declarations or logic blocks.
  **L1946 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1947 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1947 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1948 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1948 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1949 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmuh_h(__m128i _1, __m128i _2) {`.
  **L1949 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmuh_h(__m128i _1, __m128i _2) {`。
- **L1950 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmuh_h((v8i16)_1, (v8i16)_2)`.
  **L1950 CN**: 以 `(__m128i)__builtin_lsx_vmuh_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L1951 EN**: Closes the current lexical scope or compound statement.
  **L1951 CN**: 结束当前词法作用域或复合语句块。
- **L1952 EN**: Blank line separating nearby declarations or logic blocks.
  **L1952 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1953 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1953 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1954 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1954 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1955 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmuh_w(__m128i _1, __m128i _2) {`.
  **L1955 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmuh_w(__m128i _1, __m128i _2) {`。
- **L1956 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmuh_w((v4i32)_1, (v4i32)_2)`.
  **L1956 CN**: 以 `(__m128i)__builtin_lsx_vmuh_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L1957 EN**: Closes the current lexical scope or compound statement.
  **L1957 CN**: 结束当前词法作用域或复合语句块。
- **L1958 EN**: Blank line separating nearby declarations or logic blocks.
  **L1958 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1959 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1959 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1960 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1960 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1961 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmuh_d(__m128i _1, __m128i _2) {`.
  **L1961 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmuh_d(__m128i _1, __m128i _2) {`。
- **L1962 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmuh_d((v2i64)_1, (v2i64)_2)`.
  **L1962 CN**: 以 `(__m128i)__builtin_lsx_vmuh_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L1963 EN**: Closes the current lexical scope or compound statement.
  **L1963 CN**: 结束当前词法作用域或复合语句块。
- **L1964 EN**: Blank line separating nearby declarations or logic blocks.
  **L1964 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1965 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1965 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1966 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1966 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1967 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmuh_bu(__m128i _1, __m128i _2) {`.
  **L1967 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmuh_bu(__m128i _1, __m128i _2) {`。
- **L1968 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmuh_bu((v16u8)_1, (v16u8)_2)`.
  **L1968 CN**: 以 `(__m128i)__builtin_lsx_vmuh_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。

### Lines 1969-1992

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmuh_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmuh_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmuh_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmuh_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmuh_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmuh_du((v2u64)_1, (v2u64)_2);
}

#define __lsx_vsllwil_h_b(/*__m128i*/ _1, /*ui3*/ _2)                          \
  ((__m128i)__builtin_lsx_vsllwil_h_b((v16i8)(_1), (_2)))

#define __lsx_vsllwil_w_h(/*__m128i*/ _1, /*ui4*/ _2)                          \
````
- **L1969 EN**: Closes the current lexical scope or compound statement.
  **L1969 CN**: 结束当前词法作用域或复合语句块。
- **L1970 EN**: Blank line separating nearby declarations or logic blocks.
  **L1970 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1971 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1971 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1972 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1972 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1973 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmuh_hu(__m128i _1, __m128i _2) {`.
  **L1973 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmuh_hu(__m128i _1, __m128i _2) {`。
- **L1974 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmuh_hu((v8u16)_1, (v8u16)_2)`.
  **L1974 CN**: 以 `(__m128i)__builtin_lsx_vmuh_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L1975 EN**: Closes the current lexical scope or compound statement.
  **L1975 CN**: 结束当前词法作用域或复合语句块。
- **L1976 EN**: Blank line separating nearby declarations or logic blocks.
  **L1976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1977 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1977 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1978 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1978 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1979 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmuh_wu(__m128i _1, __m128i _2) {`.
  **L1979 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmuh_wu(__m128i _1, __m128i _2) {`。
- **L1980 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmuh_wu((v4u32)_1, (v4u32)_2)`.
  **L1980 CN**: 以 `(__m128i)__builtin_lsx_vmuh_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L1981 EN**: Closes the current lexical scope or compound statement.
  **L1981 CN**: 结束当前词法作用域或复合语句块。
- **L1982 EN**: Blank line separating nearby declarations or logic blocks.
  **L1982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1983 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L1983 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L1984 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L1984 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L1985 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmuh_du(__m128i _1, __m128i _2) {`.
  **L1985 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmuh_du(__m128i _1, __m128i _2) {`。
- **L1986 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmuh_du((v2u64)_1, (v2u64)_2)`.
  **L1986 CN**: 以 `(__m128i)__builtin_lsx_vmuh_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L1987 EN**: Closes the current lexical scope or compound statement.
  **L1987 CN**: 结束当前词法作用域或复合语句块。
- **L1988 EN**: Blank line separating nearby declarations or logic blocks.
  **L1988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1989 EN**: Defines macro `__lsx_vsllwil_h_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1989 CN**: 定义宏 `__lsx_vsllwil_h_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L1990 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsllwil_h_b`.
  **L1990 CN**: 继续与可调用符号 `__builtin_lsx_vsllwil_h_b` 相关的逻辑。
- **L1991 EN**: Blank line separating nearby declarations or logic blocks.
  **L1991 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1992 EN**: Defines macro `__lsx_vsllwil_w_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1992 CN**: 定义宏 `__lsx_vsllwil_w_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 1993-2016

````c
  ((__m128i)__builtin_lsx_vsllwil_w_h((v8i16)(_1), (_2)))

#define __lsx_vsllwil_d_w(/*__m128i*/ _1, /*ui5*/ _2)                          \
  ((__m128i)__builtin_lsx_vsllwil_d_w((v4i32)(_1), (_2)))

#define __lsx_vsllwil_hu_bu(/*__m128i*/ _1, /*ui3*/ _2)                        \
  ((__m128i)__builtin_lsx_vsllwil_hu_bu((v16u8)(_1), (_2)))

#define __lsx_vsllwil_wu_hu(/*__m128i*/ _1, /*ui4*/ _2)                        \
  ((__m128i)__builtin_lsx_vsllwil_wu_hu((v8u16)(_1), (_2)))

#define __lsx_vsllwil_du_wu(/*__m128i*/ _1, /*ui5*/ _2)                        \
  ((__m128i)__builtin_lsx_vsllwil_du_wu((v4u32)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsran_b_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsran_b_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsran_h_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsran_h_w((v4i32)_1, (v4i32)_2);
````
- **L1993 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsllwil_w_h`.
  **L1993 CN**: 继续与可调用符号 `__builtin_lsx_vsllwil_w_h` 相关的逻辑。
- **L1994 EN**: Blank line separating nearby declarations or logic blocks.
  **L1994 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1995 EN**: Defines macro `__lsx_vsllwil_d_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1995 CN**: 定义宏 `__lsx_vsllwil_d_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L1996 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsllwil_d_w`.
  **L1996 CN**: 继续与可调用符号 `__builtin_lsx_vsllwil_d_w` 相关的逻辑。
- **L1997 EN**: Blank line separating nearby declarations or logic blocks.
  **L1997 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1998 EN**: Defines macro `__lsx_vsllwil_hu_bu(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L1998 CN**: 定义宏 `__lsx_vsllwil_hu_bu(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L1999 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsllwil_hu_bu`.
  **L1999 CN**: 继续与可调用符号 `__builtin_lsx_vsllwil_hu_bu` 相关的逻辑。
- **L2000 EN**: Blank line separating nearby declarations or logic blocks.
  **L2000 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2001 EN**: Defines macro `__lsx_vsllwil_wu_hu(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2001 CN**: 定义宏 `__lsx_vsllwil_wu_hu(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L2002 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsllwil_wu_hu`.
  **L2002 CN**: 继续与可调用符号 `__builtin_lsx_vsllwil_wu_hu` 相关的逻辑。
- **L2003 EN**: Blank line separating nearby declarations or logic blocks.
  **L2003 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2004 EN**: Defines macro `__lsx_vsllwil_du_wu(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2004 CN**: 定义宏 `__lsx_vsllwil_du_wu(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L2005 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsllwil_du_wu`.
  **L2005 CN**: 继续与可调用符号 `__builtin_lsx_vsllwil_du_wu` 相关的逻辑。
- **L2006 EN**: Blank line separating nearby declarations or logic blocks.
  **L2006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2007 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2007 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2008 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2008 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2009 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsran_b_h(__m128i _1, __m128i _2) {`.
  **L2009 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsran_b_h(__m128i _1, __m128i _2) {`。
- **L2010 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsran_b_h((v8i16)_1, (v8i16)_2)`.
  **L2010 CN**: 以 `(__m128i)__builtin_lsx_vsran_b_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2011 EN**: Closes the current lexical scope or compound statement.
  **L2011 CN**: 结束当前词法作用域或复合语句块。
- **L2012 EN**: Blank line separating nearby declarations or logic blocks.
  **L2012 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2013 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2013 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2014 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2014 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2015 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsran_h_w(__m128i _1, __m128i _2) {`.
  **L2015 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsran_h_w(__m128i _1, __m128i _2) {`。
- **L2016 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsran_h_w((v4i32)_1, (v4i32)_2)`.
  **L2016 CN**: 以 `(__m128i)__builtin_lsx_vsran_h_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。

### Lines 2017-2040

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsran_w_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsran_w_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssran_b_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssran_b_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssran_h_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssran_h_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssran_w_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssran_w_d((v2i64)_1, (v2i64)_2);
````
- **L2017 EN**: Closes the current lexical scope or compound statement.
  **L2017 CN**: 结束当前词法作用域或复合语句块。
- **L2018 EN**: Blank line separating nearby declarations or logic blocks.
  **L2018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2019 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2019 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2020 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2020 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2021 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsran_w_d(__m128i _1, __m128i _2) {`.
  **L2021 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsran_w_d(__m128i _1, __m128i _2) {`。
- **L2022 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsran_w_d((v2i64)_1, (v2i64)_2)`.
  **L2022 CN**: 以 `(__m128i)__builtin_lsx_vsran_w_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2023 EN**: Closes the current lexical scope or compound statement.
  **L2023 CN**: 结束当前词法作用域或复合语句块。
- **L2024 EN**: Blank line separating nearby declarations or logic blocks.
  **L2024 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2025 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2025 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2026 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2026 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2027 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssran_b_h(__m128i _1, __m128i _2) {`.
  **L2027 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssran_b_h(__m128i _1, __m128i _2) {`。
- **L2028 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssran_b_h((v8i16)_1, (v8i16)_2)`.
  **L2028 CN**: 以 `(__m128i)__builtin_lsx_vssran_b_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2029 EN**: Closes the current lexical scope or compound statement.
  **L2029 CN**: 结束当前词法作用域或复合语句块。
- **L2030 EN**: Blank line separating nearby declarations or logic blocks.
  **L2030 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2031 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2031 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2032 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2032 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2033 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssran_h_w(__m128i _1, __m128i _2) {`.
  **L2033 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssran_h_w(__m128i _1, __m128i _2) {`。
- **L2034 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssran_h_w((v4i32)_1, (v4i32)_2)`.
  **L2034 CN**: 以 `(__m128i)__builtin_lsx_vssran_h_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2035 EN**: Closes the current lexical scope or compound statement.
  **L2035 CN**: 结束当前词法作用域或复合语句块。
- **L2036 EN**: Blank line separating nearby declarations or logic blocks.
  **L2036 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2037 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2037 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2038 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2038 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2039 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssran_w_d(__m128i _1, __m128i _2) {`.
  **L2039 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssran_w_d(__m128i _1, __m128i _2) {`。
- **L2040 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssran_w_d((v2i64)_1, (v2i64)_2)`.
  **L2040 CN**: 以 `(__m128i)__builtin_lsx_vssran_w_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。

### Lines 2041-2064

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssran_bu_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssran_bu_h((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssran_hu_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssran_hu_w((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssran_wu_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssran_wu_d((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrarn_b_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrarn_b_h((v8i16)_1, (v8i16)_2);
````
- **L2041 EN**: Closes the current lexical scope or compound statement.
  **L2041 CN**: 结束当前词法作用域或复合语句块。
- **L2042 EN**: Blank line separating nearby declarations or logic blocks.
  **L2042 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2043 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2043 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2044 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2044 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2045 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssran_bu_h(__m128i _1, __m128i _2) {`.
  **L2045 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssran_bu_h(__m128i _1, __m128i _2) {`。
- **L2046 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssran_bu_h((v8u16)_1, (v8u16)_2)`.
  **L2046 CN**: 以 `(__m128i)__builtin_lsx_vssran_bu_h((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2047 EN**: Closes the current lexical scope or compound statement.
  **L2047 CN**: 结束当前词法作用域或复合语句块。
- **L2048 EN**: Blank line separating nearby declarations or logic blocks.
  **L2048 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2049 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2049 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2050 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2050 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2051 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssran_hu_w(__m128i _1, __m128i _2) {`.
  **L2051 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssran_hu_w(__m128i _1, __m128i _2) {`。
- **L2052 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssran_hu_w((v4u32)_1, (v4u32)_2)`.
  **L2052 CN**: 以 `(__m128i)__builtin_lsx_vssran_hu_w((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2053 EN**: Closes the current lexical scope or compound statement.
  **L2053 CN**: 结束当前词法作用域或复合语句块。
- **L2054 EN**: Blank line separating nearby declarations or logic blocks.
  **L2054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2055 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2055 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2056 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2056 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2057 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssran_wu_d(__m128i _1, __m128i _2) {`.
  **L2057 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssran_wu_d(__m128i _1, __m128i _2) {`。
- **L2058 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssran_wu_d((v2u64)_1, (v2u64)_2)`.
  **L2058 CN**: 以 `(__m128i)__builtin_lsx_vssran_wu_d((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2059 EN**: Closes the current lexical scope or compound statement.
  **L2059 CN**: 结束当前词法作用域或复合语句块。
- **L2060 EN**: Blank line separating nearby declarations or logic blocks.
  **L2060 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2061 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2061 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2062 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2062 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2063 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrarn_b_h(__m128i _1, __m128i _2) {`.
  **L2063 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrarn_b_h(__m128i _1, __m128i _2) {`。
- **L2064 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrarn_b_h((v8i16)_1, (v8i16)_2)`.
  **L2064 CN**: 以 `(__m128i)__builtin_lsx_vsrarn_b_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。

### Lines 2065-2088

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrarn_h_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrarn_h_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrarn_w_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrarn_w_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrarn_b_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrarn_b_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrarn_h_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrarn_h_w((v4i32)_1, (v4i32)_2);
````
- **L2065 EN**: Closes the current lexical scope or compound statement.
  **L2065 CN**: 结束当前词法作用域或复合语句块。
- **L2066 EN**: Blank line separating nearby declarations or logic blocks.
  **L2066 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2067 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2067 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2068 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2068 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2069 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrarn_h_w(__m128i _1, __m128i _2) {`.
  **L2069 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrarn_h_w(__m128i _1, __m128i _2) {`。
- **L2070 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrarn_h_w((v4i32)_1, (v4i32)_2)`.
  **L2070 CN**: 以 `(__m128i)__builtin_lsx_vsrarn_h_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2071 EN**: Closes the current lexical scope or compound statement.
  **L2071 CN**: 结束当前词法作用域或复合语句块。
- **L2072 EN**: Blank line separating nearby declarations or logic blocks.
  **L2072 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2073 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2073 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2074 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2074 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2075 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrarn_w_d(__m128i _1, __m128i _2) {`.
  **L2075 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrarn_w_d(__m128i _1, __m128i _2) {`。
- **L2076 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrarn_w_d((v2i64)_1, (v2i64)_2)`.
  **L2076 CN**: 以 `(__m128i)__builtin_lsx_vsrarn_w_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2077 EN**: Closes the current lexical scope or compound statement.
  **L2077 CN**: 结束当前词法作用域或复合语句块。
- **L2078 EN**: Blank line separating nearby declarations or logic blocks.
  **L2078 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2079 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2079 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2080 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2080 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2081 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrarn_b_h(__m128i _1, __m128i _2) {`.
  **L2081 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrarn_b_h(__m128i _1, __m128i _2) {`。
- **L2082 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrarn_b_h((v8i16)_1, (v8i16)_2)`.
  **L2082 CN**: 以 `(__m128i)__builtin_lsx_vssrarn_b_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2083 EN**: Closes the current lexical scope or compound statement.
  **L2083 CN**: 结束当前词法作用域或复合语句块。
- **L2084 EN**: Blank line separating nearby declarations or logic blocks.
  **L2084 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2085 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2085 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2086 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2086 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2087 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrarn_h_w(__m128i _1, __m128i _2) {`.
  **L2087 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrarn_h_w(__m128i _1, __m128i _2) {`。
- **L2088 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrarn_h_w((v4i32)_1, (v4i32)_2)`.
  **L2088 CN**: 以 `(__m128i)__builtin_lsx_vssrarn_h_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。

### Lines 2089-2112

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrarn_w_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrarn_w_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrarn_bu_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrarn_bu_h((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrarn_hu_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrarn_hu_w((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrarn_wu_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrarn_wu_d((v2u64)_1, (v2u64)_2);
````
- **L2089 EN**: Closes the current lexical scope or compound statement.
  **L2089 CN**: 结束当前词法作用域或复合语句块。
- **L2090 EN**: Blank line separating nearby declarations or logic blocks.
  **L2090 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2091 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2091 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2092 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2092 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2093 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrarn_w_d(__m128i _1, __m128i _2) {`.
  **L2093 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrarn_w_d(__m128i _1, __m128i _2) {`。
- **L2094 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrarn_w_d((v2i64)_1, (v2i64)_2)`.
  **L2094 CN**: 以 `(__m128i)__builtin_lsx_vssrarn_w_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2095 EN**: Closes the current lexical scope or compound statement.
  **L2095 CN**: 结束当前词法作用域或复合语句块。
- **L2096 EN**: Blank line separating nearby declarations or logic blocks.
  **L2096 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2097 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2097 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2098 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2098 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2099 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrarn_bu_h(__m128i _1, __m128i _2) {`.
  **L2099 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrarn_bu_h(__m128i _1, __m128i _2) {`。
- **L2100 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrarn_bu_h((v8u16)_1, (v8u16)_2)`.
  **L2100 CN**: 以 `(__m128i)__builtin_lsx_vssrarn_bu_h((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2101 EN**: Closes the current lexical scope or compound statement.
  **L2101 CN**: 结束当前词法作用域或复合语句块。
- **L2102 EN**: Blank line separating nearby declarations or logic blocks.
  **L2102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2103 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2103 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2104 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2104 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2105 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrarn_hu_w(__m128i _1, __m128i _2) {`.
  **L2105 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrarn_hu_w(__m128i _1, __m128i _2) {`。
- **L2106 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrarn_hu_w((v4u32)_1, (v4u32)_2)`.
  **L2106 CN**: 以 `(__m128i)__builtin_lsx_vssrarn_hu_w((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2107 EN**: Closes the current lexical scope or compound statement.
  **L2107 CN**: 结束当前词法作用域或复合语句块。
- **L2108 EN**: Blank line separating nearby declarations or logic blocks.
  **L2108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2109 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2109 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2110 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2110 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2111 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrarn_wu_d(__m128i _1, __m128i _2) {`.
  **L2111 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrarn_wu_d(__m128i _1, __m128i _2) {`。
- **L2112 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrarn_wu_d((v2u64)_1, (v2u64)_2)`.
  **L2112 CN**: 以 `(__m128i)__builtin_lsx_vssrarn_wu_d((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 2113-2136

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrln_b_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrln_b_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrln_h_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrln_h_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrln_w_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrln_w_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrln_bu_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrln_bu_h((v8u16)_1, (v8u16)_2);
````
- **L2113 EN**: Closes the current lexical scope or compound statement.
  **L2113 CN**: 结束当前词法作用域或复合语句块。
- **L2114 EN**: Blank line separating nearby declarations or logic blocks.
  **L2114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2115 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2115 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2116 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2116 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2117 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrln_b_h(__m128i _1, __m128i _2) {`.
  **L2117 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrln_b_h(__m128i _1, __m128i _2) {`。
- **L2118 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrln_b_h((v8i16)_1, (v8i16)_2)`.
  **L2118 CN**: 以 `(__m128i)__builtin_lsx_vsrln_b_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2119 EN**: Closes the current lexical scope or compound statement.
  **L2119 CN**: 结束当前词法作用域或复合语句块。
- **L2120 EN**: Blank line separating nearby declarations or logic blocks.
  **L2120 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2121 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2121 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2122 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2122 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2123 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrln_h_w(__m128i _1, __m128i _2) {`.
  **L2123 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrln_h_w(__m128i _1, __m128i _2) {`。
- **L2124 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrln_h_w((v4i32)_1, (v4i32)_2)`.
  **L2124 CN**: 以 `(__m128i)__builtin_lsx_vsrln_h_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2125 EN**: Closes the current lexical scope or compound statement.
  **L2125 CN**: 结束当前词法作用域或复合语句块。
- **L2126 EN**: Blank line separating nearby declarations or logic blocks.
  **L2126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2127 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2127 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2128 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2128 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2129 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrln_w_d(__m128i _1, __m128i _2) {`.
  **L2129 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrln_w_d(__m128i _1, __m128i _2) {`。
- **L2130 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrln_w_d((v2i64)_1, (v2i64)_2)`.
  **L2130 CN**: 以 `(__m128i)__builtin_lsx_vsrln_w_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2131 EN**: Closes the current lexical scope or compound statement.
  **L2131 CN**: 结束当前词法作用域或复合语句块。
- **L2132 EN**: Blank line separating nearby declarations or logic blocks.
  **L2132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2133 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2133 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2134 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2134 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2135 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrln_bu_h(__m128i _1, __m128i _2) {`.
  **L2135 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrln_bu_h(__m128i _1, __m128i _2) {`。
- **L2136 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrln_bu_h((v8u16)_1, (v8u16)_2)`.
  **L2136 CN**: 以 `(__m128i)__builtin_lsx_vssrln_bu_h((v8u16)_1, (v8u16)_2)` 从当前函数返回。

### Lines 2137-2160

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrln_hu_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrln_hu_w((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrln_wu_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrln_wu_d((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrlrn_b_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrlrn_b_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrlrn_h_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrlrn_h_w((v4i32)_1, (v4i32)_2);
````
- **L2137 EN**: Closes the current lexical scope or compound statement.
  **L2137 CN**: 结束当前词法作用域或复合语句块。
- **L2138 EN**: Blank line separating nearby declarations or logic blocks.
  **L2138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2139 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2139 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2140 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2140 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2141 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrln_hu_w(__m128i _1, __m128i _2) {`.
  **L2141 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrln_hu_w(__m128i _1, __m128i _2) {`。
- **L2142 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrln_hu_w((v4u32)_1, (v4u32)_2)`.
  **L2142 CN**: 以 `(__m128i)__builtin_lsx_vssrln_hu_w((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2143 EN**: Closes the current lexical scope or compound statement.
  **L2143 CN**: 结束当前词法作用域或复合语句块。
- **L2144 EN**: Blank line separating nearby declarations or logic blocks.
  **L2144 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2145 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2145 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2146 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2146 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2147 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrln_wu_d(__m128i _1, __m128i _2) {`.
  **L2147 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrln_wu_d(__m128i _1, __m128i _2) {`。
- **L2148 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrln_wu_d((v2u64)_1, (v2u64)_2)`.
  **L2148 CN**: 以 `(__m128i)__builtin_lsx_vssrln_wu_d((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2149 EN**: Closes the current lexical scope or compound statement.
  **L2149 CN**: 结束当前词法作用域或复合语句块。
- **L2150 EN**: Blank line separating nearby declarations or logic blocks.
  **L2150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2151 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2151 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2152 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2152 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2153 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrlrn_b_h(__m128i _1, __m128i _2) {`.
  **L2153 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrlrn_b_h(__m128i _1, __m128i _2) {`。
- **L2154 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrlrn_b_h((v8i16)_1, (v8i16)_2)`.
  **L2154 CN**: 以 `(__m128i)__builtin_lsx_vsrlrn_b_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2155 EN**: Closes the current lexical scope or compound statement.
  **L2155 CN**: 结束当前词法作用域或复合语句块。
- **L2156 EN**: Blank line separating nearby declarations or logic blocks.
  **L2156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2157 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2157 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2158 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2158 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2159 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrlrn_h_w(__m128i _1, __m128i _2) {`.
  **L2159 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrlrn_h_w(__m128i _1, __m128i _2) {`。
- **L2160 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrlrn_h_w((v4i32)_1, (v4i32)_2)`.
  **L2160 CN**: 以 `(__m128i)__builtin_lsx_vsrlrn_h_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。

### Lines 2161-2184

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsrlrn_w_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsrlrn_w_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrlrn_bu_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrlrn_bu_h((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrlrn_hu_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrlrn_hu_w((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrlrn_wu_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrlrn_wu_d((v2u64)_1, (v2u64)_2);
````
- **L2161 EN**: Closes the current lexical scope or compound statement.
  **L2161 CN**: 结束当前词法作用域或复合语句块。
- **L2162 EN**: Blank line separating nearby declarations or logic blocks.
  **L2162 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2163 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2163 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2164 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2164 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2165 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsrlrn_w_d(__m128i _1, __m128i _2) {`.
  **L2165 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsrlrn_w_d(__m128i _1, __m128i _2) {`。
- **L2166 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsrlrn_w_d((v2i64)_1, (v2i64)_2)`.
  **L2166 CN**: 以 `(__m128i)__builtin_lsx_vsrlrn_w_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2167 EN**: Closes the current lexical scope or compound statement.
  **L2167 CN**: 结束当前词法作用域或复合语句块。
- **L2168 EN**: Blank line separating nearby declarations or logic blocks.
  **L2168 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2169 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2169 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2170 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2170 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2171 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrlrn_bu_h(__m128i _1, __m128i _2) {`.
  **L2171 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrlrn_bu_h(__m128i _1, __m128i _2) {`。
- **L2172 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrlrn_bu_h((v8u16)_1, (v8u16)_2)`.
  **L2172 CN**: 以 `(__m128i)__builtin_lsx_vssrlrn_bu_h((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2173 EN**: Closes the current lexical scope or compound statement.
  **L2173 CN**: 结束当前词法作用域或复合语句块。
- **L2174 EN**: Blank line separating nearby declarations or logic blocks.
  **L2174 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2175 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2175 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2176 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2176 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2177 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrlrn_hu_w(__m128i _1, __m128i _2) {`.
  **L2177 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrlrn_hu_w(__m128i _1, __m128i _2) {`。
- **L2178 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrlrn_hu_w((v4u32)_1, (v4u32)_2)`.
  **L2178 CN**: 以 `(__m128i)__builtin_lsx_vssrlrn_hu_w((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2179 EN**: Closes the current lexical scope or compound statement.
  **L2179 CN**: 结束当前词法作用域或复合语句块。
- **L2180 EN**: Blank line separating nearby declarations or logic blocks.
  **L2180 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2181 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2181 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2182 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2182 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2183 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrlrn_wu_d(__m128i _1, __m128i _2) {`.
  **L2183 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrlrn_wu_d(__m128i _1, __m128i _2) {`。
- **L2184 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrlrn_wu_d((v2u64)_1, (v2u64)_2)`.
  **L2184 CN**: 以 `(__m128i)__builtin_lsx_vssrlrn_wu_d((v2u64)_1, (v2u64)_2)` 从当前函数返回。

### Lines 2185-2208

````c
}

#define __lsx_vfrstpi_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)            \
  ((__m128i)__builtin_lsx_vfrstpi_b((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vfrstpi_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)            \
  ((__m128i)__builtin_lsx_vfrstpi_h((v8i16)(_1), (v8i16)(_2), (_3)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfrstp_b(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vfrstp_b((v16i8)_1, (v16i8)_2, (v16i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfrstp_h(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vfrstp_h((v8i16)_1, (v8i16)_2, (v8i16)_3);
}

#define __lsx_vshuf4i_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)            \
  ((__m128i)__builtin_lsx_vshuf4i_d((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vbsrl_v(/*__m128i*/ _1, /*ui5*/ _2)                              \
````
- **L2185 EN**: Closes the current lexical scope or compound statement.
  **L2185 CN**: 结束当前词法作用域或复合语句块。
- **L2186 EN**: Blank line separating nearby declarations or logic blocks.
  **L2186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2187 EN**: Defines macro `__lsx_vfrstpi_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2187 CN**: 定义宏 `__lsx_vfrstpi_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L2188 EN**: Continues logic associated with callable symbol `__builtin_lsx_vfrstpi_b`.
  **L2188 CN**: 继续与可调用符号 `__builtin_lsx_vfrstpi_b` 相关的逻辑。
- **L2189 EN**: Blank line separating nearby declarations or logic blocks.
  **L2189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2190 EN**: Defines macro `__lsx_vfrstpi_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2190 CN**: 定义宏 `__lsx_vfrstpi_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L2191 EN**: Continues logic associated with callable symbol `__builtin_lsx_vfrstpi_h`.
  **L2191 CN**: 继续与可调用符号 `__builtin_lsx_vfrstpi_h` 相关的逻辑。
- **L2192 EN**: Blank line separating nearby declarations or logic blocks.
  **L2192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2193 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2193 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2194 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2194 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2195 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrstp_b(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2195 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrstp_b(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2196 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfrstp_b((v16i8)_1, (v16i8)_2, (v16i8)_3)`.
  **L2196 CN**: 以 `(__m128i)__builtin_lsx_vfrstp_b((v16i8)_1, (v16i8)_2, (v16i8)_3)` 从当前函数返回。
- **L2197 EN**: Closes the current lexical scope or compound statement.
  **L2197 CN**: 结束当前词法作用域或复合语句块。
- **L2198 EN**: Blank line separating nearby declarations or logic blocks.
  **L2198 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2199 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2199 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2200 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2200 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2201 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrstp_h(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2201 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrstp_h(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2202 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfrstp_h((v8i16)_1, (v8i16)_2, (v8i16)_3)`.
  **L2202 CN**: 以 `(__m128i)__builtin_lsx_vfrstp_h((v8i16)_1, (v8i16)_2, (v8i16)_3)` 从当前函数返回。
- **L2203 EN**: Closes the current lexical scope or compound statement.
  **L2203 CN**: 结束当前词法作用域或复合语句块。
- **L2204 EN**: Blank line separating nearby declarations or logic blocks.
  **L2204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2205 EN**: Defines macro `__lsx_vshuf4i_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2205 CN**: 定义宏 `__lsx_vshuf4i_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2206 EN**: Continues logic associated with callable symbol `__builtin_lsx_vshuf4i_d`.
  **L2206 CN**: 继续与可调用符号 `__builtin_lsx_vshuf4i_d` 相关的逻辑。
- **L2207 EN**: Blank line separating nearby declarations or logic blocks.
  **L2207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2208 EN**: Defines macro `__lsx_vbsrl_v(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2208 CN**: 定义宏 `__lsx_vbsrl_v(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。

### Lines 2209-2232

````c
  ((__m128i)__builtin_lsx_vbsrl_v((v16i8)(_1), (_2)))

#define __lsx_vbsll_v(/*__m128i*/ _1, /*ui5*/ _2)                              \
  ((__m128i)__builtin_lsx_vbsll_v((v16i8)(_1), (_2)))

#define __lsx_vextrins_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)           \
  ((__m128i)__builtin_lsx_vextrins_b((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vextrins_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)           \
  ((__m128i)__builtin_lsx_vextrins_h((v8i16)(_1), (v8i16)(_2), (_3)))

#define __lsx_vextrins_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)           \
  ((__m128i)__builtin_lsx_vextrins_w((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vextrins_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)           \
  ((__m128i)__builtin_lsx_vextrins_d((v2i64)(_1), (v2i64)(_2), (_3)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmskltz_b(__m128i _1) {
  return (__m128i)__builtin_lsx_vmskltz_b((v16i8)_1);
}

extern __inline
````
- **L2209 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbsrl_v`.
  **L2209 CN**: 继续与可调用符号 `__builtin_lsx_vbsrl_v` 相关的逻辑。
- **L2210 EN**: Blank line separating nearby declarations or logic blocks.
  **L2210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2211 EN**: Defines macro `__lsx_vbsll_v(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L2211 CN**: 定义宏 `__lsx_vbsll_v(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L2212 EN**: Continues logic associated with callable symbol `__builtin_lsx_vbsll_v`.
  **L2212 CN**: 继续与可调用符号 `__builtin_lsx_vbsll_v` 相关的逻辑。
- **L2213 EN**: Blank line separating nearby declarations or logic blocks.
  **L2213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2214 EN**: Defines macro `__lsx_vextrins_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2214 CN**: 定义宏 `__lsx_vextrins_b(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2215 EN**: Continues logic associated with callable symbol `__builtin_lsx_vextrins_b`.
  **L2215 CN**: 继续与可调用符号 `__builtin_lsx_vextrins_b` 相关的逻辑。
- **L2216 EN**: Blank line separating nearby declarations or logic blocks.
  **L2216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2217 EN**: Defines macro `__lsx_vextrins_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2217 CN**: 定义宏 `__lsx_vextrins_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2218 EN**: Continues logic associated with callable symbol `__builtin_lsx_vextrins_h`.
  **L2218 CN**: 继续与可调用符号 `__builtin_lsx_vextrins_h` 相关的逻辑。
- **L2219 EN**: Blank line separating nearby declarations or logic blocks.
  **L2219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2220 EN**: Defines macro `__lsx_vextrins_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2220 CN**: 定义宏 `__lsx_vextrins_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2221 EN**: Continues logic associated with callable symbol `__builtin_lsx_vextrins_w`.
  **L2221 CN**: 继续与可调用符号 `__builtin_lsx_vextrins_w` 相关的逻辑。
- **L2222 EN**: Blank line separating nearby declarations or logic blocks.
  **L2222 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2223 EN**: Defines macro `__lsx_vextrins_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L2223 CN**: 定义宏 `__lsx_vextrins_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L2224 EN**: Continues logic associated with callable symbol `__builtin_lsx_vextrins_d`.
  **L2224 CN**: 继续与可调用符号 `__builtin_lsx_vextrins_d` 相关的逻辑。
- **L2225 EN**: Blank line separating nearby declarations or logic blocks.
  **L2225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2226 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2226 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2227 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2227 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2228 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmskltz_b(__m128i _1) {`.
  **L2228 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmskltz_b(__m128i _1) {`。
- **L2229 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmskltz_b((v16i8)_1)`.
  **L2229 CN**: 以 `(__m128i)__builtin_lsx_vmskltz_b((v16i8)_1)` 从当前函数返回。
- **L2230 EN**: Closes the current lexical scope or compound statement.
  **L2230 CN**: 结束当前词法作用域或复合语句块。
- **L2231 EN**: Blank line separating nearby declarations or logic blocks.
  **L2231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2232 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2232 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2233-2256

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmskltz_h(__m128i _1) {
  return (__m128i)__builtin_lsx_vmskltz_h((v8i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmskltz_w(__m128i _1) {
  return (__m128i)__builtin_lsx_vmskltz_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmskltz_d(__m128i _1) {
  return (__m128i)__builtin_lsx_vmskltz_d((v2i64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsigncov_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsigncov_b((v16i8)_1, (v16i8)_2);
}

extern __inline
````
- **L2233 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2233 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2234 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmskltz_h(__m128i _1) {`.
  **L2234 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmskltz_h(__m128i _1) {`。
- **L2235 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmskltz_h((v8i16)_1)`.
  **L2235 CN**: 以 `(__m128i)__builtin_lsx_vmskltz_h((v8i16)_1)` 从当前函数返回。
- **L2236 EN**: Closes the current lexical scope or compound statement.
  **L2236 CN**: 结束当前词法作用域或复合语句块。
- **L2237 EN**: Blank line separating nearby declarations or logic blocks.
  **L2237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2238 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2238 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2239 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2239 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2240 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmskltz_w(__m128i _1) {`.
  **L2240 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmskltz_w(__m128i _1) {`。
- **L2241 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmskltz_w((v4i32)_1)`.
  **L2241 CN**: 以 `(__m128i)__builtin_lsx_vmskltz_w((v4i32)_1)` 从当前函数返回。
- **L2242 EN**: Closes the current lexical scope or compound statement.
  **L2242 CN**: 结束当前词法作用域或复合语句块。
- **L2243 EN**: Blank line separating nearby declarations or logic blocks.
  **L2243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2244 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2244 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2245 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2245 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2246 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmskltz_d(__m128i _1) {`.
  **L2246 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmskltz_d(__m128i _1) {`。
- **L2247 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmskltz_d((v2i64)_1)`.
  **L2247 CN**: 以 `(__m128i)__builtin_lsx_vmskltz_d((v2i64)_1)` 从当前函数返回。
- **L2248 EN**: Closes the current lexical scope or compound statement.
  **L2248 CN**: 结束当前词法作用域或复合语句块。
- **L2249 EN**: Blank line separating nearby declarations or logic blocks.
  **L2249 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2250 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2250 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2251 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2251 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2252 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsigncov_b(__m128i _1, __m128i _2) {`.
  **L2252 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsigncov_b(__m128i _1, __m128i _2) {`。
- **L2253 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsigncov_b((v16i8)_1, (v16i8)_2)`.
  **L2253 CN**: 以 `(__m128i)__builtin_lsx_vsigncov_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L2254 EN**: Closes the current lexical scope or compound statement.
  **L2254 CN**: 结束当前词法作用域或复合语句块。
- **L2255 EN**: Blank line separating nearby declarations or logic blocks.
  **L2255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2256 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2256 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2257-2280

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsigncov_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsigncov_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsigncov_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsigncov_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsigncov_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsigncov_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfmadd_s(__m128 _1, __m128 _2, __m128 _3) {
  return (__m128)__builtin_lsx_vfmadd_s((v4f32)_1, (v4f32)_2, (v4f32)_3);
}

extern __inline
````
- **L2257 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2257 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2258 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsigncov_h(__m128i _1, __m128i _2) {`.
  **L2258 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsigncov_h(__m128i _1, __m128i _2) {`。
- **L2259 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsigncov_h((v8i16)_1, (v8i16)_2)`.
  **L2259 CN**: 以 `(__m128i)__builtin_lsx_vsigncov_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2260 EN**: Closes the current lexical scope or compound statement.
  **L2260 CN**: 结束当前词法作用域或复合语句块。
- **L2261 EN**: Blank line separating nearby declarations or logic blocks.
  **L2261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2262 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2262 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2263 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2263 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2264 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsigncov_w(__m128i _1, __m128i _2) {`.
  **L2264 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsigncov_w(__m128i _1, __m128i _2) {`。
- **L2265 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsigncov_w((v4i32)_1, (v4i32)_2)`.
  **L2265 CN**: 以 `(__m128i)__builtin_lsx_vsigncov_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2266 EN**: Closes the current lexical scope or compound statement.
  **L2266 CN**: 结束当前词法作用域或复合语句块。
- **L2267 EN**: Blank line separating nearby declarations or logic blocks.
  **L2267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2268 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2268 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2269 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2269 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2270 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsigncov_d(__m128i _1, __m128i _2) {`.
  **L2270 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsigncov_d(__m128i _1, __m128i _2) {`。
- **L2271 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsigncov_d((v2i64)_1, (v2i64)_2)`.
  **L2271 CN**: 以 `(__m128i)__builtin_lsx_vsigncov_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2272 EN**: Closes the current lexical scope or compound statement.
  **L2272 CN**: 结束当前词法作用域或复合语句块。
- **L2273 EN**: Blank line separating nearby declarations or logic blocks.
  **L2273 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2274 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2274 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2275 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2275 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2276 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmadd_s(__m128 _1, __m128 _2, __m128 _3) {`.
  **L2276 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmadd_s(__m128 _1, __m128 _2, __m128 _3) {`。
- **L2277 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfmadd_s((v4f32)_1, (v4f32)_2, (v4f32)_3)`.
  **L2277 CN**: 以 `(__m128)__builtin_lsx_vfmadd_s((v4f32)_1, (v4f32)_2, (v4f32)_3)` 从当前函数返回。
- **L2278 EN**: Closes the current lexical scope or compound statement.
  **L2278 CN**: 结束当前词法作用域或复合语句块。
- **L2279 EN**: Blank line separating nearby declarations or logic blocks.
  **L2279 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2280 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2280 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2281-2304

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfmadd_d(__m128d _1, __m128d _2, __m128d _3) {
  return (__m128d)__builtin_lsx_vfmadd_d((v2f64)_1, (v2f64)_2, (v2f64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfmsub_s(__m128 _1, __m128 _2, __m128 _3) {
  return (__m128)__builtin_lsx_vfmsub_s((v4f32)_1, (v4f32)_2, (v4f32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfmsub_d(__m128d _1, __m128d _2, __m128d _3) {
  return (__m128d)__builtin_lsx_vfmsub_d((v2f64)_1, (v2f64)_2, (v2f64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfnmadd_s(__m128 _1, __m128 _2, __m128 _3) {
  return (__m128)__builtin_lsx_vfnmadd_s((v4f32)_1, (v4f32)_2, (v4f32)_3);
}

extern __inline
````
- **L2281 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2281 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2282 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmadd_d(__m128d _1, __m128d _2, __m128d _3) {`.
  **L2282 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmadd_d(__m128d _1, __m128d _2, __m128d _3) {`。
- **L2283 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfmadd_d((v2f64)_1, (v2f64)_2, (v2f64)_3)`.
  **L2283 CN**: 以 `(__m128d)__builtin_lsx_vfmadd_d((v2f64)_1, (v2f64)_2, (v2f64)_3)` 从当前函数返回。
- **L2284 EN**: Closes the current lexical scope or compound statement.
  **L2284 CN**: 结束当前词法作用域或复合语句块。
- **L2285 EN**: Blank line separating nearby declarations or logic blocks.
  **L2285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2286 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2286 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2287 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2287 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2288 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmsub_s(__m128 _1, __m128 _2, __m128 _3) {`.
  **L2288 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmsub_s(__m128 _1, __m128 _2, __m128 _3) {`。
- **L2289 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfmsub_s((v4f32)_1, (v4f32)_2, (v4f32)_3)`.
  **L2289 CN**: 以 `(__m128)__builtin_lsx_vfmsub_s((v4f32)_1, (v4f32)_2, (v4f32)_3)` 从当前函数返回。
- **L2290 EN**: Closes the current lexical scope or compound statement.
  **L2290 CN**: 结束当前词法作用域或复合语句块。
- **L2291 EN**: Blank line separating nearby declarations or logic blocks.
  **L2291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2292 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2292 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2293 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2293 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2294 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfmsub_d(__m128d _1, __m128d _2, __m128d _3) {`.
  **L2294 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfmsub_d(__m128d _1, __m128d _2, __m128d _3) {`。
- **L2295 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfmsub_d((v2f64)_1, (v2f64)_2, (v2f64)_3)`.
  **L2295 CN**: 以 `(__m128d)__builtin_lsx_vfmsub_d((v2f64)_1, (v2f64)_2, (v2f64)_3)` 从当前函数返回。
- **L2296 EN**: Closes the current lexical scope or compound statement.
  **L2296 CN**: 结束当前词法作用域或复合语句块。
- **L2297 EN**: Blank line separating nearby declarations or logic blocks.
  **L2297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2298 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2298 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2299 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2299 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2300 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfnmadd_s(__m128 _1, __m128 _2, __m128 _3) {`.
  **L2300 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfnmadd_s(__m128 _1, __m128 _2, __m128 _3) {`。
- **L2301 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfnmadd_s((v4f32)_1, (v4f32)_2, (v4f32)_3)`.
  **L2301 CN**: 以 `(__m128)__builtin_lsx_vfnmadd_s((v4f32)_1, (v4f32)_2, (v4f32)_3)` 从当前函数返回。
- **L2302 EN**: Closes the current lexical scope or compound statement.
  **L2302 CN**: 结束当前词法作用域或复合语句块。
- **L2303 EN**: Blank line separating nearby declarations or logic blocks.
  **L2303 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2304 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2304 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2305-2328

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfnmadd_d(__m128d _1, __m128d _2, __m128d _3) {
  return (__m128d)__builtin_lsx_vfnmadd_d((v2f64)_1, (v2f64)_2, (v2f64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfnmsub_s(__m128 _1, __m128 _2, __m128 _3) {
  return (__m128)__builtin_lsx_vfnmsub_s((v4f32)_1, (v4f32)_2, (v4f32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfnmsub_d(__m128d _1, __m128d _2, __m128d _3) {
  return (__m128d)__builtin_lsx_vfnmsub_d((v2f64)_1, (v2f64)_2, (v2f64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrne_w_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrne_w_s((v4f32)_1);
}

extern __inline
````
- **L2305 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2305 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2306 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfnmadd_d(__m128d _1, __m128d _2, __m128d _3) {`.
  **L2306 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfnmadd_d(__m128d _1, __m128d _2, __m128d _3) {`。
- **L2307 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfnmadd_d((v2f64)_1, (v2f64)_2, (v2f64)_3)`.
  **L2307 CN**: 以 `(__m128d)__builtin_lsx_vfnmadd_d((v2f64)_1, (v2f64)_2, (v2f64)_3)` 从当前函数返回。
- **L2308 EN**: Closes the current lexical scope or compound statement.
  **L2308 CN**: 结束当前词法作用域或复合语句块。
- **L2309 EN**: Blank line separating nearby declarations or logic blocks.
  **L2309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2310 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2310 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2311 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2311 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2312 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfnmsub_s(__m128 _1, __m128 _2, __m128 _3) {`.
  **L2312 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfnmsub_s(__m128 _1, __m128 _2, __m128 _3) {`。
- **L2313 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfnmsub_s((v4f32)_1, (v4f32)_2, (v4f32)_3)`.
  **L2313 CN**: 以 `(__m128)__builtin_lsx_vfnmsub_s((v4f32)_1, (v4f32)_2, (v4f32)_3)` 从当前函数返回。
- **L2314 EN**: Closes the current lexical scope or compound statement.
  **L2314 CN**: 结束当前词法作用域或复合语句块。
- **L2315 EN**: Blank line separating nearby declarations or logic blocks.
  **L2315 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2316 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2316 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2317 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2317 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2318 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfnmsub_d(__m128d _1, __m128d _2, __m128d _3) {`.
  **L2318 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfnmsub_d(__m128d _1, __m128d _2, __m128d _3) {`。
- **L2319 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfnmsub_d((v2f64)_1, (v2f64)_2, (v2f64)_3)`.
  **L2319 CN**: 以 `(__m128d)__builtin_lsx_vfnmsub_d((v2f64)_1, (v2f64)_2, (v2f64)_3)` 从当前函数返回。
- **L2320 EN**: Closes the current lexical scope or compound statement.
  **L2320 CN**: 结束当前词法作用域或复合语句块。
- **L2321 EN**: Blank line separating nearby declarations or logic blocks.
  **L2321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2322 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2322 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2323 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2323 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2324 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrne_w_s(__m128 _1) {`.
  **L2324 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrne_w_s(__m128 _1) {`。
- **L2325 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrne_w_s((v4f32)_1)`.
  **L2325 CN**: 以 `(__m128i)__builtin_lsx_vftintrne_w_s((v4f32)_1)` 从当前函数返回。
- **L2326 EN**: Closes the current lexical scope or compound statement.
  **L2326 CN**: 结束当前词法作用域或复合语句块。
- **L2327 EN**: Blank line separating nearby declarations or logic blocks.
  **L2327 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2328 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2328 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2329-2352

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrne_l_d(__m128d _1) {
  return (__m128i)__builtin_lsx_vftintrne_l_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrp_w_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrp_w_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrp_l_d(__m128d _1) {
  return (__m128i)__builtin_lsx_vftintrp_l_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrm_w_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrm_w_s((v4f32)_1);
}

extern __inline
````
- **L2329 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2329 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2330 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrne_l_d(__m128d _1) {`.
  **L2330 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrne_l_d(__m128d _1) {`。
- **L2331 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrne_l_d((v2f64)_1)`.
  **L2331 CN**: 以 `(__m128i)__builtin_lsx_vftintrne_l_d((v2f64)_1)` 从当前函数返回。
- **L2332 EN**: Closes the current lexical scope or compound statement.
  **L2332 CN**: 结束当前词法作用域或复合语句块。
- **L2333 EN**: Blank line separating nearby declarations or logic blocks.
  **L2333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2334 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2334 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2335 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2335 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2336 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrp_w_s(__m128 _1) {`.
  **L2336 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrp_w_s(__m128 _1) {`。
- **L2337 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrp_w_s((v4f32)_1)`.
  **L2337 CN**: 以 `(__m128i)__builtin_lsx_vftintrp_w_s((v4f32)_1)` 从当前函数返回。
- **L2338 EN**: Closes the current lexical scope or compound statement.
  **L2338 CN**: 结束当前词法作用域或复合语句块。
- **L2339 EN**: Blank line separating nearby declarations or logic blocks.
  **L2339 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2340 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2340 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2341 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2341 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2342 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrp_l_d(__m128d _1) {`.
  **L2342 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrp_l_d(__m128d _1) {`。
- **L2343 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrp_l_d((v2f64)_1)`.
  **L2343 CN**: 以 `(__m128i)__builtin_lsx_vftintrp_l_d((v2f64)_1)` 从当前函数返回。
- **L2344 EN**: Closes the current lexical scope or compound statement.
  **L2344 CN**: 结束当前词法作用域或复合语句块。
- **L2345 EN**: Blank line separating nearby declarations or logic blocks.
  **L2345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2346 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2346 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2347 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2347 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2348 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrm_w_s(__m128 _1) {`.
  **L2348 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrm_w_s(__m128 _1) {`。
- **L2349 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrm_w_s((v4f32)_1)`.
  **L2349 CN**: 以 `(__m128i)__builtin_lsx_vftintrm_w_s((v4f32)_1)` 从当前函数返回。
- **L2350 EN**: Closes the current lexical scope or compound statement.
  **L2350 CN**: 结束当前词法作用域或复合语句块。
- **L2351 EN**: Blank line separating nearby declarations or logic blocks.
  **L2351 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2352 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2352 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2353-2376

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrm_l_d(__m128d _1) {
  return (__m128i)__builtin_lsx_vftintrm_l_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftint_w_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vftint_w_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vffint_s_l(__m128i _1, __m128i _2) {
  return (__m128)__builtin_lsx_vffint_s_l((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrz_w_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vftintrz_w_d((v2f64)_1, (v2f64)_2);
}

extern __inline
````
- **L2353 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2353 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2354 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrm_l_d(__m128d _1) {`.
  **L2354 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrm_l_d(__m128d _1) {`。
- **L2355 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrm_l_d((v2f64)_1)`.
  **L2355 CN**: 以 `(__m128i)__builtin_lsx_vftintrm_l_d((v2f64)_1)` 从当前函数返回。
- **L2356 EN**: Closes the current lexical scope or compound statement.
  **L2356 CN**: 结束当前词法作用域或复合语句块。
- **L2357 EN**: Blank line separating nearby declarations or logic blocks.
  **L2357 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2358 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2358 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2359 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2359 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2360 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftint_w_d(__m128d _1, __m128d _2) {`.
  **L2360 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftint_w_d(__m128d _1, __m128d _2) {`。
- **L2361 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftint_w_d((v2f64)_1, (v2f64)_2)`.
  **L2361 CN**: 以 `(__m128i)__builtin_lsx_vftint_w_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L2362 EN**: Closes the current lexical scope or compound statement.
  **L2362 CN**: 结束当前词法作用域或复合语句块。
- **L2363 EN**: Blank line separating nearby declarations or logic blocks.
  **L2363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2364 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2364 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2365 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2365 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2366 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vffint_s_l(__m128i _1, __m128i _2) {`.
  **L2366 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vffint_s_l(__m128i _1, __m128i _2) {`。
- **L2367 EN**: Returns from the current function with `(__m128)__builtin_lsx_vffint_s_l((v2i64)_1, (v2i64)_2)`.
  **L2367 CN**: 以 `(__m128)__builtin_lsx_vffint_s_l((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2368 EN**: Closes the current lexical scope or compound statement.
  **L2368 CN**: 结束当前词法作用域或复合语句块。
- **L2369 EN**: Blank line separating nearby declarations or logic blocks.
  **L2369 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2370 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2370 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2371 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2371 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2372 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrz_w_d(__m128d _1, __m128d _2) {`.
  **L2372 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrz_w_d(__m128d _1, __m128d _2) {`。
- **L2373 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrz_w_d((v2f64)_1, (v2f64)_2)`.
  **L2373 CN**: 以 `(__m128i)__builtin_lsx_vftintrz_w_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L2374 EN**: Closes the current lexical scope or compound statement.
  **L2374 CN**: 结束当前词法作用域或复合语句块。
- **L2375 EN**: Blank line separating nearby declarations or logic blocks.
  **L2375 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2376 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2376 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2377-2400

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrp_w_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vftintrp_w_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrm_w_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vftintrm_w_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrne_w_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vftintrne_w_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintl_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintl_l_s((v4f32)_1);
}

extern __inline
````
- **L2377 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2377 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2378 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrp_w_d(__m128d _1, __m128d _2) {`.
  **L2378 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrp_w_d(__m128d _1, __m128d _2) {`。
- **L2379 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrp_w_d((v2f64)_1, (v2f64)_2)`.
  **L2379 CN**: 以 `(__m128i)__builtin_lsx_vftintrp_w_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L2380 EN**: Closes the current lexical scope or compound statement.
  **L2380 CN**: 结束当前词法作用域或复合语句块。
- **L2381 EN**: Blank line separating nearby declarations or logic blocks.
  **L2381 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2382 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2382 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2383 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2383 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2384 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrm_w_d(__m128d _1, __m128d _2) {`.
  **L2384 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrm_w_d(__m128d _1, __m128d _2) {`。
- **L2385 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrm_w_d((v2f64)_1, (v2f64)_2)`.
  **L2385 CN**: 以 `(__m128i)__builtin_lsx_vftintrm_w_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L2386 EN**: Closes the current lexical scope or compound statement.
  **L2386 CN**: 结束当前词法作用域或复合语句块。
- **L2387 EN**: Blank line separating nearby declarations or logic blocks.
  **L2387 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2388 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2388 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2389 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2389 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2390 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrne_w_d(__m128d _1, __m128d _2) {`.
  **L2390 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrne_w_d(__m128d _1, __m128d _2) {`。
- **L2391 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrne_w_d((v2f64)_1, (v2f64)_2)`.
  **L2391 CN**: 以 `(__m128i)__builtin_lsx_vftintrne_w_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L2392 EN**: Closes the current lexical scope or compound statement.
  **L2392 CN**: 结束当前词法作用域或复合语句块。
- **L2393 EN**: Blank line separating nearby declarations or logic blocks.
  **L2393 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2394 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2394 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2395 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2395 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2396 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintl_l_s(__m128 _1) {`.
  **L2396 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintl_l_s(__m128 _1) {`。
- **L2397 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintl_l_s((v4f32)_1)`.
  **L2397 CN**: 以 `(__m128i)__builtin_lsx_vftintl_l_s((v4f32)_1)` 从当前函数返回。
- **L2398 EN**: Closes the current lexical scope or compound statement.
  **L2398 CN**: 结束当前词法作用域或复合语句块。
- **L2399 EN**: Blank line separating nearby declarations or logic blocks.
  **L2399 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2400 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2400 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2401-2424

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftinth_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftinth_l_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vffinth_d_w(__m128i _1) {
  return (__m128d)__builtin_lsx_vffinth_d_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vffintl_d_w(__m128i _1) {
  return (__m128d)__builtin_lsx_vffintl_d_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrzl_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrzl_l_s((v4f32)_1);
}

extern __inline
````
- **L2401 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2401 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2402 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftinth_l_s(__m128 _1) {`.
  **L2402 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftinth_l_s(__m128 _1) {`。
- **L2403 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftinth_l_s((v4f32)_1)`.
  **L2403 CN**: 以 `(__m128i)__builtin_lsx_vftinth_l_s((v4f32)_1)` 从当前函数返回。
- **L2404 EN**: Closes the current lexical scope or compound statement.
  **L2404 CN**: 结束当前词法作用域或复合语句块。
- **L2405 EN**: Blank line separating nearby declarations or logic blocks.
  **L2405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2406 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2406 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2407 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2407 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2408 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vffinth_d_w(__m128i _1) {`.
  **L2408 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vffinth_d_w(__m128i _1) {`。
- **L2409 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vffinth_d_w((v4i32)_1)`.
  **L2409 CN**: 以 `(__m128d)__builtin_lsx_vffinth_d_w((v4i32)_1)` 从当前函数返回。
- **L2410 EN**: Closes the current lexical scope or compound statement.
  **L2410 CN**: 结束当前词法作用域或复合语句块。
- **L2411 EN**: Blank line separating nearby declarations or logic blocks.
  **L2411 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2412 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2412 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2413 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2413 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2414 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vffintl_d_w(__m128i _1) {`.
  **L2414 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vffintl_d_w(__m128i _1) {`。
- **L2415 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vffintl_d_w((v4i32)_1)`.
  **L2415 CN**: 以 `(__m128d)__builtin_lsx_vffintl_d_w((v4i32)_1)` 从当前函数返回。
- **L2416 EN**: Closes the current lexical scope or compound statement.
  **L2416 CN**: 结束当前词法作用域或复合语句块。
- **L2417 EN**: Blank line separating nearby declarations or logic blocks.
  **L2417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2418 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2418 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2419 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2419 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2420 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrzl_l_s(__m128 _1) {`.
  **L2420 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrzl_l_s(__m128 _1) {`。
- **L2421 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrzl_l_s((v4f32)_1)`.
  **L2421 CN**: 以 `(__m128i)__builtin_lsx_vftintrzl_l_s((v4f32)_1)` 从当前函数返回。
- **L2422 EN**: Closes the current lexical scope or compound statement.
  **L2422 CN**: 结束当前词法作用域或复合语句块。
- **L2423 EN**: Blank line separating nearby declarations or logic blocks.
  **L2423 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2424 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2424 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2425-2448

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrzh_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrzh_l_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrpl_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrpl_l_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrph_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrph_l_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrml_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrml_l_s((v4f32)_1);
}

extern __inline
````
- **L2425 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2425 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2426 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrzh_l_s(__m128 _1) {`.
  **L2426 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrzh_l_s(__m128 _1) {`。
- **L2427 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrzh_l_s((v4f32)_1)`.
  **L2427 CN**: 以 `(__m128i)__builtin_lsx_vftintrzh_l_s((v4f32)_1)` 从当前函数返回。
- **L2428 EN**: Closes the current lexical scope or compound statement.
  **L2428 CN**: 结束当前词法作用域或复合语句块。
- **L2429 EN**: Blank line separating nearby declarations or logic blocks.
  **L2429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2430 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2430 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2431 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2431 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2432 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrpl_l_s(__m128 _1) {`.
  **L2432 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrpl_l_s(__m128 _1) {`。
- **L2433 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrpl_l_s((v4f32)_1)`.
  **L2433 CN**: 以 `(__m128i)__builtin_lsx_vftintrpl_l_s((v4f32)_1)` 从当前函数返回。
- **L2434 EN**: Closes the current lexical scope or compound statement.
  **L2434 CN**: 结束当前词法作用域或复合语句块。
- **L2435 EN**: Blank line separating nearby declarations or logic blocks.
  **L2435 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2436 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2436 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2437 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2437 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2438 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrph_l_s(__m128 _1) {`.
  **L2438 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrph_l_s(__m128 _1) {`。
- **L2439 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrph_l_s((v4f32)_1)`.
  **L2439 CN**: 以 `(__m128i)__builtin_lsx_vftintrph_l_s((v4f32)_1)` 从当前函数返回。
- **L2440 EN**: Closes the current lexical scope or compound statement.
  **L2440 CN**: 结束当前词法作用域或复合语句块。
- **L2441 EN**: Blank line separating nearby declarations or logic blocks.
  **L2441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2442 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2442 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2443 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2443 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2444 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrml_l_s(__m128 _1) {`.
  **L2444 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrml_l_s(__m128 _1) {`。
- **L2445 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrml_l_s((v4f32)_1)`.
  **L2445 CN**: 以 `(__m128i)__builtin_lsx_vftintrml_l_s((v4f32)_1)` 从当前函数返回。
- **L2446 EN**: Closes the current lexical scope or compound statement.
  **L2446 CN**: 结束当前词法作用域或复合语句块。
- **L2447 EN**: Blank line separating nearby declarations or logic blocks.
  **L2447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2448 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2448 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2449-2472

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrmh_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrmh_l_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrnel_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrnel_l_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vftintrneh_l_s(__m128 _1) {
  return (__m128i)__builtin_lsx_vftintrneh_l_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrintrne_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfrintrne_s((v4f32)_1);
}

extern __inline
````
- **L2449 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2449 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2450 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrmh_l_s(__m128 _1) {`.
  **L2450 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrmh_l_s(__m128 _1) {`。
- **L2451 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrmh_l_s((v4f32)_1)`.
  **L2451 CN**: 以 `(__m128i)__builtin_lsx_vftintrmh_l_s((v4f32)_1)` 从当前函数返回。
- **L2452 EN**: Closes the current lexical scope or compound statement.
  **L2452 CN**: 结束当前词法作用域或复合语句块。
- **L2453 EN**: Blank line separating nearby declarations or logic blocks.
  **L2453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2454 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2454 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2455 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2455 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2456 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrnel_l_s(__m128 _1) {`.
  **L2456 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrnel_l_s(__m128 _1) {`。
- **L2457 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrnel_l_s((v4f32)_1)`.
  **L2457 CN**: 以 `(__m128i)__builtin_lsx_vftintrnel_l_s((v4f32)_1)` 从当前函数返回。
- **L2458 EN**: Closes the current lexical scope or compound statement.
  **L2458 CN**: 结束当前词法作用域或复合语句块。
- **L2459 EN**: Blank line separating nearby declarations or logic blocks.
  **L2459 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2460 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2460 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2461 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2461 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2462 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vftintrneh_l_s(__m128 _1) {`.
  **L2462 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vftintrneh_l_s(__m128 _1) {`。
- **L2463 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vftintrneh_l_s((v4f32)_1)`.
  **L2463 CN**: 以 `(__m128i)__builtin_lsx_vftintrneh_l_s((v4f32)_1)` 从当前函数返回。
- **L2464 EN**: Closes the current lexical scope or compound statement.
  **L2464 CN**: 结束当前词法作用域或复合语句块。
- **L2465 EN**: Blank line separating nearby declarations or logic blocks.
  **L2465 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2466 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2466 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2467 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2467 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2468 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrintrne_s(__m128 _1) {`.
  **L2468 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrintrne_s(__m128 _1) {`。
- **L2469 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrintrne_s((v4f32)_1)`.
  **L2469 CN**: 以 `(__m128)__builtin_lsx_vfrintrne_s((v4f32)_1)` 从当前函数返回。
- **L2470 EN**: Closes the current lexical scope or compound statement.
  **L2470 CN**: 结束当前词法作用域或复合语句块。
- **L2471 EN**: Blank line separating nearby declarations or logic blocks.
  **L2471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2472 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2472 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2473-2496

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrintrne_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrintrne_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrintrz_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfrintrz_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrintrz_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrintrz_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrintrp_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfrintrp_s((v4f32)_1);
}

extern __inline
````
- **L2473 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2473 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2474 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrintrne_d(__m128d _1) {`.
  **L2474 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrintrne_d(__m128d _1) {`。
- **L2475 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrintrne_d((v2f64)_1)`.
  **L2475 CN**: 以 `(__m128d)__builtin_lsx_vfrintrne_d((v2f64)_1)` 从当前函数返回。
- **L2476 EN**: Closes the current lexical scope or compound statement.
  **L2476 CN**: 结束当前词法作用域或复合语句块。
- **L2477 EN**: Blank line separating nearby declarations or logic blocks.
  **L2477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2478 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2478 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2479 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2479 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2480 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrintrz_s(__m128 _1) {`.
  **L2480 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrintrz_s(__m128 _1) {`。
- **L2481 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrintrz_s((v4f32)_1)`.
  **L2481 CN**: 以 `(__m128)__builtin_lsx_vfrintrz_s((v4f32)_1)` 从当前函数返回。
- **L2482 EN**: Closes the current lexical scope or compound statement.
  **L2482 CN**: 结束当前词法作用域或复合语句块。
- **L2483 EN**: Blank line separating nearby declarations or logic blocks.
  **L2483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2484 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2484 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2485 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2485 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2486 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrintrz_d(__m128d _1) {`.
  **L2486 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrintrz_d(__m128d _1) {`。
- **L2487 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrintrz_d((v2f64)_1)`.
  **L2487 CN**: 以 `(__m128d)__builtin_lsx_vfrintrz_d((v2f64)_1)` 从当前函数返回。
- **L2488 EN**: Closes the current lexical scope or compound statement.
  **L2488 CN**: 结束当前词法作用域或复合语句块。
- **L2489 EN**: Blank line separating nearby declarations or logic blocks.
  **L2489 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2490 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2490 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2491 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2491 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2492 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrintrp_s(__m128 _1) {`.
  **L2492 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrintrp_s(__m128 _1) {`。
- **L2493 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrintrp_s((v4f32)_1)`.
  **L2493 CN**: 以 `(__m128)__builtin_lsx_vfrintrp_s((v4f32)_1)` 从当前函数返回。
- **L2494 EN**: Closes the current lexical scope or compound statement.
  **L2494 CN**: 结束当前词法作用域或复合语句块。
- **L2495 EN**: Blank line separating nearby declarations or logic blocks.
  **L2495 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2496 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2496 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2497-2520

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrintrp_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrintrp_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrintrm_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfrintrm_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrintrm_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrintrm_d((v2f64)_1);
}

#define __lsx_vstelm_b(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)  \
  ((void)__builtin_lsx_vstelm_b((v16i8)(_1), (void *)(_2), (_3), (_4)))

#define __lsx_vstelm_h(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)  \
  ((void)__builtin_lsx_vstelm_h((v8i16)(_1), (void *)(_2), (_3), (_4)))

#define __lsx_vstelm_w(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)  \
````
- **L2497 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2497 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2498 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrintrp_d(__m128d _1) {`.
  **L2498 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrintrp_d(__m128d _1) {`。
- **L2499 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrintrp_d((v2f64)_1)`.
  **L2499 CN**: 以 `(__m128d)__builtin_lsx_vfrintrp_d((v2f64)_1)` 从当前函数返回。
- **L2500 EN**: Closes the current lexical scope or compound statement.
  **L2500 CN**: 结束当前词法作用域或复合语句块。
- **L2501 EN**: Blank line separating nearby declarations or logic blocks.
  **L2501 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2502 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2502 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2503 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L2503 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L2504 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrintrm_s(__m128 _1) {`.
  **L2504 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrintrm_s(__m128 _1) {`。
- **L2505 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrintrm_s((v4f32)_1)`.
  **L2505 CN**: 以 `(__m128)__builtin_lsx_vfrintrm_s((v4f32)_1)` 从当前函数返回。
- **L2506 EN**: Closes the current lexical scope or compound statement.
  **L2506 CN**: 结束当前词法作用域或复合语句块。
- **L2507 EN**: Blank line separating nearby declarations or logic blocks.
  **L2507 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2508 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2508 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2509 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L2509 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L2510 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrintrm_d(__m128d _1) {`.
  **L2510 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrintrm_d(__m128d _1) {`。
- **L2511 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrintrm_d((v2f64)_1)`.
  **L2511 CN**: 以 `(__m128d)__builtin_lsx_vfrintrm_d((v2f64)_1)` 从当前函数返回。
- **L2512 EN**: Closes the current lexical scope or compound statement.
  **L2512 CN**: 结束当前词法作用域或复合语句块。
- **L2513 EN**: Blank line separating nearby declarations or logic blocks.
  **L2513 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2514 EN**: Defines macro `__lsx_vstelm_b(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)` for conditional compilation, shorthand, or API generation.
  **L2514 CN**: 定义宏 `__lsx_vstelm_b(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)`，用于条件编译、简写或 API 生成。
- **L2515 EN**: Continues logic associated with callable symbol `__builtin_lsx_vstelm_b`.
  **L2515 CN**: 继续与可调用符号 `__builtin_lsx_vstelm_b` 相关的逻辑。
- **L2516 EN**: Blank line separating nearby declarations or logic blocks.
  **L2516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2517 EN**: Defines macro `__lsx_vstelm_h(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)` for conditional compilation, shorthand, or API generation.
  **L2517 CN**: 定义宏 `__lsx_vstelm_h(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)`，用于条件编译、简写或 API 生成。
- **L2518 EN**: Continues logic associated with callable symbol `__builtin_lsx_vstelm_h`.
  **L2518 CN**: 继续与可调用符号 `__builtin_lsx_vstelm_h` 相关的逻辑。
- **L2519 EN**: Blank line separating nearby declarations or logic blocks.
  **L2519 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2520 EN**: Defines macro `__lsx_vstelm_w(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)` for conditional compilation, shorthand, or API generation.
  **L2520 CN**: 定义宏 `__lsx_vstelm_w(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)`，用于条件编译、简写或 API 生成。

### Lines 2521-2544

````c
  ((void)__builtin_lsx_vstelm_w((v4i32)(_1), (void *)(_2), (_3), (_4)))

#define __lsx_vstelm_d(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)  \
  ((void)__builtin_lsx_vstelm_d((v2i64)(_1), (void *)(_2), (_3), (_4)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_d_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_d_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_w_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_w_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_h_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_h_b((v16i8)_1, (v16i8)_2);
}

extern __inline
````
- **L2521 EN**: Continues logic associated with callable symbol `__builtin_lsx_vstelm_w`.
  **L2521 CN**: 继续与可调用符号 `__builtin_lsx_vstelm_w` 相关的逻辑。
- **L2522 EN**: Blank line separating nearby declarations or logic blocks.
  **L2522 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2523 EN**: Defines macro `__lsx_vstelm_d(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)` for conditional compilation, shorthand, or API generation.
  **L2523 CN**: 定义宏 `__lsx_vstelm_d(/*__m128i*/ _1, /*void **/ _2, /*si8*/ _3, /*idx*/ _4)`，用于条件编译、简写或 API 生成。
- **L2524 EN**: Continues logic associated with callable symbol `__builtin_lsx_vstelm_d`.
  **L2524 CN**: 继续与可调用符号 `__builtin_lsx_vstelm_d` 相关的逻辑。
- **L2525 EN**: Blank line separating nearby declarations or logic blocks.
  **L2525 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2526 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2526 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2527 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2527 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2528 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_d_w(__m128i _1, __m128i _2) {`.
  **L2528 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_d_w(__m128i _1, __m128i _2) {`。
- **L2529 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_d_w((v4i32)_1, (v4i32)_2)`.
  **L2529 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_d_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2530 EN**: Closes the current lexical scope or compound statement.
  **L2530 CN**: 结束当前词法作用域或复合语句块。
- **L2531 EN**: Blank line separating nearby declarations or logic blocks.
  **L2531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2532 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2532 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2533 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2533 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2534 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_w_h(__m128i _1, __m128i _2) {`.
  **L2534 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_w_h(__m128i _1, __m128i _2) {`。
- **L2535 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_w_h((v8i16)_1, (v8i16)_2)`.
  **L2535 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_w_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2536 EN**: Closes the current lexical scope or compound statement.
  **L2536 CN**: 结束当前词法作用域或复合语句块。
- **L2537 EN**: Blank line separating nearby declarations or logic blocks.
  **L2537 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2538 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2538 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2539 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2539 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2540 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_h_b(__m128i _1, __m128i _2) {`.
  **L2540 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_h_b(__m128i _1, __m128i _2) {`。
- **L2541 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_h_b((v16i8)_1, (v16i8)_2)`.
  **L2541 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_h_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L2542 EN**: Closes the current lexical scope or compound statement.
  **L2542 CN**: 结束当前词法作用域或复合语句块。
- **L2543 EN**: Blank line separating nearby declarations or logic blocks.
  **L2543 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2544 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2544 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2545-2568

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_d_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_d_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_w_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_w_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_h_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_h_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_d_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_d_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
````
- **L2545 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2545 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2546 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_d_w(__m128i _1, __m128i _2) {`.
  **L2546 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_d_w(__m128i _1, __m128i _2) {`。
- **L2547 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_d_w((v4i32)_1, (v4i32)_2)`.
  **L2547 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_d_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2548 EN**: Closes the current lexical scope or compound statement.
  **L2548 CN**: 结束当前词法作用域或复合语句块。
- **L2549 EN**: Blank line separating nearby declarations or logic blocks.
  **L2549 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2550 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2550 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2551 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2551 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2552 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_w_h(__m128i _1, __m128i _2) {`.
  **L2552 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_w_h(__m128i _1, __m128i _2) {`。
- **L2553 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_w_h((v8i16)_1, (v8i16)_2)`.
  **L2553 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_w_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2554 EN**: Closes the current lexical scope or compound statement.
  **L2554 CN**: 结束当前词法作用域或复合语句块。
- **L2555 EN**: Blank line separating nearby declarations or logic blocks.
  **L2555 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2556 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2556 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2557 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2557 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2558 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_h_b(__m128i _1, __m128i _2) {`.
  **L2558 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_h_b(__m128i _1, __m128i _2) {`。
- **L2559 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_h_b((v16i8)_1, (v16i8)_2)`.
  **L2559 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_h_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L2560 EN**: Closes the current lexical scope or compound statement.
  **L2560 CN**: 结束当前词法作用域或复合语句块。
- **L2561 EN**: Blank line separating nearby declarations or logic blocks.
  **L2561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2562 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2562 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2563 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2563 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2564 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_d_wu(__m128i _1, __m128i _2) {`.
  **L2564 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_d_wu(__m128i _1, __m128i _2) {`。
- **L2565 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_d_wu((v4u32)_1, (v4u32)_2)`.
  **L2565 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_d_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2566 EN**: Closes the current lexical scope or compound statement.
  **L2566 CN**: 结束当前词法作用域或复合语句块。
- **L2567 EN**: Blank line separating nearby declarations or logic blocks.
  **L2567 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2568 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2568 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2569-2592

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_w_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_w_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_h_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_h_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_d_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_d_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_w_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_w_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
````
- **L2569 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2569 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2570 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_w_hu(__m128i _1, __m128i _2) {`.
  **L2570 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_w_hu(__m128i _1, __m128i _2) {`。
- **L2571 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_w_hu((v8u16)_1, (v8u16)_2)`.
  **L2571 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_w_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2572 EN**: Closes the current lexical scope or compound statement.
  **L2572 CN**: 结束当前词法作用域或复合语句块。
- **L2573 EN**: Blank line separating nearby declarations or logic blocks.
  **L2573 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2574 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2574 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2575 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2575 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2576 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_h_bu(__m128i _1, __m128i _2) {`.
  **L2576 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_h_bu(__m128i _1, __m128i _2) {`。
- **L2577 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_h_bu((v16u8)_1, (v16u8)_2)`.
  **L2577 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_h_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L2578 EN**: Closes the current lexical scope or compound statement.
  **L2578 CN**: 结束当前词法作用域或复合语句块。
- **L2579 EN**: Blank line separating nearby declarations or logic blocks.
  **L2579 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2580 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2580 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2581 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2581 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2582 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_d_wu(__m128i _1, __m128i _2) {`.
  **L2582 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_d_wu(__m128i _1, __m128i _2) {`。
- **L2583 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_d_wu((v4u32)_1, (v4u32)_2)`.
  **L2583 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_d_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2584 EN**: Closes the current lexical scope or compound statement.
  **L2584 CN**: 结束当前词法作用域或复合语句块。
- **L2585 EN**: Blank line separating nearby declarations or logic blocks.
  **L2585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2586 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2586 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2587 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2587 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2588 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_w_hu(__m128i _1, __m128i _2) {`.
  **L2588 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_w_hu(__m128i _1, __m128i _2) {`。
- **L2589 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_w_hu((v8u16)_1, (v8u16)_2)`.
  **L2589 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_w_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2590 EN**: Closes the current lexical scope or compound statement.
  **L2590 CN**: 结束当前词法作用域或复合语句块。
- **L2591 EN**: Blank line separating nearby declarations or logic blocks.
  **L2591 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2592 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2592 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2593-2616

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_h_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_h_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_d_wu_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_d_wu_w((v4u32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_w_hu_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_w_hu_h((v8u16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_h_bu_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_h_bu_b((v16u8)_1, (v16i8)_2);
}

extern __inline
````
- **L2593 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2593 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2594 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_h_bu(__m128i _1, __m128i _2) {`.
  **L2594 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_h_bu(__m128i _1, __m128i _2) {`。
- **L2595 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_h_bu((v16u8)_1, (v16u8)_2)`.
  **L2595 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_h_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L2596 EN**: Closes the current lexical scope or compound statement.
  **L2596 CN**: 结束当前词法作用域或复合语句块。
- **L2597 EN**: Blank line separating nearby declarations or logic blocks.
  **L2597 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2598 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2598 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2599 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2599 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2600 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_d_wu_w(__m128i _1, __m128i _2) {`.
  **L2600 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_d_wu_w(__m128i _1, __m128i _2) {`。
- **L2601 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_d_wu_w((v4u32)_1, (v4i32)_2)`.
  **L2601 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_d_wu_w((v4u32)_1, (v4i32)_2)` 从当前函数返回。
- **L2602 EN**: Closes the current lexical scope or compound statement.
  **L2602 CN**: 结束当前词法作用域或复合语句块。
- **L2603 EN**: Blank line separating nearby declarations or logic blocks.
  **L2603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2604 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2604 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2605 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2605 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2606 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_w_hu_h(__m128i _1, __m128i _2) {`.
  **L2606 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_w_hu_h(__m128i _1, __m128i _2) {`。
- **L2607 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_w_hu_h((v8u16)_1, (v8i16)_2)`.
  **L2607 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_w_hu_h((v8u16)_1, (v8i16)_2)` 从当前函数返回。
- **L2608 EN**: Closes the current lexical scope or compound statement.
  **L2608 CN**: 结束当前词法作用域或复合语句块。
- **L2609 EN**: Blank line separating nearby declarations or logic blocks.
  **L2609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2610 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2610 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2611 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2611 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2612 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_h_bu_b(__m128i _1, __m128i _2) {`.
  **L2612 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_h_bu_b(__m128i _1, __m128i _2) {`。
- **L2613 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_h_bu_b((v16u8)_1, (v16i8)_2)`.
  **L2613 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_h_bu_b((v16u8)_1, (v16i8)_2)` 从当前函数返回。
- **L2614 EN**: Closes the current lexical scope or compound statement.
  **L2614 CN**: 结束当前词法作用域或复合语句块。
- **L2615 EN**: Blank line separating nearby declarations or logic blocks.
  **L2615 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2616 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2616 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2617-2640

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_d_wu_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_d_wu_w((v4u32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_w_hu_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_w_hu_h((v8u16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_h_bu_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_h_bu_b((v16u8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwev_d_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwev_d_w((v4i32)_1, (v4i32)_2);
}

extern __inline
````
- **L2617 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2617 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2618 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_d_wu_w(__m128i _1, __m128i _2) {`.
  **L2618 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_d_wu_w(__m128i _1, __m128i _2) {`。
- **L2619 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_d_wu_w((v4u32)_1, (v4i32)_2)`.
  **L2619 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_d_wu_w((v4u32)_1, (v4i32)_2)` 从当前函数返回。
- **L2620 EN**: Closes the current lexical scope or compound statement.
  **L2620 CN**: 结束当前词法作用域或复合语句块。
- **L2621 EN**: Blank line separating nearby declarations or logic blocks.
  **L2621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2622 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2622 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2623 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2623 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2624 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_w_hu_h(__m128i _1, __m128i _2) {`.
  **L2624 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_w_hu_h(__m128i _1, __m128i _2) {`。
- **L2625 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_w_hu_h((v8u16)_1, (v8i16)_2)`.
  **L2625 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_w_hu_h((v8u16)_1, (v8i16)_2)` 从当前函数返回。
- **L2626 EN**: Closes the current lexical scope or compound statement.
  **L2626 CN**: 结束当前词法作用域或复合语句块。
- **L2627 EN**: Blank line separating nearby declarations or logic blocks.
  **L2627 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2628 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2628 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2629 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2629 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2630 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_h_bu_b(__m128i _1, __m128i _2) {`.
  **L2630 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_h_bu_b(__m128i _1, __m128i _2) {`。
- **L2631 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_h_bu_b((v16u8)_1, (v16i8)_2)`.
  **L2631 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_h_bu_b((v16u8)_1, (v16i8)_2)` 从当前函数返回。
- **L2632 EN**: Closes the current lexical scope or compound statement.
  **L2632 CN**: 结束当前词法作用域或复合语句块。
- **L2633 EN**: Blank line separating nearby declarations or logic blocks.
  **L2633 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2634 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2634 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2635 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2635 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2636 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwev_d_w(__m128i _1, __m128i _2) {`.
  **L2636 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwev_d_w(__m128i _1, __m128i _2) {`。
- **L2637 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwev_d_w((v4i32)_1, (v4i32)_2)`.
  **L2637 CN**: 以 `(__m128i)__builtin_lsx_vsubwev_d_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2638 EN**: Closes the current lexical scope or compound statement.
  **L2638 CN**: 结束当前词法作用域或复合语句块。
- **L2639 EN**: Blank line separating nearby declarations or logic blocks.
  **L2639 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2640 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2640 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2641-2664

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwev_w_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwev_w_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwev_h_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwev_h_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwod_d_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwod_d_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwod_w_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwod_w_h((v8i16)_1, (v8i16)_2);
}

extern __inline
````
- **L2641 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2641 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2642 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwev_w_h(__m128i _1, __m128i _2) {`.
  **L2642 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwev_w_h(__m128i _1, __m128i _2) {`。
- **L2643 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwev_w_h((v8i16)_1, (v8i16)_2)`.
  **L2643 CN**: 以 `(__m128i)__builtin_lsx_vsubwev_w_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2644 EN**: Closes the current lexical scope or compound statement.
  **L2644 CN**: 结束当前词法作用域或复合语句块。
- **L2645 EN**: Blank line separating nearby declarations or logic blocks.
  **L2645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2646 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2646 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2647 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2647 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2648 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwev_h_b(__m128i _1, __m128i _2) {`.
  **L2648 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwev_h_b(__m128i _1, __m128i _2) {`。
- **L2649 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwev_h_b((v16i8)_1, (v16i8)_2)`.
  **L2649 CN**: 以 `(__m128i)__builtin_lsx_vsubwev_h_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L2650 EN**: Closes the current lexical scope or compound statement.
  **L2650 CN**: 结束当前词法作用域或复合语句块。
- **L2651 EN**: Blank line separating nearby declarations or logic blocks.
  **L2651 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2652 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2652 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2653 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2653 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2654 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwod_d_w(__m128i _1, __m128i _2) {`.
  **L2654 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwod_d_w(__m128i _1, __m128i _2) {`。
- **L2655 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwod_d_w((v4i32)_1, (v4i32)_2)`.
  **L2655 CN**: 以 `(__m128i)__builtin_lsx_vsubwod_d_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2656 EN**: Closes the current lexical scope or compound statement.
  **L2656 CN**: 结束当前词法作用域或复合语句块。
- **L2657 EN**: Blank line separating nearby declarations or logic blocks.
  **L2657 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2658 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2658 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2659 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2659 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2660 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwod_w_h(__m128i _1, __m128i _2) {`.
  **L2660 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwod_w_h(__m128i _1, __m128i _2) {`。
- **L2661 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwod_w_h((v8i16)_1, (v8i16)_2)`.
  **L2661 CN**: 以 `(__m128i)__builtin_lsx_vsubwod_w_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2662 EN**: Closes the current lexical scope or compound statement.
  **L2662 CN**: 结束当前词法作用域或复合语句块。
- **L2663 EN**: Blank line separating nearby declarations or logic blocks.
  **L2663 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2664 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2664 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2665-2688

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwod_h_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwod_h_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwev_d_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwev_d_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwev_w_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwev_w_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwev_h_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwev_h_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
````
- **L2665 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2665 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2666 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwod_h_b(__m128i _1, __m128i _2) {`.
  **L2666 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwod_h_b(__m128i _1, __m128i _2) {`。
- **L2667 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwod_h_b((v16i8)_1, (v16i8)_2)`.
  **L2667 CN**: 以 `(__m128i)__builtin_lsx_vsubwod_h_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L2668 EN**: Closes the current lexical scope or compound statement.
  **L2668 CN**: 结束当前词法作用域或复合语句块。
- **L2669 EN**: Blank line separating nearby declarations or logic blocks.
  **L2669 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2670 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2670 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2671 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2671 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2672 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwev_d_wu(__m128i _1, __m128i _2) {`.
  **L2672 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwev_d_wu(__m128i _1, __m128i _2) {`。
- **L2673 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwev_d_wu((v4u32)_1, (v4u32)_2)`.
  **L2673 CN**: 以 `(__m128i)__builtin_lsx_vsubwev_d_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2674 EN**: Closes the current lexical scope or compound statement.
  **L2674 CN**: 结束当前词法作用域或复合语句块。
- **L2675 EN**: Blank line separating nearby declarations or logic blocks.
  **L2675 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2676 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2676 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2677 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2677 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2678 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwev_w_hu(__m128i _1, __m128i _2) {`.
  **L2678 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwev_w_hu(__m128i _1, __m128i _2) {`。
- **L2679 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwev_w_hu((v8u16)_1, (v8u16)_2)`.
  **L2679 CN**: 以 `(__m128i)__builtin_lsx_vsubwev_w_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2680 EN**: Closes the current lexical scope or compound statement.
  **L2680 CN**: 结束当前词法作用域或复合语句块。
- **L2681 EN**: Blank line separating nearby declarations or logic blocks.
  **L2681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2682 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2682 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2683 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2683 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2684 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwev_h_bu(__m128i _1, __m128i _2) {`.
  **L2684 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwev_h_bu(__m128i _1, __m128i _2) {`。
- **L2685 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwev_h_bu((v16u8)_1, (v16u8)_2)`.
  **L2685 CN**: 以 `(__m128i)__builtin_lsx_vsubwev_h_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L2686 EN**: Closes the current lexical scope or compound statement.
  **L2686 CN**: 结束当前词法作用域或复合语句块。
- **L2687 EN**: Blank line separating nearby declarations or logic blocks.
  **L2687 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2688 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2688 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2689-2712

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwod_d_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwod_d_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwod_w_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwod_w_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwod_h_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwod_h_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_q_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_q_d((v2i64)_1, (v2i64)_2);
}

extern __inline
````
- **L2689 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2689 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2690 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwod_d_wu(__m128i _1, __m128i _2) {`.
  **L2690 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwod_d_wu(__m128i _1, __m128i _2) {`。
- **L2691 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwod_d_wu((v4u32)_1, (v4u32)_2)`.
  **L2691 CN**: 以 `(__m128i)__builtin_lsx_vsubwod_d_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2692 EN**: Closes the current lexical scope or compound statement.
  **L2692 CN**: 结束当前词法作用域或复合语句块。
- **L2693 EN**: Blank line separating nearby declarations or logic blocks.
  **L2693 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2694 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2694 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2695 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2695 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2696 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwod_w_hu(__m128i _1, __m128i _2) {`.
  **L2696 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwod_w_hu(__m128i _1, __m128i _2) {`。
- **L2697 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwod_w_hu((v8u16)_1, (v8u16)_2)`.
  **L2697 CN**: 以 `(__m128i)__builtin_lsx_vsubwod_w_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2698 EN**: Closes the current lexical scope or compound statement.
  **L2698 CN**: 结束当前词法作用域或复合语句块。
- **L2699 EN**: Blank line separating nearby declarations or logic blocks.
  **L2699 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2700 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2700 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2701 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2701 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2702 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwod_h_bu(__m128i _1, __m128i _2) {`.
  **L2702 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwod_h_bu(__m128i _1, __m128i _2) {`。
- **L2703 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwod_h_bu((v16u8)_1, (v16u8)_2)`.
  **L2703 CN**: 以 `(__m128i)__builtin_lsx_vsubwod_h_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L2704 EN**: Closes the current lexical scope or compound statement.
  **L2704 CN**: 结束当前词法作用域或复合语句块。
- **L2705 EN**: Blank line separating nearby declarations or logic blocks.
  **L2705 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2706 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2706 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2707 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2707 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2708 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_q_d(__m128i _1, __m128i _2) {`.
  **L2708 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_q_d(__m128i _1, __m128i _2) {`。
- **L2709 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_q_d((v2i64)_1, (v2i64)_2)`.
  **L2709 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_q_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2710 EN**: Closes the current lexical scope or compound statement.
  **L2710 CN**: 结束当前词法作用域或复合语句块。
- **L2711 EN**: Blank line separating nearby declarations or logic blocks.
  **L2711 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2712 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2712 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2713-2736

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_q_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_q_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_q_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_q_du((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_q_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_q_du((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwev_q_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwev_q_d((v2i64)_1, (v2i64)_2);
}

extern __inline
````
- **L2713 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2713 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2714 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_q_d(__m128i _1, __m128i _2) {`.
  **L2714 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_q_d(__m128i _1, __m128i _2) {`。
- **L2715 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_q_d((v2i64)_1, (v2i64)_2)`.
  **L2715 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_q_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2716 EN**: Closes the current lexical scope or compound statement.
  **L2716 CN**: 结束当前词法作用域或复合语句块。
- **L2717 EN**: Blank line separating nearby declarations or logic blocks.
  **L2717 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2718 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2718 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2719 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2719 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2720 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_q_du(__m128i _1, __m128i _2) {`.
  **L2720 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_q_du(__m128i _1, __m128i _2) {`。
- **L2721 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_q_du((v2u64)_1, (v2u64)_2)`.
  **L2721 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_q_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2722 EN**: Closes the current lexical scope or compound statement.
  **L2722 CN**: 结束当前词法作用域或复合语句块。
- **L2723 EN**: Blank line separating nearby declarations or logic blocks.
  **L2723 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2724 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2724 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2725 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2725 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2726 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_q_du(__m128i _1, __m128i _2) {`.
  **L2726 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_q_du(__m128i _1, __m128i _2) {`。
- **L2727 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_q_du((v2u64)_1, (v2u64)_2)`.
  **L2727 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_q_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2728 EN**: Closes the current lexical scope or compound statement.
  **L2728 CN**: 结束当前词法作用域或复合语句块。
- **L2729 EN**: Blank line separating nearby declarations or logic blocks.
  **L2729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2730 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2730 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2731 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2731 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2732 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwev_q_d(__m128i _1, __m128i _2) {`.
  **L2732 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwev_q_d(__m128i _1, __m128i _2) {`。
- **L2733 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwev_q_d((v2i64)_1, (v2i64)_2)`.
  **L2733 CN**: 以 `(__m128i)__builtin_lsx_vsubwev_q_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2734 EN**: Closes the current lexical scope or compound statement.
  **L2734 CN**: 结束当前词法作用域或复合语句块。
- **L2735 EN**: Blank line separating nearby declarations or logic blocks.
  **L2735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2736 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2736 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2737-2760

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwod_q_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwod_q_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwev_q_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwev_q_du((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsubwod_q_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsubwod_q_du((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwev_q_du_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwev_q_du_d((v2u64)_1, (v2i64)_2);
}

extern __inline
````
- **L2737 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2737 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2738 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwod_q_d(__m128i _1, __m128i _2) {`.
  **L2738 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwod_q_d(__m128i _1, __m128i _2) {`。
- **L2739 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwod_q_d((v2i64)_1, (v2i64)_2)`.
  **L2739 CN**: 以 `(__m128i)__builtin_lsx_vsubwod_q_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2740 EN**: Closes the current lexical scope or compound statement.
  **L2740 CN**: 结束当前词法作用域或复合语句块。
- **L2741 EN**: Blank line separating nearby declarations or logic blocks.
  **L2741 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2742 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2742 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2743 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2743 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2744 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwev_q_du(__m128i _1, __m128i _2) {`.
  **L2744 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwev_q_du(__m128i _1, __m128i _2) {`。
- **L2745 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwev_q_du((v2u64)_1, (v2u64)_2)`.
  **L2745 CN**: 以 `(__m128i)__builtin_lsx_vsubwev_q_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2746 EN**: Closes the current lexical scope or compound statement.
  **L2746 CN**: 结束当前词法作用域或复合语句块。
- **L2747 EN**: Blank line separating nearby declarations or logic blocks.
  **L2747 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2748 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2748 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2749 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2749 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2750 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsubwod_q_du(__m128i _1, __m128i _2) {`.
  **L2750 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsubwod_q_du(__m128i _1, __m128i _2) {`。
- **L2751 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsubwod_q_du((v2u64)_1, (v2u64)_2)`.
  **L2751 CN**: 以 `(__m128i)__builtin_lsx_vsubwod_q_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2752 EN**: Closes the current lexical scope or compound statement.
  **L2752 CN**: 结束当前词法作用域或复合语句块。
- **L2753 EN**: Blank line separating nearby declarations or logic blocks.
  **L2753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2754 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2754 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2755 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2755 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2756 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwev_q_du_d(__m128i _1, __m128i _2) {`.
  **L2756 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwev_q_du_d(__m128i _1, __m128i _2) {`。
- **L2757 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwev_q_du_d((v2u64)_1, (v2i64)_2)`.
  **L2757 CN**: 以 `(__m128i)__builtin_lsx_vaddwev_q_du_d((v2u64)_1, (v2i64)_2)` 从当前函数返回。
- **L2758 EN**: Closes the current lexical scope or compound statement.
  **L2758 CN**: 结束当前词法作用域或复合语句块。
- **L2759 EN**: Blank line separating nearby declarations or logic blocks.
  **L2759 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2760 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2760 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2761-2784

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vaddwod_q_du_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vaddwod_q_du_d((v2u64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_d_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_d_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_w_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_w_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_h_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_h_b((v16i8)_1, (v16i8)_2);
}

extern __inline
````
- **L2761 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2761 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2762 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vaddwod_q_du_d(__m128i _1, __m128i _2) {`.
  **L2762 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vaddwod_q_du_d(__m128i _1, __m128i _2) {`。
- **L2763 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vaddwod_q_du_d((v2u64)_1, (v2i64)_2)`.
  **L2763 CN**: 以 `(__m128i)__builtin_lsx_vaddwod_q_du_d((v2u64)_1, (v2i64)_2)` 从当前函数返回。
- **L2764 EN**: Closes the current lexical scope or compound statement.
  **L2764 CN**: 结束当前词法作用域或复合语句块。
- **L2765 EN**: Blank line separating nearby declarations or logic blocks.
  **L2765 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2766 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2766 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2767 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2767 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2768 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_d_w(__m128i _1, __m128i _2) {`.
  **L2768 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_d_w(__m128i _1, __m128i _2) {`。
- **L2769 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_d_w((v4i32)_1, (v4i32)_2)`.
  **L2769 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_d_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2770 EN**: Closes the current lexical scope or compound statement.
  **L2770 CN**: 结束当前词法作用域或复合语句块。
- **L2771 EN**: Blank line separating nearby declarations or logic blocks.
  **L2771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2772 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2772 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2773 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2773 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2774 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_w_h(__m128i _1, __m128i _2) {`.
  **L2774 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_w_h(__m128i _1, __m128i _2) {`。
- **L2775 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_w_h((v8i16)_1, (v8i16)_2)`.
  **L2775 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_w_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2776 EN**: Closes the current lexical scope or compound statement.
  **L2776 CN**: 结束当前词法作用域或复合语句块。
- **L2777 EN**: Blank line separating nearby declarations or logic blocks.
  **L2777 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2778 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2778 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2779 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2779 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2780 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_h_b(__m128i _1, __m128i _2) {`.
  **L2780 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_h_b(__m128i _1, __m128i _2) {`。
- **L2781 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_h_b((v16i8)_1, (v16i8)_2)`.
  **L2781 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_h_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L2782 EN**: Closes the current lexical scope or compound statement.
  **L2782 CN**: 结束当前词法作用域或复合语句块。
- **L2783 EN**: Blank line separating nearby declarations or logic blocks.
  **L2783 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2784 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2784 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2785-2808

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_d_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_d_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_w_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_w_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_h_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_h_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_d_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_d_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
````
- **L2785 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2785 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2786 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_d_w(__m128i _1, __m128i _2) {`.
  **L2786 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_d_w(__m128i _1, __m128i _2) {`。
- **L2787 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_d_w((v4i32)_1, (v4i32)_2)`.
  **L2787 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_d_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L2788 EN**: Closes the current lexical scope or compound statement.
  **L2788 CN**: 结束当前词法作用域或复合语句块。
- **L2789 EN**: Blank line separating nearby declarations or logic blocks.
  **L2789 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2790 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2790 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2791 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2791 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2792 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_w_h(__m128i _1, __m128i _2) {`.
  **L2792 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_w_h(__m128i _1, __m128i _2) {`。
- **L2793 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_w_h((v8i16)_1, (v8i16)_2)`.
  **L2793 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_w_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L2794 EN**: Closes the current lexical scope or compound statement.
  **L2794 CN**: 结束当前词法作用域或复合语句块。
- **L2795 EN**: Blank line separating nearby declarations or logic blocks.
  **L2795 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2796 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2796 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2797 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2797 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2798 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_h_b(__m128i _1, __m128i _2) {`.
  **L2798 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_h_b(__m128i _1, __m128i _2) {`。
- **L2799 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_h_b((v16i8)_1, (v16i8)_2)`.
  **L2799 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_h_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L2800 EN**: Closes the current lexical scope or compound statement.
  **L2800 CN**: 结束当前词法作用域或复合语句块。
- **L2801 EN**: Blank line separating nearby declarations or logic blocks.
  **L2801 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2802 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2802 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2803 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2803 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2804 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_d_wu(__m128i _1, __m128i _2) {`.
  **L2804 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_d_wu(__m128i _1, __m128i _2) {`。
- **L2805 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_d_wu((v4u32)_1, (v4u32)_2)`.
  **L2805 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_d_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2806 EN**: Closes the current lexical scope or compound statement.
  **L2806 CN**: 结束当前词法作用域或复合语句块。
- **L2807 EN**: Blank line separating nearby declarations or logic blocks.
  **L2807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2808 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2808 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2809-2832

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_w_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_w_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_h_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_h_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_d_wu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_d_wu((v4u32)_1, (v4u32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_w_hu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_w_hu((v8u16)_1, (v8u16)_2);
}

extern __inline
````
- **L2809 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2809 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2810 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_w_hu(__m128i _1, __m128i _2) {`.
  **L2810 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_w_hu(__m128i _1, __m128i _2) {`。
- **L2811 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_w_hu((v8u16)_1, (v8u16)_2)`.
  **L2811 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_w_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2812 EN**: Closes the current lexical scope or compound statement.
  **L2812 CN**: 结束当前词法作用域或复合语句块。
- **L2813 EN**: Blank line separating nearby declarations or logic blocks.
  **L2813 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2814 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2814 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2815 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2815 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2816 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_h_bu(__m128i _1, __m128i _2) {`.
  **L2816 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_h_bu(__m128i _1, __m128i _2) {`。
- **L2817 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_h_bu((v16u8)_1, (v16u8)_2)`.
  **L2817 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_h_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L2818 EN**: Closes the current lexical scope or compound statement.
  **L2818 CN**: 结束当前词法作用域或复合语句块。
- **L2819 EN**: Blank line separating nearby declarations or logic blocks.
  **L2819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2820 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2820 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2821 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2821 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2822 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_d_wu(__m128i _1, __m128i _2) {`.
  **L2822 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_d_wu(__m128i _1, __m128i _2) {`。
- **L2823 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_d_wu((v4u32)_1, (v4u32)_2)`.
  **L2823 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_d_wu((v4u32)_1, (v4u32)_2)` 从当前函数返回。
- **L2824 EN**: Closes the current lexical scope or compound statement.
  **L2824 CN**: 结束当前词法作用域或复合语句块。
- **L2825 EN**: Blank line separating nearby declarations or logic blocks.
  **L2825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2826 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2826 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2827 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2827 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2828 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_w_hu(__m128i _1, __m128i _2) {`.
  **L2828 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_w_hu(__m128i _1, __m128i _2) {`。
- **L2829 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_w_hu((v8u16)_1, (v8u16)_2)`.
  **L2829 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_w_hu((v8u16)_1, (v8u16)_2)` 从当前函数返回。
- **L2830 EN**: Closes the current lexical scope or compound statement.
  **L2830 CN**: 结束当前词法作用域或复合语句块。
- **L2831 EN**: Blank line separating nearby declarations or logic blocks.
  **L2831 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2832 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2832 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2833-2856

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_h_bu(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_h_bu((v16u8)_1, (v16u8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_d_wu_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_d_wu_w((v4u32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_w_hu_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_w_hu_h((v8u16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_h_bu_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_h_bu_b((v16u8)_1, (v16i8)_2);
}

extern __inline
````
- **L2833 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2833 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2834 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_h_bu(__m128i _1, __m128i _2) {`.
  **L2834 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_h_bu(__m128i _1, __m128i _2) {`。
- **L2835 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_h_bu((v16u8)_1, (v16u8)_2)`.
  **L2835 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_h_bu((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L2836 EN**: Closes the current lexical scope or compound statement.
  **L2836 CN**: 结束当前词法作用域或复合语句块。
- **L2837 EN**: Blank line separating nearby declarations or logic blocks.
  **L2837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2838 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2838 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2839 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2839 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2840 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_d_wu_w(__m128i _1, __m128i _2) {`.
  **L2840 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_d_wu_w(__m128i _1, __m128i _2) {`。
- **L2841 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_d_wu_w((v4u32)_1, (v4i32)_2)`.
  **L2841 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_d_wu_w((v4u32)_1, (v4i32)_2)` 从当前函数返回。
- **L2842 EN**: Closes the current lexical scope or compound statement.
  **L2842 CN**: 结束当前词法作用域或复合语句块。
- **L2843 EN**: Blank line separating nearby declarations or logic blocks.
  **L2843 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2844 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2844 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2845 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2845 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2846 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_w_hu_h(__m128i _1, __m128i _2) {`.
  **L2846 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_w_hu_h(__m128i _1, __m128i _2) {`。
- **L2847 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_w_hu_h((v8u16)_1, (v8i16)_2)`.
  **L2847 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_w_hu_h((v8u16)_1, (v8i16)_2)` 从当前函数返回。
- **L2848 EN**: Closes the current lexical scope or compound statement.
  **L2848 CN**: 结束当前词法作用域或复合语句块。
- **L2849 EN**: Blank line separating nearby declarations or logic blocks.
  **L2849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2850 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2850 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2851 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2851 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2852 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_h_bu_b(__m128i _1, __m128i _2) {`.
  **L2852 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_h_bu_b(__m128i _1, __m128i _2) {`。
- **L2853 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_h_bu_b((v16u8)_1, (v16i8)_2)`.
  **L2853 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_h_bu_b((v16u8)_1, (v16i8)_2)` 从当前函数返回。
- **L2854 EN**: Closes the current lexical scope or compound statement.
  **L2854 CN**: 结束当前词法作用域或复合语句块。
- **L2855 EN**: Blank line separating nearby declarations or logic blocks.
  **L2855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2856 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2856 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2857-2880

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_d_wu_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_d_wu_w((v4u32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_w_hu_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_w_hu_h((v8u16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_h_bu_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_h_bu_b((v16u8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_q_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_q_d((v2i64)_1, (v2i64)_2);
}

extern __inline
````
- **L2857 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2857 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2858 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_d_wu_w(__m128i _1, __m128i _2) {`.
  **L2858 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_d_wu_w(__m128i _1, __m128i _2) {`。
- **L2859 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_d_wu_w((v4u32)_1, (v4i32)_2)`.
  **L2859 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_d_wu_w((v4u32)_1, (v4i32)_2)` 从当前函数返回。
- **L2860 EN**: Closes the current lexical scope or compound statement.
  **L2860 CN**: 结束当前词法作用域或复合语句块。
- **L2861 EN**: Blank line separating nearby declarations or logic blocks.
  **L2861 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2862 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2862 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2863 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2863 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2864 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_w_hu_h(__m128i _1, __m128i _2) {`.
  **L2864 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_w_hu_h(__m128i _1, __m128i _2) {`。
- **L2865 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_w_hu_h((v8u16)_1, (v8i16)_2)`.
  **L2865 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_w_hu_h((v8u16)_1, (v8i16)_2)` 从当前函数返回。
- **L2866 EN**: Closes the current lexical scope or compound statement.
  **L2866 CN**: 结束当前词法作用域或复合语句块。
- **L2867 EN**: Blank line separating nearby declarations or logic blocks.
  **L2867 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2868 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2868 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2869 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2869 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2870 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_h_bu_b(__m128i _1, __m128i _2) {`.
  **L2870 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_h_bu_b(__m128i _1, __m128i _2) {`。
- **L2871 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_h_bu_b((v16u8)_1, (v16i8)_2)`.
  **L2871 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_h_bu_b((v16u8)_1, (v16i8)_2)` 从当前函数返回。
- **L2872 EN**: Closes the current lexical scope or compound statement.
  **L2872 CN**: 结束当前词法作用域或复合语句块。
- **L2873 EN**: Blank line separating nearby declarations or logic blocks.
  **L2873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2874 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2874 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2875 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2875 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2876 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_q_d(__m128i _1, __m128i _2) {`.
  **L2876 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_q_d(__m128i _1, __m128i _2) {`。
- **L2877 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_q_d((v2i64)_1, (v2i64)_2)`.
  **L2877 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_q_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2878 EN**: Closes the current lexical scope or compound statement.
  **L2878 CN**: 结束当前词法作用域或复合语句块。
- **L2879 EN**: Blank line separating nearby declarations or logic blocks.
  **L2879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2880 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2880 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2881-2904

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_q_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_q_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_q_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_q_du((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_q_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_q_du((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwev_q_du_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwev_q_du_d((v2u64)_1, (v2i64)_2);
}

extern __inline
````
- **L2881 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2881 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2882 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_q_d(__m128i _1, __m128i _2) {`.
  **L2882 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_q_d(__m128i _1, __m128i _2) {`。
- **L2883 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_q_d((v2i64)_1, (v2i64)_2)`.
  **L2883 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_q_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2884 EN**: Closes the current lexical scope or compound statement.
  **L2884 CN**: 结束当前词法作用域或复合语句块。
- **L2885 EN**: Blank line separating nearby declarations or logic blocks.
  **L2885 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2886 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2886 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2887 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2887 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2888 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_q_du(__m128i _1, __m128i _2) {`.
  **L2888 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_q_du(__m128i _1, __m128i _2) {`。
- **L2889 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_q_du((v2u64)_1, (v2u64)_2)`.
  **L2889 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_q_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2890 EN**: Closes the current lexical scope or compound statement.
  **L2890 CN**: 结束当前词法作用域或复合语句块。
- **L2891 EN**: Blank line separating nearby declarations or logic blocks.
  **L2891 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2892 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2892 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2893 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2893 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2894 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_q_du(__m128i _1, __m128i _2) {`.
  **L2894 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_q_du(__m128i _1, __m128i _2) {`。
- **L2895 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_q_du((v2u64)_1, (v2u64)_2)`.
  **L2895 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_q_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2896 EN**: Closes the current lexical scope or compound statement.
  **L2896 CN**: 结束当前词法作用域或复合语句块。
- **L2897 EN**: Blank line separating nearby declarations or logic blocks.
  **L2897 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2898 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2898 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2899 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2899 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2900 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwev_q_du_d(__m128i _1, __m128i _2) {`.
  **L2900 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwev_q_du_d(__m128i _1, __m128i _2) {`。
- **L2901 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwev_q_du_d((v2u64)_1, (v2i64)_2)`.
  **L2901 CN**: 以 `(__m128i)__builtin_lsx_vmulwev_q_du_d((v2u64)_1, (v2i64)_2)` 从当前函数返回。
- **L2902 EN**: Closes the current lexical scope or compound statement.
  **L2902 CN**: 结束当前词法作用域或复合语句块。
- **L2903 EN**: Blank line separating nearby declarations or logic blocks.
  **L2903 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2904 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2904 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2905-2928

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmulwod_q_du_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vmulwod_q_du_d((v2u64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhaddw_q_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhaddw_q_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhaddw_qu_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhaddw_qu_du((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhsubw_q_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhsubw_q_d((v2i64)_1, (v2i64)_2);
}

extern __inline
````
- **L2905 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2905 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2906 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmulwod_q_du_d(__m128i _1, __m128i _2) {`.
  **L2906 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmulwod_q_du_d(__m128i _1, __m128i _2) {`。
- **L2907 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmulwod_q_du_d((v2u64)_1, (v2i64)_2)`.
  **L2907 CN**: 以 `(__m128i)__builtin_lsx_vmulwod_q_du_d((v2u64)_1, (v2i64)_2)` 从当前函数返回。
- **L2908 EN**: Closes the current lexical scope or compound statement.
  **L2908 CN**: 结束当前词法作用域或复合语句块。
- **L2909 EN**: Blank line separating nearby declarations or logic blocks.
  **L2909 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2910 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2910 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2911 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2911 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2912 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhaddw_q_d(__m128i _1, __m128i _2) {`.
  **L2912 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhaddw_q_d(__m128i _1, __m128i _2) {`。
- **L2913 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhaddw_q_d((v2i64)_1, (v2i64)_2)`.
  **L2913 CN**: 以 `(__m128i)__builtin_lsx_vhaddw_q_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2914 EN**: Closes the current lexical scope or compound statement.
  **L2914 CN**: 结束当前词法作用域或复合语句块。
- **L2915 EN**: Blank line separating nearby declarations or logic blocks.
  **L2915 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2916 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2916 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2917 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2917 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2918 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhaddw_qu_du(__m128i _1, __m128i _2) {`.
  **L2918 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhaddw_qu_du(__m128i _1, __m128i _2) {`。
- **L2919 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhaddw_qu_du((v2u64)_1, (v2u64)_2)`.
  **L2919 CN**: 以 `(__m128i)__builtin_lsx_vhaddw_qu_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2920 EN**: Closes the current lexical scope or compound statement.
  **L2920 CN**: 结束当前词法作用域或复合语句块。
- **L2921 EN**: Blank line separating nearby declarations or logic blocks.
  **L2921 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2922 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2922 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2923 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2923 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2924 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhsubw_q_d(__m128i _1, __m128i _2) {`.
  **L2924 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhsubw_q_d(__m128i _1, __m128i _2) {`。
- **L2925 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhsubw_q_d((v2i64)_1, (v2i64)_2)`.
  **L2925 CN**: 以 `(__m128i)__builtin_lsx_vhsubw_q_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L2926 EN**: Closes the current lexical scope or compound statement.
  **L2926 CN**: 结束当前词法作用域或复合语句块。
- **L2927 EN**: Blank line separating nearby declarations or logic blocks.
  **L2927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2928 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2928 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2929-2952

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vhsubw_qu_du(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vhsubw_qu_du((v2u64)_1, (v2u64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_d_w(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_d_w((v2i64)_1, (v4i32)_2, (v4i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_w_h(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_w_h((v4i32)_1, (v8i16)_2, (v8i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_h_b(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_h_b((v8i16)_1, (v16i8)_2, (v16i8)_3);
}

extern __inline
````
- **L2929 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2929 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2930 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vhsubw_qu_du(__m128i _1, __m128i _2) {`.
  **L2930 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vhsubw_qu_du(__m128i _1, __m128i _2) {`。
- **L2931 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vhsubw_qu_du((v2u64)_1, (v2u64)_2)`.
  **L2931 CN**: 以 `(__m128i)__builtin_lsx_vhsubw_qu_du((v2u64)_1, (v2u64)_2)` 从当前函数返回。
- **L2932 EN**: Closes the current lexical scope or compound statement.
  **L2932 CN**: 结束当前词法作用域或复合语句块。
- **L2933 EN**: Blank line separating nearby declarations or logic blocks.
  **L2933 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2934 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2934 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2935 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2935 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2936 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_d_w(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2936 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_d_w(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2937 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_d_w((v2i64)_1, (v4i32)_2, (v4i32)_3)`.
  **L2937 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_d_w((v2i64)_1, (v4i32)_2, (v4i32)_3)` 从当前函数返回。
- **L2938 EN**: Closes the current lexical scope or compound statement.
  **L2938 CN**: 结束当前词法作用域或复合语句块。
- **L2939 EN**: Blank line separating nearby declarations or logic blocks.
  **L2939 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2940 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2940 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2941 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2941 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2942 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_w_h(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2942 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_w_h(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2943 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_w_h((v4i32)_1, (v8i16)_2, (v8i16)_3)`.
  **L2943 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_w_h((v4i32)_1, (v8i16)_2, (v8i16)_3)` 从当前函数返回。
- **L2944 EN**: Closes the current lexical scope or compound statement.
  **L2944 CN**: 结束当前词法作用域或复合语句块。
- **L2945 EN**: Blank line separating nearby declarations or logic blocks.
  **L2945 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2946 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2946 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2947 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2947 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2948 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_h_b(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2948 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_h_b(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2949 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_h_b((v8i16)_1, (v16i8)_2, (v16i8)_3)`.
  **L2949 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_h_b((v8i16)_1, (v16i8)_2, (v16i8)_3)` 从当前函数返回。
- **L2950 EN**: Closes the current lexical scope or compound statement.
  **L2950 CN**: 结束当前词法作用域或复合语句块。
- **L2951 EN**: Blank line separating nearby declarations or logic blocks.
  **L2951 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2952 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2952 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2953-2976

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_d_wu(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_d_wu((v2u64)_1, (v4u32)_2, (v4u32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_w_hu(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_w_hu((v4u32)_1, (v8u16)_2, (v8u16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_h_bu(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_h_bu((v8u16)_1, (v16u8)_2, (v16u8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_d_w(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_d_w((v2i64)_1, (v4i32)_2, (v4i32)_3);
}

extern __inline
````
- **L2953 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2953 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2954 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_d_wu(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2954 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_d_wu(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2955 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_d_wu((v2u64)_1, (v4u32)_2, (v4u32)_3)`.
  **L2955 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_d_wu((v2u64)_1, (v4u32)_2, (v4u32)_3)` 从当前函数返回。
- **L2956 EN**: Closes the current lexical scope or compound statement.
  **L2956 CN**: 结束当前词法作用域或复合语句块。
- **L2957 EN**: Blank line separating nearby declarations or logic blocks.
  **L2957 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2958 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2958 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2959 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2959 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2960 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_w_hu(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2960 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_w_hu(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2961 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_w_hu((v4u32)_1, (v8u16)_2, (v8u16)_3)`.
  **L2961 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_w_hu((v4u32)_1, (v8u16)_2, (v8u16)_3)` 从当前函数返回。
- **L2962 EN**: Closes the current lexical scope or compound statement.
  **L2962 CN**: 结束当前词法作用域或复合语句块。
- **L2963 EN**: Blank line separating nearby declarations or logic blocks.
  **L2963 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2964 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2964 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2965 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2965 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2966 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_h_bu(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2966 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_h_bu(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2967 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_h_bu((v8u16)_1, (v16u8)_2, (v16u8)_3)`.
  **L2967 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_h_bu((v8u16)_1, (v16u8)_2, (v16u8)_3)` 从当前函数返回。
- **L2968 EN**: Closes the current lexical scope or compound statement.
  **L2968 CN**: 结束当前词法作用域或复合语句块。
- **L2969 EN**: Blank line separating nearby declarations or logic blocks.
  **L2969 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2970 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2970 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2971 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2971 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2972 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_d_w(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2972 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_d_w(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2973 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_d_w((v2i64)_1, (v4i32)_2, (v4i32)_3)`.
  **L2973 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_d_w((v2i64)_1, (v4i32)_2, (v4i32)_3)` 从当前函数返回。
- **L2974 EN**: Closes the current lexical scope or compound statement.
  **L2974 CN**: 结束当前词法作用域或复合语句块。
- **L2975 EN**: Blank line separating nearby declarations or logic blocks.
  **L2975 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2976 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2976 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 2977-3000

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_w_h(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_w_h((v4i32)_1, (v8i16)_2, (v8i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_h_b(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_h_b((v8i16)_1, (v16i8)_2, (v16i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_d_wu(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_d_wu((v2u64)_1, (v4u32)_2, (v4u32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_w_hu(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_w_hu((v4u32)_1, (v8u16)_2, (v8u16)_3);
}

extern __inline
````
- **L2977 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2977 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2978 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_w_h(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2978 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_w_h(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2979 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_w_h((v4i32)_1, (v8i16)_2, (v8i16)_3)`.
  **L2979 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_w_h((v4i32)_1, (v8i16)_2, (v8i16)_3)` 从当前函数返回。
- **L2980 EN**: Closes the current lexical scope or compound statement.
  **L2980 CN**: 结束当前词法作用域或复合语句块。
- **L2981 EN**: Blank line separating nearby declarations or logic blocks.
  **L2981 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2982 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2982 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2983 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2983 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2984 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_h_b(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2984 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_h_b(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2985 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_h_b((v8i16)_1, (v16i8)_2, (v16i8)_3)`.
  **L2985 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_h_b((v8i16)_1, (v16i8)_2, (v16i8)_3)` 从当前函数返回。
- **L2986 EN**: Closes the current lexical scope or compound statement.
  **L2986 CN**: 结束当前词法作用域或复合语句块。
- **L2987 EN**: Blank line separating nearby declarations or logic blocks.
  **L2987 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2988 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2988 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2989 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2989 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2990 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_d_wu(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2990 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_d_wu(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2991 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_d_wu((v2u64)_1, (v4u32)_2, (v4u32)_3)`.
  **L2991 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_d_wu((v2u64)_1, (v4u32)_2, (v4u32)_3)` 从当前函数返回。
- **L2992 EN**: Closes the current lexical scope or compound statement.
  **L2992 CN**: 结束当前词法作用域或复合语句块。
- **L2993 EN**: Blank line separating nearby declarations or logic blocks.
  **L2993 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2994 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L2994 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L2995 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L2995 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L2996 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_w_hu(__m128i _1, __m128i _2, __m128i _3) {`.
  **L2996 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_w_hu(__m128i _1, __m128i _2, __m128i _3) {`。
- **L2997 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_w_hu((v4u32)_1, (v8u16)_2, (v8u16)_3)`.
  **L2997 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_w_hu((v4u32)_1, (v8u16)_2, (v8u16)_3)` 从当前函数返回。
- **L2998 EN**: Closes the current lexical scope or compound statement.
  **L2998 CN**: 结束当前词法作用域或复合语句块。
- **L2999 EN**: Blank line separating nearby declarations or logic blocks.
  **L2999 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3000 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3000 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 3001-3024

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_h_bu(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_h_bu((v8u16)_1, (v16u8)_2, (v16u8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_d_wu_w(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_d_wu_w((v2i64)_1, (v4u32)_2,
                                                (v4i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_w_hu_h(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_w_hu_h((v4i32)_1, (v8u16)_2,
                                                (v8i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_h_bu_b(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_h_bu_b((v8i16)_1, (v16u8)_2,
                                                (v16i8)_3);
````
- **L3001 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3001 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3002 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_h_bu(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3002 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_h_bu(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3003 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_h_bu((v8u16)_1, (v16u8)_2, (v16u8)_3)`.
  **L3003 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_h_bu((v8u16)_1, (v16u8)_2, (v16u8)_3)` 从当前函数返回。
- **L3004 EN**: Closes the current lexical scope or compound statement.
  **L3004 CN**: 结束当前词法作用域或复合语句块。
- **L3005 EN**: Blank line separating nearby declarations or logic blocks.
  **L3005 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3006 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3006 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3007 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3007 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3008 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_d_wu_w(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3008 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_d_wu_w(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3009 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_d_wu_w((v2i64)_1, (v4u32)_2,`.
  **L3009 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_d_wu_w((v2i64)_1, (v4u32)_2,` 从当前函数返回。
- **L3010 EN**: Executes a call or declaration centered on `statement`.
  **L3010 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3011 EN**: Closes the current lexical scope or compound statement.
  **L3011 CN**: 结束当前词法作用域或复合语句块。
- **L3012 EN**: Blank line separating nearby declarations or logic blocks.
  **L3012 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3013 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3013 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3014 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3014 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3015 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_w_hu_h(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3015 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_w_hu_h(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3016 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_w_hu_h((v4i32)_1, (v8u16)_2,`.
  **L3016 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_w_hu_h((v4i32)_1, (v8u16)_2,` 从当前函数返回。
- **L3017 EN**: Executes a call or declaration centered on `statement`.
  **L3017 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3018 EN**: Closes the current lexical scope or compound statement.
  **L3018 CN**: 结束当前词法作用域或复合语句块。
- **L3019 EN**: Blank line separating nearby declarations or logic blocks.
  **L3019 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3020 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3020 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3021 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3021 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3022 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_h_bu_b(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3022 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_h_bu_b(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3023 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_h_bu_b((v8i16)_1, (v16u8)_2,`.
  **L3023 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_h_bu_b((v8i16)_1, (v16u8)_2,` 从当前函数返回。
- **L3024 EN**: Executes a call or declaration centered on `statement`.
  **L3024 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 3025-3048

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_d_wu_w(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_d_wu_w((v2i64)_1, (v4u32)_2,
                                                (v4i32)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_w_hu_h(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_w_hu_h((v4i32)_1, (v8u16)_2,
                                                (v8i16)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_h_bu_b(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_h_bu_b((v8i16)_1, (v16u8)_2,
                                                (v16i8)_3);
}

extern __inline
````
- **L3025 EN**: Closes the current lexical scope or compound statement.
  **L3025 CN**: 结束当前词法作用域或复合语句块。
- **L3026 EN**: Blank line separating nearby declarations or logic blocks.
  **L3026 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3027 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3027 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3028 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3028 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3029 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_d_wu_w(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3029 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_d_wu_w(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3030 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_d_wu_w((v2i64)_1, (v4u32)_2,`.
  **L3030 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_d_wu_w((v2i64)_1, (v4u32)_2,` 从当前函数返回。
- **L3031 EN**: Executes a call or declaration centered on `statement`.
  **L3031 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3032 EN**: Closes the current lexical scope or compound statement.
  **L3032 CN**: 结束当前词法作用域或复合语句块。
- **L3033 EN**: Blank line separating nearby declarations or logic blocks.
  **L3033 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3034 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3034 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3035 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3035 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3036 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_w_hu_h(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3036 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_w_hu_h(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3037 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_w_hu_h((v4i32)_1, (v8u16)_2,`.
  **L3037 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_w_hu_h((v4i32)_1, (v8u16)_2,` 从当前函数返回。
- **L3038 EN**: Executes a call or declaration centered on `statement`.
  **L3038 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3039 EN**: Closes the current lexical scope or compound statement.
  **L3039 CN**: 结束当前词法作用域或复合语句块。
- **L3040 EN**: Blank line separating nearby declarations or logic blocks.
  **L3040 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3041 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3041 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3042 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3042 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3043 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_h_bu_b(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3043 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_h_bu_b(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3044 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_h_bu_b((v8i16)_1, (v16u8)_2,`.
  **L3044 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_h_bu_b((v8i16)_1, (v16u8)_2,` 从当前函数返回。
- **L3045 EN**: Executes a call or declaration centered on `statement`.
  **L3045 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3046 EN**: Closes the current lexical scope or compound statement.
  **L3046 CN**: 结束当前词法作用域或复合语句块。
- **L3047 EN**: Blank line separating nearby declarations or logic blocks.
  **L3047 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3048 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3048 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 3049-3072

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_q_d(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_q_d((v2i64)_1, (v2i64)_2, (v2i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_q_d(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_q_d((v2i64)_1, (v2i64)_2, (v2i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_q_du(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_q_du((v2u64)_1, (v2u64)_2, (v2u64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_q_du(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_q_du((v2u64)_1, (v2u64)_2, (v2u64)_3);
}

extern __inline
````
- **L3049 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3049 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3050 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_q_d(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3050 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_q_d(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3051 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_q_d((v2i64)_1, (v2i64)_2, (v2i64)_3)`.
  **L3051 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_q_d((v2i64)_1, (v2i64)_2, (v2i64)_3)` 从当前函数返回。
- **L3052 EN**: Closes the current lexical scope or compound statement.
  **L3052 CN**: 结束当前词法作用域或复合语句块。
- **L3053 EN**: Blank line separating nearby declarations or logic blocks.
  **L3053 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3054 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3054 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3055 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3055 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3056 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_q_d(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3056 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_q_d(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3057 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_q_d((v2i64)_1, (v2i64)_2, (v2i64)_3)`.
  **L3057 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_q_d((v2i64)_1, (v2i64)_2, (v2i64)_3)` 从当前函数返回。
- **L3058 EN**: Closes the current lexical scope or compound statement.
  **L3058 CN**: 结束当前词法作用域或复合语句块。
- **L3059 EN**: Blank line separating nearby declarations or logic blocks.
  **L3059 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3060 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3060 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3061 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3061 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3062 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_q_du(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3062 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_q_du(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3063 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_q_du((v2u64)_1, (v2u64)_2, (v2u64)_3)`.
  **L3063 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_q_du((v2u64)_1, (v2u64)_2, (v2u64)_3)` 从当前函数返回。
- **L3064 EN**: Closes the current lexical scope or compound statement.
  **L3064 CN**: 结束当前词法作用域或复合语句块。
- **L3065 EN**: Blank line separating nearby declarations or logic blocks.
  **L3065 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3066 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3066 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3067 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3067 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3068 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_q_du(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3068 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_q_du(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3069 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_q_du((v2u64)_1, (v2u64)_2, (v2u64)_3)`.
  **L3069 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_q_du((v2u64)_1, (v2u64)_2, (v2u64)_3)` 从当前函数返回。
- **L3070 EN**: Closes the current lexical scope or compound statement.
  **L3070 CN**: 结束当前词法作用域或复合语句块。
- **L3071 EN**: Blank line separating nearby declarations or logic blocks.
  **L3071 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3072 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3072 CN**: 继续构造周围的表达式或声明：`extern __inline`。

### Lines 3073-3096

````c
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwev_q_du_d(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwev_q_du_d((v2i64)_1, (v2u64)_2,
                                                (v2i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmaddwod_q_du_d(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vmaddwod_q_du_d((v2i64)_1, (v2u64)_2,
                                                (v2i64)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vrotr_b(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vrotr_b((v16i8)_1, (v16i8)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vrotr_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vrotr_h((v8i16)_1, (v8i16)_2);
}
````
- **L3073 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3073 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3074 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwev_q_du_d(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3074 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwev_q_du_d(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3075 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwev_q_du_d((v2i64)_1, (v2u64)_2,`.
  **L3075 CN**: 以 `(__m128i)__builtin_lsx_vmaddwev_q_du_d((v2i64)_1, (v2u64)_2,` 从当前函数返回。
- **L3076 EN**: Executes a call or declaration centered on `statement`.
  **L3076 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3077 EN**: Closes the current lexical scope or compound statement.
  **L3077 CN**: 结束当前词法作用域或复合语句块。
- **L3078 EN**: Blank line separating nearby declarations or logic blocks.
  **L3078 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3079 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3079 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3080 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3080 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3081 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmaddwod_q_du_d(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3081 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmaddwod_q_du_d(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3082 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmaddwod_q_du_d((v2i64)_1, (v2u64)_2,`.
  **L3082 CN**: 以 `(__m128i)__builtin_lsx_vmaddwod_q_du_d((v2i64)_1, (v2u64)_2,` 从当前函数返回。
- **L3083 EN**: Executes a call or declaration centered on `statement`.
  **L3083 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3084 EN**: Closes the current lexical scope or compound statement.
  **L3084 CN**: 结束当前词法作用域或复合语句块。
- **L3085 EN**: Blank line separating nearby declarations or logic blocks.
  **L3085 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3086 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3086 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3087 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3087 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3088 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vrotr_b(__m128i _1, __m128i _2) {`.
  **L3088 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vrotr_b(__m128i _1, __m128i _2) {`。
- **L3089 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vrotr_b((v16i8)_1, (v16i8)_2)`.
  **L3089 CN**: 以 `(__m128i)__builtin_lsx_vrotr_b((v16i8)_1, (v16i8)_2)` 从当前函数返回。
- **L3090 EN**: Closes the current lexical scope or compound statement.
  **L3090 CN**: 结束当前词法作用域或复合语句块。
- **L3091 EN**: Blank line separating nearby declarations or logic blocks.
  **L3091 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3092 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3092 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3093 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3093 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3094 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vrotr_h(__m128i _1, __m128i _2) {`.
  **L3094 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vrotr_h(__m128i _1, __m128i _2) {`。
- **L3095 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vrotr_h((v8i16)_1, (v8i16)_2)`.
  **L3095 CN**: 以 `(__m128i)__builtin_lsx_vrotr_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L3096 EN**: Closes the current lexical scope or compound statement.
  **L3096 CN**: 结束当前词法作用域或复合语句块。

### Lines 3097-3120

````c

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vrotr_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vrotr_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vrotr_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vrotr_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vadd_q(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vadd_q((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vsub_q(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vsub_q((v2i64)_1, (v2i64)_2);
}
````
- **L3097 EN**: Blank line separating nearby declarations or logic blocks.
  **L3097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3098 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3098 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3099 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3099 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3100 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vrotr_w(__m128i _1, __m128i _2) {`.
  **L3100 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vrotr_w(__m128i _1, __m128i _2) {`。
- **L3101 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vrotr_w((v4i32)_1, (v4i32)_2)`.
  **L3101 CN**: 以 `(__m128i)__builtin_lsx_vrotr_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L3102 EN**: Closes the current lexical scope or compound statement.
  **L3102 CN**: 结束当前词法作用域或复合语句块。
- **L3103 EN**: Blank line separating nearby declarations or logic blocks.
  **L3103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3104 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3104 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3105 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3105 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3106 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vrotr_d(__m128i _1, __m128i _2) {`.
  **L3106 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vrotr_d(__m128i _1, __m128i _2) {`。
- **L3107 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vrotr_d((v2i64)_1, (v2i64)_2)`.
  **L3107 CN**: 以 `(__m128i)__builtin_lsx_vrotr_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L3108 EN**: Closes the current lexical scope or compound statement.
  **L3108 CN**: 结束当前词法作用域或复合语句块。
- **L3109 EN**: Blank line separating nearby declarations or logic blocks.
  **L3109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3110 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3110 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3111 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3111 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3112 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vadd_q(__m128i _1, __m128i _2) {`.
  **L3112 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vadd_q(__m128i _1, __m128i _2) {`。
- **L3113 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vadd_q((v2i64)_1, (v2i64)_2)`.
  **L3113 CN**: 以 `(__m128i)__builtin_lsx_vadd_q((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L3114 EN**: Closes the current lexical scope or compound statement.
  **L3114 CN**: 结束当前词法作用域或复合语句块。
- **L3115 EN**: Blank line separating nearby declarations or logic blocks.
  **L3115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3116 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3116 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3117 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3117 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3118 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vsub_q(__m128i _1, __m128i _2) {`.
  **L3118 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vsub_q(__m128i _1, __m128i _2) {`。
- **L3119 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vsub_q((v2i64)_1, (v2i64)_2)`.
  **L3119 CN**: 以 `(__m128i)__builtin_lsx_vsub_q((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L3120 EN**: Closes the current lexical scope or compound statement.
  **L3120 CN**: 结束当前词法作用域或复合语句块。

### Lines 3121-3144

````c

#define __lsx_vldrepl_b(/*void **/ _1, /*si12*/ _2)                            \
  ((__m128i)__builtin_lsx_vldrepl_b((void const *)(_1), (_2)))

#define __lsx_vldrepl_h(/*void **/ _1, /*si11*/ _2)                            \
  ((__m128i)__builtin_lsx_vldrepl_h((void const *)(_1), (_2)))

#define __lsx_vldrepl_w(/*void **/ _1, /*si10*/ _2)                            \
  ((__m128i)__builtin_lsx_vldrepl_w((void const *)(_1), (_2)))

#define __lsx_vldrepl_d(/*void **/ _1, /*si9*/ _2)                             \
  ((__m128i)__builtin_lsx_vldrepl_d((void const *)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmskgez_b(__m128i _1) {
  return (__m128i)__builtin_lsx_vmskgez_b((v16i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vmsknz_b(__m128i _1) {
  return (__m128i)__builtin_lsx_vmsknz_b((v16i8)_1);
}
````
- **L3121 EN**: Blank line separating nearby declarations or logic blocks.
  **L3121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3122 EN**: Defines macro `__lsx_vldrepl_b(/*void **/ _1, /*si12*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3122 CN**: 定义宏 `__lsx_vldrepl_b(/*void **/ _1, /*si12*/ _2)`，用于条件编译、简写或 API 生成。
- **L3123 EN**: Continues logic associated with callable symbol `__builtin_lsx_vldrepl_b`.
  **L3123 CN**: 继续与可调用符号 `__builtin_lsx_vldrepl_b` 相关的逻辑。
- **L3124 EN**: Blank line separating nearby declarations or logic blocks.
  **L3124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3125 EN**: Defines macro `__lsx_vldrepl_h(/*void **/ _1, /*si11*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3125 CN**: 定义宏 `__lsx_vldrepl_h(/*void **/ _1, /*si11*/ _2)`，用于条件编译、简写或 API 生成。
- **L3126 EN**: Continues logic associated with callable symbol `__builtin_lsx_vldrepl_h`.
  **L3126 CN**: 继续与可调用符号 `__builtin_lsx_vldrepl_h` 相关的逻辑。
- **L3127 EN**: Blank line separating nearby declarations or logic blocks.
  **L3127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3128 EN**: Defines macro `__lsx_vldrepl_w(/*void **/ _1, /*si10*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3128 CN**: 定义宏 `__lsx_vldrepl_w(/*void **/ _1, /*si10*/ _2)`，用于条件编译、简写或 API 生成。
- **L3129 EN**: Continues logic associated with callable symbol `__builtin_lsx_vldrepl_w`.
  **L3129 CN**: 继续与可调用符号 `__builtin_lsx_vldrepl_w` 相关的逻辑。
- **L3130 EN**: Blank line separating nearby declarations or logic blocks.
  **L3130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3131 EN**: Defines macro `__lsx_vldrepl_d(/*void **/ _1, /*si9*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3131 CN**: 定义宏 `__lsx_vldrepl_d(/*void **/ _1, /*si9*/ _2)`，用于条件编译、简写或 API 生成。
- **L3132 EN**: Continues logic associated with callable symbol `__builtin_lsx_vldrepl_d`.
  **L3132 CN**: 继续与可调用符号 `__builtin_lsx_vldrepl_d` 相关的逻辑。
- **L3133 EN**: Blank line separating nearby declarations or logic blocks.
  **L3133 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3134 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3134 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3135 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3135 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3136 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmskgez_b(__m128i _1) {`.
  **L3136 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmskgez_b(__m128i _1) {`。
- **L3137 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmskgez_b((v16i8)_1)`.
  **L3137 CN**: 以 `(__m128i)__builtin_lsx_vmskgez_b((v16i8)_1)` 从当前函数返回。
- **L3138 EN**: Closes the current lexical scope or compound statement.
  **L3138 CN**: 结束当前词法作用域或复合语句块。
- **L3139 EN**: Blank line separating nearby declarations or logic blocks.
  **L3139 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3140 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3140 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3141 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3141 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vmsknz_b(__m128i _1) {`.
  **L3142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vmsknz_b(__m128i _1) {`。
- **L3143 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vmsknz_b((v16i8)_1)`.
  **L3143 CN**: 以 `(__m128i)__builtin_lsx_vmsknz_b((v16i8)_1)` 从当前函数返回。
- **L3144 EN**: Closes the current lexical scope or compound statement.
  **L3144 CN**: 结束当前词法作用域或复合语句块。

### Lines 3145-3168

````c

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vexth_h_b(__m128i _1) {
  return (__m128i)__builtin_lsx_vexth_h_b((v16i8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vexth_w_h(__m128i _1) {
  return (__m128i)__builtin_lsx_vexth_w_h((v8i16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vexth_d_w(__m128i _1) {
  return (__m128i)__builtin_lsx_vexth_d_w((v4i32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vexth_q_d(__m128i _1) {
  return (__m128i)__builtin_lsx_vexth_q_d((v2i64)_1);
}
````
- **L3145 EN**: Blank line separating nearby declarations or logic blocks.
  **L3145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3146 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3146 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3147 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3147 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3148 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vexth_h_b(__m128i _1) {`.
  **L3148 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vexth_h_b(__m128i _1) {`。
- **L3149 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vexth_h_b((v16i8)_1)`.
  **L3149 CN**: 以 `(__m128i)__builtin_lsx_vexth_h_b((v16i8)_1)` 从当前函数返回。
- **L3150 EN**: Closes the current lexical scope or compound statement.
  **L3150 CN**: 结束当前词法作用域或复合语句块。
- **L3151 EN**: Blank line separating nearby declarations or logic blocks.
  **L3151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3152 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3152 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3153 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3153 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3154 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vexth_w_h(__m128i _1) {`.
  **L3154 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vexth_w_h(__m128i _1) {`。
- **L3155 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vexth_w_h((v8i16)_1)`.
  **L3155 CN**: 以 `(__m128i)__builtin_lsx_vexth_w_h((v8i16)_1)` 从当前函数返回。
- **L3156 EN**: Closes the current lexical scope or compound statement.
  **L3156 CN**: 结束当前词法作用域或复合语句块。
- **L3157 EN**: Blank line separating nearby declarations or logic blocks.
  **L3157 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3158 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3158 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3159 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3159 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3160 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vexth_d_w(__m128i _1) {`.
  **L3160 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vexth_d_w(__m128i _1) {`。
- **L3161 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vexth_d_w((v4i32)_1)`.
  **L3161 CN**: 以 `(__m128i)__builtin_lsx_vexth_d_w((v4i32)_1)` 从当前函数返回。
- **L3162 EN**: Closes the current lexical scope or compound statement.
  **L3162 CN**: 结束当前词法作用域或复合语句块。
- **L3163 EN**: Blank line separating nearby declarations or logic blocks.
  **L3163 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3164 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3164 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3165 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3165 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3166 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vexth_q_d(__m128i _1) {`.
  **L3166 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vexth_q_d(__m128i _1) {`。
- **L3167 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vexth_q_d((v2i64)_1)`.
  **L3167 CN**: 以 `(__m128i)__builtin_lsx_vexth_q_d((v2i64)_1)` 从当前函数返回。
- **L3168 EN**: Closes the current lexical scope or compound statement.
  **L3168 CN**: 结束当前词法作用域或复合语句块。

### Lines 3169-3192

````c

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vexth_hu_bu(__m128i _1) {
  return (__m128i)__builtin_lsx_vexth_hu_bu((v16u8)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vexth_wu_hu(__m128i _1) {
  return (__m128i)__builtin_lsx_vexth_wu_hu((v8u16)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vexth_du_wu(__m128i _1) {
  return (__m128i)__builtin_lsx_vexth_du_wu((v4u32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vexth_qu_du(__m128i _1) {
  return (__m128i)__builtin_lsx_vexth_qu_du((v2u64)_1);
}
````
- **L3169 EN**: Blank line separating nearby declarations or logic blocks.
  **L3169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3170 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3170 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3171 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3171 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3172 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vexth_hu_bu(__m128i _1) {`.
  **L3172 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vexth_hu_bu(__m128i _1) {`。
- **L3173 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vexth_hu_bu((v16u8)_1)`.
  **L3173 CN**: 以 `(__m128i)__builtin_lsx_vexth_hu_bu((v16u8)_1)` 从当前函数返回。
- **L3174 EN**: Closes the current lexical scope or compound statement.
  **L3174 CN**: 结束当前词法作用域或复合语句块。
- **L3175 EN**: Blank line separating nearby declarations or logic blocks.
  **L3175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3176 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3176 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3177 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3177 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3178 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vexth_wu_hu(__m128i _1) {`.
  **L3178 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vexth_wu_hu(__m128i _1) {`。
- **L3179 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vexth_wu_hu((v8u16)_1)`.
  **L3179 CN**: 以 `(__m128i)__builtin_lsx_vexth_wu_hu((v8u16)_1)` 从当前函数返回。
- **L3180 EN**: Closes the current lexical scope or compound statement.
  **L3180 CN**: 结束当前词法作用域或复合语句块。
- **L3181 EN**: Blank line separating nearby declarations or logic blocks.
  **L3181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3182 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3182 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3183 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3183 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3184 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vexth_du_wu(__m128i _1) {`.
  **L3184 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vexth_du_wu(__m128i _1) {`。
- **L3185 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vexth_du_wu((v4u32)_1)`.
  **L3185 CN**: 以 `(__m128i)__builtin_lsx_vexth_du_wu((v4u32)_1)` 从当前函数返回。
- **L3186 EN**: Closes the current lexical scope or compound statement.
  **L3186 CN**: 结束当前词法作用域或复合语句块。
- **L3187 EN**: Blank line separating nearby declarations or logic blocks.
  **L3187 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3188 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3188 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3189 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3189 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3190 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vexth_qu_du(__m128i _1) {`.
  **L3190 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vexth_qu_du(__m128i _1) {`。
- **L3191 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vexth_qu_du((v2u64)_1)`.
  **L3191 CN**: 以 `(__m128i)__builtin_lsx_vexth_qu_du((v2u64)_1)` 从当前函数返回。
- **L3192 EN**: Closes the current lexical scope or compound statement.
  **L3192 CN**: 结束当前词法作用域或复合语句块。

### Lines 3193-3216

````c

#define __lsx_vrotri_b(/*__m128i*/ _1, /*ui3*/ _2)                             \
  ((__m128i)__builtin_lsx_vrotri_b((v16i8)(_1), (_2)))

#define __lsx_vrotri_h(/*__m128i*/ _1, /*ui4*/ _2)                             \
  ((__m128i)__builtin_lsx_vrotri_h((v8i16)(_1), (_2)))

#define __lsx_vrotri_w(/*__m128i*/ _1, /*ui5*/ _2)                             \
  ((__m128i)__builtin_lsx_vrotri_w((v4i32)(_1), (_2)))

#define __lsx_vrotri_d(/*__m128i*/ _1, /*ui6*/ _2)                             \
  ((__m128i)__builtin_lsx_vrotri_d((v2i64)(_1), (_2)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vextl_q_d(__m128i _1) {
  return (__m128i)__builtin_lsx_vextl_q_d((v2i64)_1);
}

#define __lsx_vsrlni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)           \
  ((__m128i)__builtin_lsx_vsrlni_b_h((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vsrlni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)           \
  ((__m128i)__builtin_lsx_vsrlni_h_w((v8i16)(_1), (v8i16)(_2), (_3)))
````
- **L3193 EN**: Blank line separating nearby declarations or logic blocks.
  **L3193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3194 EN**: Defines macro `__lsx_vrotri_b(/*__m128i*/ _1, /*ui3*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3194 CN**: 定义宏 `__lsx_vrotri_b(/*__m128i*/ _1, /*ui3*/ _2)`，用于条件编译、简写或 API 生成。
- **L3195 EN**: Continues logic associated with callable symbol `__builtin_lsx_vrotri_b`.
  **L3195 CN**: 继续与可调用符号 `__builtin_lsx_vrotri_b` 相关的逻辑。
- **L3196 EN**: Blank line separating nearby declarations or logic blocks.
  **L3196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3197 EN**: Defines macro `__lsx_vrotri_h(/*__m128i*/ _1, /*ui4*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3197 CN**: 定义宏 `__lsx_vrotri_h(/*__m128i*/ _1, /*ui4*/ _2)`，用于条件编译、简写或 API 生成。
- **L3198 EN**: Continues logic associated with callable symbol `__builtin_lsx_vrotri_h`.
  **L3198 CN**: 继续与可调用符号 `__builtin_lsx_vrotri_h` 相关的逻辑。
- **L3199 EN**: Blank line separating nearby declarations or logic blocks.
  **L3199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3200 EN**: Defines macro `__lsx_vrotri_w(/*__m128i*/ _1, /*ui5*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3200 CN**: 定义宏 `__lsx_vrotri_w(/*__m128i*/ _1, /*ui5*/ _2)`，用于条件编译、简写或 API 生成。
- **L3201 EN**: Continues logic associated with callable symbol `__builtin_lsx_vrotri_w`.
  **L3201 CN**: 继续与可调用符号 `__builtin_lsx_vrotri_w` 相关的逻辑。
- **L3202 EN**: Blank line separating nearby declarations or logic blocks.
  **L3202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3203 EN**: Defines macro `__lsx_vrotri_d(/*__m128i*/ _1, /*ui6*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3203 CN**: 定义宏 `__lsx_vrotri_d(/*__m128i*/ _1, /*ui6*/ _2)`，用于条件编译、简写或 API 生成。
- **L3204 EN**: Continues logic associated with callable symbol `__builtin_lsx_vrotri_d`.
  **L3204 CN**: 继续与可调用符号 `__builtin_lsx_vrotri_d` 相关的逻辑。
- **L3205 EN**: Blank line separating nearby declarations or logic blocks.
  **L3205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3206 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3206 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3207 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3207 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3208 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vextl_q_d(__m128i _1) {`.
  **L3208 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vextl_q_d(__m128i _1) {`。
- **L3209 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vextl_q_d((v2i64)_1)`.
  **L3209 CN**: 以 `(__m128i)__builtin_lsx_vextl_q_d((v2i64)_1)` 从当前函数返回。
- **L3210 EN**: Closes the current lexical scope or compound statement.
  **L3210 CN**: 结束当前词法作用域或复合语句块。
- **L3211 EN**: Blank line separating nearby declarations or logic blocks.
  **L3211 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3212 EN**: Defines macro `__lsx_vsrlni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3212 CN**: 定义宏 `__lsx_vsrlni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3213 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlni_b_h`.
  **L3213 CN**: 继续与可调用符号 `__builtin_lsx_vsrlni_b_h` 相关的逻辑。
- **L3214 EN**: Blank line separating nearby declarations or logic blocks.
  **L3214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3215 EN**: Defines macro `__lsx_vsrlni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3215 CN**: 定义宏 `__lsx_vsrlni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3216 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlni_h_w`.
  **L3216 CN**: 继续与可调用符号 `__builtin_lsx_vsrlni_h_w` 相关的逻辑。

### Lines 3217-3240

````c

#define __lsx_vsrlni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)           \
  ((__m128i)__builtin_lsx_vsrlni_w_d((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vsrlni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)           \
  ((__m128i)__builtin_lsx_vsrlni_d_q((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vsrlrni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)          \
  ((__m128i)__builtin_lsx_vsrlrni_b_h((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vsrlrni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)          \
  ((__m128i)__builtin_lsx_vsrlrni_h_w((v8i16)(_1), (v8i16)(_2), (_3)))

#define __lsx_vsrlrni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)          \
  ((__m128i)__builtin_lsx_vsrlrni_w_d((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vsrlrni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)          \
  ((__m128i)__builtin_lsx_vsrlrni_d_q((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vssrlni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)          \
  ((__m128i)__builtin_lsx_vssrlni_b_h((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vssrlni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)          \
  ((__m128i)__builtin_lsx_vssrlni_h_w((v8i16)(_1), (v8i16)(_2), (_3)))
````
- **L3217 EN**: Blank line separating nearby declarations or logic blocks.
  **L3217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3218 EN**: Defines macro `__lsx_vsrlni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3218 CN**: 定义宏 `__lsx_vsrlni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3219 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlni_w_d`.
  **L3219 CN**: 继续与可调用符号 `__builtin_lsx_vsrlni_w_d` 相关的逻辑。
- **L3220 EN**: Blank line separating nearby declarations or logic blocks.
  **L3220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3221 EN**: Defines macro `__lsx_vsrlni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3221 CN**: 定义宏 `__lsx_vsrlni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3222 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlni_d_q`.
  **L3222 CN**: 继续与可调用符号 `__builtin_lsx_vsrlni_d_q` 相关的逻辑。
- **L3223 EN**: Blank line separating nearby declarations or logic blocks.
  **L3223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3224 EN**: Defines macro `__lsx_vsrlrni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3224 CN**: 定义宏 `__lsx_vsrlrni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3225 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlrni_b_h`.
  **L3225 CN**: 继续与可调用符号 `__builtin_lsx_vsrlrni_b_h` 相关的逻辑。
- **L3226 EN**: Blank line separating nearby declarations or logic blocks.
  **L3226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3227 EN**: Defines macro `__lsx_vsrlrni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3227 CN**: 定义宏 `__lsx_vsrlrni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3228 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlrni_h_w`.
  **L3228 CN**: 继续与可调用符号 `__builtin_lsx_vsrlrni_h_w` 相关的逻辑。
- **L3229 EN**: Blank line separating nearby declarations or logic blocks.
  **L3229 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3230 EN**: Defines macro `__lsx_vsrlrni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3230 CN**: 定义宏 `__lsx_vsrlrni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3231 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlrni_w_d`.
  **L3231 CN**: 继续与可调用符号 `__builtin_lsx_vsrlrni_w_d` 相关的逻辑。
- **L3232 EN**: Blank line separating nearby declarations or logic blocks.
  **L3232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3233 EN**: Defines macro `__lsx_vsrlrni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3233 CN**: 定义宏 `__lsx_vsrlrni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3234 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrlrni_d_q`.
  **L3234 CN**: 继续与可调用符号 `__builtin_lsx_vsrlrni_d_q` 相关的逻辑。
- **L3235 EN**: Blank line separating nearby declarations or logic blocks.
  **L3235 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3236 EN**: Defines macro `__lsx_vssrlni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3236 CN**: 定义宏 `__lsx_vssrlni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3237 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlni_b_h`.
  **L3237 CN**: 继续与可调用符号 `__builtin_lsx_vssrlni_b_h` 相关的逻辑。
- **L3238 EN**: Blank line separating nearby declarations or logic blocks.
  **L3238 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3239 EN**: Defines macro `__lsx_vssrlni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3239 CN**: 定义宏 `__lsx_vssrlni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3240 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlni_h_w`.
  **L3240 CN**: 继续与可调用符号 `__builtin_lsx_vssrlni_h_w` 相关的逻辑。

### Lines 3241-3264

````c

#define __lsx_vssrlni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)          \
  ((__m128i)__builtin_lsx_vssrlni_w_d((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vssrlni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)          \
  ((__m128i)__builtin_lsx_vssrlni_d_q((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vssrlni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)         \
  ((__m128i)__builtin_lsx_vssrlni_bu_h((v16u8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vssrlni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)         \
  ((__m128i)__builtin_lsx_vssrlni_hu_w((v8u16)(_1), (v8i16)(_2), (_3)))

#define __lsx_vssrlni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)         \
  ((__m128i)__builtin_lsx_vssrlni_wu_d((v4u32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vssrlni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)         \
  ((__m128i)__builtin_lsx_vssrlni_du_q((v2u64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vssrlrni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)         \
  ((__m128i)__builtin_lsx_vssrlrni_b_h((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vssrlrni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)         \
  ((__m128i)__builtin_lsx_vssrlrni_h_w((v8i16)(_1), (v8i16)(_2), (_3)))
````
- **L3241 EN**: Blank line separating nearby declarations or logic blocks.
  **L3241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3242 EN**: Defines macro `__lsx_vssrlni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3242 CN**: 定义宏 `__lsx_vssrlni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3243 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlni_w_d`.
  **L3243 CN**: 继续与可调用符号 `__builtin_lsx_vssrlni_w_d` 相关的逻辑。
- **L3244 EN**: Blank line separating nearby declarations or logic blocks.
  **L3244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3245 EN**: Defines macro `__lsx_vssrlni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3245 CN**: 定义宏 `__lsx_vssrlni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3246 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlni_d_q`.
  **L3246 CN**: 继续与可调用符号 `__builtin_lsx_vssrlni_d_q` 相关的逻辑。
- **L3247 EN**: Blank line separating nearby declarations or logic blocks.
  **L3247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3248 EN**: Defines macro `__lsx_vssrlni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3248 CN**: 定义宏 `__lsx_vssrlni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3249 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlni_bu_h`.
  **L3249 CN**: 继续与可调用符号 `__builtin_lsx_vssrlni_bu_h` 相关的逻辑。
- **L3250 EN**: Blank line separating nearby declarations or logic blocks.
  **L3250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3251 EN**: Defines macro `__lsx_vssrlni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3251 CN**: 定义宏 `__lsx_vssrlni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3252 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlni_hu_w`.
  **L3252 CN**: 继续与可调用符号 `__builtin_lsx_vssrlni_hu_w` 相关的逻辑。
- **L3253 EN**: Blank line separating nearby declarations or logic blocks.
  **L3253 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3254 EN**: Defines macro `__lsx_vssrlni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3254 CN**: 定义宏 `__lsx_vssrlni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3255 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlni_wu_d`.
  **L3255 CN**: 继续与可调用符号 `__builtin_lsx_vssrlni_wu_d` 相关的逻辑。
- **L3256 EN**: Blank line separating nearby declarations or logic blocks.
  **L3256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3257 EN**: Defines macro `__lsx_vssrlni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3257 CN**: 定义宏 `__lsx_vssrlni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3258 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlni_du_q`.
  **L3258 CN**: 继续与可调用符号 `__builtin_lsx_vssrlni_du_q` 相关的逻辑。
- **L3259 EN**: Blank line separating nearby declarations or logic blocks.
  **L3259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3260 EN**: Defines macro `__lsx_vssrlrni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3260 CN**: 定义宏 `__lsx_vssrlrni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3261 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlrni_b_h`.
  **L3261 CN**: 继续与可调用符号 `__builtin_lsx_vssrlrni_b_h` 相关的逻辑。
- **L3262 EN**: Blank line separating nearby declarations or logic blocks.
  **L3262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3263 EN**: Defines macro `__lsx_vssrlrni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3263 CN**: 定义宏 `__lsx_vssrlrni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3264 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlrni_h_w`.
  **L3264 CN**: 继续与可调用符号 `__builtin_lsx_vssrlrni_h_w` 相关的逻辑。

### Lines 3265-3288

````c

#define __lsx_vssrlrni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)         \
  ((__m128i)__builtin_lsx_vssrlrni_w_d((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vssrlrni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)         \
  ((__m128i)__builtin_lsx_vssrlrni_d_q((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vssrlrni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)        \
  ((__m128i)__builtin_lsx_vssrlrni_bu_h((v16u8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vssrlrni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)        \
  ((__m128i)__builtin_lsx_vssrlrni_hu_w((v8u16)(_1), (v8i16)(_2), (_3)))

#define __lsx_vssrlrni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)        \
  ((__m128i)__builtin_lsx_vssrlrni_wu_d((v4u32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vssrlrni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)        \
  ((__m128i)__builtin_lsx_vssrlrni_du_q((v2u64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vsrani_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)           \
  ((__m128i)__builtin_lsx_vsrani_b_h((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vsrani_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)           \
  ((__m128i)__builtin_lsx_vsrani_h_w((v8i16)(_1), (v8i16)(_2), (_3)))
````
- **L3265 EN**: Blank line separating nearby declarations or logic blocks.
  **L3265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3266 EN**: Defines macro `__lsx_vssrlrni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3266 CN**: 定义宏 `__lsx_vssrlrni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3267 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlrni_w_d`.
  **L3267 CN**: 继续与可调用符号 `__builtin_lsx_vssrlrni_w_d` 相关的逻辑。
- **L3268 EN**: Blank line separating nearby declarations or logic blocks.
  **L3268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3269 EN**: Defines macro `__lsx_vssrlrni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3269 CN**: 定义宏 `__lsx_vssrlrni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3270 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlrni_d_q`.
  **L3270 CN**: 继续与可调用符号 `__builtin_lsx_vssrlrni_d_q` 相关的逻辑。
- **L3271 EN**: Blank line separating nearby declarations or logic blocks.
  **L3271 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3272 EN**: Defines macro `__lsx_vssrlrni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3272 CN**: 定义宏 `__lsx_vssrlrni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3273 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlrni_bu_h`.
  **L3273 CN**: 继续与可调用符号 `__builtin_lsx_vssrlrni_bu_h` 相关的逻辑。
- **L3274 EN**: Blank line separating nearby declarations or logic blocks.
  **L3274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3275 EN**: Defines macro `__lsx_vssrlrni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3275 CN**: 定义宏 `__lsx_vssrlrni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3276 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlrni_hu_w`.
  **L3276 CN**: 继续与可调用符号 `__builtin_lsx_vssrlrni_hu_w` 相关的逻辑。
- **L3277 EN**: Blank line separating nearby declarations or logic blocks.
  **L3277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3278 EN**: Defines macro `__lsx_vssrlrni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3278 CN**: 定义宏 `__lsx_vssrlrni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3279 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlrni_wu_d`.
  **L3279 CN**: 继续与可调用符号 `__builtin_lsx_vssrlrni_wu_d` 相关的逻辑。
- **L3280 EN**: Blank line separating nearby declarations or logic blocks.
  **L3280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3281 EN**: Defines macro `__lsx_vssrlrni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3281 CN**: 定义宏 `__lsx_vssrlrni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3282 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrlrni_du_q`.
  **L3282 CN**: 继续与可调用符号 `__builtin_lsx_vssrlrni_du_q` 相关的逻辑。
- **L3283 EN**: Blank line separating nearby declarations or logic blocks.
  **L3283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3284 EN**: Defines macro `__lsx_vsrani_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3284 CN**: 定义宏 `__lsx_vsrani_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3285 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrani_b_h`.
  **L3285 CN**: 继续与可调用符号 `__builtin_lsx_vsrani_b_h` 相关的逻辑。
- **L3286 EN**: Blank line separating nearby declarations or logic blocks.
  **L3286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3287 EN**: Defines macro `__lsx_vsrani_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3287 CN**: 定义宏 `__lsx_vsrani_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3288 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrani_h_w`.
  **L3288 CN**: 继续与可调用符号 `__builtin_lsx_vsrani_h_w` 相关的逻辑。

### Lines 3289-3312

````c

#define __lsx_vsrani_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)           \
  ((__m128i)__builtin_lsx_vsrani_w_d((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vsrani_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)           \
  ((__m128i)__builtin_lsx_vsrani_d_q((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vsrarni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)          \
  ((__m128i)__builtin_lsx_vsrarni_b_h((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vsrarni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)          \
  ((__m128i)__builtin_lsx_vsrarni_h_w((v8i16)(_1), (v8i16)(_2), (_3)))

#define __lsx_vsrarni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)          \
  ((__m128i)__builtin_lsx_vsrarni_w_d((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vsrarni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)          \
  ((__m128i)__builtin_lsx_vsrarni_d_q((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vssrani_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)          \
  ((__m128i)__builtin_lsx_vssrani_b_h((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vssrani_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)          \
  ((__m128i)__builtin_lsx_vssrani_h_w((v8i16)(_1), (v8i16)(_2), (_3)))
````
- **L3289 EN**: Blank line separating nearby declarations or logic blocks.
  **L3289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3290 EN**: Defines macro `__lsx_vsrani_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3290 CN**: 定义宏 `__lsx_vsrani_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3291 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrani_w_d`.
  **L3291 CN**: 继续与可调用符号 `__builtin_lsx_vsrani_w_d` 相关的逻辑。
- **L3292 EN**: Blank line separating nearby declarations or logic blocks.
  **L3292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3293 EN**: Defines macro `__lsx_vsrani_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3293 CN**: 定义宏 `__lsx_vsrani_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3294 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrani_d_q`.
  **L3294 CN**: 继续与可调用符号 `__builtin_lsx_vsrani_d_q` 相关的逻辑。
- **L3295 EN**: Blank line separating nearby declarations or logic blocks.
  **L3295 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3296 EN**: Defines macro `__lsx_vsrarni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3296 CN**: 定义宏 `__lsx_vsrarni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3297 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrarni_b_h`.
  **L3297 CN**: 继续与可调用符号 `__builtin_lsx_vsrarni_b_h` 相关的逻辑。
- **L3298 EN**: Blank line separating nearby declarations or logic blocks.
  **L3298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3299 EN**: Defines macro `__lsx_vsrarni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3299 CN**: 定义宏 `__lsx_vsrarni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3300 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrarni_h_w`.
  **L3300 CN**: 继续与可调用符号 `__builtin_lsx_vsrarni_h_w` 相关的逻辑。
- **L3301 EN**: Blank line separating nearby declarations or logic blocks.
  **L3301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3302 EN**: Defines macro `__lsx_vsrarni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3302 CN**: 定义宏 `__lsx_vsrarni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3303 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrarni_w_d`.
  **L3303 CN**: 继续与可调用符号 `__builtin_lsx_vsrarni_w_d` 相关的逻辑。
- **L3304 EN**: Blank line separating nearby declarations or logic blocks.
  **L3304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3305 EN**: Defines macro `__lsx_vsrarni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3305 CN**: 定义宏 `__lsx_vsrarni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3306 EN**: Continues logic associated with callable symbol `__builtin_lsx_vsrarni_d_q`.
  **L3306 CN**: 继续与可调用符号 `__builtin_lsx_vsrarni_d_q` 相关的逻辑。
- **L3307 EN**: Blank line separating nearby declarations or logic blocks.
  **L3307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3308 EN**: Defines macro `__lsx_vssrani_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3308 CN**: 定义宏 `__lsx_vssrani_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3309 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrani_b_h`.
  **L3309 CN**: 继续与可调用符号 `__builtin_lsx_vssrani_b_h` 相关的逻辑。
- **L3310 EN**: Blank line separating nearby declarations or logic blocks.
  **L3310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3311 EN**: Defines macro `__lsx_vssrani_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3311 CN**: 定义宏 `__lsx_vssrani_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3312 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrani_h_w`.
  **L3312 CN**: 继续与可调用符号 `__builtin_lsx_vssrani_h_w` 相关的逻辑。

### Lines 3313-3336

````c

#define __lsx_vssrani_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)          \
  ((__m128i)__builtin_lsx_vssrani_w_d((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vssrani_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)          \
  ((__m128i)__builtin_lsx_vssrani_d_q((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vssrani_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)         \
  ((__m128i)__builtin_lsx_vssrani_bu_h((v16u8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vssrani_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)         \
  ((__m128i)__builtin_lsx_vssrani_hu_w((v8u16)(_1), (v8i16)(_2), (_3)))

#define __lsx_vssrani_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)         \
  ((__m128i)__builtin_lsx_vssrani_wu_d((v4u32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vssrani_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)         \
  ((__m128i)__builtin_lsx_vssrani_du_q((v2u64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vssrarni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)         \
  ((__m128i)__builtin_lsx_vssrarni_b_h((v16i8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vssrarni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)         \
  ((__m128i)__builtin_lsx_vssrarni_h_w((v8i16)(_1), (v8i16)(_2), (_3)))
````
- **L3313 EN**: Blank line separating nearby declarations or logic blocks.
  **L3313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3314 EN**: Defines macro `__lsx_vssrani_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3314 CN**: 定义宏 `__lsx_vssrani_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3315 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrani_w_d`.
  **L3315 CN**: 继续与可调用符号 `__builtin_lsx_vssrani_w_d` 相关的逻辑。
- **L3316 EN**: Blank line separating nearby declarations or logic blocks.
  **L3316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3317 EN**: Defines macro `__lsx_vssrani_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3317 CN**: 定义宏 `__lsx_vssrani_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3318 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrani_d_q`.
  **L3318 CN**: 继续与可调用符号 `__builtin_lsx_vssrani_d_q` 相关的逻辑。
- **L3319 EN**: Blank line separating nearby declarations or logic blocks.
  **L3319 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3320 EN**: Defines macro `__lsx_vssrani_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3320 CN**: 定义宏 `__lsx_vssrani_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3321 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrani_bu_h`.
  **L3321 CN**: 继续与可调用符号 `__builtin_lsx_vssrani_bu_h` 相关的逻辑。
- **L3322 EN**: Blank line separating nearby declarations or logic blocks.
  **L3322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3323 EN**: Defines macro `__lsx_vssrani_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3323 CN**: 定义宏 `__lsx_vssrani_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3324 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrani_hu_w`.
  **L3324 CN**: 继续与可调用符号 `__builtin_lsx_vssrani_hu_w` 相关的逻辑。
- **L3325 EN**: Blank line separating nearby declarations or logic blocks.
  **L3325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3326 EN**: Defines macro `__lsx_vssrani_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3326 CN**: 定义宏 `__lsx_vssrani_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3327 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrani_wu_d`.
  **L3327 CN**: 继续与可调用符号 `__builtin_lsx_vssrani_wu_d` 相关的逻辑。
- **L3328 EN**: Blank line separating nearby declarations or logic blocks.
  **L3328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3329 EN**: Defines macro `__lsx_vssrani_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3329 CN**: 定义宏 `__lsx_vssrani_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3330 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrani_du_q`.
  **L3330 CN**: 继续与可调用符号 `__builtin_lsx_vssrani_du_q` 相关的逻辑。
- **L3331 EN**: Blank line separating nearby declarations or logic blocks.
  **L3331 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3332 EN**: Defines macro `__lsx_vssrarni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3332 CN**: 定义宏 `__lsx_vssrarni_b_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3333 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrarni_b_h`.
  **L3333 CN**: 继续与可调用符号 `__builtin_lsx_vssrarni_b_h` 相关的逻辑。
- **L3334 EN**: Blank line separating nearby declarations or logic blocks.
  **L3334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3335 EN**: Defines macro `__lsx_vssrarni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3335 CN**: 定义宏 `__lsx_vssrarni_h_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3336 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrarni_h_w`.
  **L3336 CN**: 继续与可调用符号 `__builtin_lsx_vssrarni_h_w` 相关的逻辑。

### Lines 3337-3360

````c

#define __lsx_vssrarni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)         \
  ((__m128i)__builtin_lsx_vssrarni_w_d((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vssrarni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)         \
  ((__m128i)__builtin_lsx_vssrarni_d_q((v2i64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vssrarni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)        \
  ((__m128i)__builtin_lsx_vssrarni_bu_h((v16u8)(_1), (v16i8)(_2), (_3)))

#define __lsx_vssrarni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)        \
  ((__m128i)__builtin_lsx_vssrarni_hu_w((v8u16)(_1), (v8i16)(_2), (_3)))

#define __lsx_vssrarni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)        \
  ((__m128i)__builtin_lsx_vssrarni_wu_d((v4u32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vssrarni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)        \
  ((__m128i)__builtin_lsx_vssrarni_du_q((v2u64)(_1), (v2i64)(_2), (_3)))

#define __lsx_vpermi_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)             \
  ((__m128i)__builtin_lsx_vpermi_w((v4i32)(_1), (v4i32)(_2), (_3)))

#define __lsx_vld(/*void **/ _1, /*si12*/ _2)                                  \
  ((__m128i)__builtin_lsx_vld((void const *)(_1), (_2)))
````
- **L3337 EN**: Blank line separating nearby declarations or logic blocks.
  **L3337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3338 EN**: Defines macro `__lsx_vssrarni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3338 CN**: 定义宏 `__lsx_vssrarni_w_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3339 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrarni_w_d`.
  **L3339 CN**: 继续与可调用符号 `__builtin_lsx_vssrarni_w_d` 相关的逻辑。
- **L3340 EN**: Blank line separating nearby declarations or logic blocks.
  **L3340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3341 EN**: Defines macro `__lsx_vssrarni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3341 CN**: 定义宏 `__lsx_vssrarni_d_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3342 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrarni_d_q`.
  **L3342 CN**: 继续与可调用符号 `__builtin_lsx_vssrarni_d_q` 相关的逻辑。
- **L3343 EN**: Blank line separating nearby declarations or logic blocks.
  **L3343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3344 EN**: Defines macro `__lsx_vssrarni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3344 CN**: 定义宏 `__lsx_vssrarni_bu_h(/*__m128i*/ _1, /*__m128i*/ _2, /*ui4*/ _3)`，用于条件编译、简写或 API 生成。
- **L3345 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrarni_bu_h`.
  **L3345 CN**: 继续与可调用符号 `__builtin_lsx_vssrarni_bu_h` 相关的逻辑。
- **L3346 EN**: Blank line separating nearby declarations or logic blocks.
  **L3346 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3347 EN**: Defines macro `__lsx_vssrarni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3347 CN**: 定义宏 `__lsx_vssrarni_hu_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui5*/ _3)`，用于条件编译、简写或 API 生成。
- **L3348 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrarni_hu_w`.
  **L3348 CN**: 继续与可调用符号 `__builtin_lsx_vssrarni_hu_w` 相关的逻辑。
- **L3349 EN**: Blank line separating nearby declarations or logic blocks.
  **L3349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3350 EN**: Defines macro `__lsx_vssrarni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3350 CN**: 定义宏 `__lsx_vssrarni_wu_d(/*__m128i*/ _1, /*__m128i*/ _2, /*ui6*/ _3)`，用于条件编译、简写或 API 生成。
- **L3351 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrarni_wu_d`.
  **L3351 CN**: 继续与可调用符号 `__builtin_lsx_vssrarni_wu_d` 相关的逻辑。
- **L3352 EN**: Blank line separating nearby declarations or logic blocks.
  **L3352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3353 EN**: Defines macro `__lsx_vssrarni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3353 CN**: 定义宏 `__lsx_vssrarni_du_q(/*__m128i*/ _1, /*__m128i*/ _2, /*ui7*/ _3)`，用于条件编译、简写或 API 生成。
- **L3354 EN**: Continues logic associated with callable symbol `__builtin_lsx_vssrarni_du_q`.
  **L3354 CN**: 继续与可调用符号 `__builtin_lsx_vssrarni_du_q` 相关的逻辑。
- **L3355 EN**: Blank line separating nearby declarations or logic blocks.
  **L3355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3356 EN**: Defines macro `__lsx_vpermi_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3356 CN**: 定义宏 `__lsx_vpermi_w(/*__m128i*/ _1, /*__m128i*/ _2, /*ui8*/ _3)`，用于条件编译、简写或 API 生成。
- **L3357 EN**: Continues logic associated with callable symbol `__builtin_lsx_vpermi_w`.
  **L3357 CN**: 继续与可调用符号 `__builtin_lsx_vpermi_w` 相关的逻辑。
- **L3358 EN**: Blank line separating nearby declarations or logic blocks.
  **L3358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3359 EN**: Defines macro `__lsx_vld(/*void **/ _1, /*si12*/ _2)` for conditional compilation, shorthand, or API generation.
  **L3359 CN**: 定义宏 `__lsx_vld(/*void **/ _1, /*si12*/ _2)`，用于条件编译、简写或 API 生成。
- **L3360 EN**: Continues logic associated with callable symbol `__builtin_lsx_vld`.
  **L3360 CN**: 继续与可调用符号 `__builtin_lsx_vld` 相关的逻辑。

### Lines 3361-3384

````c

#define __lsx_vst(/*__m128i*/ _1, /*void **/ _2, /*si12*/ _3)                  \
  ((void)__builtin_lsx_vst((v16i8)(_1), (void *)(_2), (_3)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrlrn_b_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrlrn_b_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrlrn_h_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrlrn_h_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrlrn_w_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrlrn_w_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
````
- **L3361 EN**: Blank line separating nearby declarations or logic blocks.
  **L3361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3362 EN**: Defines macro `__lsx_vst(/*__m128i*/ _1, /*void **/ _2, /*si12*/ _3)` for conditional compilation, shorthand, or API generation.
  **L3362 CN**: 定义宏 `__lsx_vst(/*__m128i*/ _1, /*void **/ _2, /*si12*/ _3)`，用于条件编译、简写或 API 生成。
- **L3363 EN**: Continues logic associated with callable symbol `__builtin_lsx_vst`.
  **L3363 CN**: 继续与可调用符号 `__builtin_lsx_vst` 相关的逻辑。
- **L3364 EN**: Blank line separating nearby declarations or logic blocks.
  **L3364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3365 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3365 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3366 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3366 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3367 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrlrn_b_h(__m128i _1, __m128i _2) {`.
  **L3367 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrlrn_b_h(__m128i _1, __m128i _2) {`。
- **L3368 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrlrn_b_h((v8i16)_1, (v8i16)_2)`.
  **L3368 CN**: 以 `(__m128i)__builtin_lsx_vssrlrn_b_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L3369 EN**: Closes the current lexical scope or compound statement.
  **L3369 CN**: 结束当前词法作用域或复合语句块。
- **L3370 EN**: Blank line separating nearby declarations or logic blocks.
  **L3370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3371 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3371 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3372 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3372 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3373 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrlrn_h_w(__m128i _1, __m128i _2) {`.
  **L3373 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrlrn_h_w(__m128i _1, __m128i _2) {`。
- **L3374 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrlrn_h_w((v4i32)_1, (v4i32)_2)`.
  **L3374 CN**: 以 `(__m128i)__builtin_lsx_vssrlrn_h_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L3375 EN**: Closes the current lexical scope or compound statement.
  **L3375 CN**: 结束当前词法作用域或复合语句块。
- **L3376 EN**: Blank line separating nearby declarations or logic blocks.
  **L3376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3377 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3377 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3378 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3378 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrlrn_w_d(__m128i _1, __m128i _2) {`.
  **L3379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrlrn_w_d(__m128i _1, __m128i _2) {`。
- **L3380 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrlrn_w_d((v2i64)_1, (v2i64)_2)`.
  **L3380 CN**: 以 `(__m128i)__builtin_lsx_vssrlrn_w_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L3381 EN**: Closes the current lexical scope or compound statement.
  **L3381 CN**: 结束当前词法作用域或复合语句块。
- **L3382 EN**: Blank line separating nearby declarations or logic blocks.
  **L3382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3383 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3383 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3384 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3384 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。

### Lines 3385-3408

````c
    __lsx_vssrln_b_h(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrln_b_h((v8i16)_1, (v8i16)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrln_h_w(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrln_h_w((v4i32)_1, (v4i32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vssrln_w_d(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vssrln_w_d((v2i64)_1, (v2i64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vorn_v(__m128i _1, __m128i _2) {
  return (__m128i)__builtin_lsx_vorn_v((v16u8)_1, (v16u8)_2);
}

#define __lsx_vldi(/*i13*/ _1) ((__m128i)__builtin_lsx_vldi((_1)))

````
- **L3385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrln_b_h(__m128i _1, __m128i _2) {`.
  **L3385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrln_b_h(__m128i _1, __m128i _2) {`。
- **L3386 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrln_b_h((v8i16)_1, (v8i16)_2)`.
  **L3386 CN**: 以 `(__m128i)__builtin_lsx_vssrln_b_h((v8i16)_1, (v8i16)_2)` 从当前函数返回。
- **L3387 EN**: Closes the current lexical scope or compound statement.
  **L3387 CN**: 结束当前词法作用域或复合语句块。
- **L3388 EN**: Blank line separating nearby declarations or logic blocks.
  **L3388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3389 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3389 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3390 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3390 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrln_h_w(__m128i _1, __m128i _2) {`.
  **L3391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrln_h_w(__m128i _1, __m128i _2) {`。
- **L3392 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrln_h_w((v4i32)_1, (v4i32)_2)`.
  **L3392 CN**: 以 `(__m128i)__builtin_lsx_vssrln_h_w((v4i32)_1, (v4i32)_2)` 从当前函数返回。
- **L3393 EN**: Closes the current lexical scope or compound statement.
  **L3393 CN**: 结束当前词法作用域或复合语句块。
- **L3394 EN**: Blank line separating nearby declarations or logic blocks.
  **L3394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3395 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3395 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3396 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3396 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3397 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vssrln_w_d(__m128i _1, __m128i _2) {`.
  **L3397 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vssrln_w_d(__m128i _1, __m128i _2) {`。
- **L3398 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vssrln_w_d((v2i64)_1, (v2i64)_2)`.
  **L3398 CN**: 以 `(__m128i)__builtin_lsx_vssrln_w_d((v2i64)_1, (v2i64)_2)` 从当前函数返回。
- **L3399 EN**: Closes the current lexical scope or compound statement.
  **L3399 CN**: 结束当前词法作用域或复合语句块。
- **L3400 EN**: Blank line separating nearby declarations or logic blocks.
  **L3400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3401 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3401 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3402 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3402 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3403 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vorn_v(__m128i _1, __m128i _2) {`.
  **L3403 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vorn_v(__m128i _1, __m128i _2) {`。
- **L3404 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vorn_v((v16u8)_1, (v16u8)_2)`.
  **L3404 CN**: 以 `(__m128i)__builtin_lsx_vorn_v((v16u8)_1, (v16u8)_2)` 从当前函数返回。
- **L3405 EN**: Closes the current lexical scope or compound statement.
  **L3405 CN**: 结束当前词法作用域或复合语句块。
- **L3406 EN**: Blank line separating nearby declarations or logic blocks.
  **L3406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3407 EN**: Defines macro `__lsx_vldi(/*i13*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3407 CN**: 定义宏 `__lsx_vldi(/*i13*/ _1)`，用于条件编译、简写或 API 生成。
- **L3408 EN**: Blank line separating nearby declarations or logic blocks.
  **L3408 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3409-3432

````c
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vshuf_b(__m128i _1, __m128i _2, __m128i _3) {
  return (__m128i)__builtin_lsx_vshuf_b((v16i8)_1, (v16i8)_2, (v16i8)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vldx(void const *_1, long int _2) {
  return (__m128i)__builtin_lsx_vldx((void const *)_1, (long int)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) void
    __lsx_vstx(__m128i _1, void *_2, long int _3) {
  return (void)__builtin_lsx_vstx((v16i8)_1, (void *)_2, (long int)_3);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vextl_qu_du(__m128i _1) {
  return (__m128i)__builtin_lsx_vextl_qu_du((v2u64)_1);
}

````
- **L3409 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3409 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3410 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3410 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3411 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vshuf_b(__m128i _1, __m128i _2, __m128i _3) {`.
  **L3411 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vshuf_b(__m128i _1, __m128i _2, __m128i _3) {`。
- **L3412 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vshuf_b((v16i8)_1, (v16i8)_2, (v16i8)_3)`.
  **L3412 CN**: 以 `(__m128i)__builtin_lsx_vshuf_b((v16i8)_1, (v16i8)_2, (v16i8)_3)` 从当前函数返回。
- **L3413 EN**: Closes the current lexical scope or compound statement.
  **L3413 CN**: 结束当前词法作用域或复合语句块。
- **L3414 EN**: Blank line separating nearby declarations or logic blocks.
  **L3414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3415 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3415 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3416 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3416 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3417 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vldx(void const *_1, long int _2) {`.
  **L3417 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vldx(void const *_1, long int _2) {`。
- **L3418 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vldx((void const *)_1, (long int)_2)`.
  **L3418 CN**: 以 `(__m128i)__builtin_lsx_vldx((void const *)_1, (long int)_2)` 从当前函数返回。
- **L3419 EN**: Closes the current lexical scope or compound statement.
  **L3419 CN**: 结束当前词法作用域或复合语句块。
- **L3420 EN**: Blank line separating nearby declarations or logic blocks.
  **L3420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3421 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3421 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3422 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) void`.
  **L3422 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) void`。
- **L3423 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vstx(__m128i _1, void *_2, long int _3) {`.
  **L3423 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vstx(__m128i _1, void *_2, long int _3) {`。
- **L3424 EN**: Returns from the current function with `(void)__builtin_lsx_vstx((v16i8)_1, (void *)_2, (long int)_3)`.
  **L3424 CN**: 以 `(void)__builtin_lsx_vstx((v16i8)_1, (void *)_2, (long int)_3)` 从当前函数返回。
- **L3425 EN**: Closes the current lexical scope or compound statement.
  **L3425 CN**: 结束当前词法作用域或复合语句块。
- **L3426 EN**: Blank line separating nearby declarations or logic blocks.
  **L3426 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3427 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3427 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3428 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3428 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3429 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vextl_qu_du(__m128i _1) {`.
  **L3429 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vextl_qu_du(__m128i _1) {`。
- **L3430 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vextl_qu_du((v2u64)_1)`.
  **L3430 CN**: 以 `(__m128i)__builtin_lsx_vextl_qu_du((v2u64)_1)` 从当前函数返回。
- **L3431 EN**: Closes the current lexical scope or compound statement.
  **L3431 CN**: 结束当前词法作用域或复合语句块。
- **L3432 EN**: Blank line separating nearby declarations or logic blocks.
  **L3432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3433-3456

````c
#define __lsx_bnz_b(/*__m128i*/ _1) ((int)__builtin_lsx_bnz_b((v16u8)(_1)))

#define __lsx_bnz_d(/*__m128i*/ _1) ((int)__builtin_lsx_bnz_d((v2u64)(_1)))

#define __lsx_bnz_h(/*__m128i*/ _1) ((int)__builtin_lsx_bnz_h((v8u16)(_1)))

#define __lsx_bnz_v(/*__m128i*/ _1) ((int)__builtin_lsx_bnz_v((v16u8)(_1)))

#define __lsx_bnz_w(/*__m128i*/ _1) ((int)__builtin_lsx_bnz_w((v4u32)(_1)))

#define __lsx_bz_b(/*__m128i*/ _1) ((int)__builtin_lsx_bz_b((v16u8)(_1)))

#define __lsx_bz_d(/*__m128i*/ _1) ((int)__builtin_lsx_bz_d((v2u64)(_1)))

#define __lsx_bz_h(/*__m128i*/ _1) ((int)__builtin_lsx_bz_h((v8u16)(_1)))

#define __lsx_bz_v(/*__m128i*/ _1) ((int)__builtin_lsx_bz_v((v16u8)(_1)))

#define __lsx_bz_w(/*__m128i*/ _1) ((int)__builtin_lsx_bz_w((v4u32)(_1)))

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_caf_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_caf_d((v2f64)_1, (v2f64)_2);
````
- **L3433 EN**: Defines macro `__lsx_bnz_b(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3433 CN**: 定义宏 `__lsx_bnz_b(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3434 EN**: Blank line separating nearby declarations or logic blocks.
  **L3434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3435 EN**: Defines macro `__lsx_bnz_d(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3435 CN**: 定义宏 `__lsx_bnz_d(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3436 EN**: Blank line separating nearby declarations or logic blocks.
  **L3436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3437 EN**: Defines macro `__lsx_bnz_h(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3437 CN**: 定义宏 `__lsx_bnz_h(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3438 EN**: Blank line separating nearby declarations or logic blocks.
  **L3438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3439 EN**: Defines macro `__lsx_bnz_v(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3439 CN**: 定义宏 `__lsx_bnz_v(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3440 EN**: Blank line separating nearby declarations or logic blocks.
  **L3440 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3441 EN**: Defines macro `__lsx_bnz_w(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3441 CN**: 定义宏 `__lsx_bnz_w(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3442 EN**: Blank line separating nearby declarations or logic blocks.
  **L3442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3443 EN**: Defines macro `__lsx_bz_b(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3443 CN**: 定义宏 `__lsx_bz_b(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3444 EN**: Blank line separating nearby declarations or logic blocks.
  **L3444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3445 EN**: Defines macro `__lsx_bz_d(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3445 CN**: 定义宏 `__lsx_bz_d(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3446 EN**: Blank line separating nearby declarations or logic blocks.
  **L3446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3447 EN**: Defines macro `__lsx_bz_h(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3447 CN**: 定义宏 `__lsx_bz_h(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3448 EN**: Blank line separating nearby declarations or logic blocks.
  **L3448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3449 EN**: Defines macro `__lsx_bz_v(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3449 CN**: 定义宏 `__lsx_bz_v(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3450 EN**: Blank line separating nearby declarations or logic blocks.
  **L3450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3451 EN**: Defines macro `__lsx_bz_w(/*__m128i*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3451 CN**: 定义宏 `__lsx_bz_w(/*__m128i*/ _1)`，用于条件编译、简写或 API 生成。
- **L3452 EN**: Blank line separating nearby declarations or logic blocks.
  **L3452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3453 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3453 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3454 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3454 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3455 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_caf_d(__m128d _1, __m128d _2) {`.
  **L3455 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_caf_d(__m128d _1, __m128d _2) {`。
- **L3456 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_caf_d((v2f64)_1, (v2f64)_2)`.
  **L3456 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_caf_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3457-3480

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_caf_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_caf_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_ceq_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_ceq_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_ceq_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_ceq_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cle_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_cle_d((v2f64)_1, (v2f64)_2);
````
- **L3457 EN**: Closes the current lexical scope or compound statement.
  **L3457 CN**: 结束当前词法作用域或复合语句块。
- **L3458 EN**: Blank line separating nearby declarations or logic blocks.
  **L3458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3459 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3459 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3460 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3460 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3461 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_caf_s(__m128 _1, __m128 _2) {`.
  **L3461 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_caf_s(__m128 _1, __m128 _2) {`。
- **L3462 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_caf_s((v4f32)_1, (v4f32)_2)`.
  **L3462 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_caf_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3463 EN**: Closes the current lexical scope or compound statement.
  **L3463 CN**: 结束当前词法作用域或复合语句块。
- **L3464 EN**: Blank line separating nearby declarations or logic blocks.
  **L3464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3465 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3465 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3466 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3466 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3467 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_ceq_d(__m128d _1, __m128d _2) {`.
  **L3467 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_ceq_d(__m128d _1, __m128d _2) {`。
- **L3468 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_ceq_d((v2f64)_1, (v2f64)_2)`.
  **L3468 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_ceq_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3469 EN**: Closes the current lexical scope or compound statement.
  **L3469 CN**: 结束当前词法作用域或复合语句块。
- **L3470 EN**: Blank line separating nearby declarations or logic blocks.
  **L3470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3471 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3471 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3472 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3472 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3473 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_ceq_s(__m128 _1, __m128 _2) {`.
  **L3473 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_ceq_s(__m128 _1, __m128 _2) {`。
- **L3474 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_ceq_s((v4f32)_1, (v4f32)_2)`.
  **L3474 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_ceq_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3475 EN**: Closes the current lexical scope or compound statement.
  **L3475 CN**: 结束当前词法作用域或复合语句块。
- **L3476 EN**: Blank line separating nearby declarations or logic blocks.
  **L3476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3477 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3477 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3478 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3478 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3479 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cle_d(__m128d _1, __m128d _2) {`.
  **L3479 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cle_d(__m128d _1, __m128d _2) {`。
- **L3480 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cle_d((v2f64)_1, (v2f64)_2)`.
  **L3480 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cle_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3481-3504

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cle_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_cle_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_clt_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_clt_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_clt_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_clt_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cne_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_cne_d((v2f64)_1, (v2f64)_2);
````
- **L3481 EN**: Closes the current lexical scope or compound statement.
  **L3481 CN**: 结束当前词法作用域或复合语句块。
- **L3482 EN**: Blank line separating nearby declarations or logic blocks.
  **L3482 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3483 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3483 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3484 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3484 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3485 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cle_s(__m128 _1, __m128 _2) {`.
  **L3485 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cle_s(__m128 _1, __m128 _2) {`。
- **L3486 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cle_s((v4f32)_1, (v4f32)_2)`.
  **L3486 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cle_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3487 EN**: Closes the current lexical scope or compound statement.
  **L3487 CN**: 结束当前词法作用域或复合语句块。
- **L3488 EN**: Blank line separating nearby declarations or logic blocks.
  **L3488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3489 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3489 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3490 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3490 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3491 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_clt_d(__m128d _1, __m128d _2) {`.
  **L3491 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_clt_d(__m128d _1, __m128d _2) {`。
- **L3492 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_clt_d((v2f64)_1, (v2f64)_2)`.
  **L3492 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_clt_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3493 EN**: Closes the current lexical scope or compound statement.
  **L3493 CN**: 结束当前词法作用域或复合语句块。
- **L3494 EN**: Blank line separating nearby declarations or logic blocks.
  **L3494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3495 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3495 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3496 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3496 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3497 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_clt_s(__m128 _1, __m128 _2) {`.
  **L3497 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_clt_s(__m128 _1, __m128 _2) {`。
- **L3498 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_clt_s((v4f32)_1, (v4f32)_2)`.
  **L3498 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_clt_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3499 EN**: Closes the current lexical scope or compound statement.
  **L3499 CN**: 结束当前词法作用域或复合语句块。
- **L3500 EN**: Blank line separating nearby declarations or logic blocks.
  **L3500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3501 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3501 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3502 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3502 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3503 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cne_d(__m128d _1, __m128d _2) {`.
  **L3503 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cne_d(__m128d _1, __m128d _2) {`。
- **L3504 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cne_d((v2f64)_1, (v2f64)_2)`.
  **L3504 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cne_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3505-3528

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cne_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_cne_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cor_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_cor_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cor_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_cor_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cueq_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_cueq_d((v2f64)_1, (v2f64)_2);
````
- **L3505 EN**: Closes the current lexical scope or compound statement.
  **L3505 CN**: 结束当前词法作用域或复合语句块。
- **L3506 EN**: Blank line separating nearby declarations or logic blocks.
  **L3506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3507 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3507 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3508 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3508 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3509 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cne_s(__m128 _1, __m128 _2) {`.
  **L3509 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cne_s(__m128 _1, __m128 _2) {`。
- **L3510 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cne_s((v4f32)_1, (v4f32)_2)`.
  **L3510 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cne_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3511 EN**: Closes the current lexical scope or compound statement.
  **L3511 CN**: 结束当前词法作用域或复合语句块。
- **L3512 EN**: Blank line separating nearby declarations or logic blocks.
  **L3512 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3513 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3513 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3514 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3514 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3515 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cor_d(__m128d _1, __m128d _2) {`.
  **L3515 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cor_d(__m128d _1, __m128d _2) {`。
- **L3516 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cor_d((v2f64)_1, (v2f64)_2)`.
  **L3516 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cor_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3517 EN**: Closes the current lexical scope or compound statement.
  **L3517 CN**: 结束当前词法作用域或复合语句块。
- **L3518 EN**: Blank line separating nearby declarations or logic blocks.
  **L3518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3519 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3519 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3520 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3520 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3521 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cor_s(__m128 _1, __m128 _2) {`.
  **L3521 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cor_s(__m128 _1, __m128 _2) {`。
- **L3522 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cor_s((v4f32)_1, (v4f32)_2)`.
  **L3522 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cor_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3523 EN**: Closes the current lexical scope or compound statement.
  **L3523 CN**: 结束当前词法作用域或复合语句块。
- **L3524 EN**: Blank line separating nearby declarations or logic blocks.
  **L3524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3525 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3525 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3526 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3526 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3527 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cueq_d(__m128d _1, __m128d _2) {`.
  **L3527 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cueq_d(__m128d _1, __m128d _2) {`。
- **L3528 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cueq_d((v2f64)_1, (v2f64)_2)`.
  **L3528 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cueq_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3529-3552

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cueq_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_cueq_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cule_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_cule_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cule_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_cule_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cult_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_cult_d((v2f64)_1, (v2f64)_2);
````
- **L3529 EN**: Closes the current lexical scope or compound statement.
  **L3529 CN**: 结束当前词法作用域或复合语句块。
- **L3530 EN**: Blank line separating nearby declarations or logic blocks.
  **L3530 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3531 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3531 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3532 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3532 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3533 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cueq_s(__m128 _1, __m128 _2) {`.
  **L3533 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cueq_s(__m128 _1, __m128 _2) {`。
- **L3534 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cueq_s((v4f32)_1, (v4f32)_2)`.
  **L3534 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cueq_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3535 EN**: Closes the current lexical scope or compound statement.
  **L3535 CN**: 结束当前词法作用域或复合语句块。
- **L3536 EN**: Blank line separating nearby declarations or logic blocks.
  **L3536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3537 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3537 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3538 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3538 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3539 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cule_d(__m128d _1, __m128d _2) {`.
  **L3539 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cule_d(__m128d _1, __m128d _2) {`。
- **L3540 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cule_d((v2f64)_1, (v2f64)_2)`.
  **L3540 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cule_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3541 EN**: Closes the current lexical scope or compound statement.
  **L3541 CN**: 结束当前词法作用域或复合语句块。
- **L3542 EN**: Blank line separating nearby declarations or logic blocks.
  **L3542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3543 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3543 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3544 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3544 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3545 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cule_s(__m128 _1, __m128 _2) {`.
  **L3545 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cule_s(__m128 _1, __m128 _2) {`。
- **L3546 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cule_s((v4f32)_1, (v4f32)_2)`.
  **L3546 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cule_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3547 EN**: Closes the current lexical scope or compound statement.
  **L3547 CN**: 结束当前词法作用域或复合语句块。
- **L3548 EN**: Blank line separating nearby declarations or logic blocks.
  **L3548 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3549 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3549 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3550 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3550 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3551 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cult_d(__m128d _1, __m128d _2) {`.
  **L3551 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cult_d(__m128d _1, __m128d _2) {`。
- **L3552 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cult_d((v2f64)_1, (v2f64)_2)`.
  **L3552 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cult_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3553-3576

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cult_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_cult_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cun_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_cun_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cune_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_cune_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cune_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_cune_s((v4f32)_1, (v4f32)_2);
````
- **L3553 EN**: Closes the current lexical scope or compound statement.
  **L3553 CN**: 结束当前词法作用域或复合语句块。
- **L3554 EN**: Blank line separating nearby declarations or logic blocks.
  **L3554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3555 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3555 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3556 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3556 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3557 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cult_s(__m128 _1, __m128 _2) {`.
  **L3557 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cult_s(__m128 _1, __m128 _2) {`。
- **L3558 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cult_s((v4f32)_1, (v4f32)_2)`.
  **L3558 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cult_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3559 EN**: Closes the current lexical scope or compound statement.
  **L3559 CN**: 结束当前词法作用域或复合语句块。
- **L3560 EN**: Blank line separating nearby declarations or logic blocks.
  **L3560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3561 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3561 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3562 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3562 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3563 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cun_d(__m128d _1, __m128d _2) {`.
  **L3563 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cun_d(__m128d _1, __m128d _2) {`。
- **L3564 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cun_d((v2f64)_1, (v2f64)_2)`.
  **L3564 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cun_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3565 EN**: Closes the current lexical scope or compound statement.
  **L3565 CN**: 结束当前词法作用域或复合语句块。
- **L3566 EN**: Blank line separating nearby declarations or logic blocks.
  **L3566 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3567 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3567 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3568 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3568 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3569 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cune_d(__m128d _1, __m128d _2) {`.
  **L3569 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cune_d(__m128d _1, __m128d _2) {`。
- **L3570 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cune_d((v2f64)_1, (v2f64)_2)`.
  **L3570 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cune_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3571 EN**: Closes the current lexical scope or compound statement.
  **L3571 CN**: 结束当前词法作用域或复合语句块。
- **L3572 EN**: Blank line separating nearby declarations or logic blocks.
  **L3572 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3573 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3573 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3574 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3574 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3575 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cune_s(__m128 _1, __m128 _2) {`.
  **L3575 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cune_s(__m128 _1, __m128 _2) {`。
- **L3576 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cune_s((v4f32)_1, (v4f32)_2)`.
  **L3576 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cune_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。

### Lines 3577-3600

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_cun_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_cun_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_saf_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_saf_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_saf_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_saf_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_seq_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_seq_d((v2f64)_1, (v2f64)_2);
````
- **L3577 EN**: Closes the current lexical scope or compound statement.
  **L3577 CN**: 结束当前词法作用域或复合语句块。
- **L3578 EN**: Blank line separating nearby declarations or logic blocks.
  **L3578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3579 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3579 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3580 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3580 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3581 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_cun_s(__m128 _1, __m128 _2) {`.
  **L3581 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_cun_s(__m128 _1, __m128 _2) {`。
- **L3582 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_cun_s((v4f32)_1, (v4f32)_2)`.
  **L3582 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_cun_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3583 EN**: Closes the current lexical scope or compound statement.
  **L3583 CN**: 结束当前词法作用域或复合语句块。
- **L3584 EN**: Blank line separating nearby declarations or logic blocks.
  **L3584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3585 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3585 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3586 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3586 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3587 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_saf_d(__m128d _1, __m128d _2) {`.
  **L3587 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_saf_d(__m128d _1, __m128d _2) {`。
- **L3588 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_saf_d((v2f64)_1, (v2f64)_2)`.
  **L3588 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_saf_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3589 EN**: Closes the current lexical scope or compound statement.
  **L3589 CN**: 结束当前词法作用域或复合语句块。
- **L3590 EN**: Blank line separating nearby declarations or logic blocks.
  **L3590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3591 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3591 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3592 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3592 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3593 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_saf_s(__m128 _1, __m128 _2) {`.
  **L3593 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_saf_s(__m128 _1, __m128 _2) {`。
- **L3594 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_saf_s((v4f32)_1, (v4f32)_2)`.
  **L3594 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_saf_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3595 EN**: Closes the current lexical scope or compound statement.
  **L3595 CN**: 结束当前词法作用域或复合语句块。
- **L3596 EN**: Blank line separating nearby declarations or logic blocks.
  **L3596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3597 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3597 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3598 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3598 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3599 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_seq_d(__m128d _1, __m128d _2) {`.
  **L3599 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_seq_d(__m128d _1, __m128d _2) {`。
- **L3600 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_seq_d((v2f64)_1, (v2f64)_2)`.
  **L3600 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_seq_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3601-3624

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_seq_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_seq_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sle_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_sle_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sle_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_sle_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_slt_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_slt_d((v2f64)_1, (v2f64)_2);
````
- **L3601 EN**: Closes the current lexical scope or compound statement.
  **L3601 CN**: 结束当前词法作用域或复合语句块。
- **L3602 EN**: Blank line separating nearby declarations or logic blocks.
  **L3602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3603 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3603 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3604 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3604 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3605 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_seq_s(__m128 _1, __m128 _2) {`.
  **L3605 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_seq_s(__m128 _1, __m128 _2) {`。
- **L3606 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_seq_s((v4f32)_1, (v4f32)_2)`.
  **L3606 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_seq_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3607 EN**: Closes the current lexical scope or compound statement.
  **L3607 CN**: 结束当前词法作用域或复合语句块。
- **L3608 EN**: Blank line separating nearby declarations or logic blocks.
  **L3608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3609 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3609 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3610 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3610 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3611 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sle_d(__m128d _1, __m128d _2) {`.
  **L3611 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sle_d(__m128d _1, __m128d _2) {`。
- **L3612 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sle_d((v2f64)_1, (v2f64)_2)`.
  **L3612 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sle_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3613 EN**: Closes the current lexical scope or compound statement.
  **L3613 CN**: 结束当前词法作用域或复合语句块。
- **L3614 EN**: Blank line separating nearby declarations or logic blocks.
  **L3614 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3615 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3615 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3616 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3616 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3617 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sle_s(__m128 _1, __m128 _2) {`.
  **L3617 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sle_s(__m128 _1, __m128 _2) {`。
- **L3618 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sle_s((v4f32)_1, (v4f32)_2)`.
  **L3618 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sle_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3619 EN**: Closes the current lexical scope or compound statement.
  **L3619 CN**: 结束当前词法作用域或复合语句块。
- **L3620 EN**: Blank line separating nearby declarations or logic blocks.
  **L3620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3621 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3621 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3622 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3622 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3623 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_slt_d(__m128d _1, __m128d _2) {`.
  **L3623 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_slt_d(__m128d _1, __m128d _2) {`。
- **L3624 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_slt_d((v2f64)_1, (v2f64)_2)`.
  **L3624 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_slt_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3625-3648

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_slt_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_slt_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sne_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_sne_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sne_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_sne_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sor_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_sor_d((v2f64)_1, (v2f64)_2);
````
- **L3625 EN**: Closes the current lexical scope or compound statement.
  **L3625 CN**: 结束当前词法作用域或复合语句块。
- **L3626 EN**: Blank line separating nearby declarations or logic blocks.
  **L3626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3627 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3627 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3628 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3628 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3629 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_slt_s(__m128 _1, __m128 _2) {`.
  **L3629 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_slt_s(__m128 _1, __m128 _2) {`。
- **L3630 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_slt_s((v4f32)_1, (v4f32)_2)`.
  **L3630 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_slt_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3631 EN**: Closes the current lexical scope or compound statement.
  **L3631 CN**: 结束当前词法作用域或复合语句块。
- **L3632 EN**: Blank line separating nearby declarations or logic blocks.
  **L3632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3633 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3633 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3634 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3634 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3635 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sne_d(__m128d _1, __m128d _2) {`.
  **L3635 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sne_d(__m128d _1, __m128d _2) {`。
- **L3636 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sne_d((v2f64)_1, (v2f64)_2)`.
  **L3636 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sne_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3637 EN**: Closes the current lexical scope or compound statement.
  **L3637 CN**: 结束当前词法作用域或复合语句块。
- **L3638 EN**: Blank line separating nearby declarations or logic blocks.
  **L3638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3639 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3639 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3640 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3640 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3641 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sne_s(__m128 _1, __m128 _2) {`.
  **L3641 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sne_s(__m128 _1, __m128 _2) {`。
- **L3642 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sne_s((v4f32)_1, (v4f32)_2)`.
  **L3642 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sne_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3643 EN**: Closes the current lexical scope or compound statement.
  **L3643 CN**: 结束当前词法作用域或复合语句块。
- **L3644 EN**: Blank line separating nearby declarations or logic blocks.
  **L3644 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3645 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3645 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3646 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3646 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3647 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sor_d(__m128d _1, __m128d _2) {`.
  **L3647 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sor_d(__m128d _1, __m128d _2) {`。
- **L3648 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sor_d((v2f64)_1, (v2f64)_2)`.
  **L3648 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sor_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3649-3672

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sor_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_sor_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sueq_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_sueq_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sueq_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_sueq_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sule_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_sule_d((v2f64)_1, (v2f64)_2);
````
- **L3649 EN**: Closes the current lexical scope or compound statement.
  **L3649 CN**: 结束当前词法作用域或复合语句块。
- **L3650 EN**: Blank line separating nearby declarations or logic blocks.
  **L3650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3651 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3651 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3652 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3652 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3653 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sor_s(__m128 _1, __m128 _2) {`.
  **L3653 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sor_s(__m128 _1, __m128 _2) {`。
- **L3654 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sor_s((v4f32)_1, (v4f32)_2)`.
  **L3654 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sor_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3655 EN**: Closes the current lexical scope or compound statement.
  **L3655 CN**: 结束当前词法作用域或复合语句块。
- **L3656 EN**: Blank line separating nearby declarations or logic blocks.
  **L3656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3657 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3657 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3658 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3658 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3659 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sueq_d(__m128d _1, __m128d _2) {`.
  **L3659 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sueq_d(__m128d _1, __m128d _2) {`。
- **L3660 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sueq_d((v2f64)_1, (v2f64)_2)`.
  **L3660 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sueq_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3661 EN**: Closes the current lexical scope or compound statement.
  **L3661 CN**: 结束当前词法作用域或复合语句块。
- **L3662 EN**: Blank line separating nearby declarations or logic blocks.
  **L3662 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3663 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3663 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3664 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3664 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3665 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sueq_s(__m128 _1, __m128 _2) {`.
  **L3665 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sueq_s(__m128 _1, __m128 _2) {`。
- **L3666 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sueq_s((v4f32)_1, (v4f32)_2)`.
  **L3666 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sueq_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3667 EN**: Closes the current lexical scope or compound statement.
  **L3667 CN**: 结束当前词法作用域或复合语句块。
- **L3668 EN**: Blank line separating nearby declarations or logic blocks.
  **L3668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3669 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3669 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3670 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3670 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3671 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sule_d(__m128d _1, __m128d _2) {`.
  **L3671 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sule_d(__m128d _1, __m128d _2) {`。
- **L3672 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sule_d((v2f64)_1, (v2f64)_2)`.
  **L3672 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sule_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3673-3696

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sule_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_sule_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sult_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_sult_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sult_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_sult_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sun_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_sun_d((v2f64)_1, (v2f64)_2);
````
- **L3673 EN**: Closes the current lexical scope or compound statement.
  **L3673 CN**: 结束当前词法作用域或复合语句块。
- **L3674 EN**: Blank line separating nearby declarations or logic blocks.
  **L3674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3675 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3675 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3676 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3676 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3677 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sule_s(__m128 _1, __m128 _2) {`.
  **L3677 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sule_s(__m128 _1, __m128 _2) {`。
- **L3678 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sule_s((v4f32)_1, (v4f32)_2)`.
  **L3678 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sule_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3679 EN**: Closes the current lexical scope or compound statement.
  **L3679 CN**: 结束当前词法作用域或复合语句块。
- **L3680 EN**: Blank line separating nearby declarations or logic blocks.
  **L3680 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3681 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3681 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3682 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3682 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3683 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sult_d(__m128d _1, __m128d _2) {`.
  **L3683 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sult_d(__m128d _1, __m128d _2) {`。
- **L3684 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sult_d((v2f64)_1, (v2f64)_2)`.
  **L3684 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sult_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3685 EN**: Closes the current lexical scope or compound statement.
  **L3685 CN**: 结束当前词法作用域或复合语句块。
- **L3686 EN**: Blank line separating nearby declarations or logic blocks.
  **L3686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3687 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3687 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3688 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3688 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3689 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sult_s(__m128 _1, __m128 _2) {`.
  **L3689 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sult_s(__m128 _1, __m128 _2) {`。
- **L3690 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sult_s((v4f32)_1, (v4f32)_2)`.
  **L3690 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sult_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3691 EN**: Closes the current lexical scope or compound statement.
  **L3691 CN**: 结束当前词法作用域或复合语句块。
- **L3692 EN**: Blank line separating nearby declarations or logic blocks.
  **L3692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3693 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3693 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3694 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3694 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3695 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sun_d(__m128d _1, __m128d _2) {`.
  **L3695 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sun_d(__m128d _1, __m128d _2) {`。
- **L3696 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sun_d((v2f64)_1, (v2f64)_2)`.
  **L3696 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sun_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。

### Lines 3697-3720

````c
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sune_d(__m128d _1, __m128d _2) {
  return (__m128i)__builtin_lsx_vfcmp_sune_d((v2f64)_1, (v2f64)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sune_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_sune_s((v4f32)_1, (v4f32)_2);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i
    __lsx_vfcmp_sun_s(__m128 _1, __m128 _2) {
  return (__m128i)__builtin_lsx_vfcmp_sun_s((v4f32)_1, (v4f32)_2);
}

#if defined(__loongarch_frecipe)
extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrecipe_s(__m128 _1) {
````
- **L3697 EN**: Closes the current lexical scope or compound statement.
  **L3697 CN**: 结束当前词法作用域或复合语句块。
- **L3698 EN**: Blank line separating nearby declarations or logic blocks.
  **L3698 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3699 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3699 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3700 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3700 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3701 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sune_d(__m128d _1, __m128d _2) {`.
  **L3701 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sune_d(__m128d _1, __m128d _2) {`。
- **L3702 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sune_d((v2f64)_1, (v2f64)_2)`.
  **L3702 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sune_d((v2f64)_1, (v2f64)_2)` 从当前函数返回。
- **L3703 EN**: Closes the current lexical scope or compound statement.
  **L3703 CN**: 结束当前词法作用域或复合语句块。
- **L3704 EN**: Blank line separating nearby declarations or logic blocks.
  **L3704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3705 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3705 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3706 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3706 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3707 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sune_s(__m128 _1, __m128 _2) {`.
  **L3707 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sune_s(__m128 _1, __m128 _2) {`。
- **L3708 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sune_s((v4f32)_1, (v4f32)_2)`.
  **L3708 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sune_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3709 EN**: Closes the current lexical scope or compound statement.
  **L3709 CN**: 结束当前词法作用域或复合语句块。
- **L3710 EN**: Blank line separating nearby declarations or logic blocks.
  **L3710 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3711 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3711 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3712 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`.
  **L3712 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128i`。
- **L3713 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfcmp_sun_s(__m128 _1, __m128 _2) {`.
  **L3713 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfcmp_sun_s(__m128 _1, __m128 _2) {`。
- **L3714 EN**: Returns from the current function with `(__m128i)__builtin_lsx_vfcmp_sun_s((v4f32)_1, (v4f32)_2)`.
  **L3714 CN**: 以 `(__m128i)__builtin_lsx_vfcmp_sun_s((v4f32)_1, (v4f32)_2)` 从当前函数返回。
- **L3715 EN**: Closes the current lexical scope or compound statement.
  **L3715 CN**: 结束当前词法作用域或复合语句块。
- **L3716 EN**: Blank line separating nearby declarations or logic blocks.
  **L3716 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3717 EN**: Starts a preprocessor conditional block: `#if defined(__loongarch_frecipe)`.
  **L3717 CN**: 开始一个预处理条件块：`#if defined(__loongarch_frecipe)`。
- **L3718 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3718 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3719 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L3719 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L3720 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrecipe_s(__m128 _1) {`.
  **L3720 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrecipe_s(__m128 _1) {`。

### Lines 3721-3744

````c
  return (__m128)__builtin_lsx_vfrecipe_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrecipe_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrecipe_d((v2f64)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128
    __lsx_vfrsqrte_s(__m128 _1) {
  return (__m128)__builtin_lsx_vfrsqrte_s((v4f32)_1);
}

extern __inline
    __attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d
    __lsx_vfrsqrte_d(__m128d _1) {
  return (__m128d)__builtin_lsx_vfrsqrte_d((v2f64)_1);
}
#endif

#define __lsx_vrepli_b(/*si10*/ _1) ((__m128i)__builtin_lsx_vrepli_b((_1)))

````
- **L3721 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrecipe_s((v4f32)_1)`.
  **L3721 CN**: 以 `(__m128)__builtin_lsx_vfrecipe_s((v4f32)_1)` 从当前函数返回。
- **L3722 EN**: Closes the current lexical scope or compound statement.
  **L3722 CN**: 结束当前词法作用域或复合语句块。
- **L3723 EN**: Blank line separating nearby declarations or logic blocks.
  **L3723 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3724 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3724 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3725 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L3725 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L3726 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrecipe_d(__m128d _1) {`.
  **L3726 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrecipe_d(__m128d _1) {`。
- **L3727 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrecipe_d((v2f64)_1)`.
  **L3727 CN**: 以 `(__m128d)__builtin_lsx_vfrecipe_d((v2f64)_1)` 从当前函数返回。
- **L3728 EN**: Closes the current lexical scope or compound statement.
  **L3728 CN**: 结束当前词法作用域或复合语句块。
- **L3729 EN**: Blank line separating nearby declarations or logic blocks.
  **L3729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3730 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3730 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3731 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`.
  **L3731 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128`。
- **L3732 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrsqrte_s(__m128 _1) {`.
  **L3732 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrsqrte_s(__m128 _1) {`。
- **L3733 EN**: Returns from the current function with `(__m128)__builtin_lsx_vfrsqrte_s((v4f32)_1)`.
  **L3733 CN**: 以 `(__m128)__builtin_lsx_vfrsqrte_s((v4f32)_1)` 从当前函数返回。
- **L3734 EN**: Closes the current lexical scope or compound statement.
  **L3734 CN**: 结束当前词法作用域或复合语句块。
- **L3735 EN**: Blank line separating nearby declarations or logic blocks.
  **L3735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3736 EN**: Continues the surrounding expression or declaration: `extern __inline`.
  **L3736 CN**: 继续构造周围的表达式或声明：`extern __inline`。
- **L3737 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`.
  **L3737 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__gnu_inline__, __always_inline__, __artificial__)) __m128d`。
- **L3738 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__lsx_vfrsqrte_d(__m128d _1) {`.
  **L3738 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__lsx_vfrsqrte_d(__m128d _1) {`。
- **L3739 EN**: Returns from the current function with `(__m128d)__builtin_lsx_vfrsqrte_d((v2f64)_1)`.
  **L3739 CN**: 以 `(__m128d)__builtin_lsx_vfrsqrte_d((v2f64)_1)` 从当前函数返回。
- **L3740 EN**: Closes the current lexical scope or compound statement.
  **L3740 CN**: 结束当前词法作用域或复合语句块。
- **L3741 EN**: Closes the current preprocessor conditional block.
  **L3741 CN**: 结束当前预处理条件块。
- **L3742 EN**: Blank line separating nearby declarations or logic blocks.
  **L3742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3743 EN**: Defines macro `__lsx_vrepli_b(/*si10*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3743 CN**: 定义宏 `__lsx_vrepli_b(/*si10*/ _1)`，用于条件编译、简写或 API 生成。
- **L3744 EN**: Blank line separating nearby declarations or logic blocks.
  **L3744 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3745-3752

````c
#define __lsx_vrepli_d(/*si10*/ _1) ((__m128i)__builtin_lsx_vrepli_d((_1)))

#define __lsx_vrepli_h(/*si10*/ _1) ((__m128i)__builtin_lsx_vrepli_h((_1)))

#define __lsx_vrepli_w(/*si10*/ _1) ((__m128i)__builtin_lsx_vrepli_w((_1)))

#endif /* defined(__loongarch_sx) */
#endif /* _LOONGSON_SXINTRIN_H */
````
- **L3745 EN**: Defines macro `__lsx_vrepli_d(/*si10*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3745 CN**: 定义宏 `__lsx_vrepli_d(/*si10*/ _1)`，用于条件编译、简写或 API 生成。
- **L3746 EN**: Blank line separating nearby declarations or logic blocks.
  **L3746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3747 EN**: Defines macro `__lsx_vrepli_h(/*si10*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3747 CN**: 定义宏 `__lsx_vrepli_h(/*si10*/ _1)`，用于条件编译、简写或 API 生成。
- **L3748 EN**: Blank line separating nearby declarations or logic blocks.
  **L3748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3749 EN**: Defines macro `__lsx_vrepli_w(/*si10*/ _1)` for conditional compilation, shorthand, or API generation.
  **L3749 CN**: 定义宏 `__lsx_vrepli_w(/*si10*/ _1)`，用于条件编译、简写或 API 生成。
- **L3750 EN**: Blank line separating nearby declarations or logic blocks.
  **L3750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3751 EN**: Closes the current preprocessor conditional block.
  **L3751 CN**: 结束当前预处理条件块。
- **L3752 EN**: Closes the current preprocessor conditional block.
  **L3752 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **LoongArch intrinsics / LoongArch intrinsic 接口**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `_LOONGSON_SXINTRIN_H`, `__loongarch_sx`, `__loongarch_frecipe`
- **External builtins / 外部 builtin**: `__builtin_lsx_vsll_b`, `__builtin_lsx_vsll_h`, `__builtin_lsx_vsll_w`, `__builtin_lsx_vsll_d`, `__builtin_lsx_vslli_b`, `__builtin_lsx_vslli_h`, `__builtin_lsx_vslli_w`, `__builtin_lsx_vslli_d`, `__builtin_lsx_vsra_b`, `__builtin_lsx_vsra_h`, `__builtin_lsx_vsra_w`, `__builtin_lsx_vsra_d`
