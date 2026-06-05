# avx512fp16intrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx512fp16intrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: AVX512-FP16 intrinsics.
- **Purpose (CN)**: 提供 AVX512-FP16 intrinsic 接口。
- **Line Count / 行数**: 3349

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===----------- avx512fp16intrin.h - AVX512-FP16 intrinsics ---------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */
#ifndef __IMMINTRIN_H
#error "Never use <avx512fp16intrin.h> directly; include <immintrin.h> instead."
#endif

#ifdef __SSE2__

#ifndef __AVX512FP16INTRIN_H
#define __AVX512FP16INTRIN_H

/* Define the default attributes for the functions in this file. */
typedef _Float16 __v32hf __attribute__((__vector_size__(64), __aligned__(64)));
typedef _Float16 __m512h __attribute__((__vector_size__(64), __aligned__(64)));
typedef _Float16 __m512h_u __attribute__((__vector_size__(64), __aligned__(1)));

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS512                                                  \
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
- **L9 EN**: Starts a preprocessor conditional block: `#ifndef __IMMINTRIN_H`.
  **L9 CN**: 开始一个预处理条件块：`#ifndef __IMMINTRIN_H`。
- **L10 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <avx512fp16intrin.h> directly; include <immintrin.h> instead."`.
  **L10 CN**: 为不受支持的配置触发编译错误：`#error "Never use <avx512fp16intrin.h> directly; include <immintrin.h> instead."`。
- **L11 EN**: Closes the current preprocessor conditional block.
  **L11 CN**: 结束当前预处理条件块。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#ifdef __SSE2__`.
  **L13 CN**: 开始一个预处理条件块：`#ifdef __SSE2__`。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Starts a preprocessor conditional block: `#ifndef __AVX512FP16INTRIN_H`.
  **L15 CN**: 开始一个预处理条件块：`#ifndef __AVX512FP16INTRIN_H`。
- **L16 EN**: Defines macro `__AVX512FP16INTRIN_H` for conditional compilation, shorthand, or API generation.
  **L16 CN**: 定义宏 `__AVX512FP16INTRIN_H`，用于条件编译、简写或 API 生成。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L19 EN**: Introduces an alias or helper declaration: `typedef _Float16 __v32hf __attribute__((__vector_size__(64), __aligned__(64)));`.
  **L19 CN**: 引入一条别名或辅助声明：`typedef _Float16 __v32hf __attribute__((__vector_size__(64), __aligned__(64)));`。
- **L20 EN**: Introduces an alias or helper declaration: `typedef _Float16 __m512h __attribute__((__vector_size__(64), __aligned__(64)));`.
  **L20 CN**: 引入一条别名或辅助声明：`typedef _Float16 __m512h __attribute__((__vector_size__(64), __aligned__(64)));`。
- **L21 EN**: Introduces an alias or helper declaration: `typedef _Float16 __m512h_u __attribute__((__vector_size__(64), __aligned__(1)));`.
  **L21 CN**: 引入一条别名或辅助声明：`typedef _Float16 __m512h_u __attribute__((__vector_size__(64), __aligned__(1)));`。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L23 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L24 EN**: Defines macro `__DEFAULT_FN_ATTRS512` for conditional compilation, shorthand, or API generation.
  **L24 CN**: 定义宏 `__DEFAULT_FN_ATTRS512`，用于条件编译、简写或 API 生成。

### Lines 25-48

````c
  __attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \
                 __min_vector_width__(512)))
#define __DEFAULT_FN_ATTRS256                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \
                 __min_vector_width__(256)))
#define __DEFAULT_FN_ATTRS128                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \
                 __min_vector_width__(128)))

#if defined(__cplusplus) && (__cplusplus >= 201103L)
#define __DEFAULT_FN_ATTRS512_CONSTEXPR __DEFAULT_FN_ATTRS512 constexpr
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256 constexpr
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128 constexpr
#else
#define __DEFAULT_FN_ATTRS512_CONSTEXPR __DEFAULT_FN_ATTRS512
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128
#endif

static __inline__ _Float16 __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_cvtsh_h(__m512h __a) {
  return __a[0];
}

````
- **L25 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \`.
  **L25 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \`。
- **L26 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L26 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L27 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L27 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L28 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \`.
  **L28 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \`。
- **L29 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L29 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L30 EN**: Defines macro `__DEFAULT_FN_ATTRS128` for conditional compilation, shorthand, or API generation.
  **L30 CN**: 定义宏 `__DEFAULT_FN_ATTRS128`，用于条件编译、简写或 API 生成。
- **L31 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \`.
  **L31 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx512fp16"),     \`。
- **L32 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L32 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L34 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L34 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L35 EN**: Defines macro `__DEFAULT_FN_ATTRS512_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L35 CN**: 定义宏 `__DEFAULT_FN_ATTRS512_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L36 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L36 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L37 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L37 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L38 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L38 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L39 EN**: Defines macro `__DEFAULT_FN_ATTRS512_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L39 CN**: 定义宏 `__DEFAULT_FN_ATTRS512_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L40 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L40 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L41 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L41 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L42 EN**: Closes the current preprocessor conditional block.
  **L42 CN**: 结束当前预处理条件块。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L44 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L44 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L45 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtsh_h(__m512h __a) {`.
  **L45 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtsh_h(__m512h __a) {`。
- **L46 EN**: Returns from the current function with `__a[0]`.
  **L46 CN**: 以 `__a[0]` 从当前函数返回。
- **L47 EN**: Closes the current lexical scope or compound statement.
  **L47 CN**: 结束当前词法作用域或复合语句块。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 49-72

````c
static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_setzero_ph(void) {
  return (__m128h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};
}

static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_setzero_ph(void) {
  return (__m256h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,
                   0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_undefined_ph(void) {
  return (__m256h)__builtin_ia32_undef256();
}

static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_setzero_ph(void) {
  return (__m512h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,
                   0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,
                   0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_undefined_ph(void) {
  return (__m128h)__builtin_ia32_undef128();
}
````
- **L49 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_setzero_ph(void) {`.
  **L49 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_setzero_ph(void) {`。
- **L50 EN**: Returns from the current function with `(__m128h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0}`.
  **L50 CN**: 以 `(__m128h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0}` 从当前函数返回。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L53 EN**: Continues the surrounding expression or declaration: `static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L53 CN**: 继续构造周围的表达式或声明：`static __inline __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L54 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_setzero_ph(void) {`.
  **L54 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_setzero_ph(void) {`。
- **L55 EN**: Returns from the current function with `(__m256h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,`.
  **L55 CN**: 以 `(__m256h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,` 从当前函数返回。
- **L56 EN**: Adds a standalone statement or declaration: `0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};`.
  **L56 CN**: 添加一条独立语句或声明：`0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};`。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L59 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_undefined_ph(void) {`.
  **L59 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256 _mm256_undefined_ph(void) {`。
- **L60 EN**: Returns from the current function with `(__m256h)__builtin_ia32_undef256()`.
  **L60 CN**: 以 `(__m256h)__builtin_ia32_undef256()` 从当前函数返回。
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Continues the surrounding expression or declaration: `static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L63 CN**: 继续构造周围的表达式或声明：`static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L64 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_setzero_ph(void) {`.
  **L64 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_setzero_ph(void) {`。
- **L65 EN**: Returns from the current function with `(__m512h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,`.
  **L65 CN**: 以 `(__m512h){0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,` 从当前函数返回。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,`。
- **L67 EN**: Adds a standalone statement or declaration: `0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};`.
  **L67 CN**: 添加一条独立语句或声明：`0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};`。
- **L68 EN**: Closes the current lexical scope or compound statement.
  **L68 CN**: 结束当前词法作用域或复合语句块。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_undefined_ph(void) {`.
  **L70 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_undefined_ph(void) {`。
- **L71 EN**: Returns from the current function with `(__m128h)__builtin_ia32_undef128()`.
  **L71 CN**: 以 `(__m128h)__builtin_ia32_undef128()` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-96

````c

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_undefined_ph(void) {
  return (__m512h)__builtin_ia32_undef512();
}

static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_set1_ph(_Float16 __h) {
  return (__m512h)(__v32hf){__h, __h, __h, __h, __h, __h, __h, __h,
                            __h, __h, __h, __h, __h, __h, __h, __h,
                            __h, __h, __h, __h, __h, __h, __h, __h,
                            __h, __h, __h, __h, __h, __h, __h, __h};
}

static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,
              _Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8,
              _Float16 __h9, _Float16 __h10, _Float16 __h11, _Float16 __h12,
              _Float16 __h13, _Float16 __h14, _Float16 __h15, _Float16 __h16,
              _Float16 __h17, _Float16 __h18, _Float16 __h19, _Float16 __h20,
              _Float16 __h21, _Float16 __h22, _Float16 __h23, _Float16 __h24,
              _Float16 __h25, _Float16 __h26, _Float16 __h27, _Float16 __h28,
              _Float16 __h29, _Float16 __h30, _Float16 __h31, _Float16 __h32) {
  return (__m512h)(__v32hf){__h32, __h31, __h30, __h29, __h28, __h27, __h26,
                            __h25, __h24, __h23, __h22, __h21, __h20, __h19,
````
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L74 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_undefined_ph(void) {`.
  **L74 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_undefined_ph(void) {`。
- **L75 EN**: Returns from the current function with `(__m512h)__builtin_ia32_undef512()`.
  **L75 CN**: 以 `(__m512h)__builtin_ia32_undef512()` 从当前函数返回。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L78 EN**: Continues the surrounding expression or declaration: `static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L78 CN**: 继续构造周围的表达式或声明：`static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L79 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_set1_ph(_Float16 __h) {`.
  **L79 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_set1_ph(_Float16 __h) {`。
- **L80 EN**: Returns from the current function with `(__m512h)(__v32hf){__h, __h, __h, __h, __h, __h, __h, __h,`.
  **L80 CN**: 以 `(__m512h)(__v32hf){__h, __h, __h, __h, __h, __h, __h, __h,` 从当前函数返回。
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__h, __h, __h, __h, __h, __h, __h, __h,`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`__h, __h, __h, __h, __h, __h, __h, __h,`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__h, __h, __h, __h, __h, __h, __h, __h,`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`__h, __h, __h, __h, __h, __h, __h, __h,`。
- **L83 EN**: Adds a standalone statement or declaration: `__h, __h, __h, __h, __h, __h, __h, __h};`.
  **L83 CN**: 添加一条独立语句或声明：`__h, __h, __h, __h, __h, __h, __h, __h};`。
- **L84 EN**: Closes the current lexical scope or compound statement.
  **L84 CN**: 结束当前词法作用域或复合语句块。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L86 EN**: Continues the surrounding expression or declaration: `static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L86 CN**: 继续构造周围的表达式或声明：`static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm512_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,`.
  **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm512_set_ph(_Float16 __h1, _Float16 __h2, _Float16 __h3, _Float16 __h4,`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8,`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 __h5, _Float16 __h6, _Float16 __h7, _Float16 __h8,`。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 __h9, _Float16 __h10, _Float16 __h11, _Float16 __h12,`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 __h9, _Float16 __h10, _Float16 __h11, _Float16 __h12,`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 __h13, _Float16 __h14, _Float16 __h15, _Float16 __h16,`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 __h13, _Float16 __h14, _Float16 __h15, _Float16 __h16,`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 __h17, _Float16 __h18, _Float16 __h19, _Float16 __h20,`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 __h17, _Float16 __h18, _Float16 __h19, _Float16 __h20,`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 __h21, _Float16 __h22, _Float16 __h23, _Float16 __h24,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 __h21, _Float16 __h22, _Float16 __h23, _Float16 __h24,`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 __h25, _Float16 __h26, _Float16 __h27, _Float16 __h28,`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 __h25, _Float16 __h26, _Float16 __h27, _Float16 __h28,`。
- **L94 EN**: Continues the surrounding expression or declaration: `_Float16 __h29, _Float16 __h30, _Float16 __h31, _Float16 __h32) {`.
  **L94 CN**: 继续构造周围的表达式或声明：`_Float16 __h29, _Float16 __h30, _Float16 __h31, _Float16 __h32) {`。
- **L95 EN**: Returns from the current function with `(__m512h)(__v32hf){__h32, __h31, __h30, __h29, __h28, __h27, __h26,`.
  **L95 CN**: 以 `(__m512h)(__v32hf){__h32, __h31, __h30, __h29, __h28, __h27, __h26,` 从当前函数返回。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__h25, __h24, __h23, __h22, __h21, __h20, __h19,`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`__h25, __h24, __h23, __h22, __h21, __h20, __h19,`。

### Lines 97-120

````c
                            __h18, __h17, __h16, __h15, __h14, __h13, __h12,
                            __h11, __h10, __h9,  __h8,  __h7,  __h6,  __h5,
                            __h4,  __h3,  __h2,  __h1};
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_setr_ph(
    _Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,
    _Float16 e5, _Float16 e6, _Float16 e7, _Float16 e8, _Float16 e9,
    _Float16 e10, _Float16 e11, _Float16 e12, _Float16 e13, _Float16 e14,
    _Float16 e15, _Float16 e16, _Float16 e17, _Float16 e18, _Float16 e19,
    _Float16 e20, _Float16 e21, _Float16 e22, _Float16 e23, _Float16 e24,
    _Float16 e25, _Float16 e26, _Float16 e27, _Float16 e28, _Float16 e29,
    _Float16 e30, _Float16 e31) {
  return _mm512_set_ph(e31, e30, e29, e28, e27, e26, e25, e24, e23, e22, e21,
                       e20, e19, e18, e17, e16, e15, e14, e13, e12, e11, e10,
                       e9, e8, e7, e6, e5, e4, e3, e2, e1, e0);
}

static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_set1_pch(_Float16 _Complex __h) {
  return (__m512h)_mm512_set1_ps(__builtin_bit_cast(float, __h));
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_ps(__m128h __a) {
````
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__h18, __h17, __h16, __h15, __h14, __h13, __h12,`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`__h18, __h17, __h16, __h15, __h14, __h13, __h12,`。
- **L98 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__h11, __h10, __h9,  __h8,  __h7,  __h6,  __h5,`.
  **L98 CN**: 继续一个多行参数列表、初始化器或聚合项：`__h11, __h10, __h9,  __h8,  __h7,  __h6,  __h5,`。
- **L99 EN**: Adds a standalone statement or declaration: `__h4,  __h3,  __h2,  __h1};`.
  **L99 CN**: 添加一条独立语句或声明：`__h4,  __h3,  __h2,  __h1};`。
- **L100 EN**: Closes the current lexical scope or compound statement.
  **L100 CN**: 结束当前词法作用域或复合语句块。
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L102 EN**: Continues logic associated with callable symbol `_mm512_setr_ph`.
  **L102 CN**: 继续与可调用符号 `_mm512_setr_ph` 相关的逻辑。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 e0, _Float16 e1, _Float16 e2, _Float16 e3, _Float16 e4,`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 e5, _Float16 e6, _Float16 e7, _Float16 e8, _Float16 e9,`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 e5, _Float16 e6, _Float16 e7, _Float16 e8, _Float16 e9,`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 e10, _Float16 e11, _Float16 e12, _Float16 e13, _Float16 e14,`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 e10, _Float16 e11, _Float16 e12, _Float16 e13, _Float16 e14,`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 e15, _Float16 e16, _Float16 e17, _Float16 e18, _Float16 e19,`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 e15, _Float16 e16, _Float16 e17, _Float16 e18, _Float16 e19,`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 e20, _Float16 e21, _Float16 e22, _Float16 e23, _Float16 e24,`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 e20, _Float16 e21, _Float16 e22, _Float16 e23, _Float16 e24,`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_Float16 e25, _Float16 e26, _Float16 e27, _Float16 e28, _Float16 e29,`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`_Float16 e25, _Float16 e26, _Float16 e27, _Float16 e28, _Float16 e29,`。
- **L109 EN**: Continues the surrounding expression or declaration: `_Float16 e30, _Float16 e31) {`.
  **L109 CN**: 继续构造周围的表达式或声明：`_Float16 e30, _Float16 e31) {`。
- **L110 EN**: Returns from the current function with `_mm512_set_ph(e31, e30, e29, e28, e27, e26, e25, e24, e23, e22, e21,`.
  **L110 CN**: 以 `_mm512_set_ph(e31, e30, e29, e28, e27, e26, e25, e24, e23, e22, e21,` 从当前函数返回。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `e20, e19, e18, e17, e16, e15, e14, e13, e12, e11, e10,`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`e20, e19, e18, e17, e16, e15, e14, e13, e12, e11, e10,`。
- **L112 EN**: Adds a standalone statement or declaration: `e9, e8, e7, e6, e5, e4, e3, e2, e1, e0);`.
  **L112 CN**: 添加一条独立语句或声明：`e9, e8, e7, e6, e5, e4, e3, e2, e1, e0);`。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Continues the surrounding expression or declaration: `static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L115 CN**: 继续构造周围的表达式或声明：`static __inline __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L116 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_set1_pch(_Float16 _Complex __h) {`.
  **L116 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_set1_pch(_Float16 _Complex __h) {`。
- **L117 EN**: Returns from the current function with `(__m512h)_mm512_set1_ps(__builtin_bit_cast(float, __h))`.
  **L117 CN**: 以 `(__m512h)_mm512_set1_ps(__builtin_bit_cast(float, __h))` 从当前函数返回。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L120 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_ps(__m128h __a) {`.
  **L120 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128 __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_ps(__m128h __a) {`。

### Lines 121-144

````c
  return (__m128)__a;
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castph_ps(__m256h __a) {
  return (__m256)__a;
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castph_ps(__m512h __a) {
  return (__m512)__a;
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_pd(__m128h __a) {
  return (__m128d)__a;
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castph_pd(__m256h __a) {
  return (__m256d)__a;
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castph_pd(__m512h __a) {
  return (__m512d)__a;
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_si128(__m128h __a) {
````
- **L121 EN**: Returns from the current function with `(__m128)__a`.
  **L121 CN**: 以 `(__m128)__a` 从当前函数返回。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Blank line separating nearby declarations or logic blocks.
  **L123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L124 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castph_ps(__m256h __a) {`.
  **L124 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castph_ps(__m256h __a) {`。
- **L125 EN**: Returns from the current function with `(__m256)__a`.
  **L125 CN**: 以 `(__m256)__a` 从当前函数返回。
- **L126 EN**: Closes the current lexical scope or compound statement.
  **L126 CN**: 结束当前词法作用域或复合语句块。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L128 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castph_ps(__m512h __a) {`.
  **L128 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512 __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castph_ps(__m512h __a) {`。
- **L129 EN**: Returns from the current function with `(__m512)__a`.
  **L129 CN**: 以 `(__m512)__a` 从当前函数返回。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L132 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_pd(__m128h __a) {`.
  **L132 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_pd(__m128h __a) {`。
- **L133 EN**: Returns from the current function with `(__m128d)__a`.
  **L133 CN**: 以 `(__m128d)__a` 从当前函数返回。
- **L134 EN**: Closes the current lexical scope or compound statement.
  **L134 CN**: 结束当前词法作用域或复合语句块。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L136 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castph_pd(__m256h __a) {`.
  **L136 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castph_pd(__m256h __a) {`。
- **L137 EN**: Returns from the current function with `(__m256d)__a`.
  **L137 CN**: 以 `(__m256d)__a` 从当前函数返回。
- **L138 EN**: Closes the current lexical scope or compound statement.
  **L138 CN**: 结束当前词法作用域或复合语句块。
- **L139 EN**: Blank line separating nearby declarations or logic blocks.
  **L139 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L140 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castph_pd(__m512h __a) {`.
  **L140 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512d __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castph_pd(__m512h __a) {`。
- **L141 EN**: Returns from the current function with `(__m512d)__a`.
  **L141 CN**: 以 `(__m512d)__a` 从当前函数返回。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L144 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_si128(__m128h __a) {`.
  **L144 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castph_si128(__m128h __a) {`。

### Lines 145-168

````c
  return (__m128i)__a;
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_castph_si256(__m256h __a) {
  return (__m256i)__a;
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_castph_si512(__m512h __a) {
  return (__m512i)__a;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castps_ph(__m128 __a) {
  return (__m128h)__a;
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castps_ph(__m256 __a) {
  return (__m256h)__a;
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castps_ph(__m512 __a) {
  return (__m512h)__a;
}
````
- **L145 EN**: Returns from the current function with `(__m128i)__a`.
  **L145 CN**: 以 `(__m128i)__a` 从当前函数返回。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L148 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castph_si256(__m256h __a) {`.
  **L149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castph_si256(__m256h __a) {`。
- **L150 EN**: Returns from the current function with `(__m256i)__a`.
  **L150 CN**: 以 `(__m256i)__a` 从当前函数返回。
- **L151 EN**: Closes the current lexical scope or compound statement.
  **L151 CN**: 结束当前词法作用域或复合语句块。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L153 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L153 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L154 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_castph_si512(__m512h __a) {`.
  **L154 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_castph_si512(__m512h __a) {`。
- **L155 EN**: Returns from the current function with `(__m512i)__a`.
  **L155 CN**: 以 `(__m512i)__a` 从当前函数返回。
- **L156 EN**: Closes the current lexical scope or compound statement.
  **L156 CN**: 结束当前词法作用域或复合语句块。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L158 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castps_ph(__m128 __a) {`.
  **L158 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castps_ph(__m128 __a) {`。
- **L159 EN**: Returns from the current function with `(__m128h)__a`.
  **L159 CN**: 以 `(__m128h)__a` 从当前函数返回。
- **L160 EN**: Closes the current lexical scope or compound statement.
  **L160 CN**: 结束当前词法作用域或复合语句块。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castps_ph(__m256 __a) {`.
  **L162 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castps_ph(__m256 __a) {`。
- **L163 EN**: Returns from the current function with `(__m256h)__a`.
  **L163 CN**: 以 `(__m256h)__a` 从当前函数返回。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L166 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castps_ph(__m512 __a) {`.
  **L166 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castps_ph(__m512 __a) {`。
- **L167 EN**: Returns from the current function with `(__m512h)__a`.
  **L167 CN**: 以 `(__m512h)__a` 从当前函数返回。
- **L168 EN**: Closes the current lexical scope or compound statement.
  **L168 CN**: 结束当前词法作用域或复合语句块。

### Lines 169-192

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castpd_ph(__m128d __a) {
  return (__m128h)__a;
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castpd_ph(__m256d __a) {
  return (__m256h)__a;
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castpd_ph(__m512d __a) {
  return (__m512h)__a;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castsi128_ph(__m128i __a) {
  return (__m128h)__a;
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_castsi256_ph(__m256i __a) {
  return (__m256h)__a;
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_castsi512_ph(__m512i __a) {
````
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L170 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castpd_ph(__m128d __a) {`.
  **L170 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castpd_ph(__m128d __a) {`。
- **L171 EN**: Returns from the current function with `(__m128h)__a`.
  **L171 CN**: 以 `(__m128h)__a` 从当前函数返回。
- **L172 EN**: Closes the current lexical scope or compound statement.
  **L172 CN**: 结束当前词法作用域或复合语句块。
- **L173 EN**: Blank line separating nearby declarations or logic blocks.
  **L173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L174 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castpd_ph(__m256d __a) {`.
  **L174 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR _mm256_castpd_ph(__m256d __a) {`。
- **L175 EN**: Returns from the current function with `(__m256h)__a`.
  **L175 CN**: 以 `(__m256h)__a` 从当前函数返回。
- **L176 EN**: Closes the current lexical scope or compound statement.
  **L176 CN**: 结束当前词法作用域或复合语句块。
- **L177 EN**: Blank line separating nearby declarations or logic blocks.
  **L177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L178 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castpd_ph(__m512d __a) {`.
  **L178 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_castpd_ph(__m512d __a) {`。
- **L179 EN**: Returns from the current function with `(__m512h)__a`.
  **L179 CN**: 以 `(__m512h)__a` 从当前函数返回。
- **L180 EN**: Closes the current lexical scope or compound statement.
  **L180 CN**: 结束当前词法作用域或复合语句块。
- **L181 EN**: Blank line separating nearby declarations or logic blocks.
  **L181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L182 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castsi128_ph(__m128i __a) {`.
  **L182 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_castsi128_ph(__m128i __a) {`。
- **L183 EN**: Returns from the current function with `(__m128h)__a`.
  **L183 CN**: 以 `(__m128h)__a` 从当前函数返回。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L186 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L187 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castsi256_ph(__m256i __a) {`.
  **L187 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castsi256_ph(__m256i __a) {`。
- **L188 EN**: Returns from the current function with `(__m256h)__a`.
  **L188 CN**: 以 `(__m256h)__a` 从当前函数返回。
- **L189 EN**: Closes the current lexical scope or compound statement.
  **L189 CN**: 结束当前词法作用域或复合语句块。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L191 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L191 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L192 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_castsi512_ph(__m512i __a) {`.
  **L192 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_castsi512_ph(__m512i __a) {`。

### Lines 193-216

````c
  return (__m512h)__a;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_castph256_ph128(__m256h __a) {
  return __builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_castph512_ph128(__m512h __a) {
  return __builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_castph512_ph256(__m512h __a) {
  return __builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,
                                 12, 13, 14, 15);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_castph128_ph256(__m128h __a) {
  return __builtin_shufflevector(__a, __builtin_nondeterministic_value(__a),
                                  0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);
}
````
- **L193 EN**: Returns from the current function with `(__m512h)__a`.
  **L193 CN**: 以 `(__m512h)__a` 从当前函数返回。
- **L194 EN**: Closes the current lexical scope or compound statement.
  **L194 CN**: 结束当前词法作用域或复合语句块。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L196 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L196 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L197 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castph256_ph128(__m256h __a) {`.
  **L197 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castph256_ph128(__m256h __a) {`。
- **L198 EN**: Returns from the current function with `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7)`.
  **L198 CN**: 以 `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7)` 从当前函数返回。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L201 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L201 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L202 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_castph512_ph128(__m512h __a) {`.
  **L202 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_castph512_ph128(__m512h __a) {`。
- **L203 EN**: Returns from the current function with `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7)`.
  **L203 CN**: 以 `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7)` 从当前函数返回。
- **L204 EN**: Closes the current lexical scope or compound statement.
  **L204 CN**: 结束当前词法作用域或复合语句块。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L206 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L207 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_castph512_ph256(__m512h __a) {`.
  **L207 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_castph512_ph256(__m512h __a) {`。
- **L208 EN**: Returns from the current function with `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,`.
  **L208 CN**: 以 `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,` 从当前函数返回。
- **L209 EN**: Adds a standalone statement or declaration: `12, 13, 14, 15);`.
  **L209 CN**: 添加一条独立语句或声明：`12, 13, 14, 15);`。
- **L210 EN**: Closes the current lexical scope or compound statement.
  **L210 CN**: 结束当前词法作用域或复合语句块。
- **L211 EN**: Blank line separating nearby declarations or logic blocks.
  **L211 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L212 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L212 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L213 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castph128_ph256(__m128h __a) {`.
  **L213 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castph128_ph256(__m128h __a) {`。
- **L214 EN**: Returns from the current function with `__builtin_shufflevector(__a, __builtin_nondeterministic_value(__a),`.
  **L214 CN**: 以 `__builtin_shufflevector(__a, __builtin_nondeterministic_value(__a),` 从当前函数返回。
- **L215 EN**: Adds a standalone statement or declaration: `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);`.
  **L215 CN**: 添加一条独立语句或声明：`0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);`。
- **L216 EN**: Closes the current lexical scope or compound statement.
  **L216 CN**: 结束当前词法作用域或复合语句块。

### Lines 217-240

````c

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_castph128_ph512(__m128h __a) {
  __m256h __b = __builtin_nondeterministic_value(__b);
  return __builtin_shufflevector(
      __builtin_shufflevector(__a, __builtin_nondeterministic_value(__a),
                              0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15),
      __b, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19,
      20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_castph256_ph512(__m256h __a) {
  return __builtin_shufflevector(__a, __builtin_nondeterministic_value(__a), 0,
                                 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14,
                                 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26,
                                 27, 28, 29, 30, 31);
}

/// Constructs a 256-bit floating-point vector of [16 x half] from a
///    128-bit floating-point vector of [8 x half]. The lower 128 bits
///    contain the value of the source vector. The upper 384 bits are set
///    to zero.
///
````
- **L217 EN**: Blank line separating nearby declarations or logic blocks.
  **L217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L218 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L218 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L219 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_castph128_ph512(__m128h __a) {`.
  **L219 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_castph128_ph512(__m128h __a) {`。
- **L220 EN**: Initializes variable `__b` from the expression on the right-hand side.
  **L220 CN**: 使用右侧表达式初始化变量 `__b`。
- **L221 EN**: Returns from the current function with `__builtin_shufflevector(`.
  **L221 CN**: 以 `__builtin_shufflevector(` 从当前函数返回。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_shufflevector(__a, __builtin_nondeterministic_value(__a),`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_shufflevector(__a, __builtin_nondeterministic_value(__a),`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15),`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15),`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__b, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19,`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`__b, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19,`。
- **L225 EN**: Adds a standalone statement or declaration: `20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31);`.
  **L225 CN**: 添加一条独立语句或声明：`20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31);`。
- **L226 EN**: Closes the current lexical scope or compound statement.
  **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Blank line separating nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L228 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L228 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L229 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_castph256_ph512(__m256h __a) {`.
  **L229 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_castph256_ph512(__m256h __a) {`。
- **L230 EN**: Returns from the current function with `__builtin_shufflevector(__a, __builtin_nondeterministic_value(__a), 0,`.
  **L230 CN**: 以 `__builtin_shufflevector(__a, __builtin_nondeterministic_value(__a), 0,` 从当前函数返回。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14,`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14,`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26,`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26,`。
- **L233 EN**: Adds a standalone statement or declaration: `27, 28, 29, 30, 31);`.
  **L233 CN**: 添加一条独立语句或声明：`27, 28, 29, 30, 31);`。
- **L234 EN**: Closes the current lexical scope or compound statement.
  **L234 CN**: 结束当前词法作用域或复合语句块。
- **L235 EN**: Blank line separating nearby declarations or logic blocks.
  **L235 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L236 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 256-bit floating-point vector of [16 x half] from a`.
  **L236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 256-bit floating-point vector of [16 x half] from a`。
- **L237 EN**: Comment explains nearby logic, constraints, or intent: `128-bit floating-point vector of [8 x half]. The lower 128 bits`.
  **L237 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit floating-point vector of [8 x half]. The lower 128 bits`。
- **L238 EN**: Comment explains nearby logic, constraints, or intent: `contain the value of the source vector. The upper 384 bits are set`.
  **L238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`contain the value of the source vector. The upper 384 bits are set`。
- **L239 EN**: Comment explains nearby logic, constraints, or intent: `to zero.`.
  **L239 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to zero.`。
- **L240 EN**: Separator comment used for visual grouping.
  **L240 CN**: 用于视觉分组的分隔注释。

### Lines 241-264

````c
/// \headerfile <x86intrin.h>
///
/// This intrinsic has no corresponding instruction.
///
/// \param __a
///    A 128-bit vector of [8 x half].
/// \returns A 512-bit floating-point vector of [16 x half]. The lower 128 bits
///    contain the value of the parameter. The upper 384 bits are set to zero.
static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_zextph128_ph256(__m128h __a) {
  return __builtin_shufflevector(__a, (__v8hf)_mm_setzero_ph(), 0, 1, 2, 3, 4,
                                 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);
}

/// Constructs a 512-bit floating-point vector of [32 x half] from a
///    128-bit floating-point vector of [8 x half]. The lower 128 bits
///    contain the value of the source vector. The upper 384 bits are set
///    to zero.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic has no corresponding instruction.
///
/// \param __a
````
- **L241 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L242 EN**: Separator comment used for visual grouping.
  **L242 CN**: 用于视觉分组的分隔注释。
- **L243 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic has no corresponding instruction.`.
  **L243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic has no corresponding instruction.`。
- **L244 EN**: Separator comment used for visual grouping.
  **L244 CN**: 用于视觉分组的分隔注释。
- **L245 EN**: Comment explains nearby logic, constraints, or intent: `param __a`.
  **L245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __a`。
- **L246 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x half].`.
  **L246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x half].`。
- **L247 EN**: Comment explains nearby logic, constraints, or intent: `returns A 512-bit floating-point vector of [16 x half]. The lower 128 bits`.
  **L247 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 512-bit floating-point vector of [16 x half]. The lower 128 bits`。
- **L248 EN**: Comment explains nearby logic, constraints, or intent: `contain the value of the parameter. The upper 384 bits are set to zero.`.
  **L248 CN**: 注释解释附近代码的逻辑、约束或设计意图：`contain the value of the parameter. The upper 384 bits are set to zero.`。
- **L249 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L249 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L250 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_zextph128_ph256(__m128h __a) {`.
  **L250 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_zextph128_ph256(__m128h __a) {`。
- **L251 EN**: Returns from the current function with `__builtin_shufflevector(__a, (__v8hf)_mm_setzero_ph(), 0, 1, 2, 3, 4,`.
  **L251 CN**: 以 `__builtin_shufflevector(__a, (__v8hf)_mm_setzero_ph(), 0, 1, 2, 3, 4,` 从当前函数返回。
- **L252 EN**: Adds a standalone statement or declaration: `5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);`.
  **L252 CN**: 添加一条独立语句或声明：`5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);`。
- **L253 EN**: Closes the current lexical scope or compound statement.
  **L253 CN**: 结束当前词法作用域或复合语句块。
- **L254 EN**: Blank line separating nearby declarations or logic blocks.
  **L254 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L255 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 512-bit floating-point vector of [32 x half] from a`.
  **L255 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 512-bit floating-point vector of [32 x half] from a`。
- **L256 EN**: Comment explains nearby logic, constraints, or intent: `128-bit floating-point vector of [8 x half]. The lower 128 bits`.
  **L256 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128-bit floating-point vector of [8 x half]. The lower 128 bits`。
- **L257 EN**: Comment explains nearby logic, constraints, or intent: `contain the value of the source vector. The upper 384 bits are set`.
  **L257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`contain the value of the source vector. The upper 384 bits are set`。
- **L258 EN**: Comment explains nearby logic, constraints, or intent: `to zero.`.
  **L258 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to zero.`。
- **L259 EN**: Separator comment used for visual grouping.
  **L259 CN**: 用于视觉分组的分隔注释。
- **L260 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L261 EN**: Separator comment used for visual grouping.
  **L261 CN**: 用于视觉分组的分隔注释。
- **L262 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic has no corresponding instruction.`.
  **L262 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic has no corresponding instruction.`。
- **L263 EN**: Separator comment used for visual grouping.
  **L263 CN**: 用于视觉分组的分隔注释。
- **L264 EN**: Comment explains nearby logic, constraints, or intent: `param __a`.
  **L264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __a`。

### Lines 265-288

````c
///    A 128-bit vector of [8 x half].
/// \returns A 512-bit floating-point vector of [32 x half]. The lower 128 bits
///    contain the value of the parameter. The upper 384 bits are set to zero.
static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_zextph128_ph512(__m128h __a) {
  return __builtin_shufflevector(
      __a, (__v8hf)_mm_setzero_ph(), 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12,
      13, 14, 15, 8, 9, 10, 11, 12, 13, 14, 15, 8, 9, 10, 11, 12, 13, 14, 15);
}

/// Constructs a 512-bit floating-point vector of [32 x half] from a
///    256-bit floating-point vector of [16 x half]. The lower 256 bits
///    contain the value of the source vector. The upper 256 bits are set
///    to zero.
///
/// \headerfile <x86intrin.h>
///
/// This intrinsic has no corresponding instruction.
///
/// \param __a
///    A 256-bit vector of [16 x half].
/// \returns A 512-bit floating-point vector of [32 x half]. The lower 256 bits
///    contain the value of the parameter. The upper 256 bits are set to zero.
static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
````
- **L265 EN**: Comment explains nearby logic, constraints, or intent: `A 128-bit vector of [8 x half].`.
  **L265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 128-bit vector of [8 x half].`。
- **L266 EN**: Comment explains nearby logic, constraints, or intent: `returns A 512-bit floating-point vector of [32 x half]. The lower 128 bits`.
  **L266 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 512-bit floating-point vector of [32 x half]. The lower 128 bits`。
- **L267 EN**: Comment explains nearby logic, constraints, or intent: `contain the value of the parameter. The upper 384 bits are set to zero.`.
  **L267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`contain the value of the parameter. The upper 384 bits are set to zero.`。
- **L268 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L268 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L269 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_zextph128_ph512(__m128h __a) {`.
  **L269 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_zextph128_ph512(__m128h __a) {`。
- **L270 EN**: Returns from the current function with `__builtin_shufflevector(`.
  **L270 CN**: 以 `__builtin_shufflevector(` 从当前函数返回。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__a, (__v8hf)_mm_setzero_ph(), 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12,`.
  **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`__a, (__v8hf)_mm_setzero_ph(), 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12,`。
- **L272 EN**: Adds a standalone statement or declaration: `13, 14, 15, 8, 9, 10, 11, 12, 13, 14, 15, 8, 9, 10, 11, 12, 13, 14, 15);`.
  **L272 CN**: 添加一条独立语句或声明：`13, 14, 15, 8, 9, 10, 11, 12, 13, 14, 15, 8, 9, 10, 11, 12, 13, 14, 15);`。
- **L273 EN**: Closes the current lexical scope or compound statement.
  **L273 CN**: 结束当前词法作用域或复合语句块。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L275 EN**: Comment explains nearby logic, constraints, or intent: `Constructs a 512-bit floating-point vector of [32 x half] from a`.
  **L275 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructs a 512-bit floating-point vector of [32 x half] from a`。
- **L276 EN**: Comment explains nearby logic, constraints, or intent: `256-bit floating-point vector of [16 x half]. The lower 256 bits`.
  **L276 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256-bit floating-point vector of [16 x half]. The lower 256 bits`。
- **L277 EN**: Comment explains nearby logic, constraints, or intent: `contain the value of the source vector. The upper 256 bits are set`.
  **L277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`contain the value of the source vector. The upper 256 bits are set`。
- **L278 EN**: Comment explains nearby logic, constraints, or intent: `to zero.`.
  **L278 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to zero.`。
- **L279 EN**: Separator comment used for visual grouping.
  **L279 CN**: 用于视觉分组的分隔注释。
- **L280 EN**: Comment explains nearby logic, constraints, or intent: `headerfile <x86intrin.h>`.
  **L280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`headerfile <x86intrin.h>`。
- **L281 EN**: Separator comment used for visual grouping.
  **L281 CN**: 用于视觉分组的分隔注释。
- **L282 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic has no corresponding instruction.`.
  **L282 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic has no corresponding instruction.`。
- **L283 EN**: Separator comment used for visual grouping.
  **L283 CN**: 用于视觉分组的分隔注释。
- **L284 EN**: Comment explains nearby logic, constraints, or intent: `param __a`.
  **L284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`param __a`。
- **L285 EN**: Comment explains nearby logic, constraints, or intent: `A 256-bit vector of [16 x half].`.
  **L285 CN**: 注释解释附近代码的逻辑、约束或设计意图：`A 256-bit vector of [16 x half].`。
- **L286 EN**: Comment explains nearby logic, constraints, or intent: `returns A 512-bit floating-point vector of [32 x half]. The lower 256 bits`.
  **L286 CN**: 注释解释附近代码的逻辑、约束或设计意图：`returns A 512-bit floating-point vector of [32 x half]. The lower 256 bits`。
- **L287 EN**: Comment explains nearby logic, constraints, or intent: `contain the value of the parameter. The upper 256 bits are set to zero.`.
  **L287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`contain the value of the parameter. The upper 256 bits are set to zero.`。
- **L288 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L288 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。

### Lines 289-312

````c
_mm512_zextph256_ph512(__m256h __a) {
  return __builtin_shufflevector(__a, (__v16hf)_mm256_setzero_ph(), 0, 1, 2, 3,
                                 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,
                                 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28,
                                 29, 30, 31);
}

#define _mm_comi_round_sh(A, B, P, R)                                          \
  __builtin_ia32_vcomish((__v8hf)A, (__v8hf)B, (int)(P), (int)(R))

#define _mm_comi_sh(A, B, pred)                                                \
  _mm_comi_round_sh((A), (B), (pred), _MM_FROUND_CUR_DIRECTION)

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comieq_sh(__m128h __A,
                                                          __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_EQ_OS,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comilt_sh(__m128h __A,
                                                          __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LT_OS,
                                _MM_FROUND_CUR_DIRECTION);
}
````
- **L289 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_zextph256_ph512(__m256h __a) {`.
  **L289 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_zextph256_ph512(__m256h __a) {`。
- **L290 EN**: Returns from the current function with `__builtin_shufflevector(__a, (__v16hf)_mm256_setzero_ph(), 0, 1, 2, 3,`.
  **L290 CN**: 以 `__builtin_shufflevector(__a, (__v16hf)_mm256_setzero_ph(), 0, 1, 2, 3,` 从当前函数返回。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28,`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28,`。
- **L293 EN**: Adds a standalone statement or declaration: `29, 30, 31);`.
  **L293 CN**: 添加一条独立语句或声明：`29, 30, 31);`。
- **L294 EN**: Closes the current lexical scope or compound statement.
  **L294 CN**: 结束当前词法作用域或复合语句块。
- **L295 EN**: Blank line separating nearby declarations or logic blocks.
  **L295 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L296 EN**: Defines macro `_mm_comi_round_sh(A, B, P, R)` for conditional compilation, shorthand, or API generation.
  **L296 CN**: 定义宏 `_mm_comi_round_sh(A, B, P, R)`，用于条件编译、简写或 API 生成。
- **L297 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcomish`.
  **L297 CN**: 继续与可调用符号 `__builtin_ia32_vcomish` 相关的逻辑。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L299 EN**: Defines macro `_mm_comi_sh(A, B, pred)` for conditional compilation, shorthand, or API generation.
  **L299 CN**: 定义宏 `_mm_comi_sh(A, B, pred)`，用于条件编译、简写或 API 生成。
- **L300 EN**: Continues logic associated with callable symbol `_mm_comi_round_sh`.
  **L300 CN**: 继续与可调用符号 `_mm_comi_round_sh` 相关的逻辑。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comieq_sh(__m128h __A,`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comieq_sh(__m128h __A,`。
- **L303 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L303 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L304 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_EQ_OS,`.
  **L304 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_EQ_OS,` 从当前函数返回。
- **L305 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L305 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L306 EN**: Closes the current lexical scope or compound statement.
  **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comilt_sh(__m128h __A,`.
  **L308 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comilt_sh(__m128h __A,`。
- **L309 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L309 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L310 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LT_OS,`.
  **L310 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LT_OS,` 从当前函数返回。
- **L311 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L311 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L312 EN**: Closes the current lexical scope or compound statement.
  **L312 CN**: 结束当前词法作用域或复合语句块。

### Lines 313-336

````c

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comile_sh(__m128h __A,
                                                          __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LE_OS,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comigt_sh(__m128h __A,
                                                          __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GT_OS,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comige_sh(__m128h __A,
                                                          __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GE_OS,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comineq_sh(__m128h __A,
                                                           __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_NEQ_US,
                                _MM_FROUND_CUR_DIRECTION);
}
````
- **L313 EN**: Blank line separating nearby declarations or logic blocks.
  **L313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comile_sh(__m128h __A,`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comile_sh(__m128h __A,`。
- **L315 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L315 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L316 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LE_OS,`.
  **L316 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LE_OS,` 从当前函数返回。
- **L317 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L317 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L318 EN**: Closes the current lexical scope or compound statement.
  **L318 CN**: 结束当前词法作用域或复合语句块。
- **L319 EN**: Blank line separating nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comigt_sh(__m128h __A,`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comigt_sh(__m128h __A,`。
- **L321 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L321 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L322 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GT_OS,`.
  **L322 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GT_OS,` 从当前函数返回。
- **L323 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L323 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comige_sh(__m128h __A,`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comige_sh(__m128h __A,`。
- **L327 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L327 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L328 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GE_OS,`.
  **L328 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GE_OS,` 从当前函数返回。
- **L329 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L329 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L330 EN**: Closes the current lexical scope or compound statement.
  **L330 CN**: 结束当前词法作用域或复合语句块。
- **L331 EN**: Blank line separating nearby declarations or logic blocks.
  **L331 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comineq_sh(__m128h __A,`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comineq_sh(__m128h __A,`。
- **L333 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L333 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L334 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_NEQ_US,`.
  **L334 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_NEQ_US,` 从当前函数返回。
- **L335 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L335 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。

### Lines 337-360

````c

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomieq_sh(__m128h __A,
                                                           __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_EQ_OQ,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomilt_sh(__m128h __A,
                                                           __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LT_OQ,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomile_sh(__m128h __A,
                                                           __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LE_OQ,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomigt_sh(__m128h __A,
                                                           __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GT_OQ,
                                _MM_FROUND_CUR_DIRECTION);
}
````
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomieq_sh(__m128h __A,`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomieq_sh(__m128h __A,`。
- **L339 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L339 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L340 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_EQ_OQ,`.
  **L340 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_EQ_OQ,` 从当前函数返回。
- **L341 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L341 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomilt_sh(__m128h __A,`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomilt_sh(__m128h __A,`。
- **L345 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L345 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L346 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LT_OQ,`.
  **L346 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LT_OQ,` 从当前函数返回。
- **L347 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L347 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomile_sh(__m128h __A,`.
  **L350 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomile_sh(__m128h __A,`。
- **L351 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L351 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L352 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LE_OQ,`.
  **L352 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_LE_OQ,` 从当前函数返回。
- **L353 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L353 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L354 EN**: Closes the current lexical scope or compound statement.
  **L354 CN**: 结束当前词法作用域或复合语句块。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomigt_sh(__m128h __A,`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomigt_sh(__m128h __A,`。
- **L357 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L357 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L358 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GT_OQ,`.
  **L358 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GT_OQ,` 从当前函数返回。
- **L359 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L359 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384

````c

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomige_sh(__m128h __A,
                                                           __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GE_OQ,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomineq_sh(__m128h __A,
                                                            __m128h __B) {
  return __builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_NEQ_UQ,
                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_add_ph(__m512h __A,
                                                              __m512h __B) {
  return (__m512h)((__v32hf)__A + (__v32hf)__B);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_add_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512(
      (__mmask32)__U, (__v32hf)_mm512_add_ph(__A, __B), (__v32hf)__W);
}

````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomige_sh(__m128h __A,`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomige_sh(__m128h __A,`。
- **L363 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L363 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L364 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GE_OQ,`.
  **L364 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_GE_OQ,` 从当前函数返回。
- **L365 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L365 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L366 EN**: Closes the current lexical scope or compound statement.
  **L366 CN**: 结束当前词法作用域或复合语句块。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomineq_sh(__m128h __A,`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_ucomineq_sh(__m128h __A,`。
- **L369 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L369 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L370 EN**: Returns from the current function with `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_NEQ_UQ,`.
  **L370 CN**: 以 `__builtin_ia32_vcomish((__v8hf)__A, (__v8hf)__B, _CMP_NEQ_UQ,` 从当前函数返回。
- **L371 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L371 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L372 EN**: Closes the current lexical scope or compound statement.
  **L372 CN**: 结束当前词法作用域或复合语句块。
- **L373 EN**: Blank line separating nearby declarations or logic blocks.
  **L373 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_add_ph(__m512h __A,`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_add_ph(__m512h __A,`。
- **L375 EN**: Continues the surrounding expression or declaration: `__m512h __B) {`.
  **L375 CN**: 继续构造周围的表达式或声明：`__m512h __B) {`。
- **L376 EN**: Returns from the current function with `(__m512h)((__v32hf)__A + (__v32hf)__B)`.
  **L376 CN**: 以 `(__m512h)((__v32hf)__A + (__v32hf)__B)` 从当前函数返回。
- **L377 EN**: Closes the current lexical scope or compound statement.
  **L377 CN**: 结束当前词法作用域或复合语句块。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L379 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L379 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L380 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_add_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`.
  **L380 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_add_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`。
- **L381 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512(`.
  **L381 CN**: 以 `(__m512h)__builtin_ia32_selectph_512(` 从当前函数返回。
- **L382 EN**: Executes a call or declaration centered on `statement`.
  **L382 CN**: 执行以 `statement` 为核心的调用或声明。
- **L383 EN**: Closes the current lexical scope or compound statement.
  **L383 CN**: 结束当前词法作用域或复合语句块。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````c
static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_add_ph(__mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512((__mmask32)__U,
                                              (__v32hf)_mm512_add_ph(__A, __B),
                                              (__v32hf)_mm512_setzero_ph());
}

#define _mm512_add_round_ph(A, B, R)                                           \
  ((__m512h)__builtin_ia32_addph512((__v32hf)(__m512h)(A),                     \
                                    (__v32hf)(__m512h)(B), (int)(R)))

#define _mm512_mask_add_round_ph(W, U, A, B, R)                                \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_add_round_ph((A), (B), (R)),             \
      (__v32hf)(__m512h)(W)))

#define _mm512_maskz_add_round_ph(U, A, B, R)                                  \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_add_round_ph((A), (B), (R)),             \
      (__v32hf)_mm512_setzero_ph()))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_sub_ph(__m512h __A,
                                                              __m512h __B) {
  return (__m512h)((__v32hf)__A - (__v32hf)__B);
````
- **L385 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L385 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L386 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_add_ph(__mmask32 __U, __m512h __A, __m512h __B) {`.
  **L386 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_add_ph(__mmask32 __U, __m512h __A, __m512h __B) {`。
- **L387 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,`.
  **L387 CN**: 以 `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,` 从当前函数返回。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)_mm512_add_ph(__A, __B),`.
  **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)_mm512_add_ph(__A, __B),`。
- **L389 EN**: Executes a call or declaration centered on `statement`.
  **L389 CN**: 执行以 `statement` 为核心的调用或声明。
- **L390 EN**: Closes the current lexical scope or compound statement.
  **L390 CN**: 结束当前词法作用域或复合语句块。
- **L391 EN**: Blank line separating nearby declarations or logic blocks.
  **L391 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L392 EN**: Defines macro `_mm512_add_round_ph(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L392 CN**: 定义宏 `_mm512_add_round_ph(A, B, R)`，用于条件编译、简写或 API 生成。
- **L393 EN**: Continues logic associated with callable symbol `__builtin_ia32_addph512`.
  **L393 CN**: 继续与可调用符号 `__builtin_ia32_addph512` 相关的逻辑。
- **L394 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(B), (int)(R)))`.
  **L394 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(B), (int)(R)))`。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Defines macro `_mm512_mask_add_round_ph(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L396 CN**: 定义宏 `_mm512_mask_add_round_ph(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L397 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L397 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L398 EN**: Continues logic associated with callable symbol `_mm512_add_round_ph`.
  **L398 CN**: 继续与可调用符号 `_mm512_add_round_ph` 相关的逻辑。
- **L399 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(W)))`.
  **L399 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(W)))`。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L401 EN**: Defines macro `_mm512_maskz_add_round_ph(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L401 CN**: 定义宏 `_mm512_maskz_add_round_ph(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L402 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L402 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L403 EN**: Continues logic associated with callable symbol `_mm512_add_round_ph`.
  **L403 CN**: 继续与可调用符号 `_mm512_add_round_ph` 相关的逻辑。
- **L404 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L404 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_sub_ph(__m512h __A,`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_sub_ph(__m512h __A,`。
- **L407 EN**: Continues the surrounding expression or declaration: `__m512h __B) {`.
  **L407 CN**: 继续构造周围的表达式或声明：`__m512h __B) {`。
- **L408 EN**: Returns from the current function with `(__m512h)((__v32hf)__A - (__v32hf)__B)`.
  **L408 CN**: 以 `(__m512h)((__v32hf)__A - (__v32hf)__B)` 从当前函数返回。

### Lines 409-432

````c
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_sub_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512(
      (__mmask32)__U, (__v32hf)_mm512_sub_ph(__A, __B), (__v32hf)__W);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_sub_ph(__mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512((__mmask32)__U,
                                              (__v32hf)_mm512_sub_ph(__A, __B),
                                              (__v32hf)_mm512_setzero_ph());
}

#define _mm512_sub_round_ph(A, B, R)                                           \
  ((__m512h)__builtin_ia32_subph512((__v32hf)(__m512h)(A),                     \
                                    (__v32hf)(__m512h)(B), (int)(R)))

#define _mm512_mask_sub_round_ph(W, U, A, B, R)                                \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_sub_round_ph((A), (B), (R)),             \
      (__v32hf)(__m512h)(W)))

````
- **L409 EN**: Closes the current lexical scope or compound statement.
  **L409 CN**: 结束当前词法作用域或复合语句块。
- **L410 EN**: Blank line separating nearby declarations or logic blocks.
  **L410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L411 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L411 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L412 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_sub_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`.
  **L412 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_sub_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`。
- **L413 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512(`.
  **L413 CN**: 以 `(__m512h)__builtin_ia32_selectph_512(` 从当前函数返回。
- **L414 EN**: Executes a call or declaration centered on `statement`.
  **L414 CN**: 执行以 `statement` 为核心的调用或声明。
- **L415 EN**: Closes the current lexical scope or compound statement.
  **L415 CN**: 结束当前词法作用域或复合语句块。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L417 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L417 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L418 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_sub_ph(__mmask32 __U, __m512h __A, __m512h __B) {`.
  **L418 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_sub_ph(__mmask32 __U, __m512h __A, __m512h __B) {`。
- **L419 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,`.
  **L419 CN**: 以 `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,` 从当前函数返回。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)_mm512_sub_ph(__A, __B),`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)_mm512_sub_ph(__A, __B),`。
- **L421 EN**: Executes a call or declaration centered on `statement`.
  **L421 CN**: 执行以 `statement` 为核心的调用或声明。
- **L422 EN**: Closes the current lexical scope or compound statement.
  **L422 CN**: 结束当前词法作用域或复合语句块。
- **L423 EN**: Blank line separating nearby declarations or logic blocks.
  **L423 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L424 EN**: Defines macro `_mm512_sub_round_ph(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L424 CN**: 定义宏 `_mm512_sub_round_ph(A, B, R)`，用于条件编译、简写或 API 生成。
- **L425 EN**: Continues logic associated with callable symbol `__builtin_ia32_subph512`.
  **L425 CN**: 继续与可调用符号 `__builtin_ia32_subph512` 相关的逻辑。
- **L426 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(B), (int)(R)))`.
  **L426 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(B), (int)(R)))`。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L428 EN**: Defines macro `_mm512_mask_sub_round_ph(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L428 CN**: 定义宏 `_mm512_mask_sub_round_ph(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L429 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L429 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L430 EN**: Continues logic associated with callable symbol `_mm512_sub_round_ph`.
  **L430 CN**: 继续与可调用符号 `_mm512_sub_round_ph` 相关的逻辑。
- **L431 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(W)))`.
  **L431 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(W)))`。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 433-456

````c
#define _mm512_maskz_sub_round_ph(U, A, B, R)                                  \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_sub_round_ph((A), (B), (R)),             \
      (__v32hf)_mm512_setzero_ph()))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_mul_ph(__m512h __A,
                                                              __m512h __B) {
  return (__m512h)((__v32hf)__A * (__v32hf)__B);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_mul_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512(
      (__mmask32)__U, (__v32hf)_mm512_mul_ph(__A, __B), (__v32hf)__W);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_mul_ph(__mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512((__mmask32)__U,
                                              (__v32hf)_mm512_mul_ph(__A, __B),
                                              (__v32hf)_mm512_setzero_ph());
}

#define _mm512_mul_round_ph(A, B, R)                                           \
````
- **L433 EN**: Defines macro `_mm512_maskz_sub_round_ph(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L433 CN**: 定义宏 `_mm512_maskz_sub_round_ph(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L434 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L434 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L435 EN**: Continues logic associated with callable symbol `_mm512_sub_round_ph`.
  **L435 CN**: 继续与可调用符号 `_mm512_sub_round_ph` 相关的逻辑。
- **L436 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L436 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_mul_ph(__m512h __A,`.
  **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_mul_ph(__m512h __A,`。
- **L439 EN**: Continues the surrounding expression or declaration: `__m512h __B) {`.
  **L439 CN**: 继续构造周围的表达式或声明：`__m512h __B) {`。
- **L440 EN**: Returns from the current function with `(__m512h)((__v32hf)__A * (__v32hf)__B)`.
  **L440 CN**: 以 `(__m512h)((__v32hf)__A * (__v32hf)__B)` 从当前函数返回。
- **L441 EN**: Closes the current lexical scope or compound statement.
  **L441 CN**: 结束当前词法作用域或复合语句块。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L443 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L443 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L444 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_mul_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`.
  **L444 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_mul_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`。
- **L445 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512(`.
  **L445 CN**: 以 `(__m512h)__builtin_ia32_selectph_512(` 从当前函数返回。
- **L446 EN**: Executes a call or declaration centered on `statement`.
  **L446 CN**: 执行以 `statement` 为核心的调用或声明。
- **L447 EN**: Closes the current lexical scope or compound statement.
  **L447 CN**: 结束当前词法作用域或复合语句块。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L449 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L449 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L450 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_mul_ph(__mmask32 __U, __m512h __A, __m512h __B) {`.
  **L450 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_mul_ph(__mmask32 __U, __m512h __A, __m512h __B) {`。
- **L451 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,`.
  **L451 CN**: 以 `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,` 从当前函数返回。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)_mm512_mul_ph(__A, __B),`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)_mm512_mul_ph(__A, __B),`。
- **L453 EN**: Executes a call or declaration centered on `statement`.
  **L453 CN**: 执行以 `statement` 为核心的调用或声明。
- **L454 EN**: Closes the current lexical scope or compound statement.
  **L454 CN**: 结束当前词法作用域或复合语句块。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L456 EN**: Defines macro `_mm512_mul_round_ph(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L456 CN**: 定义宏 `_mm512_mul_round_ph(A, B, R)`，用于条件编译、简写或 API 生成。

### Lines 457-480

````c
  ((__m512h)__builtin_ia32_mulph512((__v32hf)(__m512h)(A),                     \
                                    (__v32hf)(__m512h)(B), (int)(R)))

#define _mm512_mask_mul_round_ph(W, U, A, B, R)                                \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_mul_round_ph((A), (B), (R)),             \
      (__v32hf)(__m512h)(W)))

#define _mm512_maskz_mul_round_ph(U, A, B, R)                                  \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_mul_round_ph((A), (B), (R)),             \
      (__v32hf)_mm512_setzero_ph()))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_div_ph(__m512h __A,
                                                              __m512h __B) {
  return (__m512h)((__v32hf)__A / (__v32hf)__B);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_div_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512(
      (__mmask32)__U, (__v32hf)_mm512_div_ph(__A, __B), (__v32hf)__W);
}

````
- **L457 EN**: Continues logic associated with callable symbol `__builtin_ia32_mulph512`.
  **L457 CN**: 继续与可调用符号 `__builtin_ia32_mulph512` 相关的逻辑。
- **L458 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(B), (int)(R)))`.
  **L458 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(B), (int)(R)))`。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L460 EN**: Defines macro `_mm512_mask_mul_round_ph(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L460 CN**: 定义宏 `_mm512_mask_mul_round_ph(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L461 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L461 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L462 EN**: Continues logic associated with callable symbol `_mm512_mul_round_ph`.
  **L462 CN**: 继续与可调用符号 `_mm512_mul_round_ph` 相关的逻辑。
- **L463 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(W)))`.
  **L463 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(W)))`。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L465 EN**: Defines macro `_mm512_maskz_mul_round_ph(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L465 CN**: 定义宏 `_mm512_maskz_mul_round_ph(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L466 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L466 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L467 EN**: Continues logic associated with callable symbol `_mm512_mul_round_ph`.
  **L467 CN**: 继续与可调用符号 `_mm512_mul_round_ph` 相关的逻辑。
- **L468 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L468 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L469 EN**: Blank line separating nearby declarations or logic blocks.
  **L469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_div_ph(__m512h __A,`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_div_ph(__m512h __A,`。
- **L471 EN**: Continues the surrounding expression or declaration: `__m512h __B) {`.
  **L471 CN**: 继续构造周围的表达式或声明：`__m512h __B) {`。
- **L472 EN**: Returns from the current function with `(__m512h)((__v32hf)__A / (__v32hf)__B)`.
  **L472 CN**: 以 `(__m512h)((__v32hf)__A / (__v32hf)__B)` 从当前函数返回。
- **L473 EN**: Closes the current lexical scope or compound statement.
  **L473 CN**: 结束当前词法作用域或复合语句块。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L475 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L476 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_div_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`.
  **L476 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_div_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`。
- **L477 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512(`.
  **L477 CN**: 以 `(__m512h)__builtin_ia32_selectph_512(` 从当前函数返回。
- **L478 EN**: Executes a call or declaration centered on `statement`.
  **L478 CN**: 执行以 `statement` 为核心的调用或声明。
- **L479 EN**: Closes the current lexical scope or compound statement.
  **L479 CN**: 结束当前词法作用域或复合语句块。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 481-504

````c
static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_div_ph(__mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512((__mmask32)__U,
                                              (__v32hf)_mm512_div_ph(__A, __B),
                                              (__v32hf)_mm512_setzero_ph());
}

#define _mm512_div_round_ph(A, B, R)                                           \
  ((__m512h)__builtin_ia32_divph512((__v32hf)(__m512h)(A),                     \
                                    (__v32hf)(__m512h)(B), (int)(R)))

#define _mm512_mask_div_round_ph(W, U, A, B, R)                                \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_div_round_ph((A), (B), (R)),             \
      (__v32hf)(__m512h)(W)))

#define _mm512_maskz_div_round_ph(U, A, B, R)                                  \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_div_round_ph((A), (B), (R)),             \
      (__v32hf)_mm512_setzero_ph()))

static __inline__ __m512h
    __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_min_ph(__m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_minph512((__v32hf)__A, (__v32hf)__B,
````
- **L481 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L481 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L482 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_div_ph(__mmask32 __U, __m512h __A, __m512h __B) {`.
  **L482 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_div_ph(__mmask32 __U, __m512h __A, __m512h __B) {`。
- **L483 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,`.
  **L483 CN**: 以 `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,` 从当前函数返回。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)_mm512_div_ph(__A, __B),`.
  **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)_mm512_div_ph(__A, __B),`。
- **L485 EN**: Executes a call or declaration centered on `statement`.
  **L485 CN**: 执行以 `statement` 为核心的调用或声明。
- **L486 EN**: Closes the current lexical scope or compound statement.
  **L486 CN**: 结束当前词法作用域或复合语句块。
- **L487 EN**: Blank line separating nearby declarations or logic blocks.
  **L487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L488 EN**: Defines macro `_mm512_div_round_ph(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L488 CN**: 定义宏 `_mm512_div_round_ph(A, B, R)`，用于条件编译、简写或 API 生成。
- **L489 EN**: Continues logic associated with callable symbol `__builtin_ia32_divph512`.
  **L489 CN**: 继续与可调用符号 `__builtin_ia32_divph512` 相关的逻辑。
- **L490 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(B), (int)(R)))`.
  **L490 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(B), (int)(R)))`。
- **L491 EN**: Blank line separating nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L492 EN**: Defines macro `_mm512_mask_div_round_ph(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L492 CN**: 定义宏 `_mm512_mask_div_round_ph(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L493 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L493 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L494 EN**: Continues logic associated with callable symbol `_mm512_div_round_ph`.
  **L494 CN**: 继续与可调用符号 `_mm512_div_round_ph` 相关的逻辑。
- **L495 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(W)))`.
  **L495 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(W)))`。
- **L496 EN**: Blank line separating nearby declarations or logic blocks.
  **L496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L497 EN**: Defines macro `_mm512_maskz_div_round_ph(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L497 CN**: 定义宏 `_mm512_maskz_div_round_ph(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L498 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L498 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L499 EN**: Continues logic associated with callable symbol `_mm512_div_round_ph`.
  **L499 CN**: 继续与可调用符号 `_mm512_div_round_ph` 相关的逻辑。
- **L500 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L500 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L501 EN**: Blank line separating nearby declarations or logic blocks.
  **L501 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L502 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h`.
  **L502 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h`。
- **L503 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_min_ph(__m512h __A, __m512h __B) {`.
  **L503 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_min_ph(__m512h __A, __m512h __B) {`。
- **L504 EN**: Returns from the current function with `(__m512h)__builtin_ia32_minph512((__v32hf)__A, (__v32hf)__B,`.
  **L504 CN**: 以 `(__m512h)__builtin_ia32_minph512((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。

### Lines 505-528

````c
                                          _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_min_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512(
      (__mmask32)__U, (__v32hf)_mm512_min_ph(__A, __B), (__v32hf)__W);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_min_ph(__mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512((__mmask32)__U,
                                              (__v32hf)_mm512_min_ph(__A, __B),
                                              (__v32hf)_mm512_setzero_ph());
}

#define _mm512_min_round_ph(A, B, R)                                           \
  ((__m512h)__builtin_ia32_minph512((__v32hf)(__m512h)(A),                     \
                                    (__v32hf)(__m512h)(B), (int)(R)))

#define _mm512_mask_min_round_ph(W, U, A, B, R)                                \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_min_round_ph((A), (B), (R)),             \
      (__v32hf)(__m512h)(W)))
````
- **L505 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L505 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L506 EN**: Closes the current lexical scope or compound statement.
  **L506 CN**: 结束当前词法作用域或复合语句块。
- **L507 EN**: Blank line separating nearby declarations or logic blocks.
  **L507 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L508 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L508 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L509 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_min_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`.
  **L509 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_min_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`。
- **L510 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512(`.
  **L510 CN**: 以 `(__m512h)__builtin_ia32_selectph_512(` 从当前函数返回。
- **L511 EN**: Executes a call or declaration centered on `statement`.
  **L511 CN**: 执行以 `statement` 为核心的调用或声明。
- **L512 EN**: Closes the current lexical scope or compound statement.
  **L512 CN**: 结束当前词法作用域或复合语句块。
- **L513 EN**: Blank line separating nearby declarations or logic blocks.
  **L513 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L514 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L514 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L515 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_min_ph(__mmask32 __U, __m512h __A, __m512h __B) {`.
  **L515 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_min_ph(__mmask32 __U, __m512h __A, __m512h __B) {`。
- **L516 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,`.
  **L516 CN**: 以 `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,` 从当前函数返回。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)_mm512_min_ph(__A, __B),`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)_mm512_min_ph(__A, __B),`。
- **L518 EN**: Executes a call or declaration centered on `statement`.
  **L518 CN**: 执行以 `statement` 为核心的调用或声明。
- **L519 EN**: Closes the current lexical scope or compound statement.
  **L519 CN**: 结束当前词法作用域或复合语句块。
- **L520 EN**: Blank line separating nearby declarations or logic blocks.
  **L520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L521 EN**: Defines macro `_mm512_min_round_ph(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L521 CN**: 定义宏 `_mm512_min_round_ph(A, B, R)`，用于条件编译、简写或 API 生成。
- **L522 EN**: Continues logic associated with callable symbol `__builtin_ia32_minph512`.
  **L522 CN**: 继续与可调用符号 `__builtin_ia32_minph512` 相关的逻辑。
- **L523 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(B), (int)(R)))`.
  **L523 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(B), (int)(R)))`。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L525 EN**: Defines macro `_mm512_mask_min_round_ph(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L525 CN**: 定义宏 `_mm512_mask_min_round_ph(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L526 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L526 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L527 EN**: Continues logic associated with callable symbol `_mm512_min_round_ph`.
  **L527 CN**: 继续与可调用符号 `_mm512_min_round_ph` 相关的逻辑。
- **L528 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(W)))`.
  **L528 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(W)))`。

### Lines 529-552

````c

#define _mm512_maskz_min_round_ph(U, A, B, R)                                  \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_min_round_ph((A), (B), (R)),             \
      (__v32hf)_mm512_setzero_ph()))

static __inline__ __m512h
    __DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_max_ph(__m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_maxph512((__v32hf)__A, (__v32hf)__B,
                                          _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_mask_max_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512(
      (__mmask32)__U, (__v32hf)_mm512_max_ph(__A, __B), (__v32hf)__W);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_maskz_max_ph(__mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_selectph_512((__mmask32)__U,
                                              (__v32hf)_mm512_max_ph(__A, __B),
                                              (__v32hf)_mm512_setzero_ph());
}
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Defines macro `_mm512_maskz_min_round_ph(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L530 CN**: 定义宏 `_mm512_maskz_min_round_ph(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L531 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L531 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L532 EN**: Continues logic associated with callable symbol `_mm512_min_round_ph`.
  **L532 CN**: 继续与可调用符号 `_mm512_min_round_ph` 相关的逻辑。
- **L533 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L533 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L534 EN**: Blank line separating nearby declarations or logic blocks.
  **L534 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L535 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h`.
  **L535 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h`。
- **L536 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_max_ph(__m512h __A, __m512h __B) {`.
  **L536 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS512_CONSTEXPR _mm512_max_ph(__m512h __A, __m512h __B) {`。
- **L537 EN**: Returns from the current function with `(__m512h)__builtin_ia32_maxph512((__v32hf)__A, (__v32hf)__B,`.
  **L537 CN**: 以 `(__m512h)__builtin_ia32_maxph512((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L538 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L538 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L539 EN**: Closes the current lexical scope or compound statement.
  **L539 CN**: 结束当前词法作用域或复合语句块。
- **L540 EN**: Blank line separating nearby declarations or logic blocks.
  **L540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L541 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L541 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L542 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_max_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`.
  **L542 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_max_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`。
- **L543 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512(`.
  **L543 CN**: 以 `(__m512h)__builtin_ia32_selectph_512(` 从当前函数返回。
- **L544 EN**: Executes a call or declaration centered on `statement`.
  **L544 CN**: 执行以 `statement` 为核心的调用或声明。
- **L545 EN**: Closes the current lexical scope or compound statement.
  **L545 CN**: 结束当前词法作用域或复合语句块。
- **L546 EN**: Blank line separating nearby declarations or logic blocks.
  **L546 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L547 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L547 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L548 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_max_ph(__mmask32 __U, __m512h __A, __m512h __B) {`.
  **L548 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_max_ph(__mmask32 __U, __m512h __A, __m512h __B) {`。
- **L549 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,`.
  **L549 CN**: 以 `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U,` 从当前函数返回。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)_mm512_max_ph(__A, __B),`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)_mm512_max_ph(__A, __B),`。
- **L551 EN**: Executes a call or declaration centered on `statement`.
  **L551 CN**: 执行以 `statement` 为核心的调用或声明。
- **L552 EN**: Closes the current lexical scope or compound statement.
  **L552 CN**: 结束当前词法作用域或复合语句块。

### Lines 553-576

````c

#define _mm512_max_round_ph(A, B, R)                                           \
  ((__m512h)__builtin_ia32_maxph512((__v32hf)(__m512h)(A),                     \
                                    (__v32hf)(__m512h)(B), (int)(R)))

#define _mm512_mask_max_round_ph(W, U, A, B, R)                                \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_max_round_ph((A), (B), (R)),             \
      (__v32hf)(__m512h)(W)))

#define _mm512_maskz_max_round_ph(U, A, B, R)                                  \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_max_round_ph((A), (B), (R)),             \
      (__v32hf)_mm512_setzero_ph()))

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_abs_ph(__m512h __A) {
  return (__m512h)_mm512_and_epi32(_mm512_set1_epi32(0x7FFF7FFF), (__m512i)__A);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_conj_pch(__m512h __A) {
  return (__m512h)_mm512_xor_epi32((__m512i)__A,
                                   _mm512_set1_epi32(-2147483648));
}
````
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L554 EN**: Defines macro `_mm512_max_round_ph(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L554 CN**: 定义宏 `_mm512_max_round_ph(A, B, R)`，用于条件编译、简写或 API 生成。
- **L555 EN**: Continues logic associated with callable symbol `__builtin_ia32_maxph512`.
  **L555 CN**: 继续与可调用符号 `__builtin_ia32_maxph512` 相关的逻辑。
- **L556 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(B), (int)(R)))`.
  **L556 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(B), (int)(R)))`。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L558 EN**: Defines macro `_mm512_mask_max_round_ph(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L558 CN**: 定义宏 `_mm512_mask_max_round_ph(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L559 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L559 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L560 EN**: Continues logic associated with callable symbol `_mm512_max_round_ph`.
  **L560 CN**: 继续与可调用符号 `_mm512_max_round_ph` 相关的逻辑。
- **L561 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(W)))`.
  **L561 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(W)))`。
- **L562 EN**: Blank line separating nearby declarations or logic blocks.
  **L562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L563 EN**: Defines macro `_mm512_maskz_max_round_ph(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L563 CN**: 定义宏 `_mm512_maskz_max_round_ph(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L564 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L564 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L565 EN**: Continues logic associated with callable symbol `_mm512_max_round_ph`.
  **L565 CN**: 继续与可调用符号 `_mm512_max_round_ph` 相关的逻辑。
- **L566 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L566 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L567 EN**: Blank line separating nearby declarations or logic blocks.
  **L567 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L568 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L568 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L569 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_abs_ph(__m512h __A) {`.
  **L569 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_abs_ph(__m512h __A) {`。
- **L570 EN**: Returns from the current function with `(__m512h)_mm512_and_epi32(_mm512_set1_epi32(0x7FFF7FFF), (__m512i)__A)`.
  **L570 CN**: 以 `(__m512h)_mm512_and_epi32(_mm512_set1_epi32(0x7FFF7FFF), (__m512i)__A)` 从当前函数返回。
- **L571 EN**: Closes the current lexical scope or compound statement.
  **L571 CN**: 结束当前词法作用域或复合语句块。
- **L572 EN**: Blank line separating nearby declarations or logic blocks.
  **L572 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L573 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_conj_pch(__m512h __A) {`.
  **L573 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_conj_pch(__m512h __A) {`。
- **L574 EN**: Returns from the current function with `(__m512h)_mm512_xor_epi32((__m512i)__A,`.
  **L574 CN**: 以 `(__m512h)_mm512_xor_epi32((__m512i)__A,` 从当前函数返回。
- **L575 EN**: Executes a call or declaration centered on `_mm512_set1_epi32`.
  **L575 CN**: 执行以 `_mm512_set1_epi32` 为核心的调用或声明。
- **L576 EN**: Closes the current lexical scope or compound statement.
  **L576 CN**: 结束当前词法作用域或复合语句块。

### Lines 577-600

````c

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_conj_pch(__m512h __W, __mmask16 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_selectps_512(
      (__mmask16)__U, (__v16sf)_mm512_conj_pch(__A), (__v16sf)__W);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_conj_pch(__mmask16 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_selectps_512((__mmask16)__U,
                                              (__v16sf)_mm512_conj_pch(__A),
                                              (__v16sf)_mm512_setzero_ps());
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_add_sh(__m128h __A, __m128h __B) {
  __A[0] += __B[0];
  return __A;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_add_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  __A = _mm_add_sh(__A, __B);
  return __builtin_ia32_selectsh_128(__U, __A, __W);
````
- **L577 EN**: Blank line separating nearby declarations or logic blocks.
  **L577 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L578 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L578 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L579 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_conj_pch(__m512h __W, __mmask16 __U, __m512h __A) {`.
  **L579 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_conj_pch(__m512h __W, __mmask16 __U, __m512h __A) {`。
- **L580 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectps_512(`.
  **L580 CN**: 以 `(__m512h)__builtin_ia32_selectps_512(` 从当前函数返回。
- **L581 EN**: Executes a call or declaration centered on `statement`.
  **L581 CN**: 执行以 `statement` 为核心的调用或声明。
- **L582 EN**: Closes the current lexical scope or compound statement.
  **L582 CN**: 结束当前词法作用域或复合语句块。
- **L583 EN**: Blank line separating nearby declarations or logic blocks.
  **L583 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L584 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L584 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L585 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_conj_pch(__mmask16 __U, __m512h __A) {`.
  **L585 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_conj_pch(__mmask16 __U, __m512h __A) {`。
- **L586 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectps_512((__mmask16)__U,`.
  **L586 CN**: 以 `(__m512h)__builtin_ia32_selectps_512((__mmask16)__U,` 从当前函数返回。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)_mm512_conj_pch(__A),`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)_mm512_conj_pch(__A),`。
- **L588 EN**: Executes a call or declaration centered on `statement`.
  **L588 CN**: 执行以 `statement` 为核心的调用或声明。
- **L589 EN**: Closes the current lexical scope or compound statement.
  **L589 CN**: 结束当前词法作用域或复合语句块。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L591 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L591 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L592 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_add_sh(__m128h __A, __m128h __B) {`.
  **L592 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_add_sh(__m128h __A, __m128h __B) {`。
- **L593 EN**: Adds a standalone statement or declaration: `__A[0] += __B[0];`.
  **L593 CN**: 添加一条独立语句或声明：`__A[0] += __B[0];`。
- **L594 EN**: Returns from the current function with `__A`.
  **L594 CN**: 以 `__A` 从当前函数返回。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L597 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L597 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L598 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_add_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L598 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_add_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L599 EN**: Executes a call or declaration centered on `_mm_add_sh`.
  **L599 CN**: 执行以 `_mm_add_sh` 为核心的调用或声明。
- **L600 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, __A, __W)`.
  **L600 CN**: 以 `__builtin_ia32_selectsh_128(__U, __A, __W)` 从当前函数返回。

### Lines 601-624

````c
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_add_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  __A = _mm_add_sh(__A, __B);
  return __builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph());
}

#define _mm_add_round_sh(A, B, R)                                              \
  ((__m128h)__builtin_ia32_addsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_add_round_sh(W, U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_addsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_add_round_sh(U, A, B, R)                                     \
  ((__m128h)__builtin_ia32_addsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h
````
- **L601 EN**: Closes the current lexical scope or compound statement.
  **L601 CN**: 结束当前词法作用域或复合语句块。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L603 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L603 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L604 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_add_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L604 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_add_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L605 EN**: Executes a call or declaration centered on `_mm_add_sh`.
  **L605 CN**: 执行以 `_mm_add_sh` 为核心的调用或声明。
- **L606 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph())`.
  **L606 CN**: 以 `__builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph())` 从当前函数返回。
- **L607 EN**: Closes the current lexical scope or compound statement.
  **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Defines macro `_mm_add_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L609 CN**: 定义宏 `_mm_add_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L610 EN**: Continues logic associated with callable symbol `__builtin_ia32_addsh_round_mask`.
  **L610 CN**: 继续与可调用符号 `__builtin_ia32_addsh_round_mask` 相关的逻辑。
- **L611 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L611 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L612 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L612 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L613 EN**: Blank line separating nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L614 EN**: Defines macro `_mm_mask_add_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L614 CN**: 定义宏 `_mm_mask_add_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L615 EN**: Continues logic associated with callable symbol `__builtin_ia32_addsh_round_mask`.
  **L615 CN**: 继续与可调用符号 `__builtin_ia32_addsh_round_mask` 相关的逻辑。
- **L616 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L616 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L617 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L617 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L618 EN**: Blank line separating nearby declarations or logic blocks.
  **L618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L619 EN**: Defines macro `_mm_maskz_add_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L619 CN**: 定义宏 `_mm_maskz_add_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L620 EN**: Continues logic associated with callable symbol `__builtin_ia32_addsh_round_mask`.
  **L620 CN**: 继续与可调用符号 `__builtin_ia32_addsh_round_mask` 相关的逻辑。
- **L621 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L621 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L622 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L622 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L623 EN**: Blank line separating nearby declarations or logic blocks.
  **L623 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L624 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h`.
  **L624 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h`。

### Lines 625-648

````c
    __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_sub_sh(__m128h __A, __m128h __B) {
  __A[0] -= __B[0];
  return __A;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_sub_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  __A = _mm_sub_sh(__A, __B);
  return __builtin_ia32_selectsh_128(__U, __A, __W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_sub_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  __A = _mm_sub_sh(__A, __B);
  return __builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph());
}

#define _mm_sub_round_sh(A, B, R)                                              \
  ((__m128h)__builtin_ia32_subsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_sub_round_sh(W, U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_subsh_round_mask(                                   \
````
- **L625 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_sub_sh(__m128h __A, __m128h __B) {`.
  **L625 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_sub_sh(__m128h __A, __m128h __B) {`。
- **L626 EN**: Adds a standalone statement or declaration: `__A[0] -= __B[0];`.
  **L626 CN**: 添加一条独立语句或声明：`__A[0] -= __B[0];`。
- **L627 EN**: Returns from the current function with `__A`.
  **L627 CN**: 以 `__A` 从当前函数返回。
- **L628 EN**: Closes the current lexical scope or compound statement.
  **L628 CN**: 结束当前词法作用域或复合语句块。
- **L629 EN**: Blank line separating nearby declarations or logic blocks.
  **L629 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L630 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L630 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L631 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_sub_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L631 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_sub_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L632 EN**: Executes a call or declaration centered on `_mm_sub_sh`.
  **L632 CN**: 执行以 `_mm_sub_sh` 为核心的调用或声明。
- **L633 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, __A, __W)`.
  **L633 CN**: 以 `__builtin_ia32_selectsh_128(__U, __A, __W)` 从当前函数返回。
- **L634 EN**: Closes the current lexical scope or compound statement.
  **L634 CN**: 结束当前词法作用域或复合语句块。
- **L635 EN**: Blank line separating nearby declarations or logic blocks.
  **L635 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L636 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L636 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L637 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_sub_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L637 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_sub_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L638 EN**: Executes a call or declaration centered on `_mm_sub_sh`.
  **L638 CN**: 执行以 `_mm_sub_sh` 为核心的调用或声明。
- **L639 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph())`.
  **L639 CN**: 以 `__builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph())` 从当前函数返回。
- **L640 EN**: Closes the current lexical scope or compound statement.
  **L640 CN**: 结束当前词法作用域或复合语句块。
- **L641 EN**: Blank line separating nearby declarations or logic blocks.
  **L641 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L642 EN**: Defines macro `_mm_sub_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L642 CN**: 定义宏 `_mm_sub_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L643 EN**: Continues logic associated with callable symbol `__builtin_ia32_subsh_round_mask`.
  **L643 CN**: 继续与可调用符号 `__builtin_ia32_subsh_round_mask` 相关的逻辑。
- **L644 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L644 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L645 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L645 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L646 EN**: Blank line separating nearby declarations or logic blocks.
  **L646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L647 EN**: Defines macro `_mm_mask_sub_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L647 CN**: 定义宏 `_mm_mask_sub_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L648 EN**: Continues logic associated with callable symbol `__builtin_ia32_subsh_round_mask`.
  **L648 CN**: 继续与可调用符号 `__builtin_ia32_subsh_round_mask` 相关的逻辑。

### Lines 649-672

````c
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_sub_round_sh(U, A, B, R)                                     \
  ((__m128h)__builtin_ia32_subsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h
    __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_mul_sh(__m128h __A, __m128h __B) {
  __A[0] *= __B[0];
  return __A;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_mul_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  __A = _mm_mul_sh(__A, __B);
  return __builtin_ia32_selectsh_128(__U, __A, __W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_mul_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  __A = _mm_mul_sh(__A, __B);
  return __builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph());
````
- **L649 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L649 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L650 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L650 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L651 EN**: Blank line separating nearby declarations or logic blocks.
  **L651 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L652 EN**: Defines macro `_mm_maskz_sub_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L652 CN**: 定义宏 `_mm_maskz_sub_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L653 EN**: Continues logic associated with callable symbol `__builtin_ia32_subsh_round_mask`.
  **L653 CN**: 继续与可调用符号 `__builtin_ia32_subsh_round_mask` 相关的逻辑。
- **L654 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L654 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L655 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L655 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h`.
  **L657 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h`。
- **L658 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_mul_sh(__m128h __A, __m128h __B) {`.
  **L658 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_mul_sh(__m128h __A, __m128h __B) {`。
- **L659 EN**: Adds a standalone statement or declaration: `__A[0] *= __B[0];`.
  **L659 CN**: 添加一条独立语句或声明：`__A[0] *= __B[0];`。
- **L660 EN**: Returns from the current function with `__A`.
  **L660 CN**: 以 `__A` 从当前函数返回。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Blank line separating nearby declarations or logic blocks.
  **L662 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L663 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L663 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L664 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mul_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L664 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mul_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L665 EN**: Executes a call or declaration centered on `_mm_mul_sh`.
  **L665 CN**: 执行以 `_mm_mul_sh` 为核心的调用或声明。
- **L666 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, __A, __W)`.
  **L666 CN**: 以 `__builtin_ia32_selectsh_128(__U, __A, __W)` 从当前函数返回。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L669 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L670 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mul_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L670 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mul_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L671 EN**: Executes a call or declaration centered on `_mm_mul_sh`.
  **L671 CN**: 执行以 `_mm_mul_sh` 为核心的调用或声明。
- **L672 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph())`.
  **L672 CN**: 以 `__builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph())` 从当前函数返回。

### Lines 673-696

````c
}

#define _mm_mul_round_sh(A, B, R)                                              \
  ((__m128h)__builtin_ia32_mulsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_mul_round_sh(W, U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_mulsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_mul_round_sh(U, A, B, R)                                     \
  ((__m128h)__builtin_ia32_mulsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h
    __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_div_sh(__m128h __A, __m128h __B) {
  __A[0] /= __B[0];
  return __A;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L675 EN**: Defines macro `_mm_mul_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L675 CN**: 定义宏 `_mm_mul_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L676 EN**: Continues logic associated with callable symbol `__builtin_ia32_mulsh_round_mask`.
  **L676 CN**: 继续与可调用符号 `__builtin_ia32_mulsh_round_mask` 相关的逻辑。
- **L677 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L677 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L678 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L678 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L680 EN**: Defines macro `_mm_mask_mul_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L680 CN**: 定义宏 `_mm_mask_mul_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L681 EN**: Continues logic associated with callable symbol `__builtin_ia32_mulsh_round_mask`.
  **L681 CN**: 继续与可调用符号 `__builtin_ia32_mulsh_round_mask` 相关的逻辑。
- **L682 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L682 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L683 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L683 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L684 EN**: Blank line separating nearby declarations or logic blocks.
  **L684 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L685 EN**: Defines macro `_mm_maskz_mul_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L685 CN**: 定义宏 `_mm_maskz_mul_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L686 EN**: Continues logic associated with callable symbol `__builtin_ia32_mulsh_round_mask`.
  **L686 CN**: 继续与可调用符号 `__builtin_ia32_mulsh_round_mask` 相关的逻辑。
- **L687 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L687 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L688 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L688 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L689 EN**: Blank line separating nearby declarations or logic blocks.
  **L689 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L690 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h`.
  **L690 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h`。
- **L691 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_div_sh(__m128h __A, __m128h __B) {`.
  **L691 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_div_sh(__m128h __A, __m128h __B) {`。
- **L692 EN**: Adds a standalone statement or declaration: `__A[0] /= __B[0];`.
  **L692 CN**: 添加一条独立语句或声明：`__A[0] /= __B[0];`。
- **L693 EN**: Returns from the current function with `__A`.
  **L693 CN**: 以 `__A` 从当前函数返回。
- **L694 EN**: Closes the current lexical scope or compound statement.
  **L694 CN**: 结束当前词法作用域或复合语句块。
- **L695 EN**: Blank line separating nearby declarations or logic blocks.
  **L695 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L696 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L696 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。

### Lines 697-720

````c
_mm_mask_div_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  __A = _mm_div_sh(__A, __B);
  return __builtin_ia32_selectsh_128(__U, __A, __W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_div_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  __A = _mm_div_sh(__A, __B);
  return __builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph());
}

#define _mm_div_round_sh(A, B, R)                                              \
  ((__m128h)__builtin_ia32_divsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_div_round_sh(W, U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_divsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_div_round_sh(U, A, B, R)                                     \
  ((__m128h)__builtin_ia32_divsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
````
- **L697 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_div_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L697 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_div_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L698 EN**: Executes a call or declaration centered on `_mm_div_sh`.
  **L698 CN**: 执行以 `_mm_div_sh` 为核心的调用或声明。
- **L699 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, __A, __W)`.
  **L699 CN**: 以 `__builtin_ia32_selectsh_128(__U, __A, __W)` 从当前函数返回。
- **L700 EN**: Closes the current lexical scope or compound statement.
  **L700 CN**: 结束当前词法作用域或复合语句块。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L702 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L703 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_div_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L703 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_div_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L704 EN**: Executes a call or declaration centered on `_mm_div_sh`.
  **L704 CN**: 执行以 `_mm_div_sh` 为核心的调用或声明。
- **L705 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph())`.
  **L705 CN**: 以 `__builtin_ia32_selectsh_128(__U, __A, _mm_setzero_ph())` 从当前函数返回。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L708 EN**: Defines macro `_mm_div_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L708 CN**: 定义宏 `_mm_div_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L709 EN**: Continues logic associated with callable symbol `__builtin_ia32_divsh_round_mask`.
  **L709 CN**: 继续与可调用符号 `__builtin_ia32_divsh_round_mask` 相关的逻辑。
- **L710 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L710 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L711 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L711 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L712 EN**: Blank line separating nearby declarations or logic blocks.
  **L712 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L713 EN**: Defines macro `_mm_mask_div_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L713 CN**: 定义宏 `_mm_mask_div_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L714 EN**: Continues logic associated with callable symbol `__builtin_ia32_divsh_round_mask`.
  **L714 CN**: 继续与可调用符号 `__builtin_ia32_divsh_round_mask` 相关的逻辑。
- **L715 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L715 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L716 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L716 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L717 EN**: Blank line separating nearby declarations or logic blocks.
  **L717 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L718 EN**: Defines macro `_mm_maskz_div_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L718 CN**: 定义宏 `_mm_maskz_div_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L719 EN**: Continues logic associated with callable symbol `__builtin_ia32_divsh_round_mask`.
  **L719 CN**: 继续与可调用符号 `__builtin_ia32_divsh_round_mask` 相关的逻辑。
- **L720 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L720 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。

### Lines 721-744

````c
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h
    __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_min_sh(__m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_minsh_round_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_min_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_minsh_round_mask((__v8hf)__A, (__v8hf)__B,
                                                  (__v8hf)__W, (__mmask8)__U,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_min_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_minsh_round_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_min_round_sh(A, B, R)                                              \
````
- **L721 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L721 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L722 EN**: Blank line separating nearby declarations or logic blocks.
  **L722 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L723 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h`.
  **L723 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h`。
- **L724 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_min_sh(__m128h __A, __m128h __B) {`.
  **L724 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_min_sh(__m128h __A, __m128h __B) {`。
- **L725 EN**: Returns from the current function with `(__m128h)__builtin_ia32_minsh_round_mask(`.
  **L725 CN**: 以 `(__m128h)__builtin_ia32_minsh_round_mask(` 从当前函数返回。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`。
- **L727 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L727 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L728 EN**: Closes the current lexical scope or compound statement.
  **L728 CN**: 结束当前词法作用域或复合语句块。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L730 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L730 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L731 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_min_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L731 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_min_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L732 EN**: Returns from the current function with `(__m128h)__builtin_ia32_minsh_round_mask((__v8hf)__A, (__v8hf)__B,`.
  **L732 CN**: 以 `(__m128h)__builtin_ia32_minsh_round_mask((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__W, (__mmask8)__U,`.
  **L733 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__W, (__mmask8)__U,`。
- **L734 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L734 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L735 EN**: Closes the current lexical scope or compound statement.
  **L735 CN**: 结束当前词法作用域或复合语句块。
- **L736 EN**: Blank line separating nearby declarations or logic blocks.
  **L736 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L737 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L737 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L738 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_min_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L738 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_min_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L739 EN**: Returns from the current function with `(__m128h)__builtin_ia32_minsh_round_mask(`.
  **L739 CN**: 以 `(__m128h)__builtin_ia32_minsh_round_mask(` 从当前函数返回。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L741 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L741 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L742 EN**: Closes the current lexical scope or compound statement.
  **L742 CN**: 结束当前词法作用域或复合语句块。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L744 EN**: Defines macro `_mm_min_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L744 CN**: 定义宏 `_mm_min_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。

### Lines 745-768

````c
  ((__m128h)__builtin_ia32_minsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_min_round_sh(W, U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_minsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_min_round_sh(U, A, B, R)                                     \
  ((__m128h)__builtin_ia32_minsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h
    __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_max_sh(__m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_maxsh_round_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_max_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_maxsh_round_mask((__v8hf)__A, (__v8hf)__B,
````
- **L745 EN**: Continues logic associated with callable symbol `__builtin_ia32_minsh_round_mask`.
  **L745 CN**: 继续与可调用符号 `__builtin_ia32_minsh_round_mask` 相关的逻辑。
- **L746 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L746 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L747 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L747 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L748 EN**: Blank line separating nearby declarations or logic blocks.
  **L748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L749 EN**: Defines macro `_mm_mask_min_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L749 CN**: 定义宏 `_mm_mask_min_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L750 EN**: Continues logic associated with callable symbol `__builtin_ia32_minsh_round_mask`.
  **L750 CN**: 继续与可调用符号 `__builtin_ia32_minsh_round_mask` 相关的逻辑。
- **L751 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L751 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L752 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L752 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L753 EN**: Blank line separating nearby declarations or logic blocks.
  **L753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L754 EN**: Defines macro `_mm_maskz_min_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L754 CN**: 定义宏 `_mm_maskz_min_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L755 EN**: Continues logic associated with callable symbol `__builtin_ia32_minsh_round_mask`.
  **L755 CN**: 继续与可调用符号 `__builtin_ia32_minsh_round_mask` 相关的逻辑。
- **L756 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L756 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L757 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L757 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L759 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h`.
  **L759 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h`。
- **L760 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_max_sh(__m128h __A, __m128h __B) {`.
  **L760 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_max_sh(__m128h __A, __m128h __B) {`。
- **L761 EN**: Returns from the current function with `(__m128h)__builtin_ia32_maxsh_round_mask(`.
  **L761 CN**: 以 `(__m128h)__builtin_ia32_maxsh_round_mask(` 从当前函数返回。
- **L762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`.
  **L762 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`。
- **L763 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L763 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L764 EN**: Closes the current lexical scope or compound statement.
  **L764 CN**: 结束当前词法作用域或复合语句块。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L766 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L766 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L767 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_max_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L767 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_max_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L768 EN**: Returns from the current function with `(__m128h)__builtin_ia32_maxsh_round_mask((__v8hf)__A, (__v8hf)__B,`.
  **L768 CN**: 以 `(__m128h)__builtin_ia32_maxsh_round_mask((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。

### Lines 769-792

````c
                                                  (__v8hf)__W, (__mmask8)__U,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_max_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_maxsh_round_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_max_round_sh(A, B, R)                                              \
  ((__m128h)__builtin_ia32_maxsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_max_round_sh(W, U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_maxsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_max_round_sh(U, A, B, R)                                     \
  ((__m128h)__builtin_ia32_maxsh_round_mask(                                   \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__W, (__mmask8)__U,`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__W, (__mmask8)__U,`。
- **L770 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L770 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L771 EN**: Closes the current lexical scope or compound statement.
  **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L773 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L773 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L774 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_max_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L774 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_max_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L775 EN**: Returns from the current function with `(__m128h)__builtin_ia32_maxsh_round_mask(`.
  **L775 CN**: 以 `(__m128h)__builtin_ia32_maxsh_round_mask(` 从当前函数返回。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L777 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L777 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L778 EN**: Closes the current lexical scope or compound statement.
  **L778 CN**: 结束当前词法作用域或复合语句块。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L780 EN**: Defines macro `_mm_max_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L780 CN**: 定义宏 `_mm_max_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L781 EN**: Continues logic associated with callable symbol `__builtin_ia32_maxsh_round_mask`.
  **L781 CN**: 继续与可调用符号 `__builtin_ia32_maxsh_round_mask` 相关的逻辑。
- **L782 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L782 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L783 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L783 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L785 EN**: Defines macro `_mm_mask_max_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L785 CN**: 定义宏 `_mm_mask_max_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L786 EN**: Continues logic associated with callable symbol `__builtin_ia32_maxsh_round_mask`.
  **L786 CN**: 继续与可调用符号 `__builtin_ia32_maxsh_round_mask` 相关的逻辑。
- **L787 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L787 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L788 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L788 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L789 EN**: Blank line separating nearby declarations or logic blocks.
  **L789 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L790 EN**: Defines macro `_mm_maskz_max_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L790 CN**: 定义宏 `_mm_maskz_max_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L791 EN**: Continues logic associated with callable symbol `__builtin_ia32_maxsh_round_mask`.
  **L791 CN**: 继续与可调用符号 `__builtin_ia32_maxsh_round_mask` 相关的逻辑。
- **L792 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L792 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。

### Lines 793-816

````c
      (__mmask8)(U), (int)(R)))

#define _mm512_cmp_round_ph_mask(A, B, P, R)                                   \
  ((__mmask32)__builtin_ia32_cmpph512_mask((__v32hf)(__m512h)(A),              \
                                           (__v32hf)(__m512h)(B), (int)(P),    \
                                           (__mmask32)-1, (int)(R)))

#define _mm512_mask_cmp_round_ph_mask(U, A, B, P, R)                           \
  ((__mmask32)__builtin_ia32_cmpph512_mask((__v32hf)(__m512h)(A),              \
                                           (__v32hf)(__m512h)(B), (int)(P),    \
                                           (__mmask32)(U), (int)(R)))

#define _mm512_cmp_ph_mask(A, B, P)                                            \
  _mm512_cmp_round_ph_mask((A), (B), (P), _MM_FROUND_CUR_DIRECTION)

#define _mm512_mask_cmp_ph_mask(U, A, B, P)                                    \
  _mm512_mask_cmp_round_ph_mask((U), (A), (B), (P), _MM_FROUND_CUR_DIRECTION)

#define _mm_cmp_round_sh_mask(X, Y, P, R)                                      \
  ((__mmask8)__builtin_ia32_cmpsh_mask((__v8hf)(__m128h)(X),                   \
                                       (__v8hf)(__m128h)(Y), (int)(P),         \
                                       (__mmask8)-1, (int)(R)))

#define _mm_mask_cmp_round_sh_mask(M, X, Y, P, R)                              \
````
- **L793 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L793 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L795 EN**: Defines macro `_mm512_cmp_round_ph_mask(A, B, P, R)` for conditional compilation, shorthand, or API generation.
  **L795 CN**: 定义宏 `_mm512_cmp_round_ph_mask(A, B, P, R)`，用于条件编译、简写或 API 生成。
- **L796 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpph512_mask`.
  **L796 CN**: 继续与可调用符号 `__builtin_ia32_cmpph512_mask` 相关的逻辑。
- **L797 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(B), (int)(P),    \`.
  **L797 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(B), (int)(P),    \`。
- **L798 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L798 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L800 EN**: Defines macro `_mm512_mask_cmp_round_ph_mask(U, A, B, P, R)` for conditional compilation, shorthand, or API generation.
  **L800 CN**: 定义宏 `_mm512_mask_cmp_round_ph_mask(U, A, B, P, R)`，用于条件编译、简写或 API 生成。
- **L801 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpph512_mask`.
  **L801 CN**: 继续与可调用符号 `__builtin_ia32_cmpph512_mask` 相关的逻辑。
- **L802 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(B), (int)(P),    \`.
  **L802 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(B), (int)(P),    \`。
- **L803 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L803 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L804 EN**: Blank line separating nearby declarations or logic blocks.
  **L804 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L805 EN**: Defines macro `_mm512_cmp_ph_mask(A, B, P)` for conditional compilation, shorthand, or API generation.
  **L805 CN**: 定义宏 `_mm512_cmp_ph_mask(A, B, P)`，用于条件编译、简写或 API 生成。
- **L806 EN**: Continues logic associated with callable symbol `_mm512_cmp_round_ph_mask`.
  **L806 CN**: 继续与可调用符号 `_mm512_cmp_round_ph_mask` 相关的逻辑。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L808 EN**: Defines macro `_mm512_mask_cmp_ph_mask(U, A, B, P)` for conditional compilation, shorthand, or API generation.
  **L808 CN**: 定义宏 `_mm512_mask_cmp_ph_mask(U, A, B, P)`，用于条件编译、简写或 API 生成。
- **L809 EN**: Continues logic associated with callable symbol `_mm512_mask_cmp_round_ph_mask`.
  **L809 CN**: 继续与可调用符号 `_mm512_mask_cmp_round_ph_mask` 相关的逻辑。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L811 EN**: Defines macro `_mm_cmp_round_sh_mask(X, Y, P, R)` for conditional compilation, shorthand, or API generation.
  **L811 CN**: 定义宏 `_mm_cmp_round_sh_mask(X, Y, P, R)`，用于条件编译、简写或 API 生成。
- **L812 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpsh_mask`.
  **L812 CN**: 继续与可调用符号 `__builtin_ia32_cmpsh_mask` 相关的逻辑。
- **L813 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(Y), (int)(P),         \`.
  **L813 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(Y), (int)(P),         \`。
- **L814 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L814 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L815 EN**: Blank line separating nearby declarations or logic blocks.
  **L815 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L816 EN**: Defines macro `_mm_mask_cmp_round_sh_mask(M, X, Y, P, R)` for conditional compilation, shorthand, or API generation.
  **L816 CN**: 定义宏 `_mm_mask_cmp_round_sh_mask(M, X, Y, P, R)`，用于条件编译、简写或 API 生成。

### Lines 817-840

````c
  ((__mmask8)__builtin_ia32_cmpsh_mask((__v8hf)(__m128h)(X),                   \
                                       (__v8hf)(__m128h)(Y), (int)(P),         \
                                       (__mmask8)(M), (int)(R)))

#define _mm_cmp_sh_mask(X, Y, P)                                               \
  ((__mmask8)__builtin_ia32_cmpsh_mask(                                        \
      (__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y), (int)(P), (__mmask8)-1,      \
      _MM_FROUND_CUR_DIRECTION))

#define _mm_mask_cmp_sh_mask(M, X, Y, P)                                       \
  ((__mmask8)__builtin_ia32_cmpsh_mask(                                        \
      (__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y), (int)(P), (__mmask8)(M),     \
      _MM_FROUND_CUR_DIRECTION))
// loads with vmovsh:
static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_load_sh(void const *__dp) {
  struct __mm_load_sh_struct {
    _Float16 __u;
  } __attribute__((__packed__, __may_alias__));
  _Float16 __u = ((const struct __mm_load_sh_struct *)__dp)->__u;
  return (__m128h){__u, 0, 0, 0, 0, 0, 0, 0};
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_load_sh(__m128h __W, __mmask8 __U, const void *__A) {
````
- **L817 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpsh_mask`.
  **L817 CN**: 继续与可调用符号 `__builtin_ia32_cmpsh_mask` 相关的逻辑。
- **L818 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(Y), (int)(P),         \`.
  **L818 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(Y), (int)(P),         \`。
- **L819 EN**: Continues the surrounding expression or declaration: `(__mmask8)(M), (int)(R)))`.
  **L819 CN**: 继续构造周围的表达式或声明：`(__mmask8)(M), (int)(R)))`。
- **L820 EN**: Blank line separating nearby declarations or logic blocks.
  **L820 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L821 EN**: Defines macro `_mm_cmp_sh_mask(X, Y, P)` for conditional compilation, shorthand, or API generation.
  **L821 CN**: 定义宏 `_mm_cmp_sh_mask(X, Y, P)`，用于条件编译、简写或 API 生成。
- **L822 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpsh_mask`.
  **L822 CN**: 继续与可调用符号 `__builtin_ia32_cmpsh_mask` 相关的逻辑。
- **L823 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y), (int)(P), (__mmask8)-1,      \`.
  **L823 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y), (int)(P), (__mmask8)-1,      \`。
- **L824 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L824 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Defines macro `_mm_mask_cmp_sh_mask(M, X, Y, P)` for conditional compilation, shorthand, or API generation.
  **L826 CN**: 定义宏 `_mm_mask_cmp_sh_mask(M, X, Y, P)`，用于条件编译、简写或 API 生成。
- **L827 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpsh_mask`.
  **L827 CN**: 继续与可调用符号 `__builtin_ia32_cmpsh_mask` 相关的逻辑。
- **L828 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y), (int)(P), (__mmask8)(M),     \`.
  **L828 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y), (int)(P), (__mmask8)(M),     \`。
- **L829 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L829 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L830 EN**: Comment explains nearby logic, constraints, or intent: `loads with vmovsh:`.
  **L830 CN**: 注释解释附近代码的逻辑、约束或设计意图：`loads with vmovsh:`。
- **L831 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_load_sh(void const *__dp) {`.
  **L831 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_load_sh(void const *__dp) {`。
- **L832 EN**: Declares struct `__mm_load_sh_struct`.
  **L832 CN**: 声明 struct `__mm_load_sh_struct`。
- **L833 EN**: Adds a standalone statement or declaration: `_Float16 __u;`.
  **L833 CN**: 添加一条独立语句或声明：`_Float16 __u;`。
- **L834 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L834 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L835 EN**: Initializes variable `__u` from the expression on the right-hand side.
  **L835 CN**: 使用右侧表达式初始化变量 `__u`。
- **L836 EN**: Returns from the current function with `(__m128h){__u, 0, 0, 0, 0, 0, 0, 0}`.
  **L836 CN**: 以 `(__m128h){__u, 0, 0, 0, 0, 0, 0, 0}` 从当前函数返回。
- **L837 EN**: Closes the current lexical scope or compound statement.
  **L837 CN**: 结束当前词法作用域或复合语句块。
- **L838 EN**: Blank line separating nearby declarations or logic blocks.
  **L838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L839 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L839 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L840 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_load_sh(__m128h __W, __mmask8 __U, const void *__A) {`.
  **L840 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_load_sh(__m128h __W, __mmask8 __U, const void *__A) {`。

### Lines 841-864

````c
  __m128h src = (__v8hf)__builtin_shufflevector(
      (__v8hf)__W, (__v8hf)_mm_setzero_ph(), 0, 8, 8, 8, 8, 8, 8, 8);

  return (__m128h)__builtin_ia32_loadsh128_mask((const __v8hf *)__A, src, __U & 1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_load_sh(__mmask8 __U, const void *__A) {
  return (__m128h)__builtin_ia32_loadsh128_mask(
      (const __v8hf *)__A, (__v8hf)_mm_setzero_ph(), __U & 1);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_load_ph(void const *__p) {
  return *(const __m512h *)__p;
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_load_ph(void const *__p) {
  return *(const __m256h *)__p;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_load_ph(void const *__p) {
  return *(const __m128h *)__p;
````
- **L841 EN**: Continues logic associated with callable symbol `__builtin_shufflevector`.
  **L841 CN**: 继续与可调用符号 `__builtin_shufflevector` 相关的逻辑。
- **L842 EN**: Executes a call or declaration centered on `statement`.
  **L842 CN**: 执行以 `statement` 为核心的调用或声明。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L844 EN**: Returns from the current function with `(__m128h)__builtin_ia32_loadsh128_mask((const __v8hf *)__A, src, __U & 1)`.
  **L844 CN**: 以 `(__m128h)__builtin_ia32_loadsh128_mask((const __v8hf *)__A, src, __U & 1)` 从当前函数返回。
- **L845 EN**: Closes the current lexical scope or compound statement.
  **L845 CN**: 结束当前词法作用域或复合语句块。
- **L846 EN**: Blank line separating nearby declarations or logic blocks.
  **L846 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L847 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L847 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L848 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_load_sh(__mmask8 __U, const void *__A) {`.
  **L848 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_load_sh(__mmask8 __U, const void *__A) {`。
- **L849 EN**: Returns from the current function with `(__m128h)__builtin_ia32_loadsh128_mask(`.
  **L849 CN**: 以 `(__m128h)__builtin_ia32_loadsh128_mask(` 从当前函数返回。
- **L850 EN**: Executes a call or declaration centered on `statement`.
  **L850 CN**: 执行以 `statement` 为核心的调用或声明。
- **L851 EN**: Closes the current lexical scope or compound statement.
  **L851 CN**: 结束当前词法作用域或复合语句块。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L853 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L854 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_load_ph(void const *__p) {`.
  **L854 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_load_ph(void const *__p) {`。
- **L855 EN**: Returns from the current function with `*(const __m512h *)__p`.
  **L855 CN**: 以 `*(const __m512h *)__p` 从当前函数返回。
- **L856 EN**: Closes the current lexical scope or compound statement.
  **L856 CN**: 结束当前词法作用域或复合语句块。
- **L857 EN**: Blank line separating nearby declarations or logic blocks.
  **L857 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L858 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L858 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L859 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_load_ph(void const *__p) {`.
  **L859 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_load_ph(void const *__p) {`。
- **L860 EN**: Returns from the current function with `*(const __m256h *)__p`.
  **L860 CN**: 以 `*(const __m256h *)__p` 从当前函数返回。
- **L861 EN**: Closes the current lexical scope or compound statement.
  **L861 CN**: 结束当前词法作用域或复合语句块。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L863 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_load_ph(void const *__p) {`.
  **L863 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_load_ph(void const *__p) {`。
- **L864 EN**: Returns from the current function with `*(const __m128h *)__p`.
  **L864 CN**: 以 `*(const __m128h *)__p` 从当前函数返回。

### Lines 865-888

````c
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_loadu_ph(void const *__p) {
  struct __loadu_ph {
    __m512h_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_ph *)__p)->__v;
}

static __inline__ __m256h __DEFAULT_FN_ATTRS256
_mm256_loadu_ph(void const *__p) {
  struct __loadu_ph {
    __m256h_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_ph *)__p)->__v;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_loadu_ph(void const *__p) {
  struct __loadu_ph {
    __m128h_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_ph *)__p)->__v;
}
````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L867 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L868 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_loadu_ph(void const *__p) {`.
  **L868 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_loadu_ph(void const *__p) {`。
- **L869 EN**: Declares struct `__loadu_ph`.
  **L869 CN**: 声明 struct `__loadu_ph`。
- **L870 EN**: Adds a standalone statement or declaration: `__m512h_u __v;`.
  **L870 CN**: 添加一条独立语句或声明：`__m512h_u __v;`。
- **L871 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L871 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L872 EN**: Returns from the current function with `((const struct __loadu_ph *)__p)->__v`.
  **L872 CN**: 以 `((const struct __loadu_ph *)__p)->__v` 从当前函数返回。
- **L873 EN**: Closes the current lexical scope or compound statement.
  **L873 CN**: 结束当前词法作用域或复合语句块。
- **L874 EN**: Blank line separating nearby declarations or logic blocks.
  **L874 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L875 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS256`.
  **L875 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS256`。
- **L876 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_loadu_ph(void const *__p) {`.
  **L876 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_loadu_ph(void const *__p) {`。
- **L877 EN**: Declares struct `__loadu_ph`.
  **L877 CN**: 声明 struct `__loadu_ph`。
- **L878 EN**: Adds a standalone statement or declaration: `__m256h_u __v;`.
  **L878 CN**: 添加一条独立语句或声明：`__m256h_u __v;`。
- **L879 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L879 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L880 EN**: Returns from the current function with `((const struct __loadu_ph *)__p)->__v`.
  **L880 CN**: 以 `((const struct __loadu_ph *)__p)->__v` 从当前函数返回。
- **L881 EN**: Closes the current lexical scope or compound statement.
  **L881 CN**: 结束当前词法作用域或复合语句块。
- **L882 EN**: Blank line separating nearby declarations or logic blocks.
  **L882 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L883 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_loadu_ph(void const *__p) {`.
  **L883 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_loadu_ph(void const *__p) {`。
- **L884 EN**: Declares struct `__loadu_ph`.
  **L884 CN**: 声明 struct `__loadu_ph`。
- **L885 EN**: Adds a standalone statement or declaration: `__m128h_u __v;`.
  **L885 CN**: 添加一条独立语句或声明：`__m128h_u __v;`。
- **L886 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L886 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L887 EN**: Returns from the current function with `((const struct __loadu_ph *)__p)->__v`.
  **L887 CN**: 以 `((const struct __loadu_ph *)__p)->__v` 从当前函数返回。
- **L888 EN**: Closes the current lexical scope or compound statement.
  **L888 CN**: 结束当前词法作用域或复合语句块。

### Lines 889-912

````c

// stores with vmovsh:
static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_sh(void *__dp,
                                                          __m128h __a) {
  struct __mm_store_sh_struct {
    _Float16 __u;
  } __attribute__((__packed__, __may_alias__));
  ((struct __mm_store_sh_struct *)__dp)->__u = __a[0];
}

static __inline__ void __DEFAULT_FN_ATTRS128 _mm_mask_store_sh(void *__W,
                                                               __mmask8 __U,
                                                               __m128h __A) {
  __builtin_ia32_storesh128_mask((__v8hf *)__W, __A, __U & 1);
}

static __inline__ void __DEFAULT_FN_ATTRS512 _mm512_store_ph(void *__P,
                                                             __m512h __A) {
  *(__m512h *)__P = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_store_ph(void *__P,
                                                             __m256h __A) {
  *(__m256h *)__P = __A;
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L890 EN**: Comment explains nearby logic, constraints, or intent: `stores with vmovsh:`.
  **L890 CN**: 注释解释附近代码的逻辑、约束或设计意图：`stores with vmovsh:`。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_sh(void *__dp,`.
  **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_sh(void *__dp,`。
- **L892 EN**: Continues the surrounding expression or declaration: `__m128h __a) {`.
  **L892 CN**: 继续构造周围的表达式或声明：`__m128h __a) {`。
- **L893 EN**: Declares struct `__mm_store_sh_struct`.
  **L893 CN**: 声明 struct `__mm_store_sh_struct`。
- **L894 EN**: Adds a standalone statement or declaration: `_Float16 __u;`.
  **L894 CN**: 添加一条独立语句或声明：`_Float16 __u;`。
- **L895 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L895 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L896 EN**: Executes a call or declaration centered on `statement`.
  **L896 CN**: 执行以 `statement` 为核心的调用或声明。
- **L897 EN**: Closes the current lexical scope or compound statement.
  **L897 CN**: 结束当前词法作用域或复合语句块。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS128 _mm_mask_store_sh(void *__W,`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS128 _mm_mask_store_sh(void *__W,`。
- **L900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L900 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L901 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L901 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L902 EN**: Executes a call or declaration centered on `__builtin_ia32_storesh128_mask`.
  **L902 CN**: 执行以 `__builtin_ia32_storesh128_mask` 为核心的调用或声明。
- **L903 EN**: Closes the current lexical scope or compound statement.
  **L903 CN**: 结束当前词法作用域或复合语句块。
- **L904 EN**: Blank line separating nearby declarations or logic blocks.
  **L904 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS512 _mm512_store_ph(void *__P,`.
  **L905 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS512 _mm512_store_ph(void *__P,`。
- **L906 EN**: Continues the surrounding expression or declaration: `__m512h __A) {`.
  **L906 CN**: 继续构造周围的表达式或声明：`__m512h __A) {`。
- **L907 EN**: Comment explains nearby logic, constraints, or intent: `(__m512h *)__P __A;`.
  **L907 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__m512h *)__P __A;`。
- **L908 EN**: Closes the current lexical scope or compound statement.
  **L908 CN**: 结束当前词法作用域或复合语句块。
- **L909 EN**: Blank line separating nearby declarations or logic blocks.
  **L909 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_store_ph(void *__P,`.
  **L910 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_store_ph(void *__P,`。
- **L911 EN**: Continues the surrounding expression or declaration: `__m256h __A) {`.
  **L911 CN**: 继续构造周围的表达式或声明：`__m256h __A) {`。
- **L912 EN**: Comment explains nearby logic, constraints, or intent: `(__m256h *)__P __A;`.
  **L912 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__m256h *)__P __A;`。

### Lines 913-936

````c
}

static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_ph(void *__P,
                                                          __m128h __A) {
  *(__m128h *)__P = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS512 _mm512_storeu_ph(void *__P,
                                                              __m512h __A) {
  struct __storeu_ph {
    __m512h_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_ph *)__P)->__v = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_storeu_ph(void *__P,
                                                              __m256h __A) {
  struct __storeu_ph {
    __m256h_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_ph *)__P)->__v = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS128 _mm_storeu_ph(void *__P,
````
- **L913 EN**: Closes the current lexical scope or compound statement.
  **L913 CN**: 结束当前词法作用域或复合语句块。
- **L914 EN**: Blank line separating nearby declarations or logic blocks.
  **L914 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_ph(void *__P,`.
  **L915 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_ph(void *__P,`。
- **L916 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L916 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L917 EN**: Comment explains nearby logic, constraints, or intent: `(__m128h *)__P __A;`.
  **L917 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__m128h *)__P __A;`。
- **L918 EN**: Closes the current lexical scope or compound statement.
  **L918 CN**: 结束当前词法作用域或复合语句块。
- **L919 EN**: Blank line separating nearby declarations or logic blocks.
  **L919 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS512 _mm512_storeu_ph(void *__P,`.
  **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS512 _mm512_storeu_ph(void *__P,`。
- **L921 EN**: Continues the surrounding expression or declaration: `__m512h __A) {`.
  **L921 CN**: 继续构造周围的表达式或声明：`__m512h __A) {`。
- **L922 EN**: Declares struct `__storeu_ph`.
  **L922 CN**: 声明 struct `__storeu_ph`。
- **L923 EN**: Adds a standalone statement or declaration: `__m512h_u __v;`.
  **L923 CN**: 添加一条独立语句或声明：`__m512h_u __v;`。
- **L924 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L924 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L925 EN**: Executes a call or declaration centered on `statement`.
  **L925 CN**: 执行以 `statement` 为核心的调用或声明。
- **L926 EN**: Closes the current lexical scope or compound statement.
  **L926 CN**: 结束当前词法作用域或复合语句块。
- **L927 EN**: Blank line separating nearby declarations or logic blocks.
  **L927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_storeu_ph(void *__P,`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_storeu_ph(void *__P,`。
- **L929 EN**: Continues the surrounding expression or declaration: `__m256h __A) {`.
  **L929 CN**: 继续构造周围的表达式或声明：`__m256h __A) {`。
- **L930 EN**: Declares struct `__storeu_ph`.
  **L930 CN**: 声明 struct `__storeu_ph`。
- **L931 EN**: Adds a standalone statement or declaration: `__m256h_u __v;`.
  **L931 CN**: 添加一条独立语句或声明：`__m256h_u __v;`。
- **L932 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L932 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L933 EN**: Executes a call or declaration centered on `statement`.
  **L933 CN**: 执行以 `statement` 为核心的调用或声明。
- **L934 EN**: Closes the current lexical scope or compound statement.
  **L934 CN**: 结束当前词法作用域或复合语句块。
- **L935 EN**: Blank line separating nearby declarations or logic blocks.
  **L935 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS128 _mm_storeu_ph(void *__P,`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS128 _mm_storeu_ph(void *__P,`。

### Lines 937-960

````c
                                                           __m128h __A) {
  struct __storeu_ph {
    __m128h_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_ph *)__P)->__v = __A;
}

// moves with vmovsh:
static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_move_sh(__m128h __a, __m128h __b) {
  __a[0] = __b[0];
  return __a;
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_move_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return __builtin_ia32_selectsh_128(__U, _mm_move_sh(__A, __B), __W);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_move_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  return __builtin_ia32_selectsh_128(__U, _mm_move_sh(__A, __B),
                                     _mm_setzero_ph());
}
````
- **L937 EN**: Continues the surrounding expression or declaration: `__m128h __A) {`.
  **L937 CN**: 继续构造周围的表达式或声明：`__m128h __A) {`。
- **L938 EN**: Declares struct `__storeu_ph`.
  **L938 CN**: 声明 struct `__storeu_ph`。
- **L939 EN**: Adds a standalone statement or declaration: `__m128h_u __v;`.
  **L939 CN**: 添加一条独立语句或声明：`__m128h_u __v;`。
- **L940 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L940 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L941 EN**: Executes a call or declaration centered on `statement`.
  **L941 CN**: 执行以 `statement` 为核心的调用或声明。
- **L942 EN**: Closes the current lexical scope or compound statement.
  **L942 CN**: 结束当前词法作用域或复合语句块。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Comment explains nearby logic, constraints, or intent: `moves with vmovsh:`.
  **L944 CN**: 注释解释附近代码的逻辑、约束或设计意图：`moves with vmovsh:`。
- **L945 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L945 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L946 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_move_sh(__m128h __a, __m128h __b) {`.
  **L946 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_move_sh(__m128h __a, __m128h __b) {`。
- **L947 EN**: Adds a standalone statement or declaration: `__a[0] = __b[0];`.
  **L947 CN**: 添加一条独立语句或声明：`__a[0] = __b[0];`。
- **L948 EN**: Returns from the current function with `__a`.
  **L948 CN**: 以 `__a` 从当前函数返回。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Blank line separating nearby declarations or logic blocks.
  **L950 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L951 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L951 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_move_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_move_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L953 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, _mm_move_sh(__A, __B), __W)`.
  **L953 CN**: 以 `__builtin_ia32_selectsh_128(__U, _mm_move_sh(__A, __B), __W)` 从当前函数返回。
- **L954 EN**: Closes the current lexical scope or compound statement.
  **L954 CN**: 结束当前词法作用域或复合语句块。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L956 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L956 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L957 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_move_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L957 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_move_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L958 EN**: Returns from the current function with `__builtin_ia32_selectsh_128(__U, _mm_move_sh(__A, __B),`.
  **L958 CN**: 以 `__builtin_ia32_selectsh_128(__U, _mm_move_sh(__A, __B),` 从当前函数返回。
- **L959 EN**: Executes a call or declaration centered on `_mm_setzero_ph`.
  **L959 CN**: 执行以 `_mm_setzero_ph` 为核心的调用或声明。
- **L960 EN**: Closes the current lexical scope or compound statement.
  **L960 CN**: 结束当前词法作用域或复合语句块。

### Lines 961-984

````c

// vmovw:
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtsi16_si128(short __a) {
  return (__m128i)(__v8hi){__a, 0, 0, 0, 0, 0, 0, 0};
}

static __inline__ short __DEFAULT_FN_ATTRS128 _mm_cvtsi128_si16(__m128i __a) {
  __v8hi __b = (__v8hi)__a;
  return __b[0];
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_rcp_ph(__m512h __A) {
  return (__m512h)__builtin_ia32_rcpph512_mask(
      (__v32hf)__A, (__v32hf)_mm512_undefined_ph(), (__mmask32)-1);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_rcp_ph(__m512h __W, __mmask32 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_rcpph512_mask((__v32hf)__A, (__v32hf)__W,
                                               (__mmask32)__U);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_rcp_ph(__mmask32 __U, __m512h __A) {
````
- **L961 EN**: Blank line separating nearby declarations or logic blocks.
  **L961 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L962 EN**: Comment explains nearby logic, constraints, or intent: `vmovw:`.
  **L962 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vmovw:`。
- **L963 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtsi16_si128(short __a) {`.
  **L963 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtsi16_si128(short __a) {`。
- **L964 EN**: Returns from the current function with `(__m128i)(__v8hi){__a, 0, 0, 0, 0, 0, 0, 0}`.
  **L964 CN**: 以 `(__m128i)(__v8hi){__a, 0, 0, 0, 0, 0, 0, 0}` 从当前函数返回。
- **L965 EN**: Closes the current lexical scope or compound statement.
  **L965 CN**: 结束当前词法作用域或复合语句块。
- **L966 EN**: Blank line separating nearby declarations or logic blocks.
  **L966 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L967 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ short __DEFAULT_FN_ATTRS128 _mm_cvtsi128_si16(__m128i __a) {`.
  **L967 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ short __DEFAULT_FN_ATTRS128 _mm_cvtsi128_si16(__m128i __a) {`。
- **L968 EN**: Initializes variable `__b` from the expression on the right-hand side.
  **L968 CN**: 使用右侧表达式初始化变量 `__b`。
- **L969 EN**: Returns from the current function with `__b[0]`.
  **L969 CN**: 以 `__b[0]` 从当前函数返回。
- **L970 EN**: Closes the current lexical scope or compound statement.
  **L970 CN**: 结束当前词法作用域或复合语句块。
- **L971 EN**: Blank line separating nearby declarations or logic blocks.
  **L971 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L972 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_rcp_ph(__m512h __A) {`.
  **L972 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_rcp_ph(__m512h __A) {`。
- **L973 EN**: Returns from the current function with `(__m512h)__builtin_ia32_rcpph512_mask(`.
  **L973 CN**: 以 `(__m512h)__builtin_ia32_rcpph512_mask(` 从当前函数返回。
- **L974 EN**: Executes a call or declaration centered on `statement`.
  **L974 CN**: 执行以 `statement` 为核心的调用或声明。
- **L975 EN**: Closes the current lexical scope or compound statement.
  **L975 CN**: 结束当前词法作用域或复合语句块。
- **L976 EN**: Blank line separating nearby declarations or logic blocks.
  **L976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L977 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L977 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L978 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_rcp_ph(__m512h __W, __mmask32 __U, __m512h __A) {`.
  **L978 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_rcp_ph(__m512h __W, __mmask32 __U, __m512h __A) {`。
- **L979 EN**: Returns from the current function with `(__m512h)__builtin_ia32_rcpph512_mask((__v32hf)__A, (__v32hf)__W,`.
  **L979 CN**: 以 `(__m512h)__builtin_ia32_rcpph512_mask((__v32hf)__A, (__v32hf)__W,` 从当前函数返回。
- **L980 EN**: Executes a call or declaration centered on `statement`.
  **L980 CN**: 执行以 `statement` 为核心的调用或声明。
- **L981 EN**: Closes the current lexical scope or compound statement.
  **L981 CN**: 结束当前词法作用域或复合语句块。
- **L982 EN**: Blank line separating nearby declarations or logic blocks.
  **L982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L983 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L983 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L984 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_rcp_ph(__mmask32 __U, __m512h __A) {`.
  **L984 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_rcp_ph(__mmask32 __U, __m512h __A) {`。

### Lines 985-1008

````c
  return (__m512h)__builtin_ia32_rcpph512_mask(
      (__v32hf)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_rsqrt_ph(__m512h __A) {
  return (__m512h)__builtin_ia32_rsqrtph512_mask(
      (__v32hf)__A, (__v32hf)_mm512_undefined_ph(), (__mmask32)-1);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_rsqrt_ph(__m512h __W, __mmask32 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_rsqrtph512_mask((__v32hf)__A, (__v32hf)__W,
                                                 (__mmask32)__U);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_rsqrt_ph(__mmask32 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_rsqrtph512_mask(
      (__v32hf)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U);
}

#define _mm512_getmant_ph(A, B, C)                                             \
  ((__m512h)__builtin_ia32_getmantph512_mask(                                  \
      (__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \
````
- **L985 EN**: Returns from the current function with `(__m512h)__builtin_ia32_rcpph512_mask(`.
  **L985 CN**: 以 `(__m512h)__builtin_ia32_rcpph512_mask(` 从当前函数返回。
- **L986 EN**: Executes a call or declaration centered on `statement`.
  **L986 CN**: 执行以 `statement` 为核心的调用或声明。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Blank line separating nearby declarations or logic blocks.
  **L988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L989 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_rsqrt_ph(__m512h __A) {`.
  **L989 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_rsqrt_ph(__m512h __A) {`。
- **L990 EN**: Returns from the current function with `(__m512h)__builtin_ia32_rsqrtph512_mask(`.
  **L990 CN**: 以 `(__m512h)__builtin_ia32_rsqrtph512_mask(` 从当前函数返回。
- **L991 EN**: Executes a call or declaration centered on `statement`.
  **L991 CN**: 执行以 `statement` 为核心的调用或声明。
- **L992 EN**: Closes the current lexical scope or compound statement.
  **L992 CN**: 结束当前词法作用域或复合语句块。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L994 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L994 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L995 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_rsqrt_ph(__m512h __W, __mmask32 __U, __m512h __A) {`.
  **L995 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_rsqrt_ph(__m512h __W, __mmask32 __U, __m512h __A) {`。
- **L996 EN**: Returns from the current function with `(__m512h)__builtin_ia32_rsqrtph512_mask((__v32hf)__A, (__v32hf)__W,`.
  **L996 CN**: 以 `(__m512h)__builtin_ia32_rsqrtph512_mask((__v32hf)__A, (__v32hf)__W,` 从当前函数返回。
- **L997 EN**: Executes a call or declaration centered on `statement`.
  **L997 CN**: 执行以 `statement` 为核心的调用或声明。
- **L998 EN**: Closes the current lexical scope or compound statement.
  **L998 CN**: 结束当前词法作用域或复合语句块。
- **L999 EN**: Blank line separating nearby declarations or logic blocks.
  **L999 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1000 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1000 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1001 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_rsqrt_ph(__mmask32 __U, __m512h __A) {`.
  **L1001 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_rsqrt_ph(__mmask32 __U, __m512h __A) {`。
- **L1002 EN**: Returns from the current function with `(__m512h)__builtin_ia32_rsqrtph512_mask(`.
  **L1002 CN**: 以 `(__m512h)__builtin_ia32_rsqrtph512_mask(` 从当前函数返回。
- **L1003 EN**: Executes a call or declaration centered on `statement`.
  **L1003 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1004 EN**: Closes the current lexical scope or compound statement.
  **L1004 CN**: 结束当前词法作用域或复合语句块。
- **L1005 EN**: Blank line separating nearby declarations or logic blocks.
  **L1005 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1006 EN**: Defines macro `_mm512_getmant_ph(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1006 CN**: 定义宏 `_mm512_getmant_ph(A, B, C)`，用于条件编译、简写或 API 生成。
- **L1007 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph512_mask`.
  **L1007 CN**: 继续与可调用符号 `__builtin_ia32_getmantph512_mask` 相关的逻辑。
- **L1008 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \`.
  **L1008 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \`。

### Lines 1009-1032

````c
      (__v32hf)_mm512_undefined_ph(), (__mmask32)-1,                           \
      _MM_FROUND_CUR_DIRECTION))

#define _mm512_mask_getmant_ph(W, U, A, B, C)                                  \
  ((__m512h)__builtin_ia32_getmantph512_mask(                                  \
      (__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)), (__v32hf)(__m512h)(W),   \
      (__mmask32)(U), _MM_FROUND_CUR_DIRECTION))

#define _mm512_maskz_getmant_ph(U, A, B, C)                                    \
  ((__m512h)__builtin_ia32_getmantph512_mask(                                  \
      (__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \
      (__v32hf)_mm512_setzero_ph(), (__mmask32)(U), _MM_FROUND_CUR_DIRECTION))

#define _mm512_getmant_round_ph(A, B, C, R)                                    \
  ((__m512h)__builtin_ia32_getmantph512_mask(                                  \
      (__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \
      (__v32hf)_mm512_undefined_ph(), (__mmask32)-1, (int)(R)))

#define _mm512_mask_getmant_round_ph(W, U, A, B, C, R)                         \
  ((__m512h)__builtin_ia32_getmantph512_mask(                                  \
      (__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)), (__v32hf)(__m512h)(W),   \
      (__mmask32)(U), (int)(R)))

#define _mm512_maskz_getmant_round_ph(U, A, B, C, R)                           \
````
- **L1009 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1009 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1010 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L1010 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L1011 EN**: Blank line separating nearby declarations or logic blocks.
  **L1011 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1012 EN**: Defines macro `_mm512_mask_getmant_ph(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1012 CN**: 定义宏 `_mm512_mask_getmant_ph(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L1013 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph512_mask`.
  **L1013 CN**: 继续与可调用符号 `__builtin_ia32_getmantph512_mask` 相关的逻辑。
- **L1014 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)), (__v32hf)(__m512h)(W),   \`.
  **L1014 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)), (__v32hf)(__m512h)(W),   \`。
- **L1015 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), _MM_FROUND_CUR_DIRECTION))`.
  **L1015 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), _MM_FROUND_CUR_DIRECTION))`。
- **L1016 EN**: Blank line separating nearby declarations or logic blocks.
  **L1016 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1017 EN**: Defines macro `_mm512_maskz_getmant_ph(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1017 CN**: 定义宏 `_mm512_maskz_getmant_ph(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L1018 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph512_mask`.
  **L1018 CN**: 继续与可调用符号 `__builtin_ia32_getmantph512_mask` 相关的逻辑。
- **L1019 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \`.
  **L1019 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \`。
- **L1020 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1020 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1021 EN**: Blank line separating nearby declarations or logic blocks.
  **L1021 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1022 EN**: Defines macro `_mm512_getmant_round_ph(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1022 CN**: 定义宏 `_mm512_getmant_round_ph(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1023 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph512_mask`.
  **L1023 CN**: 继续与可调用符号 `__builtin_ia32_getmantph512_mask` 相关的逻辑。
- **L1024 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \`.
  **L1024 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \`。
- **L1025 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1025 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1026 EN**: Blank line separating nearby declarations or logic blocks.
  **L1026 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1027 EN**: Defines macro `_mm512_mask_getmant_round_ph(W, U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1027 CN**: 定义宏 `_mm512_mask_getmant_round_ph(W, U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1028 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph512_mask`.
  **L1028 CN**: 继续与可调用符号 `__builtin_ia32_getmantph512_mask` 相关的逻辑。
- **L1029 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)), (__v32hf)(__m512h)(W),   \`.
  **L1029 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)), (__v32hf)(__m512h)(W),   \`。
- **L1030 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1030 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1031 EN**: Blank line separating nearby declarations or logic blocks.
  **L1031 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1032 EN**: Defines macro `_mm512_maskz_getmant_round_ph(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1032 CN**: 定义宏 `_mm512_maskz_getmant_round_ph(U, A, B, C, R)`，用于条件编译、简写或 API 生成。

### Lines 1033-1056

````c
  ((__m512h)__builtin_ia32_getmantph512_mask(                                  \
      (__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \
      (__v32hf)_mm512_setzero_ph(), (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_getexp_ph(__m512h __A) {
  return (__m512h)__builtin_ia32_getexpph512_mask(
      (__v32hf)__A, (__v32hf)_mm512_undefined_ph(), (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_getexp_ph(__m512h __W, __mmask32 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_getexpph512_mask(
      (__v32hf)__A, (__v32hf)__W, (__mmask32)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_getexp_ph(__mmask32 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_getexpph512_mask(
      (__v32hf)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_getexp_round_ph(A, R)                                           \
````
- **L1033 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantph512_mask`.
  **L1033 CN**: 继续与可调用符号 `__builtin_ia32_getmantph512_mask` 相关的逻辑。
- **L1034 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \`.
  **L1034 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (int)(((C) << 2) | (B)),                          \`。
- **L1035 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1035 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1036 EN**: Blank line separating nearby declarations or logic blocks.
  **L1036 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1037 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_getexp_ph(__m512h __A) {`.
  **L1037 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_getexp_ph(__m512h __A) {`。
- **L1038 EN**: Returns from the current function with `(__m512h)__builtin_ia32_getexpph512_mask(`.
  **L1038 CN**: 以 `(__m512h)__builtin_ia32_getexpph512_mask(` 从当前函数返回。
- **L1039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)_mm512_undefined_ph(), (__mmask32)-1,`.
  **L1039 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)_mm512_undefined_ph(), (__mmask32)-1,`。
- **L1040 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1040 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1041 EN**: Closes the current lexical scope or compound statement.
  **L1041 CN**: 结束当前词法作用域或复合语句块。
- **L1042 EN**: Blank line separating nearby declarations or logic blocks.
  **L1042 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1043 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1043 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1044 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_getexp_ph(__m512h __W, __mmask32 __U, __m512h __A) {`.
  **L1044 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_getexp_ph(__m512h __W, __mmask32 __U, __m512h __A) {`。
- **L1045 EN**: Returns from the current function with `(__m512h)__builtin_ia32_getexpph512_mask(`.
  **L1045 CN**: 以 `(__m512h)__builtin_ia32_getexpph512_mask(` 从当前函数返回。
- **L1046 EN**: Executes a call or declaration centered on `statement`.
  **L1046 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1047 EN**: Closes the current lexical scope or compound statement.
  **L1047 CN**: 结束当前词法作用域或复合语句块。
- **L1048 EN**: Blank line separating nearby declarations or logic blocks.
  **L1048 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1049 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1049 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1050 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_getexp_ph(__mmask32 __U, __m512h __A) {`.
  **L1050 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_getexp_ph(__mmask32 __U, __m512h __A) {`。
- **L1051 EN**: Returns from the current function with `(__m512h)__builtin_ia32_getexpph512_mask(`.
  **L1051 CN**: 以 `(__m512h)__builtin_ia32_getexpph512_mask(` 从当前函数返回。
- **L1052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,`.
  **L1052 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,`。
- **L1053 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1053 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1054 EN**: Closes the current lexical scope or compound statement.
  **L1054 CN**: 结束当前词法作用域或复合语句块。
- **L1055 EN**: Blank line separating nearby declarations or logic blocks.
  **L1055 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1056 EN**: Defines macro `_mm512_getexp_round_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L1056 CN**: 定义宏 `_mm512_getexp_round_ph(A, R)`，用于条件编译、简写或 API 生成。

### Lines 1057-1080

````c
  ((__m512h)__builtin_ia32_getexpph512_mask((__v32hf)(__m512h)(A),             \
                                            (__v32hf)_mm512_undefined_ph(),    \
                                            (__mmask32)-1, (int)(R)))

#define _mm512_mask_getexp_round_ph(W, U, A, R)                                \
  ((__m512h)__builtin_ia32_getexpph512_mask(                                   \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(W), (__mmask32)(U), (int)(R)))

#define _mm512_maskz_getexp_round_ph(U, A, R)                                  \
  ((__m512h)__builtin_ia32_getexpph512_mask((__v32hf)(__m512h)(A),             \
                                            (__v32hf)_mm512_setzero_ph(),      \
                                            (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_scalef_ph(__m512h __A,
                                                                 __m512h __B) {
  return (__m512h)__builtin_ia32_scalefph512_mask(
      (__v32hf)__A, (__v32hf)__B, (__v32hf)_mm512_undefined_ph(), (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_scalef_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_scalefph512_mask((__v32hf)__A, (__v32hf)__B,
                                                  (__v32hf)__W, (__mmask32)__U,
````
- **L1057 EN**: Continues logic associated with callable symbol `__builtin_ia32_getexpph512_mask`.
  **L1057 CN**: 继续与可调用符号 `__builtin_ia32_getexpph512_mask` 相关的逻辑。
- **L1058 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1058 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1059 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L1059 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1061 EN**: Defines macro `_mm512_mask_getexp_round_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1061 CN**: 定义宏 `_mm512_mask_getexp_round_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1062 EN**: Continues logic associated with callable symbol `__builtin_ia32_getexpph512_mask`.
  **L1062 CN**: 继续与可调用符号 `__builtin_ia32_getexpph512_mask` 相关的逻辑。
- **L1063 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(W), (__mmask32)(U), (int)(R)))`.
  **L1063 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(W), (__mmask32)(U), (int)(R)))`。
- **L1064 EN**: Blank line separating nearby declarations or logic blocks.
  **L1064 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1065 EN**: Defines macro `_mm512_maskz_getexp_round_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1065 CN**: 定义宏 `_mm512_maskz_getexp_round_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1066 EN**: Continues logic associated with callable symbol `__builtin_ia32_getexpph512_mask`.
  **L1066 CN**: 继续与可调用符号 `__builtin_ia32_getexpph512_mask` 相关的逻辑。
- **L1067 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1067 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1068 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1068 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1069 EN**: Blank line separating nearby declarations or logic blocks.
  **L1069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_scalef_ph(__m512h __A,`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_scalef_ph(__m512h __A,`。
- **L1071 EN**: Continues the surrounding expression or declaration: `__m512h __B) {`.
  **L1071 CN**: 继续构造周围的表达式或声明：`__m512h __B) {`。
- **L1072 EN**: Returns from the current function with `(__m512h)__builtin_ia32_scalefph512_mask(`.
  **L1072 CN**: 以 `(__m512h)__builtin_ia32_scalefph512_mask(` 从当前函数返回。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, (__v32hf)_mm512_undefined_ph(), (__mmask32)-1,`.
  **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, (__v32hf)_mm512_undefined_ph(), (__mmask32)-1,`。
- **L1074 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1074 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Blank line separating nearby declarations or logic blocks.
  **L1076 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1077 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1077 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1078 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_scalef_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`.
  **L1078 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_scalef_ph(__m512h __W, __mmask32 __U, __m512h __A, __m512h __B) {`。
- **L1079 EN**: Returns from the current function with `(__m512h)__builtin_ia32_scalefph512_mask((__v32hf)__A, (__v32hf)__B,`.
  **L1079 CN**: 以 `(__m512h)__builtin_ia32_scalefph512_mask((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__W, (__mmask32)__U,`.
  **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__W, (__mmask32)__U,`。

### Lines 1081-1104

````c
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_scalef_ph(__mmask32 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_scalefph512_mask(
      (__v32hf)__A, (__v32hf)__B, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_scalef_round_ph(A, B, R)                                        \
  ((__m512h)__builtin_ia32_scalefph512_mask(                                   \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B),                            \
      (__v32hf)_mm512_undefined_ph(), (__mmask32)-1, (int)(R)))

#define _mm512_mask_scalef_round_ph(W, U, A, B, R)                             \
  ((__m512h)__builtin_ia32_scalefph512_mask(                                   \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(W),     \
      (__mmask32)(U), (int)(R)))

#define _mm512_maskz_scalef_round_ph(U, A, B, R)                               \
  ((__m512h)__builtin_ia32_scalefph512_mask(                                   \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B),                            \
      (__v32hf)_mm512_setzero_ph(), (__mmask32)(U), (int)(R)))
````
- **L1081 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1081 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1082 EN**: Closes the current lexical scope or compound statement.
  **L1082 CN**: 结束当前词法作用域或复合语句块。
- **L1083 EN**: Blank line separating nearby declarations or logic blocks.
  **L1083 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1084 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1084 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1085 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_scalef_ph(__mmask32 __U, __m512h __A, __m512h __B) {`.
  **L1085 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_scalef_ph(__mmask32 __U, __m512h __A, __m512h __B) {`。
- **L1086 EN**: Returns from the current function with `(__m512h)__builtin_ia32_scalefph512_mask(`.
  **L1086 CN**: 以 `(__m512h)__builtin_ia32_scalefph512_mask(` 从当前函数返回。
- **L1087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,`.
  **L1087 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,`。
- **L1088 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1088 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1089 EN**: Closes the current lexical scope or compound statement.
  **L1089 CN**: 结束当前词法作用域或复合语句块。
- **L1090 EN**: Blank line separating nearby declarations or logic blocks.
  **L1090 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1091 EN**: Defines macro `_mm512_scalef_round_ph(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1091 CN**: 定义宏 `_mm512_scalef_round_ph(A, B, R)`，用于条件编译、简写或 API 生成。
- **L1092 EN**: Continues logic associated with callable symbol `__builtin_ia32_scalefph512_mask`.
  **L1092 CN**: 继续与可调用符号 `__builtin_ia32_scalefph512_mask` 相关的逻辑。
- **L1093 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B),                            \`.
  **L1093 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B),                            \`。
- **L1094 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1094 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1095 EN**: Blank line separating nearby declarations or logic blocks.
  **L1095 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1096 EN**: Defines macro `_mm512_mask_scalef_round_ph(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1096 CN**: 定义宏 `_mm512_mask_scalef_round_ph(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1097 EN**: Continues logic associated with callable symbol `__builtin_ia32_scalefph512_mask`.
  **L1097 CN**: 继续与可调用符号 `__builtin_ia32_scalefph512_mask` 相关的逻辑。
- **L1098 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(W),     \`.
  **L1098 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(W),     \`。
- **L1099 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1099 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1101 EN**: Defines macro `_mm512_maskz_scalef_round_ph(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1101 CN**: 定义宏 `_mm512_maskz_scalef_round_ph(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1102 EN**: Continues logic associated with callable symbol `__builtin_ia32_scalefph512_mask`.
  **L1102 CN**: 继续与可调用符号 `__builtin_ia32_scalefph512_mask` 相关的逻辑。
- **L1103 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B),                            \`.
  **L1103 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B),                            \`。
- **L1104 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1104 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。

### Lines 1105-1128

````c

#define _mm512_roundscale_ph(A, B)                                             \
  ((__m512h)__builtin_ia32_rndscaleph_mask(                                    \
      (__v32hf)(__m512h)(A), (int)(B), (__v32hf)(__m512h)(A), (__mmask32)-1,   \
      _MM_FROUND_CUR_DIRECTION))

#define _mm512_mask_roundscale_ph(A, B, C, imm)                                \
  ((__m512h)__builtin_ia32_rndscaleph_mask(                                    \
      (__v32hf)(__m512h)(C), (int)(imm), (__v32hf)(__m512h)(A),                \
      (__mmask32)(B), _MM_FROUND_CUR_DIRECTION))

#define _mm512_maskz_roundscale_ph(A, B, imm)                                  \
  ((__m512h)__builtin_ia32_rndscaleph_mask(                                    \
      (__v32hf)(__m512h)(B), (int)(imm), (__v32hf)_mm512_setzero_ph(),         \
      (__mmask32)(A), _MM_FROUND_CUR_DIRECTION))

#define _mm512_mask_roundscale_round_ph(A, B, C, imm, R)                       \
  ((__m512h)__builtin_ia32_rndscaleph_mask((__v32hf)(__m512h)(C), (int)(imm),  \
                                           (__v32hf)(__m512h)(A),              \
                                           (__mmask32)(B), (int)(R)))

#define _mm512_maskz_roundscale_round_ph(A, B, imm, R)                         \
  ((__m512h)__builtin_ia32_rndscaleph_mask((__v32hf)(__m512h)(B), (int)(imm),  \
                                           (__v32hf)_mm512_setzero_ph(),       \
````
- **L1105 EN**: Blank line separating nearby declarations or logic blocks.
  **L1105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1106 EN**: Defines macro `_mm512_roundscale_ph(A, B)` for conditional compilation, shorthand, or API generation.
  **L1106 CN**: 定义宏 `_mm512_roundscale_ph(A, B)`，用于条件编译、简写或 API 生成。
- **L1107 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_mask`.
  **L1107 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_mask` 相关的逻辑。
- **L1108 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (int)(B), (__v32hf)(__m512h)(A), (__mmask32)-1,   \`.
  **L1108 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (int)(B), (__v32hf)(__m512h)(A), (__mmask32)-1,   \`。
- **L1109 EN**: Continues the surrounding expression or declaration: `_MM_FROUND_CUR_DIRECTION))`.
  **L1109 CN**: 继续构造周围的表达式或声明：`_MM_FROUND_CUR_DIRECTION))`。
- **L1110 EN**: Blank line separating nearby declarations or logic blocks.
  **L1110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1111 EN**: Defines macro `_mm512_mask_roundscale_ph(A, B, C, imm)` for conditional compilation, shorthand, or API generation.
  **L1111 CN**: 定义宏 `_mm512_mask_roundscale_ph(A, B, C, imm)`，用于条件编译、简写或 API 生成。
- **L1112 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_mask`.
  **L1112 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_mask` 相关的逻辑。
- **L1113 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(C), (int)(imm), (__v32hf)(__m512h)(A),                \`.
  **L1113 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(C), (int)(imm), (__v32hf)(__m512h)(A),                \`。
- **L1114 EN**: Continues the surrounding expression or declaration: `(__mmask32)(B), _MM_FROUND_CUR_DIRECTION))`.
  **L1114 CN**: 继续构造周围的表达式或声明：`(__mmask32)(B), _MM_FROUND_CUR_DIRECTION))`。
- **L1115 EN**: Blank line separating nearby declarations or logic blocks.
  **L1115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1116 EN**: Defines macro `_mm512_maskz_roundscale_ph(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1116 CN**: 定义宏 `_mm512_maskz_roundscale_ph(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1117 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_mask`.
  **L1117 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_mask` 相关的逻辑。
- **L1118 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1118 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1119 EN**: Continues the surrounding expression or declaration: `(__mmask32)(A), _MM_FROUND_CUR_DIRECTION))`.
  **L1119 CN**: 继续构造周围的表达式或声明：`(__mmask32)(A), _MM_FROUND_CUR_DIRECTION))`。
- **L1120 EN**: Blank line separating nearby declarations or logic blocks.
  **L1120 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1121 EN**: Defines macro `_mm512_mask_roundscale_round_ph(A, B, C, imm, R)` for conditional compilation, shorthand, or API generation.
  **L1121 CN**: 定义宏 `_mm512_mask_roundscale_round_ph(A, B, C, imm, R)`，用于条件编译、简写或 API 生成。
- **L1122 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_mask`.
  **L1122 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_mask` 相关的逻辑。
- **L1123 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A),              \`.
  **L1123 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A),              \`。
- **L1124 EN**: Continues the surrounding expression or declaration: `(__mmask32)(B), (int)(R)))`.
  **L1124 CN**: 继续构造周围的表达式或声明：`(__mmask32)(B), (int)(R)))`。
- **L1125 EN**: Blank line separating nearby declarations or logic blocks.
  **L1125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1126 EN**: Defines macro `_mm512_maskz_roundscale_round_ph(A, B, imm, R)` for conditional compilation, shorthand, or API generation.
  **L1126 CN**: 定义宏 `_mm512_maskz_roundscale_round_ph(A, B, imm, R)`，用于条件编译、简写或 API 生成。
- **L1127 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_mask`.
  **L1127 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_mask` 相关的逻辑。
- **L1128 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1128 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。

### Lines 1129-1152

````c
                                           (__mmask32)(A), (int)(R)))

#define _mm512_roundscale_round_ph(A, imm, R)                                  \
  ((__m512h)__builtin_ia32_rndscaleph_mask((__v32hf)(__m512h)(A), (int)(imm),  \
                                           (__v32hf)_mm512_undefined_ph(),     \
                                           (__mmask32)-1, (int)(R)))

#define _mm512_reduce_ph(A, imm)                                               \
  ((__m512h)__builtin_ia32_reduceph512_mask(                                   \
      (__v32hf)(__m512h)(A), (int)(imm), (__v32hf)_mm512_undefined_ph(),       \
      (__mmask32)-1, _MM_FROUND_CUR_DIRECTION))

#define _mm512_mask_reduce_ph(W, U, A, imm)                                    \
  ((__m512h)__builtin_ia32_reduceph512_mask(                                   \
      (__v32hf)(__m512h)(A), (int)(imm), (__v32hf)(__m512h)(W),                \
      (__mmask32)(U), _MM_FROUND_CUR_DIRECTION))

#define _mm512_maskz_reduce_ph(U, A, imm)                                      \
  ((__m512h)__builtin_ia32_reduceph512_mask(                                   \
      (__v32hf)(__m512h)(A), (int)(imm), (__v32hf)_mm512_setzero_ph(),         \
      (__mmask32)(U), _MM_FROUND_CUR_DIRECTION))

#define _mm512_mask_reduce_round_ph(W, U, A, imm, R)                           \
  ((__m512h)__builtin_ia32_reduceph512_mask((__v32hf)(__m512h)(A), (int)(imm), \
````
- **L1129 EN**: Continues the surrounding expression or declaration: `(__mmask32)(A), (int)(R)))`.
  **L1129 CN**: 继续构造周围的表达式或声明：`(__mmask32)(A), (int)(R)))`。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1131 EN**: Defines macro `_mm512_roundscale_round_ph(A, imm, R)` for conditional compilation, shorthand, or API generation.
  **L1131 CN**: 定义宏 `_mm512_roundscale_round_ph(A, imm, R)`，用于条件编译、简写或 API 生成。
- **L1132 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscaleph_mask`.
  **L1132 CN**: 继续与可调用符号 `__builtin_ia32_rndscaleph_mask` 相关的逻辑。
- **L1133 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1133 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1134 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L1134 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L1135 EN**: Blank line separating nearby declarations or logic blocks.
  **L1135 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1136 EN**: Defines macro `_mm512_reduce_ph(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1136 CN**: 定义宏 `_mm512_reduce_ph(A, imm)`，用于条件编译、简写或 API 生成。
- **L1137 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph512_mask`.
  **L1137 CN**: 继续与可调用符号 `__builtin_ia32_reduceph512_mask` 相关的逻辑。
- **L1138 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1138 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1139 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, _MM_FROUND_CUR_DIRECTION))`.
  **L1139 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, _MM_FROUND_CUR_DIRECTION))`。
- **L1140 EN**: Blank line separating nearby declarations or logic blocks.
  **L1140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1141 EN**: Defines macro `_mm512_mask_reduce_ph(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1141 CN**: 定义宏 `_mm512_mask_reduce_ph(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1142 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph512_mask`.
  **L1142 CN**: 继续与可调用符号 `__builtin_ia32_reduceph512_mask` 相关的逻辑。
- **L1143 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (int)(imm), (__v32hf)(__m512h)(W),                \`.
  **L1143 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (int)(imm), (__v32hf)(__m512h)(W),                \`。
- **L1144 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), _MM_FROUND_CUR_DIRECTION))`.
  **L1144 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), _MM_FROUND_CUR_DIRECTION))`。
- **L1145 EN**: Blank line separating nearby declarations or logic blocks.
  **L1145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1146 EN**: Defines macro `_mm512_maskz_reduce_ph(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1146 CN**: 定义宏 `_mm512_maskz_reduce_ph(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1147 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph512_mask`.
  **L1147 CN**: 继续与可调用符号 `__builtin_ia32_reduceph512_mask` 相关的逻辑。
- **L1148 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1148 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1149 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), _MM_FROUND_CUR_DIRECTION))`.
  **L1149 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), _MM_FROUND_CUR_DIRECTION))`。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1151 EN**: Defines macro `_mm512_mask_reduce_round_ph(W, U, A, imm, R)` for conditional compilation, shorthand, or API generation.
  **L1151 CN**: 定义宏 `_mm512_mask_reduce_round_ph(W, U, A, imm, R)`，用于条件编译、简写或 API 生成。
- **L1152 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph512_mask`.
  **L1152 CN**: 继续与可调用符号 `__builtin_ia32_reduceph512_mask` 相关的逻辑。

### Lines 1153-1176

````c
                                            (__v32hf)(__m512h)(W),             \
                                            (__mmask32)(U), (int)(R)))

#define _mm512_maskz_reduce_round_ph(U, A, imm, R)                             \
  ((__m512h)__builtin_ia32_reduceph512_mask((__v32hf)(__m512h)(A), (int)(imm), \
                                            (__v32hf)_mm512_setzero_ph(),      \
                                            (__mmask32)(U), (int)(R)))

#define _mm512_reduce_round_ph(A, imm, R)                                      \
  ((__m512h)__builtin_ia32_reduceph512_mask((__v32hf)(__m512h)(A), (int)(imm), \
                                            (__v32hf)_mm512_undefined_ph(),    \
                                            (__mmask32)-1, (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rcp_sh(__m128h __A,
                                                           __m128h __B) {
  return (__m128h)__builtin_ia32_rcpsh_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rcp_sh(__m128h __W,
                                                                __mmask8 __U,
                                                                __m128h __A,
                                                                __m128h __B) {
  return (__m128h)__builtin_ia32_rcpsh_mask((__v8hf)__A, (__v8hf)__B,
````
- **L1153 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(W),             \`.
  **L1153 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(W),             \`。
- **L1154 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1154 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1155 EN**: Blank line separating nearby declarations or logic blocks.
  **L1155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1156 EN**: Defines macro `_mm512_maskz_reduce_round_ph(U, A, imm, R)` for conditional compilation, shorthand, or API generation.
  **L1156 CN**: 定义宏 `_mm512_maskz_reduce_round_ph(U, A, imm, R)`，用于条件编译、简写或 API 生成。
- **L1157 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph512_mask`.
  **L1157 CN**: 继续与可调用符号 `__builtin_ia32_reduceph512_mask` 相关的逻辑。
- **L1158 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1158 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1159 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1159 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1160 EN**: Blank line separating nearby declarations or logic blocks.
  **L1160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1161 EN**: Defines macro `_mm512_reduce_round_ph(A, imm, R)` for conditional compilation, shorthand, or API generation.
  **L1161 CN**: 定义宏 `_mm512_reduce_round_ph(A, imm, R)`，用于条件编译、简写或 API 生成。
- **L1162 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceph512_mask`.
  **L1162 CN**: 继续与可调用符号 `__builtin_ia32_reduceph512_mask` 相关的逻辑。
- **L1163 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1163 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1164 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L1164 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rcp_sh(__m128h __A,`.
  **L1166 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rcp_sh(__m128h __A,`。
- **L1167 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1167 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1168 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rcpsh_mask(`.
  **L1168 CN**: 以 `(__m128h)__builtin_ia32_rcpsh_mask(` 从当前函数返回。
- **L1169 EN**: Executes a call or declaration centered on `statement`.
  **L1169 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1170 EN**: Closes the current lexical scope or compound statement.
  **L1170 CN**: 结束当前词法作用域或复合语句块。
- **L1171 EN**: Blank line separating nearby declarations or logic blocks.
  **L1171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rcp_sh(__m128h __W,`.
  **L1172 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rcp_sh(__m128h __W,`。
- **L1173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1173 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1174 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1175 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1175 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1176 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rcpsh_mask((__v8hf)__A, (__v8hf)__B,`.
  **L1176 CN**: 以 `(__m128h)__builtin_ia32_rcpsh_mask((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。

### Lines 1177-1200

````c
                                            (__v8hf)__W, (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_rcp_sh(__mmask8 __U,
                                                                 __m128h __A,
                                                                 __m128h __B) {
  return (__m128h)__builtin_ia32_rcpsh_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rsqrt_sh(__m128h __A,
                                                             __m128h __B) {
  return (__m128h)__builtin_ia32_rsqrtsh_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rsqrt_sh(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A,
                                                                  __m128h __B) {
  return (__m128h)__builtin_ia32_rsqrtsh_mask((__v8hf)__A, (__v8hf)__B,
                                              (__v8hf)__W, (__mmask8)__U);
}

````
- **L1177 EN**: Executes a call or declaration centered on `statement`.
  **L1177 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1178 EN**: Closes the current lexical scope or compound statement.
  **L1178 CN**: 结束当前词法作用域或复合语句块。
- **L1179 EN**: Blank line separating nearby declarations or logic blocks.
  **L1179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_rcp_sh(__mmask8 __U,`.
  **L1180 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_rcp_sh(__mmask8 __U,`。
- **L1181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1181 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1182 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1182 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1183 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rcpsh_mask(`.
  **L1183 CN**: 以 `(__m128h)__builtin_ia32_rcpsh_mask(` 从当前函数返回。
- **L1184 EN**: Executes a call or declaration centered on `statement`.
  **L1184 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1185 EN**: Closes the current lexical scope or compound statement.
  **L1185 CN**: 结束当前词法作用域或复合语句块。
- **L1186 EN**: Blank line separating nearby declarations or logic blocks.
  **L1186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rsqrt_sh(__m128h __A,`.
  **L1187 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_rsqrt_sh(__m128h __A,`。
- **L1188 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1188 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1189 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rsqrtsh_mask(`.
  **L1189 CN**: 以 `(__m128h)__builtin_ia32_rsqrtsh_mask(` 从当前函数返回。
- **L1190 EN**: Executes a call or declaration centered on `statement`.
  **L1190 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1191 EN**: Closes the current lexical scope or compound statement.
  **L1191 CN**: 结束当前词法作用域或复合语句块。
- **L1192 EN**: Blank line separating nearby declarations or logic blocks.
  **L1192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rsqrt_sh(__m128h __W,`.
  **L1193 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_rsqrt_sh(__m128h __W,`。
- **L1194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1194 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1195 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1196 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1196 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1197 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rsqrtsh_mask((__v8hf)__A, (__v8hf)__B,`.
  **L1197 CN**: 以 `(__m128h)__builtin_ia32_rsqrtsh_mask((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L1198 EN**: Executes a call or declaration centered on `statement`.
  **L1198 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1199 EN**: Closes the current lexical scope or compound statement.
  **L1199 CN**: 结束当前词法作用域或复合语句块。
- **L1200 EN**: Blank line separating nearby declarations or logic blocks.
  **L1200 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1201-1224

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_rsqrt_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_rsqrtsh_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U);
}

#define _mm_getmant_round_sh(A, B, C, D, R)                                    \
  ((__m128h)__builtin_ia32_getmantsh_round_mask(                               \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \
      (__v8hf)_mm_setzero_ph(), (__mmask8)-1, (int)(R)))

#define _mm_getmant_sh(A, B, C, D)                                             \
  ((__m128h)__builtin_ia32_getmantsh_round_mask(                               \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \
      (__v8hf)_mm_setzero_ph(), (__mmask8)-1, _MM_FROUND_CUR_DIRECTION))

#define _mm_mask_getmant_sh(W, U, A, B, C, D)                                  \
  ((__m128h)__builtin_ia32_getmantsh_round_mask(                               \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \
      (__v8hf)(__m128h)(W), (__mmask8)(U), _MM_FROUND_CUR_DIRECTION))

#define _mm_mask_getmant_round_sh(W, U, A, B, C, D, R)                         \
  ((__m128h)__builtin_ia32_getmantsh_round_mask(                               \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \
````
- **L1201 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1201 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1202 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_rsqrt_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L1202 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_rsqrt_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L1203 EN**: Returns from the current function with `(__m128h)__builtin_ia32_rsqrtsh_mask(`.
  **L1203 CN**: 以 `(__m128h)__builtin_ia32_rsqrtsh_mask(` 从当前函数返回。
- **L1204 EN**: Executes a call or declaration centered on `statement`.
  **L1204 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1205 EN**: Closes the current lexical scope or compound statement.
  **L1205 CN**: 结束当前词法作用域或复合语句块。
- **L1206 EN**: Blank line separating nearby declarations or logic blocks.
  **L1206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1207 EN**: Defines macro `_mm_getmant_round_sh(A, B, C, D, R)` for conditional compilation, shorthand, or API generation.
  **L1207 CN**: 定义宏 `_mm_getmant_round_sh(A, B, C, D, R)`，用于条件编译、简写或 API 生成。
- **L1208 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantsh_round_mask`.
  **L1208 CN**: 继续与可调用符号 `__builtin_ia32_getmantsh_round_mask` 相关的逻辑。
- **L1209 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`.
  **L1209 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`。
- **L1210 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1210 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1211 EN**: Blank line separating nearby declarations or logic blocks.
  **L1211 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1212 EN**: Defines macro `_mm_getmant_sh(A, B, C, D)` for conditional compilation, shorthand, or API generation.
  **L1212 CN**: 定义宏 `_mm_getmant_sh(A, B, C, D)`，用于条件编译、简写或 API 生成。
- **L1213 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantsh_round_mask`.
  **L1213 CN**: 继续与可调用符号 `__builtin_ia32_getmantsh_round_mask` 相关的逻辑。
- **L1214 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`.
  **L1214 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`。
- **L1215 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1215 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1216 EN**: Blank line separating nearby declarations or logic blocks.
  **L1216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1217 EN**: Defines macro `_mm_mask_getmant_sh(W, U, A, B, C, D)` for conditional compilation, shorthand, or API generation.
  **L1217 CN**: 定义宏 `_mm_mask_getmant_sh(W, U, A, B, C, D)`，用于条件编译、简写或 API 生成。
- **L1218 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantsh_round_mask`.
  **L1218 CN**: 继续与可调用符号 `__builtin_ia32_getmantsh_round_mask` 相关的逻辑。
- **L1219 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`.
  **L1219 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`。
- **L1220 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), (__mmask8)(U), _MM_FROUND_CUR_DIRECTION))`.
  **L1220 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), (__mmask8)(U), _MM_FROUND_CUR_DIRECTION))`。
- **L1221 EN**: Blank line separating nearby declarations or logic blocks.
  **L1221 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1222 EN**: Defines macro `_mm_mask_getmant_round_sh(W, U, A, B, C, D, R)` for conditional compilation, shorthand, or API generation.
  **L1222 CN**: 定义宏 `_mm_mask_getmant_round_sh(W, U, A, B, C, D, R)`，用于条件编译、简写或 API 生成。
- **L1223 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantsh_round_mask`.
  **L1223 CN**: 继续与可调用符号 `__builtin_ia32_getmantsh_round_mask` 相关的逻辑。
- **L1224 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`.
  **L1224 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`。

### Lines 1225-1248

````c
      (__v8hf)(__m128h)(W), (__mmask8)(U), (int)(R)))

#define _mm_maskz_getmant_sh(U, A, B, C, D)                                    \
  ((__m128h)__builtin_ia32_getmantsh_round_mask(                               \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \
      (__v8hf)_mm_setzero_ph(), (__mmask8)(U), _MM_FROUND_CUR_DIRECTION))

#define _mm_maskz_getmant_round_sh(U, A, B, C, D, R)                           \
  ((__m128h)__builtin_ia32_getmantsh_round_mask(                               \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \
      (__v8hf)_mm_setzero_ph(), (__mmask8)(U), (int)(R)))

#define _mm_getexp_round_sh(A, B, R)                                           \
  ((__m128h)__builtin_ia32_getexpsh128_round_mask(                             \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_getexp_sh(__m128h __A,
                                                              __m128h __B) {
  return (__m128h)__builtin_ia32_getexpsh128_round_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

````
- **L1225 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), (__mmask8)(U), (int)(R)))`.
  **L1225 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), (__mmask8)(U), (int)(R)))`。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1227 EN**: Defines macro `_mm_maskz_getmant_sh(U, A, B, C, D)` for conditional compilation, shorthand, or API generation.
  **L1227 CN**: 定义宏 `_mm_maskz_getmant_sh(U, A, B, C, D)`，用于条件编译、简写或 API 生成。
- **L1228 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantsh_round_mask`.
  **L1228 CN**: 继续与可调用符号 `__builtin_ia32_getmantsh_round_mask` 相关的逻辑。
- **L1229 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`.
  **L1229 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`。
- **L1230 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1230 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1231 EN**: Blank line separating nearby declarations or logic blocks.
  **L1231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1232 EN**: Defines macro `_mm_maskz_getmant_round_sh(U, A, B, C, D, R)` for conditional compilation, shorthand, or API generation.
  **L1232 CN**: 定义宏 `_mm_maskz_getmant_round_sh(U, A, B, C, D, R)`，用于条件编译、简写或 API 生成。
- **L1233 EN**: Continues logic associated with callable symbol `__builtin_ia32_getmantsh_round_mask`.
  **L1233 CN**: 继续与可调用符号 `__builtin_ia32_getmantsh_round_mask` 相关的逻辑。
- **L1234 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`.
  **L1234 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (int)(((D) << 2) | (C)),     \`。
- **L1235 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1235 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1236 EN**: Blank line separating nearby declarations or logic blocks.
  **L1236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1237 EN**: Defines macro `_mm_getexp_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1237 CN**: 定义宏 `_mm_getexp_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L1238 EN**: Continues logic associated with callable symbol `__builtin_ia32_getexpsh128_round_mask`.
  **L1238 CN**: 继续与可调用符号 `__builtin_ia32_getexpsh128_round_mask` 相关的逻辑。
- **L1239 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1239 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1240 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L1240 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L1241 EN**: Blank line separating nearby declarations or logic blocks.
  **L1241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_getexp_sh(__m128h __A,`.
  **L1242 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_getexp_sh(__m128h __A,`。
- **L1243 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1243 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1244 EN**: Returns from the current function with `(__m128h)__builtin_ia32_getexpsh128_round_mask(`.
  **L1244 CN**: 以 `(__m128h)__builtin_ia32_getexpsh128_round_mask(` 从当前函数返回。
- **L1245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`.
  **L1245 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`。
- **L1246 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1246 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1247 EN**: Closes the current lexical scope or compound statement.
  **L1247 CN**: 结束当前词法作用域或复合语句块。
- **L1248 EN**: Blank line separating nearby declarations or logic blocks.
  **L1248 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1249-1272

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_getexp_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_getexpsh128_round_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)__W, (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_mask_getexp_round_sh(W, U, A, B, R)                                \
  ((__m128h)__builtin_ia32_getexpsh128_round_mask(                             \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_getexp_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_getexpsh128_round_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_maskz_getexp_round_sh(U, A, B, R)                                  \
  ((__m128h)__builtin_ia32_getexpsh128_round_mask(                             \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(R)))

````
- **L1249 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1249 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1250 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_getexp_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L1250 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_getexp_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L1251 EN**: Returns from the current function with `(__m128h)__builtin_ia32_getexpsh128_round_mask(`.
  **L1251 CN**: 以 `(__m128h)__builtin_ia32_getexpsh128_round_mask(` 从当前函数返回。
- **L1252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)__B, (__v8hf)__W, (__mmask8)__U,`.
  **L1252 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)__B, (__v8hf)__W, (__mmask8)__U,`。
- **L1253 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1253 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1254 EN**: Closes the current lexical scope or compound statement.
  **L1254 CN**: 结束当前词法作用域或复合语句块。
- **L1255 EN**: Blank line separating nearby declarations or logic blocks.
  **L1255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1256 EN**: Defines macro `_mm_mask_getexp_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1256 CN**: 定义宏 `_mm_mask_getexp_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1257 EN**: Continues logic associated with callable symbol `__builtin_ia32_getexpsh128_round_mask`.
  **L1257 CN**: 继续与可调用符号 `__builtin_ia32_getexpsh128_round_mask` 相关的逻辑。
- **L1258 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L1258 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L1259 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1259 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1261 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1261 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1262 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_getexp_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L1262 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_getexp_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L1263 EN**: Returns from the current function with `(__m128h)__builtin_ia32_getexpsh128_round_mask(`.
  **L1263 CN**: 以 `(__m128h)__builtin_ia32_getexpsh128_round_mask(` 从当前函数返回。
- **L1264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L1264 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L1265 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1265 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1266 EN**: Closes the current lexical scope or compound statement.
  **L1266 CN**: 结束当前词法作用域或复合语句块。
- **L1267 EN**: Blank line separating nearby declarations or logic blocks.
  **L1267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1268 EN**: Defines macro `_mm_maskz_getexp_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1268 CN**: 定义宏 `_mm_maskz_getexp_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1269 EN**: Continues logic associated with callable symbol `__builtin_ia32_getexpsh128_round_mask`.
  **L1269 CN**: 继续与可调用符号 `__builtin_ia32_getexpsh128_round_mask` 相关的逻辑。
- **L1270 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1270 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1271 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1271 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1272 EN**: Blank line separating nearby declarations or logic blocks.
  **L1272 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1273-1296

````c
#define _mm_scalef_round_sh(A, B, R)                                           \
  ((__m128h)__builtin_ia32_scalefsh_round_mask(                                \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_scalef_sh(__m128h __A,
                                                              __m128h __B) {
  return (__m128h)__builtin_ia32_scalefsh_round_mask(
      (__v8hf)__A, (__v8hf)(__B), (__v8hf)_mm_setzero_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_scalef_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_scalefsh_round_mask((__v8hf)__A, (__v8hf)__B,
                                                     (__v8hf)__W, (__mmask8)__U,
                                                     _MM_FROUND_CUR_DIRECTION);
}

#define _mm_mask_scalef_round_sh(W, U, A, B, R)                                \
  ((__m128h)__builtin_ia32_scalefsh_round_mask(                                \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

````
- **L1273 EN**: Defines macro `_mm_scalef_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1273 CN**: 定义宏 `_mm_scalef_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L1274 EN**: Continues logic associated with callable symbol `__builtin_ia32_scalefsh_round_mask`.
  **L1274 CN**: 继续与可调用符号 `__builtin_ia32_scalefsh_round_mask` 相关的逻辑。
- **L1275 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1275 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1276 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L1276 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L1277 EN**: Blank line separating nearby declarations or logic blocks.
  **L1277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_scalef_sh(__m128h __A,`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_scalef_sh(__m128h __A,`。
- **L1279 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1279 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1280 EN**: Returns from the current function with `(__m128h)__builtin_ia32_scalefsh_round_mask(`.
  **L1280 CN**: 以 `(__m128h)__builtin_ia32_scalefsh_round_mask(` 从当前函数返回。
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)(__B), (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`.
  **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)(__B), (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`。
- **L1282 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1282 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1283 EN**: Closes the current lexical scope or compound statement.
  **L1283 CN**: 结束当前词法作用域或复合语句块。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1285 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1285 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1286 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_scalef_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L1286 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_scalef_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L1287 EN**: Returns from the current function with `(__m128h)__builtin_ia32_scalefsh_round_mask((__v8hf)__A, (__v8hf)__B,`.
  **L1287 CN**: 以 `(__m128h)__builtin_ia32_scalefsh_round_mask((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L1288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__W, (__mmask8)__U,`.
  **L1288 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__W, (__mmask8)__U,`。
- **L1289 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1289 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1290 EN**: Closes the current lexical scope or compound statement.
  **L1290 CN**: 结束当前词法作用域或复合语句块。
- **L1291 EN**: Blank line separating nearby declarations or logic blocks.
  **L1291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1292 EN**: Defines macro `_mm_mask_scalef_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1292 CN**: 定义宏 `_mm_mask_scalef_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1293 EN**: Continues logic associated with callable symbol `__builtin_ia32_scalefsh_round_mask`.
  **L1293 CN**: 继续与可调用符号 `__builtin_ia32_scalefsh_round_mask` 相关的逻辑。
- **L1294 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L1294 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L1295 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1295 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1297-1320

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_scalef_sh(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_scalefsh_round_mask(
      (__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_maskz_scalef_round_sh(U, A, B, R)                                  \
  ((__m128h)__builtin_ia32_scalefsh_round_mask(                                \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(R)))

#define _mm_roundscale_round_sh(A, B, imm, R)                                  \
  ((__m128h)__builtin_ia32_rndscalesh_round_mask(                              \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(imm), (int)(R)))

#define _mm_roundscale_sh(A, B, imm)                                           \
  ((__m128h)__builtin_ia32_rndscalesh_round_mask(                              \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(imm), _MM_FROUND_CUR_DIRECTION))

#define _mm_mask_roundscale_sh(W, U, A, B, I)                                  \
  ((__m128h)__builtin_ia32_rndscalesh_round_mask(                              \
````
- **L1297 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1297 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1298 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_scalef_sh(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L1298 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_scalef_sh(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L1299 EN**: Returns from the current function with `(__m128h)__builtin_ia32_scalefsh_round_mask(`.
  **L1299 CN**: 以 `(__m128h)__builtin_ia32_scalefsh_round_mask(` 从当前函数返回。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8hf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L1301 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1301 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1302 EN**: Closes the current lexical scope or compound statement.
  **L1302 CN**: 结束当前词法作用域或复合语句块。
- **L1303 EN**: Blank line separating nearby declarations or logic blocks.
  **L1303 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1304 EN**: Defines macro `_mm_maskz_scalef_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1304 CN**: 定义宏 `_mm_maskz_scalef_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1305 EN**: Continues logic associated with callable symbol `__builtin_ia32_scalefsh_round_mask`.
  **L1305 CN**: 继续与可调用符号 `__builtin_ia32_scalefsh_round_mask` 相关的逻辑。
- **L1306 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1306 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1307 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1307 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1308 EN**: Blank line separating nearby declarations or logic blocks.
  **L1308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1309 EN**: Defines macro `_mm_roundscale_round_sh(A, B, imm, R)` for conditional compilation, shorthand, or API generation.
  **L1309 CN**: 定义宏 `_mm_roundscale_round_sh(A, B, imm, R)`，用于条件编译、简写或 API 生成。
- **L1310 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscalesh_round_mask`.
  **L1310 CN**: 继续与可调用符号 `__builtin_ia32_rndscalesh_round_mask` 相关的逻辑。
- **L1311 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1311 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1312 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(imm), (int)(R)))`.
  **L1312 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(imm), (int)(R)))`。
- **L1313 EN**: Blank line separating nearby declarations or logic blocks.
  **L1313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1314 EN**: Defines macro `_mm_roundscale_sh(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1314 CN**: 定义宏 `_mm_roundscale_sh(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1315 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscalesh_round_mask`.
  **L1315 CN**: 继续与可调用符号 `__builtin_ia32_rndscalesh_round_mask` 相关的逻辑。
- **L1316 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1316 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1317 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(imm), _MM_FROUND_CUR_DIRECTION))`.
  **L1317 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(imm), _MM_FROUND_CUR_DIRECTION))`。
- **L1318 EN**: Blank line separating nearby declarations or logic blocks.
  **L1318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1319 EN**: Defines macro `_mm_mask_roundscale_sh(W, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L1319 CN**: 定义宏 `_mm_mask_roundscale_sh(W, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L1320 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscalesh_round_mask`.
  **L1320 CN**: 继续与可调用符号 `__builtin_ia32_rndscalesh_round_mask` 相关的逻辑。

### Lines 1321-1344

````c
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(I), _MM_FROUND_CUR_DIRECTION))

#define _mm_mask_roundscale_round_sh(W, U, A, B, I, R)                         \
  ((__m128h)__builtin_ia32_rndscalesh_round_mask(                              \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(I), (int)(R)))

#define _mm_maskz_roundscale_sh(U, A, B, I)                                    \
  ((__m128h)__builtin_ia32_rndscalesh_round_mask(                              \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(I), _MM_FROUND_CUR_DIRECTION))

#define _mm_maskz_roundscale_round_sh(U, A, B, I, R)                           \
  ((__m128h)__builtin_ia32_rndscalesh_round_mask(                              \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(I), (int)(R)))

#define _mm_reduce_sh(A, B, C)                                                 \
  ((__m128h)__builtin_ia32_reducesh_mask(                                      \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(C), _MM_FROUND_CUR_DIRECTION))

#define _mm_mask_reduce_sh(W, U, A, B, C)                                      \
````
- **L1321 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L1321 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L1322 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(I), _MM_FROUND_CUR_DIRECTION))`.
  **L1322 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(I), _MM_FROUND_CUR_DIRECTION))`。
- **L1323 EN**: Blank line separating nearby declarations or logic blocks.
  **L1323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1324 EN**: Defines macro `_mm_mask_roundscale_round_sh(W, U, A, B, I, R)` for conditional compilation, shorthand, or API generation.
  **L1324 CN**: 定义宏 `_mm_mask_roundscale_round_sh(W, U, A, B, I, R)`，用于条件编译、简写或 API 生成。
- **L1325 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscalesh_round_mask`.
  **L1325 CN**: 继续与可调用符号 `__builtin_ia32_rndscalesh_round_mask` 相关的逻辑。
- **L1326 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L1326 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L1327 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(I), (int)(R)))`.
  **L1327 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(I), (int)(R)))`。
- **L1328 EN**: Blank line separating nearby declarations or logic blocks.
  **L1328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1329 EN**: Defines macro `_mm_maskz_roundscale_sh(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L1329 CN**: 定义宏 `_mm_maskz_roundscale_sh(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L1330 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscalesh_round_mask`.
  **L1330 CN**: 继续与可调用符号 `__builtin_ia32_rndscalesh_round_mask` 相关的逻辑。
- **L1331 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1331 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1332 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(I), _MM_FROUND_CUR_DIRECTION))`.
  **L1332 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(I), _MM_FROUND_CUR_DIRECTION))`。
- **L1333 EN**: Blank line separating nearby declarations or logic blocks.
  **L1333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1334 EN**: Defines macro `_mm_maskz_roundscale_round_sh(U, A, B, I, R)` for conditional compilation, shorthand, or API generation.
  **L1334 CN**: 定义宏 `_mm_maskz_roundscale_round_sh(U, A, B, I, R)`，用于条件编译、简写或 API 生成。
- **L1335 EN**: Continues logic associated with callable symbol `__builtin_ia32_rndscalesh_round_mask`.
  **L1335 CN**: 继续与可调用符号 `__builtin_ia32_rndscalesh_round_mask` 相关的逻辑。
- **L1336 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1336 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1337 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(I), (int)(R)))`.
  **L1337 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(I), (int)(R)))`。
- **L1338 EN**: Blank line separating nearby declarations or logic blocks.
  **L1338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1339 EN**: Defines macro `_mm_reduce_sh(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1339 CN**: 定义宏 `_mm_reduce_sh(A, B, C)`，用于条件编译、简写或 API 生成。
- **L1340 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesh_mask`.
  **L1340 CN**: 继续与可调用符号 `__builtin_ia32_reducesh_mask` 相关的逻辑。
- **L1341 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1341 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1342 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(C), _MM_FROUND_CUR_DIRECTION))`.
  **L1342 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(C), _MM_FROUND_CUR_DIRECTION))`。
- **L1343 EN**: Blank line separating nearby declarations or logic blocks.
  **L1343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1344 EN**: Defines macro `_mm_mask_reduce_sh(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1344 CN**: 定义宏 `_mm_mask_reduce_sh(W, U, A, B, C)`，用于条件编译、简写或 API 生成。

### Lines 1345-1368

````c
  ((__m128h)__builtin_ia32_reducesh_mask(                                      \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(C), _MM_FROUND_CUR_DIRECTION))

#define _mm_maskz_reduce_sh(U, A, B, C)                                        \
  ((__m128h)__builtin_ia32_reducesh_mask(                                      \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(C), _MM_FROUND_CUR_DIRECTION))

#define _mm_reduce_round_sh(A, B, C, R)                                        \
  ((__m128h)__builtin_ia32_reducesh_mask(                                      \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(C), (int)(R)))

#define _mm_mask_reduce_round_sh(W, U, A, B, C, R)                             \
  ((__m128h)__builtin_ia32_reducesh_mask(                                      \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(C), (int)(R)))

#define _mm_maskz_reduce_round_sh(U, A, B, C, R)                               \
  ((__m128h)__builtin_ia32_reducesh_mask(                                      \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(C), (int)(R)))

````
- **L1345 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesh_mask`.
  **L1345 CN**: 继续与可调用符号 `__builtin_ia32_reducesh_mask` 相关的逻辑。
- **L1346 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L1346 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L1347 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C), _MM_FROUND_CUR_DIRECTION))`.
  **L1347 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C), _MM_FROUND_CUR_DIRECTION))`。
- **L1348 EN**: Blank line separating nearby declarations or logic blocks.
  **L1348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1349 EN**: Defines macro `_mm_maskz_reduce_sh(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L1349 CN**: 定义宏 `_mm_maskz_reduce_sh(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L1350 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesh_mask`.
  **L1350 CN**: 继续与可调用符号 `__builtin_ia32_reducesh_mask` 相关的逻辑。
- **L1351 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1351 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1352 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C), _MM_FROUND_CUR_DIRECTION))`.
  **L1352 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C), _MM_FROUND_CUR_DIRECTION))`。
- **L1353 EN**: Blank line separating nearby declarations or logic blocks.
  **L1353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1354 EN**: Defines macro `_mm_reduce_round_sh(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1354 CN**: 定义宏 `_mm_reduce_round_sh(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1355 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesh_mask`.
  **L1355 CN**: 继续与可调用符号 `__builtin_ia32_reducesh_mask` 相关的逻辑。
- **L1356 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1356 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1357 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(C), (int)(R)))`.
  **L1357 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(C), (int)(R)))`。
- **L1358 EN**: Blank line separating nearby declarations or logic blocks.
  **L1358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1359 EN**: Defines macro `_mm_mask_reduce_round_sh(W, U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1359 CN**: 定义宏 `_mm_mask_reduce_round_sh(W, U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1360 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesh_mask`.
  **L1360 CN**: 继续与可调用符号 `__builtin_ia32_reducesh_mask` 相关的逻辑。
- **L1361 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L1361 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L1362 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C), (int)(R)))`.
  **L1362 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C), (int)(R)))`。
- **L1363 EN**: Blank line separating nearby declarations or logic blocks.
  **L1363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1364 EN**: Defines macro `_mm_maskz_reduce_round_sh(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L1364 CN**: 定义宏 `_mm_maskz_reduce_round_sh(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L1365 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducesh_mask`.
  **L1365 CN**: 继续与可调用符号 `__builtin_ia32_reducesh_mask` 相关的逻辑。
- **L1366 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1366 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1367 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(C), (int)(R)))`.
  **L1367 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(C), (int)(R)))`。
- **L1368 EN**: Blank line separating nearby declarations or logic blocks.
  **L1368 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1369-1392

````c
#define _mm512_sqrt_round_ph(A, R)                                             \
  ((__m512h)__builtin_ia32_sqrtph512((__v32hf)(__m512h)(A), (int)(R)))

#define _mm512_mask_sqrt_round_ph(W, U, A, R)                                  \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_sqrt_round_ph((A), (R)),                 \
      (__v32hf)(__m512h)(W)))

#define _mm512_maskz_sqrt_round_ph(U, A, R)                                    \
  ((__m512h)__builtin_ia32_selectph_512(                                       \
      (__mmask32)(U), (__v32hf)_mm512_sqrt_round_ph((A), (R)),                 \
      (__v32hf)_mm512_setzero_ph()))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_sqrt_ph(__m512h __A) {
  return (__m512h)__builtin_elementwise_sqrt((__v32hf)__A);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_sqrt_ph(__m512h __W, __mmask32 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_selectph_512(
      (__mmask32)(__U), (__v32hf)_mm512_sqrt_ph(__A), (__v32hf)(__m512h)(__W));
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
````
- **L1369 EN**: Defines macro `_mm512_sqrt_round_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L1369 CN**: 定义宏 `_mm512_sqrt_round_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L1370 EN**: Continues logic associated with callable symbol `__builtin_ia32_sqrtph512`.
  **L1370 CN**: 继续与可调用符号 `__builtin_ia32_sqrtph512` 相关的逻辑。
- **L1371 EN**: Blank line separating nearby declarations or logic blocks.
  **L1371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1372 EN**: Defines macro `_mm512_mask_sqrt_round_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1372 CN**: 定义宏 `_mm512_mask_sqrt_round_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1373 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L1373 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L1374 EN**: Continues logic associated with callable symbol `_mm512_sqrt_round_ph`.
  **L1374 CN**: 继续与可调用符号 `_mm512_sqrt_round_ph` 相关的逻辑。
- **L1375 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(W)))`.
  **L1375 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(W)))`。
- **L1376 EN**: Blank line separating nearby declarations or logic blocks.
  **L1376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1377 EN**: Defines macro `_mm512_maskz_sqrt_round_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1377 CN**: 定义宏 `_mm512_maskz_sqrt_round_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1378 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectph_512`.
  **L1378 CN**: 继续与可调用符号 `__builtin_ia32_selectph_512` 相关的逻辑。
- **L1379 EN**: Continues logic associated with callable symbol `_mm512_sqrt_round_ph`.
  **L1379 CN**: 继续与可调用符号 `_mm512_sqrt_round_ph` 相关的逻辑。
- **L1380 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1380 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1381 EN**: Blank line separating nearby declarations or logic blocks.
  **L1381 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1382 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_sqrt_ph(__m512h __A) {`.
  **L1382 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_sqrt_ph(__m512h __A) {`。
- **L1383 EN**: Returns from the current function with `(__m512h)__builtin_elementwise_sqrt((__v32hf)__A)`.
  **L1383 CN**: 以 `(__m512h)__builtin_elementwise_sqrt((__v32hf)__A)` 从当前函数返回。
- **L1384 EN**: Closes the current lexical scope or compound statement.
  **L1384 CN**: 结束当前词法作用域或复合语句块。
- **L1385 EN**: Blank line separating nearby declarations or logic blocks.
  **L1385 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1386 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1386 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1387 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_sqrt_ph(__m512h __W, __mmask32 __U, __m512h __A) {`.
  **L1387 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_sqrt_ph(__m512h __W, __mmask32 __U, __m512h __A) {`。
- **L1388 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512(`.
  **L1388 CN**: 以 `(__m512h)__builtin_ia32_selectph_512(` 从当前函数返回。
- **L1389 EN**: Executes a call or declaration centered on `statement`.
  **L1389 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1390 EN**: Closes the current lexical scope or compound statement.
  **L1390 CN**: 结束当前词法作用域或复合语句块。
- **L1391 EN**: Blank line separating nearby declarations or logic blocks.
  **L1391 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1392 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1392 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。

### Lines 1393-1416

````c
_mm512_maskz_sqrt_ph(__mmask32 __U, __m512h __A) {
  return (__m512h)__builtin_ia32_selectph_512((__mmask32)(__U),
                                              (__v32hf)_mm512_sqrt_ph(__A),
                                              (__v32hf)_mm512_setzero_ph());
}

#define _mm_sqrt_round_sh(A, B, R)                                             \
  ((__m128h)__builtin_ia32_sqrtsh_round_mask(                                  \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_sqrt_round_sh(W, U, A, B, R)                                  \
  ((__m128h)__builtin_ia32_sqrtsh_round_mask(                                  \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_sqrt_round_sh(U, A, B, R)                                    \
  ((__m128h)__builtin_ia32_sqrtsh_round_mask(                                  \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)_mm_setzero_ph(),    \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sqrt_sh(__m128h __A,
                                                            __m128h __B) {
  return (__m128h)__builtin_ia32_sqrtsh_round_mask(
````
- **L1393 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_sqrt_ph(__mmask32 __U, __m512h __A) {`.
  **L1393 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_sqrt_ph(__mmask32 __U, __m512h __A) {`。
- **L1394 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512((__mmask32)(__U),`.
  **L1394 CN**: 以 `(__m512h)__builtin_ia32_selectph_512((__mmask32)(__U),` 从当前函数返回。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)_mm512_sqrt_ph(__A),`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)_mm512_sqrt_ph(__A),`。
- **L1396 EN**: Executes a call or declaration centered on `statement`.
  **L1396 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1397 EN**: Closes the current lexical scope or compound statement.
  **L1397 CN**: 结束当前词法作用域或复合语句块。
- **L1398 EN**: Blank line separating nearby declarations or logic blocks.
  **L1398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1399 EN**: Defines macro `_mm_sqrt_round_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1399 CN**: 定义宏 `_mm_sqrt_round_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L1400 EN**: Continues logic associated with callable symbol `__builtin_ia32_sqrtsh_round_mask`.
  **L1400 CN**: 继续与可调用符号 `__builtin_ia32_sqrtsh_round_mask` 相关的逻辑。
- **L1401 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1401 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1402 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L1402 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L1403 EN**: Blank line separating nearby declarations or logic blocks.
  **L1403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1404 EN**: Defines macro `_mm_mask_sqrt_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1404 CN**: 定义宏 `_mm_mask_sqrt_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1405 EN**: Continues logic associated with callable symbol `__builtin_ia32_sqrtsh_round_mask`.
  **L1405 CN**: 继续与可调用符号 `__builtin_ia32_sqrtsh_round_mask` 相关的逻辑。
- **L1406 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`.
  **L1406 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(W),        \`。
- **L1407 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1407 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1408 EN**: Blank line separating nearby declarations or logic blocks.
  **L1408 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1409 EN**: Defines macro `_mm_maskz_sqrt_round_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1409 CN**: 定义宏 `_mm_maskz_sqrt_round_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1410 EN**: Continues logic associated with callable symbol `__builtin_ia32_sqrtsh_round_mask`.
  **L1410 CN**: 继续与可调用符号 `__builtin_ia32_sqrtsh_round_mask` 相关的逻辑。
- **L1411 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1411 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1412 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1412 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1413 EN**: Blank line separating nearby declarations or logic blocks.
  **L1413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sqrt_sh(__m128h __A,`.
  **L1414 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_sqrt_sh(__m128h __A,`。
- **L1415 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1415 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1416 EN**: Returns from the current function with `(__m128h)__builtin_ia32_sqrtsh_round_mask(`.
  **L1416 CN**: 以 `(__m128h)__builtin_ia32_sqrtsh_round_mask(` 从当前函数返回。

### Lines 1417-1440

````c
      (__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)_mm_setzero_ph(),
      (__mmask8)-1, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sqrt_sh(__m128h __W,
                                                                 __mmask32 __U,
                                                                 __m128h __A,
                                                                 __m128h __B) {
  return (__m128h)__builtin_ia32_sqrtsh_round_mask(
      (__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)(__m128h)(__W),
      (__mmask8)(__U), _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sqrt_sh(__mmask32 __U,
                                                                  __m128h __A,
                                                                  __m128h __B) {
  return (__m128h)__builtin_ia32_sqrtsh_round_mask(
      (__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)_mm_setzero_ph(),
      (__mmask8)(__U), _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_mask_fpclass_ph_mask(U, A, imm)                                 \
  ((__mmask32)__builtin_ia32_fpclassph512_mask((__v32hf)(__m512h)(A),          \
                                               (int)(imm), (__mmask32)(U)))
````
- **L1417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)_mm_setzero_ph(),`.
  **L1417 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)_mm_setzero_ph(),`。
- **L1418 EN**: Executes a call or declaration centered on `statement`.
  **L1418 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1419 EN**: Closes the current lexical scope or compound statement.
  **L1419 CN**: 结束当前词法作用域或复合语句块。
- **L1420 EN**: Blank line separating nearby declarations or logic blocks.
  **L1420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sqrt_sh(__m128h __W,`.
  **L1421 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_sqrt_sh(__m128h __W,`。
- **L1422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask32 __U,`.
  **L1422 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask32 __U,`。
- **L1423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1423 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1424 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1424 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1425 EN**: Returns from the current function with `(__m128h)__builtin_ia32_sqrtsh_round_mask(`.
  **L1425 CN**: 以 `(__m128h)__builtin_ia32_sqrtsh_round_mask(` 从当前函数返回。
- **L1426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)(__m128h)(__W),`.
  **L1426 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)(__m128h)(__W),`。
- **L1427 EN**: Executes a call or declaration centered on `statement`.
  **L1427 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1428 EN**: Closes the current lexical scope or compound statement.
  **L1428 CN**: 结束当前词法作用域或复合语句块。
- **L1429 EN**: Blank line separating nearby declarations or logic blocks.
  **L1429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sqrt_sh(__mmask32 __U,`.
  **L1430 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_sqrt_sh(__mmask32 __U,`。
- **L1431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1431 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1432 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1432 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1433 EN**: Returns from the current function with `(__m128h)__builtin_ia32_sqrtsh_round_mask(`.
  **L1433 CN**: 以 `(__m128h)__builtin_ia32_sqrtsh_round_mask(` 从当前函数返回。
- **L1434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)_mm_setzero_ph(),`.
  **L1434 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)(__m128h)(__A), (__v8hf)(__m128h)(__B), (__v8hf)_mm_setzero_ph(),`。
- **L1435 EN**: Executes a call or declaration centered on `statement`.
  **L1435 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1436 EN**: Closes the current lexical scope or compound statement.
  **L1436 CN**: 结束当前词法作用域或复合语句块。
- **L1437 EN**: Blank line separating nearby declarations or logic blocks.
  **L1437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1438 EN**: Defines macro `_mm512_mask_fpclass_ph_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1438 CN**: 定义宏 `_mm512_mask_fpclass_ph_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1439 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassph512_mask`.
  **L1439 CN**: 继续与可调用符号 `__builtin_ia32_fpclassph512_mask` 相关的逻辑。
- **L1440 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask32)(U)))`.
  **L1440 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask32)(U)))`。

### Lines 1441-1464

````c

#define _mm512_fpclass_ph_mask(A, imm)                                         \
  ((__mmask32)__builtin_ia32_fpclassph512_mask((__v32hf)(__m512h)(A),          \
                                               (int)(imm), (__mmask32)-1))

#define _mm_fpclass_sh_mask(A, imm)                                            \
  ((__mmask8)__builtin_ia32_fpclasssh_mask((__v8hf)(__m128h)(A), (int)(imm),   \
                                           (__mmask8)-1))

#define _mm_mask_fpclass_sh_mask(U, A, imm)                                    \
  ((__mmask8)__builtin_ia32_fpclasssh_mask((__v8hf)(__m128h)(A), (int)(imm),   \
                                           (__mmask8)(U)))

#define _mm512_cvt_roundpd_ph(A, R)                                            \
  ((__m128h)__builtin_ia32_vcvtpd2ph512_mask(                                  \
      (__v8df)(A), (__v8hf)_mm_undefined_ph(), (__mmask8)(-1), (int)(R)))

#define _mm512_mask_cvt_roundpd_ph(W, U, A, R)                                 \
  ((__m128h)__builtin_ia32_vcvtpd2ph512_mask((__v8df)(A), (__v8hf)(W),         \
                                             (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundpd_ph(U, A, R)                                   \
  ((__m128h)__builtin_ia32_vcvtpd2ph512_mask(                                  \
      (__v8df)(A), (__v8hf)_mm_setzero_ph(), (__mmask8)(U), (int)(R)))
````
- **L1441 EN**: Blank line separating nearby declarations or logic blocks.
  **L1441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1442 EN**: Defines macro `_mm512_fpclass_ph_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1442 CN**: 定义宏 `_mm512_fpclass_ph_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1443 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassph512_mask`.
  **L1443 CN**: 继续与可调用符号 `__builtin_ia32_fpclassph512_mask` 相关的逻辑。
- **L1444 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask32)-1))`.
  **L1444 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask32)-1))`。
- **L1445 EN**: Blank line separating nearby declarations or logic blocks.
  **L1445 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1446 EN**: Defines macro `_mm_fpclass_sh_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1446 CN**: 定义宏 `_mm_fpclass_sh_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1447 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasssh_mask`.
  **L1447 CN**: 继续与可调用符号 `__builtin_ia32_fpclasssh_mask` 相关的逻辑。
- **L1448 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L1448 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L1449 EN**: Blank line separating nearby declarations or logic blocks.
  **L1449 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1450 EN**: Defines macro `_mm_mask_fpclass_sh_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1450 CN**: 定义宏 `_mm_mask_fpclass_sh_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1451 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasssh_mask`.
  **L1451 CN**: 继续与可调用符号 `__builtin_ia32_fpclasssh_mask` 相关的逻辑。
- **L1452 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1452 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1453 EN**: Blank line separating nearby declarations or logic blocks.
  **L1453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1454 EN**: Defines macro `_mm512_cvt_roundpd_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L1454 CN**: 定义宏 `_mm512_cvt_roundpd_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L1455 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtpd2ph512_mask`.
  **L1455 CN**: 继续与可调用符号 `__builtin_ia32_vcvtpd2ph512_mask` 相关的逻辑。
- **L1456 EN**: Continues logic associated with callable symbol `_mm_undefined_ph`.
  **L1456 CN**: 继续与可调用符号 `_mm_undefined_ph` 相关的逻辑。
- **L1457 EN**: Blank line separating nearby declarations or logic blocks.
  **L1457 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1458 EN**: Defines macro `_mm512_mask_cvt_roundpd_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1458 CN**: 定义宏 `_mm512_mask_cvt_roundpd_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1459 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtpd2ph512_mask`.
  **L1459 CN**: 继续与可调用符号 `__builtin_ia32_vcvtpd2ph512_mask` 相关的逻辑。
- **L1460 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1460 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1461 EN**: Blank line separating nearby declarations or logic blocks.
  **L1461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1462 EN**: Defines macro `_mm512_maskz_cvt_roundpd_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1462 CN**: 定义宏 `_mm512_maskz_cvt_roundpd_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1463 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtpd2ph512_mask`.
  **L1463 CN**: 继续与可调用符号 `__builtin_ia32_vcvtpd2ph512_mask` 相关的逻辑。
- **L1464 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1464 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。

### Lines 1465-1488

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS512 _mm512_cvtpd_ph(__m512d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph512_mask(
      (__v8df)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS512
_mm512_mask_cvtpd_ph(__m128h __W, __mmask8 __U, __m512d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph512_mask(
      (__v8df)__A, (__v8hf)__W, (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtpd_ph(__mmask8 __U, __m512d __A) {
  return (__m128h)__builtin_ia32_vcvtpd2ph512_mask(
      (__v8df)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundph_pd(A, R)                                            \
  ((__m512d)__builtin_ia32_vcvtph2pd512_mask(                                  \
      (__v8hf)(A), (__v8df)_mm512_undefined_pd(), (__mmask8)(-1), (int)(R)))

````
- **L1465 EN**: Blank line separating nearby declarations or logic blocks.
  **L1465 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1466 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128h __DEFAULT_FN_ATTRS512 _mm512_cvtpd_ph(__m512d __A) {`.
  **L1466 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128h __DEFAULT_FN_ATTRS512 _mm512_cvtpd_ph(__m512d __A) {`。
- **L1467 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph512_mask(`.
  **L1467 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph512_mask(` 从当前函数返回。
- **L1468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`.
  **L1468 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`。
- **L1469 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1469 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1470 EN**: Closes the current lexical scope or compound statement.
  **L1470 CN**: 结束当前词法作用域或复合语句块。
- **L1471 EN**: Blank line separating nearby declarations or logic blocks.
  **L1471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1472 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512`.
  **L1472 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512`。
- **L1473 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtpd_ph(__m128h __W, __mmask8 __U, __m512d __A) {`.
  **L1473 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtpd_ph(__m128h __W, __mmask8 __U, __m512d __A) {`。
- **L1474 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph512_mask(`.
  **L1474 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph512_mask(` 从当前函数返回。
- **L1475 EN**: Executes a call or declaration centered on `statement`.
  **L1475 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1476 EN**: Closes the current lexical scope or compound statement.
  **L1476 CN**: 结束当前词法作用域或复合语句块。
- **L1477 EN**: Blank line separating nearby declarations or logic blocks.
  **L1477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1478 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512`.
  **L1478 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512`。
- **L1479 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtpd_ph(__mmask8 __U, __m512d __A) {`.
  **L1479 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtpd_ph(__mmask8 __U, __m512d __A) {`。
- **L1480 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtpd2ph512_mask(`.
  **L1480 CN**: 以 `(__m128h)__builtin_ia32_vcvtpd2ph512_mask(` 从当前函数返回。
- **L1481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8df)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L1481 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8df)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L1482 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1482 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1483 EN**: Closes the current lexical scope or compound statement.
  **L1483 CN**: 结束当前词法作用域或复合语句块。
- **L1484 EN**: Blank line separating nearby declarations or logic blocks.
  **L1484 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1485 EN**: Defines macro `_mm512_cvt_roundph_pd(A, R)` for conditional compilation, shorthand, or API generation.
  **L1485 CN**: 定义宏 `_mm512_cvt_roundph_pd(A, R)`，用于条件编译、简写或 API 生成。
- **L1486 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2pd512_mask`.
  **L1486 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2pd512_mask` 相关的逻辑。
- **L1487 EN**: Continues logic associated with callable symbol `_mm512_undefined_pd`.
  **L1487 CN**: 继续与可调用符号 `_mm512_undefined_pd` 相关的逻辑。
- **L1488 EN**: Blank line separating nearby declarations or logic blocks.
  **L1488 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1489-1512

````c
#define _mm512_mask_cvt_roundph_pd(W, U, A, R)                                 \
  ((__m512d)__builtin_ia32_vcvtph2pd512_mask((__v8hf)(A), (__v8df)(W),         \
                                             (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundph_pd(U, A, R)                                   \
  ((__m512d)__builtin_ia32_vcvtph2pd512_mask(                                  \
      (__v8hf)(A), (__v8df)_mm512_setzero_pd(), (__mmask8)(U), (int)(R)))

static __inline__ __m512d __DEFAULT_FN_ATTRS512 _mm512_cvtph_pd(__m128h __A) {
  return (__m512d)__builtin_ia32_vcvtph2pd512_mask(
      (__v8hf)__A, (__v8df)_mm512_setzero_pd(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_mask_cvtph_pd(__m512d __W, __mmask8 __U, __m128h __A) {
  return (__m512d)__builtin_ia32_vcvtph2pd512_mask(
      (__v8hf)__A, (__v8df)__W, (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512d __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {
  return (__m512d)__builtin_ia32_vcvtph2pd512_mask(
      (__v8hf)__A, (__v8df)_mm512_setzero_pd(), (__mmask8)__U,
````
- **L1489 EN**: Defines macro `_mm512_mask_cvt_roundph_pd(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1489 CN**: 定义宏 `_mm512_mask_cvt_roundph_pd(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1490 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2pd512_mask`.
  **L1490 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2pd512_mask` 相关的逻辑。
- **L1491 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1491 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1492 EN**: Blank line separating nearby declarations or logic blocks.
  **L1492 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1493 EN**: Defines macro `_mm512_maskz_cvt_roundph_pd(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1493 CN**: 定义宏 `_mm512_maskz_cvt_roundph_pd(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1494 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2pd512_mask`.
  **L1494 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2pd512_mask` 相关的逻辑。
- **L1495 EN**: Continues logic associated with callable symbol `_mm512_setzero_pd`.
  **L1495 CN**: 继续与可调用符号 `_mm512_setzero_pd` 相关的逻辑。
- **L1496 EN**: Blank line separating nearby declarations or logic blocks.
  **L1496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1497 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512d __DEFAULT_FN_ATTRS512 _mm512_cvtph_pd(__m128h __A) {`.
  **L1497 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512d __DEFAULT_FN_ATTRS512 _mm512_cvtph_pd(__m128h __A) {`。
- **L1498 EN**: Returns from the current function with `(__m512d)__builtin_ia32_vcvtph2pd512_mask(`.
  **L1498 CN**: 以 `(__m512d)__builtin_ia32_vcvtph2pd512_mask(` 从当前函数返回。
- **L1499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8df)_mm512_setzero_pd(), (__mmask8)-1,`.
  **L1499 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8df)_mm512_setzero_pd(), (__mmask8)-1,`。
- **L1500 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1500 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1501 EN**: Closes the current lexical scope or compound statement.
  **L1501 CN**: 结束当前词法作用域或复合语句块。
- **L1502 EN**: Blank line separating nearby declarations or logic blocks.
  **L1502 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1503 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L1503 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L1504 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtph_pd(__m512d __W, __mmask8 __U, __m128h __A) {`.
  **L1504 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtph_pd(__m512d __W, __mmask8 __U, __m128h __A) {`。
- **L1505 EN**: Returns from the current function with `(__m512d)__builtin_ia32_vcvtph2pd512_mask(`.
  **L1505 CN**: 以 `(__m512d)__builtin_ia32_vcvtph2pd512_mask(` 从当前函数返回。
- **L1506 EN**: Executes a call or declaration centered on `statement`.
  **L1506 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1507 EN**: Closes the current lexical scope or compound statement.
  **L1507 CN**: 结束当前词法作用域或复合语句块。
- **L1508 EN**: Blank line separating nearby declarations or logic blocks.
  **L1508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1509 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512d __DEFAULT_FN_ATTRS512`.
  **L1509 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512d __DEFAULT_FN_ATTRS512`。
- **L1510 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {`.
  **L1510 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtph_pd(__mmask8 __U, __m128h __A) {`。
- **L1511 EN**: Returns from the current function with `(__m512d)__builtin_ia32_vcvtph2pd512_mask(`.
  **L1511 CN**: 以 `(__m512d)__builtin_ia32_vcvtph2pd512_mask(` 从当前函数返回。
- **L1512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8df)_mm512_setzero_pd(), (__mmask8)__U,`.
  **L1512 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8df)_mm512_setzero_pd(), (__mmask8)__U,`。

### Lines 1513-1536

````c
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_cvt_roundsh_ss(A, B, R)                                            \
  ((__m128)__builtin_ia32_vcvtsh2ss_round_mask((__v4sf)(A), (__v8hf)(B),       \
                                               (__v4sf)_mm_undefined_ps(),     \
                                               (__mmask8)(-1), (int)(R)))

#define _mm_mask_cvt_roundsh_ss(W, U, A, B, R)                                 \
  ((__m128)__builtin_ia32_vcvtsh2ss_round_mask(                                \
      (__v4sf)(A), (__v8hf)(B), (__v4sf)(W), (__mmask8)(U), (int)(R)))

#define _mm_maskz_cvt_roundsh_ss(U, A, B, R)                                   \
  ((__m128)__builtin_ia32_vcvtsh2ss_round_mask((__v4sf)(A), (__v8hf)(B),       \
                                               (__v4sf)_mm_setzero_ps(),       \
                                               (__mmask8)(U), (int)(R)))

static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_cvtsh_ss(__m128 __A,
                                                            __m128h __B) {
  return (__m128)__builtin_ia32_vcvtsh2ss_round_mask(
      (__v4sf)__A, (__v8hf)__B, (__v4sf)_mm_undefined_ps(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

````
- **L1513 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1513 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1514 EN**: Closes the current lexical scope or compound statement.
  **L1514 CN**: 结束当前词法作用域或复合语句块。
- **L1515 EN**: Blank line separating nearby declarations or logic blocks.
  **L1515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1516 EN**: Defines macro `_mm_cvt_roundsh_ss(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1516 CN**: 定义宏 `_mm_cvt_roundsh_ss(A, B, R)`，用于条件编译、简写或 API 生成。
- **L1517 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2ss_round_mask`.
  **L1517 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2ss_round_mask` 相关的逻辑。
- **L1518 EN**: Continues logic associated with callable symbol `_mm_undefined_ps`.
  **L1518 CN**: 继续与可调用符号 `_mm_undefined_ps` 相关的逻辑。
- **L1519 EN**: Continues the surrounding expression or declaration: `(__mmask8)(-1), (int)(R)))`.
  **L1519 CN**: 继续构造周围的表达式或声明：`(__mmask8)(-1), (int)(R)))`。
- **L1520 EN**: Blank line separating nearby declarations or logic blocks.
  **L1520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1521 EN**: Defines macro `_mm_mask_cvt_roundsh_ss(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1521 CN**: 定义宏 `_mm_mask_cvt_roundsh_ss(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1522 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2ss_round_mask`.
  **L1522 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2ss_round_mask` 相关的逻辑。
- **L1523 EN**: Continues the surrounding expression or declaration: `(__v4sf)(A), (__v8hf)(B), (__v4sf)(W), (__mmask8)(U), (int)(R)))`.
  **L1523 CN**: 继续构造周围的表达式或声明：`(__v4sf)(A), (__v8hf)(B), (__v4sf)(W), (__mmask8)(U), (int)(R)))`。
- **L1524 EN**: Blank line separating nearby declarations or logic blocks.
  **L1524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1525 EN**: Defines macro `_mm_maskz_cvt_roundsh_ss(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1525 CN**: 定义宏 `_mm_maskz_cvt_roundsh_ss(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1526 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2ss_round_mask`.
  **L1526 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2ss_round_mask` 相关的逻辑。
- **L1527 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L1527 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L1528 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1528 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1529 EN**: Blank line separating nearby declarations or logic blocks.
  **L1529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_cvtsh_ss(__m128 __A,`.
  **L1530 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_cvtsh_ss(__m128 __A,`。
- **L1531 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1531 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1532 EN**: Returns from the current function with `(__m128)__builtin_ia32_vcvtsh2ss_round_mask(`.
  **L1532 CN**: 以 `(__m128)__builtin_ia32_vcvtsh2ss_round_mask(` 从当前函数返回。
- **L1533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__A, (__v8hf)__B, (__v4sf)_mm_undefined_ps(), (__mmask8)-1,`.
  **L1533 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__A, (__v8hf)__B, (__v4sf)_mm_undefined_ps(), (__mmask8)-1,`。
- **L1534 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1534 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1535 EN**: Closes the current lexical scope or compound statement.
  **L1535 CN**: 结束当前词法作用域或复合语句块。
- **L1536 EN**: Blank line separating nearby declarations or logic blocks.
  **L1536 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1537-1560

````c
static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_mask_cvtsh_ss(__m128 __W,
                                                                 __mmask8 __U,
                                                                 __m128 __A,
                                                                 __m128h __B) {
  return (__m128)__builtin_ia32_vcvtsh2ss_round_mask((__v4sf)__A, (__v8hf)__B,
                                                     (__v4sf)__W, (__mmask8)__U,
                                                     _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_maskz_cvtsh_ss(__mmask8 __U,
                                                                  __m128 __A,
                                                                  __m128h __B) {
  return (__m128)__builtin_ia32_vcvtsh2ss_round_mask(
      (__v4sf)__A, (__v8hf)__B, (__v4sf)_mm_setzero_ps(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_cvt_roundss_sh(A, B, R)                                            \
  ((__m128h)__builtin_ia32_vcvtss2sh_round_mask((__v8hf)(A), (__v4sf)(B),      \
                                                (__v8hf)_mm_undefined_ph(),    \
                                                (__mmask8)(-1), (int)(R)))

#define _mm_mask_cvt_roundss_sh(W, U, A, B, R)                                 \
  ((__m128h)__builtin_ia32_vcvtss2sh_round_mask(                               \
````
- **L1537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_mask_cvtsh_ss(__m128 __W,`.
  **L1537 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_mask_cvtsh_ss(__m128 __W,`。
- **L1538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1538 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128 __A,`.
  **L1539 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128 __A,`。
- **L1540 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1540 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1541 EN**: Returns from the current function with `(__m128)__builtin_ia32_vcvtsh2ss_round_mask((__v4sf)__A, (__v8hf)__B,`.
  **L1541 CN**: 以 `(__m128)__builtin_ia32_vcvtsh2ss_round_mask((__v4sf)__A, (__v8hf)__B,` 从当前函数返回。
- **L1542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__W, (__mmask8)__U,`.
  **L1542 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__W, (__mmask8)__U,`。
- **L1543 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1543 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1544 EN**: Closes the current lexical scope or compound statement.
  **L1544 CN**: 结束当前词法作用域或复合语句块。
- **L1545 EN**: Blank line separating nearby declarations or logic blocks.
  **L1545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_maskz_cvtsh_ss(__mmask8 __U,`.
  **L1546 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_maskz_cvtsh_ss(__mmask8 __U,`。
- **L1547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128 __A,`.
  **L1547 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128 __A,`。
- **L1548 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1548 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1549 EN**: Returns from the current function with `(__m128)__builtin_ia32_vcvtsh2ss_round_mask(`.
  **L1549 CN**: 以 `(__m128)__builtin_ia32_vcvtsh2ss_round_mask(` 从当前函数返回。
- **L1550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__A, (__v8hf)__B, (__v4sf)_mm_setzero_ps(), (__mmask8)__U,`.
  **L1550 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__A, (__v8hf)__B, (__v4sf)_mm_setzero_ps(), (__mmask8)__U,`。
- **L1551 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1551 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1552 EN**: Closes the current lexical scope or compound statement.
  **L1552 CN**: 结束当前词法作用域或复合语句块。
- **L1553 EN**: Blank line separating nearby declarations or logic blocks.
  **L1553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1554 EN**: Defines macro `_mm_cvt_roundss_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1554 CN**: 定义宏 `_mm_cvt_roundss_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L1555 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtss2sh_round_mask`.
  **L1555 CN**: 继续与可调用符号 `__builtin_ia32_vcvtss2sh_round_mask` 相关的逻辑。
- **L1556 EN**: Continues logic associated with callable symbol `_mm_undefined_ph`.
  **L1556 CN**: 继续与可调用符号 `_mm_undefined_ph` 相关的逻辑。
- **L1557 EN**: Continues the surrounding expression or declaration: `(__mmask8)(-1), (int)(R)))`.
  **L1557 CN**: 继续构造周围的表达式或声明：`(__mmask8)(-1), (int)(R)))`。
- **L1558 EN**: Blank line separating nearby declarations or logic blocks.
  **L1558 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1559 EN**: Defines macro `_mm_mask_cvt_roundss_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1559 CN**: 定义宏 `_mm_mask_cvt_roundss_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1560 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtss2sh_round_mask`.
  **L1560 CN**: 继续与可调用符号 `__builtin_ia32_vcvtss2sh_round_mask` 相关的逻辑。

### Lines 1561-1584

````c
      (__v8hf)(A), (__v4sf)(B), (__v8hf)(W), (__mmask8)(U), (int)(R)))

#define _mm_maskz_cvt_roundss_sh(U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_vcvtss2sh_round_mask((__v8hf)(A), (__v4sf)(B),      \
                                                (__v8hf)_mm_setzero_ph(),      \
                                                (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtss_sh(__m128h __A,
                                                             __m128 __B) {
  return (__m128h)__builtin_ia32_vcvtss2sh_round_mask(
      (__v8hf)__A, (__v4sf)__B, (__v8hf)_mm_undefined_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtss_sh(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A,
                                                                  __m128 __B) {
  return (__m128h)__builtin_ia32_vcvtss2sh_round_mask(
      (__v8hf)__A, (__v4sf)__B, (__v8hf)__W, (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_cvtss_sh(__mmask8 __U,
````
- **L1561 EN**: Continues the surrounding expression or declaration: `(__v8hf)(A), (__v4sf)(B), (__v8hf)(W), (__mmask8)(U), (int)(R)))`.
  **L1561 CN**: 继续构造周围的表达式或声明：`(__v8hf)(A), (__v4sf)(B), (__v8hf)(W), (__mmask8)(U), (int)(R)))`。
- **L1562 EN**: Blank line separating nearby declarations or logic blocks.
  **L1562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1563 EN**: Defines macro `_mm_maskz_cvt_roundss_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1563 CN**: 定义宏 `_mm_maskz_cvt_roundss_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1564 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtss2sh_round_mask`.
  **L1564 CN**: 继续与可调用符号 `__builtin_ia32_vcvtss2sh_round_mask` 相关的逻辑。
- **L1565 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1565 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1566 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1566 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1567 EN**: Blank line separating nearby declarations or logic blocks.
  **L1567 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtss_sh(__m128h __A,`.
  **L1568 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtss_sh(__m128h __A,`。
- **L1569 EN**: Continues the surrounding expression or declaration: `__m128 __B) {`.
  **L1569 CN**: 继续构造周围的表达式或声明：`__m128 __B) {`。
- **L1570 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtss2sh_round_mask(`.
  **L1570 CN**: 以 `(__m128h)__builtin_ia32_vcvtss2sh_round_mask(` 从当前函数返回。
- **L1571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v4sf)__B, (__v8hf)_mm_undefined_ph(), (__mmask8)-1,`.
  **L1571 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v4sf)__B, (__v8hf)_mm_undefined_ph(), (__mmask8)-1,`。
- **L1572 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1572 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1573 EN**: Closes the current lexical scope or compound statement.
  **L1573 CN**: 结束当前词法作用域或复合语句块。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtss_sh(__m128h __W,`.
  **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtss_sh(__m128h __W,`。
- **L1576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1576 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1577 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1578 EN**: Continues the surrounding expression or declaration: `__m128 __B) {`.
  **L1578 CN**: 继续构造周围的表达式或声明：`__m128 __B) {`。
- **L1579 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtss2sh_round_mask(`.
  **L1579 CN**: 以 `(__m128h)__builtin_ia32_vcvtss2sh_round_mask(` 从当前函数返回。
- **L1580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v4sf)__B, (__v8hf)__W, (__mmask8)__U,`.
  **L1580 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v4sf)__B, (__v8hf)__W, (__mmask8)__U,`。
- **L1581 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1581 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1582 EN**: Closes the current lexical scope or compound statement.
  **L1582 CN**: 结束当前词法作用域或复合语句块。
- **L1583 EN**: Blank line separating nearby declarations or logic blocks.
  **L1583 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_cvtss_sh(__mmask8 __U,`.
  **L1584 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_maskz_cvtss_sh(__mmask8 __U,`。

### Lines 1585-1608

````c
                                                                   __m128h __A,
                                                                   __m128 __B) {
  return (__m128h)__builtin_ia32_vcvtss2sh_round_mask(
      (__v8hf)__A, (__v4sf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_cvt_roundsd_sh(A, B, R)                                            \
  ((__m128h)__builtin_ia32_vcvtsd2sh_round_mask((__v8hf)(A), (__v2df)(B),      \
                                                (__v8hf)_mm_undefined_ph(),    \
                                                (__mmask8)(-1), (int)(R)))

#define _mm_mask_cvt_roundsd_sh(W, U, A, B, R)                                 \
  ((__m128h)__builtin_ia32_vcvtsd2sh_round_mask(                               \
      (__v8hf)(A), (__v2df)(B), (__v8hf)(W), (__mmask8)(U), (int)(R)))

#define _mm_maskz_cvt_roundsd_sh(U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_vcvtsd2sh_round_mask((__v8hf)(A), (__v2df)(B),      \
                                                (__v8hf)_mm_setzero_ph(),      \
                                                (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtsd_sh(__m128h __A,
                                                             __m128d __B) {
  return (__m128h)__builtin_ia32_vcvtsd2sh_round_mask(
````
- **L1585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1585 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1586 EN**: Continues the surrounding expression or declaration: `__m128 __B) {`.
  **L1586 CN**: 继续构造周围的表达式或声明：`__m128 __B) {`。
- **L1587 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtss2sh_round_mask(`.
  **L1587 CN**: 以 `(__m128h)__builtin_ia32_vcvtss2sh_round_mask(` 从当前函数返回。
- **L1588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v4sf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L1588 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v4sf)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L1589 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1589 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1590 EN**: Closes the current lexical scope or compound statement.
  **L1590 CN**: 结束当前词法作用域或复合语句块。
- **L1591 EN**: Blank line separating nearby declarations or logic blocks.
  **L1591 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1592 EN**: Defines macro `_mm_cvt_roundsd_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1592 CN**: 定义宏 `_mm_cvt_roundsd_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L1593 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsd2sh_round_mask`.
  **L1593 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsd2sh_round_mask` 相关的逻辑。
- **L1594 EN**: Continues logic associated with callable symbol `_mm_undefined_ph`.
  **L1594 CN**: 继续与可调用符号 `_mm_undefined_ph` 相关的逻辑。
- **L1595 EN**: Continues the surrounding expression or declaration: `(__mmask8)(-1), (int)(R)))`.
  **L1595 CN**: 继续构造周围的表达式或声明：`(__mmask8)(-1), (int)(R)))`。
- **L1596 EN**: Blank line separating nearby declarations or logic blocks.
  **L1596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1597 EN**: Defines macro `_mm_mask_cvt_roundsd_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1597 CN**: 定义宏 `_mm_mask_cvt_roundsd_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1598 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsd2sh_round_mask`.
  **L1598 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsd2sh_round_mask` 相关的逻辑。
- **L1599 EN**: Continues the surrounding expression or declaration: `(__v8hf)(A), (__v2df)(B), (__v8hf)(W), (__mmask8)(U), (int)(R)))`.
  **L1599 CN**: 继续构造周围的表达式或声明：`(__v8hf)(A), (__v2df)(B), (__v8hf)(W), (__mmask8)(U), (int)(R)))`。
- **L1600 EN**: Blank line separating nearby declarations or logic blocks.
  **L1600 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1601 EN**: Defines macro `_mm_maskz_cvt_roundsd_sh(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1601 CN**: 定义宏 `_mm_maskz_cvt_roundsd_sh(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1602 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsd2sh_round_mask`.
  **L1602 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsd2sh_round_mask` 相关的逻辑。
- **L1603 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L1603 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L1604 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1604 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1605 EN**: Blank line separating nearby declarations or logic blocks.
  **L1605 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtsd_sh(__m128h __A,`.
  **L1606 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvtsd_sh(__m128h __A,`。
- **L1607 EN**: Continues the surrounding expression or declaration: `__m128d __B) {`.
  **L1607 CN**: 继续构造周围的表达式或声明：`__m128d __B) {`。
- **L1608 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtsd2sh_round_mask(`.
  **L1608 CN**: 以 `(__m128h)__builtin_ia32_vcvtsd2sh_round_mask(` 从当前函数返回。

### Lines 1609-1632

````c
      (__v8hf)__A, (__v2df)__B, (__v8hf)_mm_undefined_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtsd_sh(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A,
                                                                  __m128d __B) {
  return (__m128h)__builtin_ia32_vcvtsd2sh_round_mask(
      (__v8hf)__A, (__v2df)__B, (__v8hf)__W, (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_cvtsd_sh(__mmask8 __U, __m128h __A, __m128d __B) {
  return (__m128h)__builtin_ia32_vcvtsd2sh_round_mask(
      (__v8hf)__A, (__v2df)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_cvt_roundsh_sd(A, B, R)                                            \
  ((__m128d)__builtin_ia32_vcvtsh2sd_round_mask((__v2df)(A), (__v8hf)(B),      \
                                                (__v2df)_mm_undefined_pd(),    \
                                                (__mmask8)(-1), (int)(R)))
````
- **L1609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v2df)__B, (__v8hf)_mm_undefined_ph(), (__mmask8)-1,`.
  **L1609 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v2df)__B, (__v8hf)_mm_undefined_ph(), (__mmask8)-1,`。
- **L1610 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1610 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1611 EN**: Closes the current lexical scope or compound statement.
  **L1611 CN**: 结束当前词法作用域或复合语句块。
- **L1612 EN**: Blank line separating nearby declarations or logic blocks.
  **L1612 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtsd_sh(__m128h __W,`.
  **L1613 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_cvtsd_sh(__m128h __W,`。
- **L1614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1614 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L1615 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L1616 EN**: Continues the surrounding expression or declaration: `__m128d __B) {`.
  **L1616 CN**: 继续构造周围的表达式或声明：`__m128d __B) {`。
- **L1617 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtsd2sh_round_mask(`.
  **L1617 CN**: 以 `(__m128h)__builtin_ia32_vcvtsd2sh_round_mask(` 从当前函数返回。
- **L1618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v2df)__B, (__v8hf)__W, (__mmask8)__U,`.
  **L1618 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v2df)__B, (__v8hf)__W, (__mmask8)__U,`。
- **L1619 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1619 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1620 EN**: Closes the current lexical scope or compound statement.
  **L1620 CN**: 结束当前词法作用域或复合语句块。
- **L1621 EN**: Blank line separating nearby declarations or logic blocks.
  **L1621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1622 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L1622 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L1623 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtsd_sh(__mmask8 __U, __m128h __A, __m128d __B) {`.
  **L1623 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtsd_sh(__mmask8 __U, __m128h __A, __m128d __B) {`。
- **L1624 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtsd2sh_round_mask(`.
  **L1624 CN**: 以 `(__m128h)__builtin_ia32_vcvtsd2sh_round_mask(` 从当前函数返回。
- **L1625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v2df)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L1625 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v2df)__B, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L1626 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1626 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1627 EN**: Closes the current lexical scope or compound statement.
  **L1627 CN**: 结束当前词法作用域或复合语句块。
- **L1628 EN**: Blank line separating nearby declarations or logic blocks.
  **L1628 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1629 EN**: Defines macro `_mm_cvt_roundsh_sd(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1629 CN**: 定义宏 `_mm_cvt_roundsh_sd(A, B, R)`，用于条件编译、简写或 API 生成。
- **L1630 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2sd_round_mask`.
  **L1630 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2sd_round_mask` 相关的逻辑。
- **L1631 EN**: Continues logic associated with callable symbol `_mm_undefined_pd`.
  **L1631 CN**: 继续与可调用符号 `_mm_undefined_pd` 相关的逻辑。
- **L1632 EN**: Continues the surrounding expression or declaration: `(__mmask8)(-1), (int)(R)))`.
  **L1632 CN**: 继续构造周围的表达式或声明：`(__mmask8)(-1), (int)(R)))`。

### Lines 1633-1656

````c

#define _mm_mask_cvt_roundsh_sd(W, U, A, B, R)                                 \
  ((__m128d)__builtin_ia32_vcvtsh2sd_round_mask(                               \
      (__v2df)(A), (__v8hf)(B), (__v2df)(W), (__mmask8)(U), (int)(R)))

#define _mm_maskz_cvt_roundsh_sd(U, A, B, R)                                   \
  ((__m128d)__builtin_ia32_vcvtsh2sd_round_mask((__v2df)(A), (__v8hf)(B),      \
                                                (__v2df)_mm_setzero_pd(),      \
                                                (__mmask8)(U), (int)(R)))

static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_cvtsh_sd(__m128d __A,
                                                             __m128h __B) {
  return (__m128d)__builtin_ia32_vcvtsh2sd_round_mask(
      (__v2df)__A, (__v8hf)__B, (__v2df)_mm_undefined_pd(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_mask_cvtsh_sd(__m128d __W,
                                                                  __mmask8 __U,
                                                                  __m128d __A,
                                                                  __m128h __B) {
  return (__m128d)__builtin_ia32_vcvtsh2sd_round_mask(
      (__v2df)__A, (__v8hf)__B, (__v2df)__W, (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
````
- **L1633 EN**: Blank line separating nearby declarations or logic blocks.
  **L1633 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1634 EN**: Defines macro `_mm_mask_cvt_roundsh_sd(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1634 CN**: 定义宏 `_mm_mask_cvt_roundsh_sd(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1635 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2sd_round_mask`.
  **L1635 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2sd_round_mask` 相关的逻辑。
- **L1636 EN**: Continues the surrounding expression or declaration: `(__v2df)(A), (__v8hf)(B), (__v2df)(W), (__mmask8)(U), (int)(R)))`.
  **L1636 CN**: 继续构造周围的表达式或声明：`(__v2df)(A), (__v8hf)(B), (__v2df)(W), (__mmask8)(U), (int)(R)))`。
- **L1637 EN**: Blank line separating nearby declarations or logic blocks.
  **L1637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1638 EN**: Defines macro `_mm_maskz_cvt_roundsh_sd(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L1638 CN**: 定义宏 `_mm_maskz_cvt_roundsh_sd(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L1639 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2sd_round_mask`.
  **L1639 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2sd_round_mask` 相关的逻辑。
- **L1640 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1640 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1641 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L1641 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L1642 EN**: Blank line separating nearby declarations or logic blocks.
  **L1642 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_cvtsh_sd(__m128d __A,`.
  **L1643 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_cvtsh_sd(__m128d __A,`。
- **L1644 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1644 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1645 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vcvtsh2sd_round_mask(`.
  **L1645 CN**: 以 `(__m128d)__builtin_ia32_vcvtsh2sd_round_mask(` 从当前函数返回。
- **L1646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)__A, (__v8hf)__B, (__v2df)_mm_undefined_pd(), (__mmask8)-1,`.
  **L1646 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)__A, (__v8hf)__B, (__v2df)_mm_undefined_pd(), (__mmask8)-1,`。
- **L1647 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1647 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1648 EN**: Closes the current lexical scope or compound statement.
  **L1648 CN**: 结束当前词法作用域或复合语句块。
- **L1649 EN**: Blank line separating nearby declarations or logic blocks.
  **L1649 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_mask_cvtsh_sd(__m128d __W,`.
  **L1650 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_mask_cvtsh_sd(__m128d __W,`。
- **L1651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L1651 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L1652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128d __A,`.
  **L1652 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128d __A,`。
- **L1653 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L1653 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L1654 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vcvtsh2sd_round_mask(`.
  **L1654 CN**: 以 `(__m128d)__builtin_ia32_vcvtsh2sd_round_mask(` 从当前函数返回。
- **L1655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)__A, (__v8hf)__B, (__v2df)__W, (__mmask8)__U,`.
  **L1655 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)__A, (__v8hf)__B, (__v2df)__W, (__mmask8)__U,`。
- **L1656 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1656 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。

### Lines 1657-1680

````c
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_maskz_cvtsh_sd(__mmask8 __U, __m128d __A, __m128h __B) {
  return (__m128d)__builtin_ia32_vcvtsh2sd_round_mask(
      (__v2df)__A, (__v8hf)__B, (__v2df)_mm_setzero_pd(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundph_epi16(A, R)                                         \
  ((__m512i)__builtin_ia32_vcvtph2w512_mask((__v32hf)(A),                      \
                                            (__v32hi)_mm512_undefined_epi32(), \
                                            (__mmask32)(-1), (int)(R)))

#define _mm512_mask_cvt_roundph_epi16(W, U, A, R)                              \
  ((__m512i)__builtin_ia32_vcvtph2w512_mask((__v32hf)(A), (__v32hi)(W),        \
                                            (__mmask32)(U), (int)(R)))

#define _mm512_maskz_cvt_roundph_epi16(U, A, R)                                \
  ((__m512i)__builtin_ia32_vcvtph2w512_mask((__v32hf)(A),                      \
                                            (__v32hi)_mm512_setzero_epi32(),   \
                                            (__mmask32)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
````
- **L1657 EN**: Closes the current lexical scope or compound statement.
  **L1657 CN**: 结束当前词法作用域或复合语句块。
- **L1658 EN**: Blank line separating nearby declarations or logic blocks.
  **L1658 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1659 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L1659 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L1660 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtsh_sd(__mmask8 __U, __m128d __A, __m128h __B) {`.
  **L1660 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtsh_sd(__mmask8 __U, __m128d __A, __m128h __B) {`。
- **L1661 EN**: Returns from the current function with `(__m128d)__builtin_ia32_vcvtsh2sd_round_mask(`.
  **L1661 CN**: 以 `(__m128d)__builtin_ia32_vcvtsh2sd_round_mask(` 从当前函数返回。
- **L1662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)__A, (__v8hf)__B, (__v2df)_mm_setzero_pd(), (__mmask8)__U,`.
  **L1662 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)__A, (__v8hf)__B, (__v2df)_mm_setzero_pd(), (__mmask8)__U,`。
- **L1663 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1663 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1664 EN**: Closes the current lexical scope or compound statement.
  **L1664 CN**: 结束当前词法作用域或复合语句块。
- **L1665 EN**: Blank line separating nearby declarations or logic blocks.
  **L1665 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1666 EN**: Defines macro `_mm512_cvt_roundph_epi16(A, R)` for conditional compilation, shorthand, or API generation.
  **L1666 CN**: 定义宏 `_mm512_cvt_roundph_epi16(A, R)`，用于条件编译、简写或 API 生成。
- **L1667 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2w512_mask`.
  **L1667 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2w512_mask` 相关的逻辑。
- **L1668 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L1668 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L1669 EN**: Continues the surrounding expression or declaration: `(__mmask32)(-1), (int)(R)))`.
  **L1669 CN**: 继续构造周围的表达式或声明：`(__mmask32)(-1), (int)(R)))`。
- **L1670 EN**: Blank line separating nearby declarations or logic blocks.
  **L1670 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1671 EN**: Defines macro `_mm512_mask_cvt_roundph_epi16(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1671 CN**: 定义宏 `_mm512_mask_cvt_roundph_epi16(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1672 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2w512_mask`.
  **L1672 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2w512_mask` 相关的逻辑。
- **L1673 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1673 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1674 EN**: Blank line separating nearby declarations or logic blocks.
  **L1674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1675 EN**: Defines macro `_mm512_maskz_cvt_roundph_epi16(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1675 CN**: 定义宏 `_mm512_maskz_cvt_roundph_epi16(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1676 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2w512_mask`.
  **L1676 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2w512_mask` 相关的逻辑。
- **L1677 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L1677 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L1678 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1678 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1679 EN**: Blank line separating nearby declarations or logic blocks.
  **L1679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1680 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1680 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。

### Lines 1681-1704

````c
_mm512_cvtph_epi16(__m512h __A) {
  return (__m512i)__builtin_ia32_vcvtph2w512_mask(
      (__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtph_epi16(__m512i __W, __mmask32 __U, __m512h __A) {
  return (__m512i)__builtin_ia32_vcvtph2w512_mask(
      (__v32hf)__A, (__v32hi)__W, (__mmask32)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtph_epi16(__mmask32 __U, __m512h __A) {
  return (__m512i)__builtin_ia32_vcvtph2w512_mask(
      (__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvtt_roundph_epi16(A, R)                                        \
  ((__m512i)__builtin_ia32_vcvttph2w512_mask(                                  \
      (__v32hf)(A), (__v32hi)_mm512_undefined_epi32(), (__mmask32)(-1),        \
      (int)(R)))

````
- **L1681 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtph_epi16(__m512h __A) {`.
  **L1681 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtph_epi16(__m512h __A) {`。
- **L1682 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2w512_mask(`.
  **L1682 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2w512_mask(` 从当前函数返回。
- **L1683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)-1,`.
  **L1683 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)-1,`。
- **L1684 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1684 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1685 EN**: Closes the current lexical scope or compound statement.
  **L1685 CN**: 结束当前词法作用域或复合语句块。
- **L1686 EN**: Blank line separating nearby declarations or logic blocks.
  **L1686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1687 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1687 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1688 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtph_epi16(__m512i __W, __mmask32 __U, __m512h __A) {`.
  **L1688 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtph_epi16(__m512i __W, __mmask32 __U, __m512h __A) {`。
- **L1689 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2w512_mask(`.
  **L1689 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2w512_mask(` 从当前函数返回。
- **L1690 EN**: Executes a call or declaration centered on `statement`.
  **L1690 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1691 EN**: Closes the current lexical scope or compound statement.
  **L1691 CN**: 结束当前词法作用域或复合语句块。
- **L1692 EN**: Blank line separating nearby declarations or logic blocks.
  **L1692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1693 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1693 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1694 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtph_epi16(__mmask32 __U, __m512h __A) {`.
  **L1694 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtph_epi16(__mmask32 __U, __m512h __A) {`。
- **L1695 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2w512_mask(`.
  **L1695 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2w512_mask(` 从当前函数返回。
- **L1696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)__U,`.
  **L1696 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)__U,`。
- **L1697 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1697 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1698 EN**: Closes the current lexical scope or compound statement.
  **L1698 CN**: 结束当前词法作用域或复合语句块。
- **L1699 EN**: Blank line separating nearby declarations or logic blocks.
  **L1699 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1700 EN**: Defines macro `_mm512_cvtt_roundph_epi16(A, R)` for conditional compilation, shorthand, or API generation.
  **L1700 CN**: 定义宏 `_mm512_cvtt_roundph_epi16(A, R)`，用于条件编译、简写或 API 生成。
- **L1701 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2w512_mask`.
  **L1701 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2w512_mask` 相关的逻辑。
- **L1702 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L1702 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L1703 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L1703 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L1704 EN**: Blank line separating nearby declarations or logic blocks.
  **L1704 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1705-1728

````c
#define _mm512_mask_cvtt_roundph_epi16(W, U, A, R)                             \
  ((__m512i)__builtin_ia32_vcvttph2w512_mask((__v32hf)(A), (__v32hi)(W),       \
                                             (__mmask32)(U), (int)(R)))

#define _mm512_maskz_cvtt_roundph_epi16(U, A, R)                               \
  ((__m512i)__builtin_ia32_vcvttph2w512_mask((__v32hf)(A),                     \
                                             (__v32hi)_mm512_setzero_epi32(),  \
                                             (__mmask32)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttph_epi16(__m512h __A) {
  return (__m512i)__builtin_ia32_vcvttph2w512_mask(
      (__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttph_epi16(__m512i __W, __mmask32 __U, __m512h __A) {
  return (__m512i)__builtin_ia32_vcvttph2w512_mask(
      (__v32hf)__A, (__v32hi)__W, (__mmask32)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttph_epi16(__mmask32 __U, __m512h __A) {
````
- **L1705 EN**: Defines macro `_mm512_mask_cvtt_roundph_epi16(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1705 CN**: 定义宏 `_mm512_mask_cvtt_roundph_epi16(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1706 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2w512_mask`.
  **L1706 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2w512_mask` 相关的逻辑。
- **L1707 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1707 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1708 EN**: Blank line separating nearby declarations or logic blocks.
  **L1708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1709 EN**: Defines macro `_mm512_maskz_cvtt_roundph_epi16(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1709 CN**: 定义宏 `_mm512_maskz_cvtt_roundph_epi16(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1710 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2w512_mask`.
  **L1710 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2w512_mask` 相关的逻辑。
- **L1711 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L1711 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L1712 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1712 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1713 EN**: Blank line separating nearby declarations or logic blocks.
  **L1713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1714 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1714 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1715 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttph_epi16(__m512h __A) {`.
  **L1715 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttph_epi16(__m512h __A) {`。
- **L1716 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2w512_mask(`.
  **L1716 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2w512_mask(` 从当前函数返回。
- **L1717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)-1,`.
  **L1717 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)-1,`。
- **L1718 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1718 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1719 EN**: Closes the current lexical scope or compound statement.
  **L1719 CN**: 结束当前词法作用域或复合语句块。
- **L1720 EN**: Blank line separating nearby declarations or logic blocks.
  **L1720 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1721 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1721 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1722 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttph_epi16(__m512i __W, __mmask32 __U, __m512h __A) {`.
  **L1722 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttph_epi16(__m512i __W, __mmask32 __U, __m512h __A) {`。
- **L1723 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2w512_mask(`.
  **L1723 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2w512_mask(` 从当前函数返回。
- **L1724 EN**: Executes a call or declaration centered on `statement`.
  **L1724 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1725 EN**: Closes the current lexical scope or compound statement.
  **L1725 CN**: 结束当前词法作用域或复合语句块。
- **L1726 EN**: Blank line separating nearby declarations or logic blocks.
  **L1726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1727 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1727 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1728 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttph_epi16(__mmask32 __U, __m512h __A) {`.
  **L1728 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttph_epi16(__mmask32 __U, __m512h __A) {`。

### Lines 1729-1752

````c
  return (__m512i)__builtin_ia32_vcvttph2w512_mask(
      (__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundepi16_ph(A, R)                                         \
  ((__m512h)__builtin_ia32_vcvtw2ph512_mask((__v32hi)(A),                      \
                                            (__v32hf)_mm512_undefined_ph(),    \
                                            (__mmask32)(-1), (int)(R)))

#define _mm512_mask_cvt_roundepi16_ph(W, U, A, R)                              \
  ((__m512h)__builtin_ia32_vcvtw2ph512_mask((__v32hi)(A), (__v32hf)(W),        \
                                            (__mmask32)(U), (int)(R)))

#define _mm512_maskz_cvt_roundepi16_ph(U, A, R)                                \
  ((__m512h)__builtin_ia32_vcvtw2ph512_mask(                                   \
      (__v32hi)(A), (__v32hf)_mm512_setzero_ph(), (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_cvtepi16_ph(__m512i __A) {
  return (__m512h)__builtin_ia32_vcvtw2ph512_mask(
      (__v32hi)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}
````
- **L1729 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2w512_mask(`.
  **L1729 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2w512_mask(` 从当前函数返回。
- **L1730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)__U,`.
  **L1730 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hi)_mm512_setzero_epi32(), (__mmask32)__U,`。
- **L1731 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1731 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1732 EN**: Closes the current lexical scope or compound statement.
  **L1732 CN**: 结束当前词法作用域或复合语句块。
- **L1733 EN**: Blank line separating nearby declarations or logic blocks.
  **L1733 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1734 EN**: Defines macro `_mm512_cvt_roundepi16_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L1734 CN**: 定义宏 `_mm512_cvt_roundepi16_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L1735 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtw2ph512_mask`.
  **L1735 CN**: 继续与可调用符号 `__builtin_ia32_vcvtw2ph512_mask` 相关的逻辑。
- **L1736 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1736 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1737 EN**: Continues the surrounding expression or declaration: `(__mmask32)(-1), (int)(R)))`.
  **L1737 CN**: 继续构造周围的表达式或声明：`(__mmask32)(-1), (int)(R)))`。
- **L1738 EN**: Blank line separating nearby declarations or logic blocks.
  **L1738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1739 EN**: Defines macro `_mm512_mask_cvt_roundepi16_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1739 CN**: 定义宏 `_mm512_mask_cvt_roundepi16_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1740 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtw2ph512_mask`.
  **L1740 CN**: 继续与可调用符号 `__builtin_ia32_vcvtw2ph512_mask` 相关的逻辑。
- **L1741 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1741 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1742 EN**: Blank line separating nearby declarations or logic blocks.
  **L1742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1743 EN**: Defines macro `_mm512_maskz_cvt_roundepi16_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1743 CN**: 定义宏 `_mm512_maskz_cvt_roundepi16_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1744 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtw2ph512_mask`.
  **L1744 CN**: 继续与可调用符号 `__builtin_ia32_vcvtw2ph512_mask` 相关的逻辑。
- **L1745 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1745 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1746 EN**: Blank line separating nearby declarations or logic blocks.
  **L1746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1747 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1747 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1748 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepi16_ph(__m512i __A) {`.
  **L1748 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepi16_ph(__m512i __A) {`。
- **L1749 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vcvtw2ph512_mask(`.
  **L1749 CN**: 以 `(__m512h)__builtin_ia32_vcvtw2ph512_mask(` 从当前函数返回。
- **L1750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hi)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)-1,`.
  **L1750 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hi)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)-1,`。
- **L1751 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1751 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1752 EN**: Closes the current lexical scope or compound statement.
  **L1752 CN**: 结束当前词法作用域或复合语句块。

### Lines 1753-1776

````c

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_cvtepi16_ph(__m512h __W, __mmask32 __U, __m512i __A) {
  return (__m512h)__builtin_ia32_vcvtw2ph512_mask(
      (__v32hi)__A, (__v32hf)__W, (__mmask32)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtepi16_ph(__mmask32 __U, __m512i __A) {
  return (__m512h)__builtin_ia32_vcvtw2ph512_mask(
      (__v32hi)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundph_epu16(A, R)                                         \
  ((__m512i)__builtin_ia32_vcvtph2uw512_mask(                                  \
      (__v32hf)(A), (__v32hu)_mm512_undefined_epi32(), (__mmask32)(-1),        \
      (int)(R)))

#define _mm512_mask_cvt_roundph_epu16(W, U, A, R)                              \
  ((__m512i)__builtin_ia32_vcvtph2uw512_mask((__v32hf)(A), (__v32hu)(W),       \
                                             (__mmask32)(U), (int)(R)))

#define _mm512_maskz_cvt_roundph_epu16(U, A, R)                                \
````
- **L1753 EN**: Blank line separating nearby declarations or logic blocks.
  **L1753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1754 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1754 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1755 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepi16_ph(__m512h __W, __mmask32 __U, __m512i __A) {`.
  **L1755 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepi16_ph(__m512h __W, __mmask32 __U, __m512i __A) {`。
- **L1756 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vcvtw2ph512_mask(`.
  **L1756 CN**: 以 `(__m512h)__builtin_ia32_vcvtw2ph512_mask(` 从当前函数返回。
- **L1757 EN**: Executes a call or declaration centered on `statement`.
  **L1757 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1758 EN**: Closes the current lexical scope or compound statement.
  **L1758 CN**: 结束当前词法作用域或复合语句块。
- **L1759 EN**: Blank line separating nearby declarations or logic blocks.
  **L1759 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1760 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1760 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1761 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepi16_ph(__mmask32 __U, __m512i __A) {`.
  **L1761 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepi16_ph(__mmask32 __U, __m512i __A) {`。
- **L1762 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vcvtw2ph512_mask(`.
  **L1762 CN**: 以 `(__m512h)__builtin_ia32_vcvtw2ph512_mask(` 从当前函数返回。
- **L1763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hi)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,`.
  **L1763 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hi)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,`。
- **L1764 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1764 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1765 EN**: Closes the current lexical scope or compound statement.
  **L1765 CN**: 结束当前词法作用域或复合语句块。
- **L1766 EN**: Blank line separating nearby declarations or logic blocks.
  **L1766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1767 EN**: Defines macro `_mm512_cvt_roundph_epu16(A, R)` for conditional compilation, shorthand, or API generation.
  **L1767 CN**: 定义宏 `_mm512_cvt_roundph_epu16(A, R)`，用于条件编译、简写或 API 生成。
- **L1768 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2uw512_mask`.
  **L1768 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2uw512_mask` 相关的逻辑。
- **L1769 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L1769 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L1770 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L1770 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L1771 EN**: Blank line separating nearby declarations or logic blocks.
  **L1771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1772 EN**: Defines macro `_mm512_mask_cvt_roundph_epu16(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1772 CN**: 定义宏 `_mm512_mask_cvt_roundph_epu16(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1773 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2uw512_mask`.
  **L1773 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2uw512_mask` 相关的逻辑。
- **L1774 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1774 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1775 EN**: Blank line separating nearby declarations or logic blocks.
  **L1775 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1776 EN**: Defines macro `_mm512_maskz_cvt_roundph_epu16(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1776 CN**: 定义宏 `_mm512_maskz_cvt_roundph_epu16(U, A, R)`，用于条件编译、简写或 API 生成。

### Lines 1777-1800

````c
  ((__m512i)__builtin_ia32_vcvtph2uw512_mask((__v32hf)(A),                     \
                                             (__v32hu)_mm512_setzero_epi32(),  \
                                             (__mmask32)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtph_epu16(__m512h __A) {
  return (__m512i)__builtin_ia32_vcvtph2uw512_mask(
      (__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtph_epu16(__m512i __W, __mmask32 __U, __m512h __A) {
  return (__m512i)__builtin_ia32_vcvtph2uw512_mask(
      (__v32hf)__A, (__v32hu)__W, (__mmask32)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtph_epu16(__mmask32 __U, __m512h __A) {
  return (__m512i)__builtin_ia32_vcvtph2uw512_mask(
      (__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

````
- **L1777 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2uw512_mask`.
  **L1777 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2uw512_mask` 相关的逻辑。
- **L1778 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L1778 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L1779 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1779 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1780 EN**: Blank line separating nearby declarations or logic blocks.
  **L1780 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1781 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1781 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1782 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtph_epu16(__m512h __A) {`.
  **L1782 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtph_epu16(__m512h __A) {`。
- **L1783 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2uw512_mask(`.
  **L1783 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2uw512_mask(` 从当前函数返回。
- **L1784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)-1,`.
  **L1784 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)-1,`。
- **L1785 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1785 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1786 EN**: Closes the current lexical scope or compound statement.
  **L1786 CN**: 结束当前词法作用域或复合语句块。
- **L1787 EN**: Blank line separating nearby declarations or logic blocks.
  **L1787 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1788 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1788 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1789 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtph_epu16(__m512i __W, __mmask32 __U, __m512h __A) {`.
  **L1789 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtph_epu16(__m512i __W, __mmask32 __U, __m512h __A) {`。
- **L1790 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2uw512_mask(`.
  **L1790 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2uw512_mask(` 从当前函数返回。
- **L1791 EN**: Executes a call or declaration centered on `statement`.
  **L1791 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1792 EN**: Closes the current lexical scope or compound statement.
  **L1792 CN**: 结束当前词法作用域或复合语句块。
- **L1793 EN**: Blank line separating nearby declarations or logic blocks.
  **L1793 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1794 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1794 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1795 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtph_epu16(__mmask32 __U, __m512h __A) {`.
  **L1795 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtph_epu16(__mmask32 __U, __m512h __A) {`。
- **L1796 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2uw512_mask(`.
  **L1796 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2uw512_mask(` 从当前函数返回。
- **L1797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)__U,`.
  **L1797 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)__U,`。
- **L1798 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1798 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1799 EN**: Closes the current lexical scope or compound statement.
  **L1799 CN**: 结束当前词法作用域或复合语句块。
- **L1800 EN**: Blank line separating nearby declarations or logic blocks.
  **L1800 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1801-1824

````c
#define _mm512_cvtt_roundph_epu16(A, R)                                        \
  ((__m512i)__builtin_ia32_vcvttph2uw512_mask(                                 \
      (__v32hf)(A), (__v32hu)_mm512_undefined_epi32(), (__mmask32)(-1),        \
      (int)(R)))

#define _mm512_mask_cvtt_roundph_epu16(W, U, A, R)                             \
  ((__m512i)__builtin_ia32_vcvttph2uw512_mask((__v32hf)(A), (__v32hu)(W),      \
                                              (__mmask32)(U), (int)(R)))

#define _mm512_maskz_cvtt_roundph_epu16(U, A, R)                               \
  ((__m512i)__builtin_ia32_vcvttph2uw512_mask((__v32hf)(A),                    \
                                              (__v32hu)_mm512_setzero_epi32(), \
                                              (__mmask32)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttph_epu16(__m512h __A) {
  return (__m512i)__builtin_ia32_vcvttph2uw512_mask(
      (__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttph_epu16(__m512i __W, __mmask32 __U, __m512h __A) {
  return (__m512i)__builtin_ia32_vcvttph2uw512_mask(
````
- **L1801 EN**: Defines macro `_mm512_cvtt_roundph_epu16(A, R)` for conditional compilation, shorthand, or API generation.
  **L1801 CN**: 定义宏 `_mm512_cvtt_roundph_epu16(A, R)`，用于条件编译、简写或 API 生成。
- **L1802 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2uw512_mask`.
  **L1802 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2uw512_mask` 相关的逻辑。
- **L1803 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L1803 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L1804 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L1804 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L1805 EN**: Blank line separating nearby declarations or logic blocks.
  **L1805 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1806 EN**: Defines macro `_mm512_mask_cvtt_roundph_epu16(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1806 CN**: 定义宏 `_mm512_mask_cvtt_roundph_epu16(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1807 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2uw512_mask`.
  **L1807 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2uw512_mask` 相关的逻辑。
- **L1808 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1808 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1809 EN**: Blank line separating nearby declarations or logic blocks.
  **L1809 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1810 EN**: Defines macro `_mm512_maskz_cvtt_roundph_epu16(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1810 CN**: 定义宏 `_mm512_maskz_cvtt_roundph_epu16(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1811 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2uw512_mask`.
  **L1811 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2uw512_mask` 相关的逻辑。
- **L1812 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L1812 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L1813 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1813 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1814 EN**: Blank line separating nearby declarations or logic blocks.
  **L1814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1815 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1815 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1816 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttph_epu16(__m512h __A) {`.
  **L1816 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttph_epu16(__m512h __A) {`。
- **L1817 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2uw512_mask(`.
  **L1817 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2uw512_mask(` 从当前函数返回。
- **L1818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)-1,`.
  **L1818 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)-1,`。
- **L1819 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1819 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1820 EN**: Closes the current lexical scope or compound statement.
  **L1820 CN**: 结束当前词法作用域或复合语句块。
- **L1821 EN**: Blank line separating nearby declarations or logic blocks.
  **L1821 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1822 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1822 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1823 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttph_epu16(__m512i __W, __mmask32 __U, __m512h __A) {`.
  **L1823 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttph_epu16(__m512i __W, __mmask32 __U, __m512h __A) {`。
- **L1824 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2uw512_mask(`.
  **L1824 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2uw512_mask(` 从当前函数返回。

### Lines 1825-1848

````c
      (__v32hf)__A, (__v32hu)__W, (__mmask32)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttph_epu16(__mmask32 __U, __m512h __A) {
  return (__m512i)__builtin_ia32_vcvttph2uw512_mask(
      (__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundepu16_ph(A, R)                                         \
  ((__m512h)__builtin_ia32_vcvtuw2ph512_mask((__v32hu)(A),                     \
                                             (__v32hf)_mm512_undefined_ph(),   \
                                             (__mmask32)(-1), (int)(R)))

#define _mm512_mask_cvt_roundepu16_ph(W, U, A, R)                              \
  ((__m512h)__builtin_ia32_vcvtuw2ph512_mask((__v32hu)(A), (__v32hf)(W),       \
                                             (__mmask32)(U), (int)(R)))

#define _mm512_maskz_cvt_roundepu16_ph(U, A, R)                                \
  ((__m512h)__builtin_ia32_vcvtuw2ph512_mask(                                  \
      (__v32hu)(A), (__v32hf)_mm512_setzero_ph(), (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512
````
- **L1825 EN**: Executes a call or declaration centered on `statement`.
  **L1825 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1826 EN**: Closes the current lexical scope or compound statement.
  **L1826 CN**: 结束当前词法作用域或复合语句块。
- **L1827 EN**: Blank line separating nearby declarations or logic blocks.
  **L1827 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1828 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1828 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1829 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttph_epu16(__mmask32 __U, __m512h __A) {`.
  **L1829 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttph_epu16(__mmask32 __U, __m512h __A) {`。
- **L1830 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2uw512_mask(`.
  **L1830 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2uw512_mask(` 从当前函数返回。
- **L1831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)__U,`.
  **L1831 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hu)_mm512_setzero_epi32(), (__mmask32)__U,`。
- **L1832 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1832 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1833 EN**: Closes the current lexical scope or compound statement.
  **L1833 CN**: 结束当前词法作用域或复合语句块。
- **L1834 EN**: Blank line separating nearby declarations or logic blocks.
  **L1834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1835 EN**: Defines macro `_mm512_cvt_roundepu16_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L1835 CN**: 定义宏 `_mm512_cvt_roundepu16_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L1836 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtuw2ph512_mask`.
  **L1836 CN**: 继续与可调用符号 `__builtin_ia32_vcvtuw2ph512_mask` 相关的逻辑。
- **L1837 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L1837 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L1838 EN**: Continues the surrounding expression or declaration: `(__mmask32)(-1), (int)(R)))`.
  **L1838 CN**: 继续构造周围的表达式或声明：`(__mmask32)(-1), (int)(R)))`。
- **L1839 EN**: Blank line separating nearby declarations or logic blocks.
  **L1839 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1840 EN**: Defines macro `_mm512_mask_cvt_roundepu16_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1840 CN**: 定义宏 `_mm512_mask_cvt_roundepu16_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1841 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtuw2ph512_mask`.
  **L1841 CN**: 继续与可调用符号 `__builtin_ia32_vcvtuw2ph512_mask` 相关的逻辑。
- **L1842 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L1842 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L1843 EN**: Blank line separating nearby declarations or logic blocks.
  **L1843 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1844 EN**: Defines macro `_mm512_maskz_cvt_roundepu16_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1844 CN**: 定义宏 `_mm512_maskz_cvt_roundepu16_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1845 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtuw2ph512_mask`.
  **L1845 CN**: 继续与可调用符号 `__builtin_ia32_vcvtuw2ph512_mask` 相关的逻辑。
- **L1846 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L1846 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L1847 EN**: Blank line separating nearby declarations or logic blocks.
  **L1847 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1848 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1848 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。

### Lines 1849-1872

````c
_mm512_cvtepu16_ph(__m512i __A) {
  return (__m512h)__builtin_ia32_vcvtuw2ph512_mask(
      (__v32hu)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_cvtepu16_ph(__m512h __W, __mmask32 __U, __m512i __A) {
  return (__m512h)__builtin_ia32_vcvtuw2ph512_mask(
      (__v32hu)__A, (__v32hf)__W, (__mmask32)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtepu16_ph(__mmask32 __U, __m512i __A) {
  return (__m512h)__builtin_ia32_vcvtuw2ph512_mask(
      (__v32hu)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundph_epi32(A, R)                                         \
  ((__m512i)__builtin_ia32_vcvtph2dq512_mask(                                  \
      (__v16hf)(A), (__v16si)_mm512_undefined_epi32(), (__mmask16)(-1),        \
      (int)(R)))

````
- **L1849 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepu16_ph(__m512i __A) {`.
  **L1849 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepu16_ph(__m512i __A) {`。
- **L1850 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vcvtuw2ph512_mask(`.
  **L1850 CN**: 以 `(__m512h)__builtin_ia32_vcvtuw2ph512_mask(` 从当前函数返回。
- **L1851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hu)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)-1,`.
  **L1851 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hu)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)-1,`。
- **L1852 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1852 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1853 EN**: Closes the current lexical scope or compound statement.
  **L1853 CN**: 结束当前词法作用域或复合语句块。
- **L1854 EN**: Blank line separating nearby declarations or logic blocks.
  **L1854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1855 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1855 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1856 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepu16_ph(__m512h __W, __mmask32 __U, __m512i __A) {`.
  **L1856 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepu16_ph(__m512h __W, __mmask32 __U, __m512i __A) {`。
- **L1857 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vcvtuw2ph512_mask(`.
  **L1857 CN**: 以 `(__m512h)__builtin_ia32_vcvtuw2ph512_mask(` 从当前函数返回。
- **L1858 EN**: Executes a call or declaration centered on `statement`.
  **L1858 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1859 EN**: Closes the current lexical scope or compound statement.
  **L1859 CN**: 结束当前词法作用域或复合语句块。
- **L1860 EN**: Blank line separating nearby declarations or logic blocks.
  **L1860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1861 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L1861 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L1862 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepu16_ph(__mmask32 __U, __m512i __A) {`.
  **L1862 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepu16_ph(__mmask32 __U, __m512i __A) {`。
- **L1863 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vcvtuw2ph512_mask(`.
  **L1863 CN**: 以 `(__m512h)__builtin_ia32_vcvtuw2ph512_mask(` 从当前函数返回。
- **L1864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hu)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,`.
  **L1864 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hu)__A, (__v32hf)_mm512_setzero_ph(), (__mmask32)__U,`。
- **L1865 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1865 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1866 EN**: Closes the current lexical scope or compound statement.
  **L1866 CN**: 结束当前词法作用域或复合语句块。
- **L1867 EN**: Blank line separating nearby declarations or logic blocks.
  **L1867 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1868 EN**: Defines macro `_mm512_cvt_roundph_epi32(A, R)` for conditional compilation, shorthand, or API generation.
  **L1868 CN**: 定义宏 `_mm512_cvt_roundph_epi32(A, R)`，用于条件编译、简写或 API 生成。
- **L1869 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2dq512_mask`.
  **L1869 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2dq512_mask` 相关的逻辑。
- **L1870 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L1870 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L1871 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L1871 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L1872 EN**: Blank line separating nearby declarations or logic blocks.
  **L1872 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1873-1896

````c
#define _mm512_mask_cvt_roundph_epi32(W, U, A, R)                              \
  ((__m512i)__builtin_ia32_vcvtph2dq512_mask((__v16hf)(A), (__v16si)(W),       \
                                             (__mmask16)(U), (int)(R)))

#define _mm512_maskz_cvt_roundph_epi32(U, A, R)                                \
  ((__m512i)__builtin_ia32_vcvtph2dq512_mask((__v16hf)(A),                     \
                                             (__v16si)_mm512_setzero_epi32(),  \
                                             (__mmask16)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtph_epi32(__m256h __A) {
  return (__m512i)__builtin_ia32_vcvtph2dq512_mask(
      (__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtph_epi32(__m512i __W, __mmask16 __U, __m256h __A) {
  return (__m512i)__builtin_ia32_vcvtph2dq512_mask(
      (__v16hf)__A, (__v16si)__W, (__mmask16)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtph_epi32(__mmask16 __U, __m256h __A) {
````
- **L1873 EN**: Defines macro `_mm512_mask_cvt_roundph_epi32(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1873 CN**: 定义宏 `_mm512_mask_cvt_roundph_epi32(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1874 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2dq512_mask`.
  **L1874 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2dq512_mask` 相关的逻辑。
- **L1875 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L1875 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L1876 EN**: Blank line separating nearby declarations or logic blocks.
  **L1876 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1877 EN**: Defines macro `_mm512_maskz_cvt_roundph_epi32(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1877 CN**: 定义宏 `_mm512_maskz_cvt_roundph_epi32(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1878 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2dq512_mask`.
  **L1878 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2dq512_mask` 相关的逻辑。
- **L1879 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L1879 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L1880 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L1880 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L1881 EN**: Blank line separating nearby declarations or logic blocks.
  **L1881 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1882 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1882 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1883 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtph_epi32(__m256h __A) {`.
  **L1883 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtph_epi32(__m256h __A) {`。
- **L1884 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2dq512_mask(`.
  **L1884 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2dq512_mask(` 从当前函数返回。
- **L1885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)-1,`.
  **L1885 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)-1,`。
- **L1886 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1886 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1887 EN**: Closes the current lexical scope or compound statement.
  **L1887 CN**: 结束当前词法作用域或复合语句块。
- **L1888 EN**: Blank line separating nearby declarations or logic blocks.
  **L1888 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1889 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1889 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1890 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtph_epi32(__m512i __W, __mmask16 __U, __m256h __A) {`.
  **L1890 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtph_epi32(__m512i __W, __mmask16 __U, __m256h __A) {`。
- **L1891 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2dq512_mask(`.
  **L1891 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2dq512_mask(` 从当前函数返回。
- **L1892 EN**: Executes a call or declaration centered on `statement`.
  **L1892 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1893 EN**: Closes the current lexical scope or compound statement.
  **L1893 CN**: 结束当前词法作用域或复合语句块。
- **L1894 EN**: Blank line separating nearby declarations or logic blocks.
  **L1894 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1895 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1895 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1896 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtph_epi32(__mmask16 __U, __m256h __A) {`.
  **L1896 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtph_epi32(__mmask16 __U, __m256h __A) {`。

### Lines 1897-1920

````c
  return (__m512i)__builtin_ia32_vcvtph2dq512_mask(
      (__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundph_epu32(A, R)                                         \
  ((__m512i)__builtin_ia32_vcvtph2udq512_mask(                                 \
      (__v16hf)(A), (__v16su)_mm512_undefined_epi32(), (__mmask16)(-1),        \
      (int)(R)))

#define _mm512_mask_cvt_roundph_epu32(W, U, A, R)                              \
  ((__m512i)__builtin_ia32_vcvtph2udq512_mask((__v16hf)(A), (__v16su)(W),      \
                                              (__mmask16)(U), (int)(R)))

#define _mm512_maskz_cvt_roundph_epu32(U, A, R)                                \
  ((__m512i)__builtin_ia32_vcvtph2udq512_mask((__v16hf)(A),                    \
                                              (__v16su)_mm512_setzero_epi32(), \
                                              (__mmask16)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtph_epu32(__m256h __A) {
  return (__m512i)__builtin_ia32_vcvtph2udq512_mask(
      (__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
````
- **L1897 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2dq512_mask(`.
  **L1897 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2dq512_mask(` 从当前函数返回。
- **L1898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)__U,`.
  **L1898 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)__U,`。
- **L1899 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1899 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1900 EN**: Closes the current lexical scope or compound statement.
  **L1900 CN**: 结束当前词法作用域或复合语句块。
- **L1901 EN**: Blank line separating nearby declarations or logic blocks.
  **L1901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1902 EN**: Defines macro `_mm512_cvt_roundph_epu32(A, R)` for conditional compilation, shorthand, or API generation.
  **L1902 CN**: 定义宏 `_mm512_cvt_roundph_epu32(A, R)`，用于条件编译、简写或 API 生成。
- **L1903 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2udq512_mask`.
  **L1903 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2udq512_mask` 相关的逻辑。
- **L1904 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L1904 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L1905 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L1905 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L1906 EN**: Blank line separating nearby declarations or logic blocks.
  **L1906 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1907 EN**: Defines macro `_mm512_mask_cvt_roundph_epu32(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1907 CN**: 定义宏 `_mm512_mask_cvt_roundph_epu32(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1908 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2udq512_mask`.
  **L1908 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2udq512_mask` 相关的逻辑。
- **L1909 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L1909 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L1910 EN**: Blank line separating nearby declarations or logic blocks.
  **L1910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1911 EN**: Defines macro `_mm512_maskz_cvt_roundph_epu32(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1911 CN**: 定义宏 `_mm512_maskz_cvt_roundph_epu32(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1912 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2udq512_mask`.
  **L1912 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2udq512_mask` 相关的逻辑。
- **L1913 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L1913 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L1914 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L1914 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L1915 EN**: Blank line separating nearby declarations or logic blocks.
  **L1915 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1916 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1916 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1917 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtph_epu32(__m256h __A) {`.
  **L1917 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtph_epu32(__m256h __A) {`。
- **L1918 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2udq512_mask(`.
  **L1918 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2udq512_mask(` 从当前函数返回。
- **L1919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)-1,`.
  **L1919 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)-1,`。
- **L1920 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1920 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。

### Lines 1921-1944

````c
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtph_epu32(__m512i __W, __mmask16 __U, __m256h __A) {
  return (__m512i)__builtin_ia32_vcvtph2udq512_mask(
      (__v16hf)__A, (__v16su)__W, (__mmask16)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtph_epu32(__mmask16 __U, __m256h __A) {
  return (__m512i)__builtin_ia32_vcvtph2udq512_mask(
      (__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundepi32_ph(A, R)                                         \
  ((__m256h)__builtin_ia32_vcvtdq2ph512_mask((__v16si)(A),                     \
                                             (__v16hf)_mm256_undefined_ph(),   \
                                             (__mmask16)(-1), (int)(R)))

#define _mm512_mask_cvt_roundepi32_ph(W, U, A, R)                              \
  ((__m256h)__builtin_ia32_vcvtdq2ph512_mask((__v16si)(A), (__v16hf)(W),       \
                                             (__mmask16)(U), (int)(R)))

````
- **L1921 EN**: Closes the current lexical scope or compound statement.
  **L1921 CN**: 结束当前词法作用域或复合语句块。
- **L1922 EN**: Blank line separating nearby declarations or logic blocks.
  **L1922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1923 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1923 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1924 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtph_epu32(__m512i __W, __mmask16 __U, __m256h __A) {`.
  **L1924 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtph_epu32(__m512i __W, __mmask16 __U, __m256h __A) {`。
- **L1925 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2udq512_mask(`.
  **L1925 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2udq512_mask(` 从当前函数返回。
- **L1926 EN**: Executes a call or declaration centered on `statement`.
  **L1926 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1927 EN**: Closes the current lexical scope or compound statement.
  **L1927 CN**: 结束当前词法作用域或复合语句块。
- **L1928 EN**: Blank line separating nearby declarations or logic blocks.
  **L1928 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1929 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L1929 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L1930 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtph_epu32(__mmask16 __U, __m256h __A) {`.
  **L1930 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtph_epu32(__mmask16 __U, __m256h __A) {`。
- **L1931 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2udq512_mask(`.
  **L1931 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2udq512_mask(` 从当前函数返回。
- **L1932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)__U,`.
  **L1932 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)__U,`。
- **L1933 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1933 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1934 EN**: Closes the current lexical scope or compound statement.
  **L1934 CN**: 结束当前词法作用域或复合语句块。
- **L1935 EN**: Blank line separating nearby declarations or logic blocks.
  **L1935 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1936 EN**: Defines macro `_mm512_cvt_roundepi32_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L1936 CN**: 定义宏 `_mm512_cvt_roundepi32_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L1937 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtdq2ph512_mask`.
  **L1937 CN**: 继续与可调用符号 `__builtin_ia32_vcvtdq2ph512_mask` 相关的逻辑。
- **L1938 EN**: Continues logic associated with callable symbol `_mm256_undefined_ph`.
  **L1938 CN**: 继续与可调用符号 `_mm256_undefined_ph` 相关的逻辑。
- **L1939 EN**: Continues the surrounding expression or declaration: `(__mmask16)(-1), (int)(R)))`.
  **L1939 CN**: 继续构造周围的表达式或声明：`(__mmask16)(-1), (int)(R)))`。
- **L1940 EN**: Blank line separating nearby declarations or logic blocks.
  **L1940 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1941 EN**: Defines macro `_mm512_mask_cvt_roundepi32_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1941 CN**: 定义宏 `_mm512_mask_cvt_roundepi32_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1942 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtdq2ph512_mask`.
  **L1942 CN**: 继续与可调用符号 `__builtin_ia32_vcvtdq2ph512_mask` 相关的逻辑。
- **L1943 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L1943 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L1944 EN**: Blank line separating nearby declarations or logic blocks.
  **L1944 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1945-1968

````c
#define _mm512_maskz_cvt_roundepi32_ph(U, A, R)                                \
  ((__m256h)__builtin_ia32_vcvtdq2ph512_mask(                                  \
      (__v16si)(A), (__v16hf)_mm256_setzero_ph(), (__mmask16)(U), (int)(R)))

static __inline__ __m256h __DEFAULT_FN_ATTRS512
_mm512_cvtepi32_ph(__m512i __A) {
  return (__m256h)__builtin_ia32_vcvtdq2ph512_mask(
      (__v16si)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS512
_mm512_mask_cvtepi32_ph(__m256h __W, __mmask16 __U, __m512i __A) {
  return (__m256h)__builtin_ia32_vcvtdq2ph512_mask(
      (__v16si)__A, (__v16hf)__W, (__mmask16)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtepi32_ph(__mmask16 __U, __m512i __A) {
  return (__m256h)__builtin_ia32_vcvtdq2ph512_mask(
      (__v16si)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

````
- **L1945 EN**: Defines macro `_mm512_maskz_cvt_roundepi32_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1945 CN**: 定义宏 `_mm512_maskz_cvt_roundepi32_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1946 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtdq2ph512_mask`.
  **L1946 CN**: 继续与可调用符号 `__builtin_ia32_vcvtdq2ph512_mask` 相关的逻辑。
- **L1947 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L1947 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L1948 EN**: Blank line separating nearby declarations or logic blocks.
  **L1948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1949 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512`.
  **L1949 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512`。
- **L1950 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepi32_ph(__m512i __A) {`.
  **L1950 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepi32_ph(__m512i __A) {`。
- **L1951 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtdq2ph512_mask(`.
  **L1951 CN**: 以 `(__m256h)__builtin_ia32_vcvtdq2ph512_mask(` 从当前函数返回。
- **L1952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16si)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,`.
  **L1952 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16si)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,`。
- **L1953 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1953 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1954 EN**: Closes the current lexical scope or compound statement.
  **L1954 CN**: 结束当前词法作用域或复合语句块。
- **L1955 EN**: Blank line separating nearby declarations or logic blocks.
  **L1955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1956 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512`.
  **L1956 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512`。
- **L1957 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepi32_ph(__m256h __W, __mmask16 __U, __m512i __A) {`.
  **L1957 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepi32_ph(__m256h __W, __mmask16 __U, __m512i __A) {`。
- **L1958 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtdq2ph512_mask(`.
  **L1958 CN**: 以 `(__m256h)__builtin_ia32_vcvtdq2ph512_mask(` 从当前函数返回。
- **L1959 EN**: Executes a call or declaration centered on `statement`.
  **L1959 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1960 EN**: Closes the current lexical scope or compound statement.
  **L1960 CN**: 结束当前词法作用域或复合语句块。
- **L1961 EN**: Blank line separating nearby declarations or logic blocks.
  **L1961 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1962 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512`.
  **L1962 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512`。
- **L1963 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepi32_ph(__mmask16 __U, __m512i __A) {`.
  **L1963 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepi32_ph(__mmask16 __U, __m512i __A) {`。
- **L1964 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtdq2ph512_mask(`.
  **L1964 CN**: 以 `(__m256h)__builtin_ia32_vcvtdq2ph512_mask(` 从当前函数返回。
- **L1965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16si)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,`.
  **L1965 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16si)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,`。
- **L1966 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1966 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1967 EN**: Closes the current lexical scope or compound statement.
  **L1967 CN**: 结束当前词法作用域或复合语句块。
- **L1968 EN**: Blank line separating nearby declarations or logic blocks.
  **L1968 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1969-1992

````c
#define _mm512_cvt_roundepu32_ph(A, R)                                         \
  ((__m256h)__builtin_ia32_vcvtudq2ph512_mask((__v16su)(A),                    \
                                              (__v16hf)_mm256_undefined_ph(),  \
                                              (__mmask16)(-1), (int)(R)))

#define _mm512_mask_cvt_roundepu32_ph(W, U, A, R)                              \
  ((__m256h)__builtin_ia32_vcvtudq2ph512_mask((__v16su)(A), (__v16hf)(W),      \
                                              (__mmask16)(U), (int)(R)))

#define _mm512_maskz_cvt_roundepu32_ph(U, A, R)                                \
  ((__m256h)__builtin_ia32_vcvtudq2ph512_mask(                                 \
      (__v16su)(A), (__v16hf)_mm256_setzero_ph(), (__mmask16)(U), (int)(R)))

static __inline__ __m256h __DEFAULT_FN_ATTRS512
_mm512_cvtepu32_ph(__m512i __A) {
  return (__m256h)__builtin_ia32_vcvtudq2ph512_mask(
      (__v16su)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS512
_mm512_mask_cvtepu32_ph(__m256h __W, __mmask16 __U, __m512i __A) {
  return (__m256h)__builtin_ia32_vcvtudq2ph512_mask(
      (__v16su)__A, (__v16hf)__W, (__mmask16)__U, _MM_FROUND_CUR_DIRECTION);
````
- **L1969 EN**: Defines macro `_mm512_cvt_roundepu32_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L1969 CN**: 定义宏 `_mm512_cvt_roundepu32_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L1970 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtudq2ph512_mask`.
  **L1970 CN**: 继续与可调用符号 `__builtin_ia32_vcvtudq2ph512_mask` 相关的逻辑。
- **L1971 EN**: Continues logic associated with callable symbol `_mm256_undefined_ph`.
  **L1971 CN**: 继续与可调用符号 `_mm256_undefined_ph` 相关的逻辑。
- **L1972 EN**: Continues the surrounding expression or declaration: `(__mmask16)(-1), (int)(R)))`.
  **L1972 CN**: 继续构造周围的表达式或声明：`(__mmask16)(-1), (int)(R)))`。
- **L1973 EN**: Blank line separating nearby declarations or logic blocks.
  **L1973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1974 EN**: Defines macro `_mm512_mask_cvt_roundepu32_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1974 CN**: 定义宏 `_mm512_mask_cvt_roundepu32_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L1975 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtudq2ph512_mask`.
  **L1975 CN**: 继续与可调用符号 `__builtin_ia32_vcvtudq2ph512_mask` 相关的逻辑。
- **L1976 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L1976 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L1977 EN**: Blank line separating nearby declarations or logic blocks.
  **L1977 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1978 EN**: Defines macro `_mm512_maskz_cvt_roundepu32_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L1978 CN**: 定义宏 `_mm512_maskz_cvt_roundepu32_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L1979 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtudq2ph512_mask`.
  **L1979 CN**: 继续与可调用符号 `__builtin_ia32_vcvtudq2ph512_mask` 相关的逻辑。
- **L1980 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L1980 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L1981 EN**: Blank line separating nearby declarations or logic blocks.
  **L1981 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1982 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512`.
  **L1982 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512`。
- **L1983 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepu32_ph(__m512i __A) {`.
  **L1983 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepu32_ph(__m512i __A) {`。
- **L1984 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtudq2ph512_mask(`.
  **L1984 CN**: 以 `(__m256h)__builtin_ia32_vcvtudq2ph512_mask(` 从当前函数返回。
- **L1985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16su)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,`.
  **L1985 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16su)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,`。
- **L1986 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1986 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L1987 EN**: Closes the current lexical scope or compound statement.
  **L1987 CN**: 结束当前词法作用域或复合语句块。
- **L1988 EN**: Blank line separating nearby declarations or logic blocks.
  **L1988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1989 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512`.
  **L1989 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512`。
- **L1990 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepu32_ph(__m256h __W, __mmask16 __U, __m512i __A) {`.
  **L1990 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepu32_ph(__m256h __W, __mmask16 __U, __m512i __A) {`。
- **L1991 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtudq2ph512_mask(`.
  **L1991 CN**: 以 `(__m256h)__builtin_ia32_vcvtudq2ph512_mask(` 从当前函数返回。
- **L1992 EN**: Executes a call or declaration centered on `statement`.
  **L1992 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1993-2016

````c
}

static __inline__ __m256h __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtepu32_ph(__mmask16 __U, __m512i __A) {
  return (__m256h)__builtin_ia32_vcvtudq2ph512_mask(
      (__v16su)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvtt_roundph_epi32(A, R)                                        \
  ((__m512i)__builtin_ia32_vcvttph2dq512_mask(                                 \
      (__v16hf)(A), (__v16si)_mm512_undefined_epi32(), (__mmask16)(-1),        \
      (int)(R)))

#define _mm512_mask_cvtt_roundph_epi32(W, U, A, R)                             \
  ((__m512i)__builtin_ia32_vcvttph2dq512_mask((__v16hf)(A), (__v16si)(W),      \
                                              (__mmask16)(U), (int)(R)))

#define _mm512_maskz_cvtt_roundph_epi32(U, A, R)                               \
  ((__m512i)__builtin_ia32_vcvttph2dq512_mask((__v16hf)(A),                    \
                                              (__v16si)_mm512_setzero_epi32(), \
                                              (__mmask16)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
````
- **L1993 EN**: Closes the current lexical scope or compound statement.
  **L1993 CN**: 结束当前词法作用域或复合语句块。
- **L1994 EN**: Blank line separating nearby declarations or logic blocks.
  **L1994 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1995 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512`.
  **L1995 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512`。
- **L1996 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepu32_ph(__mmask16 __U, __m512i __A) {`.
  **L1996 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepu32_ph(__mmask16 __U, __m512i __A) {`。
- **L1997 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtudq2ph512_mask(`.
  **L1997 CN**: 以 `(__m256h)__builtin_ia32_vcvtudq2ph512_mask(` 从当前函数返回。
- **L1998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16su)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,`.
  **L1998 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16su)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,`。
- **L1999 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L1999 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2000 EN**: Closes the current lexical scope or compound statement.
  **L2000 CN**: 结束当前词法作用域或复合语句块。
- **L2001 EN**: Blank line separating nearby declarations or logic blocks.
  **L2001 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2002 EN**: Defines macro `_mm512_cvtt_roundph_epi32(A, R)` for conditional compilation, shorthand, or API generation.
  **L2002 CN**: 定义宏 `_mm512_cvtt_roundph_epi32(A, R)`，用于条件编译、简写或 API 生成。
- **L2003 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2dq512_mask`.
  **L2003 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2dq512_mask` 相关的逻辑。
- **L2004 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L2004 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L2005 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L2005 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L2006 EN**: Blank line separating nearby declarations or logic blocks.
  **L2006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2007 EN**: Defines macro `_mm512_mask_cvtt_roundph_epi32(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2007 CN**: 定义宏 `_mm512_mask_cvtt_roundph_epi32(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2008 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2dq512_mask`.
  **L2008 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2dq512_mask` 相关的逻辑。
- **L2009 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L2009 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L2010 EN**: Blank line separating nearby declarations or logic blocks.
  **L2010 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2011 EN**: Defines macro `_mm512_maskz_cvtt_roundph_epi32(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2011 CN**: 定义宏 `_mm512_maskz_cvtt_roundph_epi32(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2012 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2dq512_mask`.
  **L2012 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2dq512_mask` 相关的逻辑。
- **L2013 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L2013 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L2014 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L2014 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L2015 EN**: Blank line separating nearby declarations or logic blocks.
  **L2015 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2016 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2016 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。

### Lines 2017-2040

````c
_mm512_cvttph_epi32(__m256h __A) {
  return (__m512i)__builtin_ia32_vcvttph2dq512_mask(
      (__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttph_epi32(__m512i __W, __mmask16 __U, __m256h __A) {
  return (__m512i)__builtin_ia32_vcvttph2dq512_mask(
      (__v16hf)__A, (__v16si)__W, (__mmask16)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttph_epi32(__mmask16 __U, __m256h __A) {
  return (__m512i)__builtin_ia32_vcvttph2dq512_mask(
      (__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvtt_roundph_epu32(A, R)                                        \
  ((__m512i)__builtin_ia32_vcvttph2udq512_mask(                                \
      (__v16hf)(A), (__v16su)_mm512_undefined_epi32(), (__mmask16)(-1),        \
      (int)(R)))

````
- **L2017 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttph_epi32(__m256h __A) {`.
  **L2017 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttph_epi32(__m256h __A) {`。
- **L2018 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2dq512_mask(`.
  **L2018 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2dq512_mask(` 从当前函数返回。
- **L2019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)-1,`.
  **L2019 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)-1,`。
- **L2020 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2020 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2021 EN**: Closes the current lexical scope or compound statement.
  **L2021 CN**: 结束当前词法作用域或复合语句块。
- **L2022 EN**: Blank line separating nearby declarations or logic blocks.
  **L2022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2023 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2023 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2024 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttph_epi32(__m512i __W, __mmask16 __U, __m256h __A) {`.
  **L2024 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttph_epi32(__m512i __W, __mmask16 __U, __m256h __A) {`。
- **L2025 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2dq512_mask(`.
  **L2025 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2dq512_mask(` 从当前函数返回。
- **L2026 EN**: Executes a call or declaration centered on `statement`.
  **L2026 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2027 EN**: Closes the current lexical scope or compound statement.
  **L2027 CN**: 结束当前词法作用域或复合语句块。
- **L2028 EN**: Blank line separating nearby declarations or logic blocks.
  **L2028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2029 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2029 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2030 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttph_epi32(__mmask16 __U, __m256h __A) {`.
  **L2030 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttph_epi32(__mmask16 __U, __m256h __A) {`。
- **L2031 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2dq512_mask(`.
  **L2031 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2dq512_mask(` 从当前函数返回。
- **L2032 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)__U,`.
  **L2032 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16si)_mm512_setzero_epi32(), (__mmask16)__U,`。
- **L2033 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2033 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2034 EN**: Closes the current lexical scope or compound statement.
  **L2034 CN**: 结束当前词法作用域或复合语句块。
- **L2035 EN**: Blank line separating nearby declarations or logic blocks.
  **L2035 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2036 EN**: Defines macro `_mm512_cvtt_roundph_epu32(A, R)` for conditional compilation, shorthand, or API generation.
  **L2036 CN**: 定义宏 `_mm512_cvtt_roundph_epu32(A, R)`，用于条件编译、简写或 API 生成。
- **L2037 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2udq512_mask`.
  **L2037 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2udq512_mask` 相关的逻辑。
- **L2038 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L2038 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L2039 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L2039 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L2040 EN**: Blank line separating nearby declarations or logic blocks.
  **L2040 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2041-2064

````c
#define _mm512_mask_cvtt_roundph_epu32(W, U, A, R)                             \
  ((__m512i)__builtin_ia32_vcvttph2udq512_mask((__v16hf)(A), (__v16su)(W),     \
                                               (__mmask16)(U), (int)(R)))

#define _mm512_maskz_cvtt_roundph_epu32(U, A, R)                               \
  ((__m512i)__builtin_ia32_vcvttph2udq512_mask(                                \
      (__v16hf)(A), (__v16su)_mm512_setzero_epi32(), (__mmask16)(U),           \
      (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttph_epu32(__m256h __A) {
  return (__m512i)__builtin_ia32_vcvttph2udq512_mask(
      (__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttph_epu32(__m512i __W, __mmask16 __U, __m256h __A) {
  return (__m512i)__builtin_ia32_vcvttph2udq512_mask(
      (__v16hf)__A, (__v16su)__W, (__mmask16)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttph_epu32(__mmask16 __U, __m256h __A) {
````
- **L2041 EN**: Defines macro `_mm512_mask_cvtt_roundph_epu32(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2041 CN**: 定义宏 `_mm512_mask_cvtt_roundph_epu32(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2042 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2udq512_mask`.
  **L2042 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2udq512_mask` 相关的逻辑。
- **L2043 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L2043 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L2044 EN**: Blank line separating nearby declarations or logic blocks.
  **L2044 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2045 EN**: Defines macro `_mm512_maskz_cvtt_roundph_epu32(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2045 CN**: 定义宏 `_mm512_maskz_cvtt_roundph_epu32(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2046 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2udq512_mask`.
  **L2046 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2udq512_mask` 相关的逻辑。
- **L2047 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L2047 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L2048 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L2048 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L2049 EN**: Blank line separating nearby declarations or logic blocks.
  **L2049 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2050 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2050 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2051 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttph_epu32(__m256h __A) {`.
  **L2051 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttph_epu32(__m256h __A) {`。
- **L2052 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2udq512_mask(`.
  **L2052 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2udq512_mask(` 从当前函数返回。
- **L2053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)-1,`.
  **L2053 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)-1,`。
- **L2054 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2054 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2055 EN**: Closes the current lexical scope or compound statement.
  **L2055 CN**: 结束当前词法作用域或复合语句块。
- **L2056 EN**: Blank line separating nearby declarations or logic blocks.
  **L2056 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2057 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2057 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2058 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttph_epu32(__m512i __W, __mmask16 __U, __m256h __A) {`.
  **L2058 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttph_epu32(__m512i __W, __mmask16 __U, __m256h __A) {`。
- **L2059 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2udq512_mask(`.
  **L2059 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2udq512_mask(` 从当前函数返回。
- **L2060 EN**: Executes a call or declaration centered on `statement`.
  **L2060 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2061 EN**: Closes the current lexical scope or compound statement.
  **L2061 CN**: 结束当前词法作用域或复合语句块。
- **L2062 EN**: Blank line separating nearby declarations or logic blocks.
  **L2062 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2063 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2063 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2064 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttph_epu32(__mmask16 __U, __m256h __A) {`.
  **L2064 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttph_epu32(__mmask16 __U, __m256h __A) {`。

### Lines 2065-2088

````c
  return (__m512i)__builtin_ia32_vcvttph2udq512_mask(
      (__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundepi64_ph(A, R)                                         \
  ((__m128h)__builtin_ia32_vcvtqq2ph512_mask(                                  \
      (__v8di)(A), (__v8hf)_mm_undefined_ph(), (__mmask8)(-1), (int)(R)))

#define _mm512_mask_cvt_roundepi64_ph(W, U, A, R)                              \
  ((__m128h)__builtin_ia32_vcvtqq2ph512_mask((__v8di)(A), (__v8hf)(W),         \
                                             (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundepi64_ph(U, A, R)                                \
  ((__m128h)__builtin_ia32_vcvtqq2ph512_mask(                                  \
      (__v8di)(A), (__v8hf)_mm_setzero_ph(), (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS512
_mm512_cvtepi64_ph(__m512i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph512_mask(
      (__v8di)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

````
- **L2065 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2udq512_mask(`.
  **L2065 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2udq512_mask(` 从当前函数返回。
- **L2066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)__U,`.
  **L2066 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16su)_mm512_setzero_epi32(), (__mmask16)__U,`。
- **L2067 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2067 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2068 EN**: Closes the current lexical scope or compound statement.
  **L2068 CN**: 结束当前词法作用域或复合语句块。
- **L2069 EN**: Blank line separating nearby declarations or logic blocks.
  **L2069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2070 EN**: Defines macro `_mm512_cvt_roundepi64_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L2070 CN**: 定义宏 `_mm512_cvt_roundepi64_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L2071 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtqq2ph512_mask`.
  **L2071 CN**: 继续与可调用符号 `__builtin_ia32_vcvtqq2ph512_mask` 相关的逻辑。
- **L2072 EN**: Continues logic associated with callable symbol `_mm_undefined_ph`.
  **L2072 CN**: 继续与可调用符号 `_mm_undefined_ph` 相关的逻辑。
- **L2073 EN**: Blank line separating nearby declarations or logic blocks.
  **L2073 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2074 EN**: Defines macro `_mm512_mask_cvt_roundepi64_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2074 CN**: 定义宏 `_mm512_mask_cvt_roundepi64_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2075 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtqq2ph512_mask`.
  **L2075 CN**: 继续与可调用符号 `__builtin_ia32_vcvtqq2ph512_mask` 相关的逻辑。
- **L2076 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2076 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2077 EN**: Blank line separating nearby declarations or logic blocks.
  **L2077 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2078 EN**: Defines macro `_mm512_maskz_cvt_roundepi64_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2078 CN**: 定义宏 `_mm512_maskz_cvt_roundepi64_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2079 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtqq2ph512_mask`.
  **L2079 CN**: 继续与可调用符号 `__builtin_ia32_vcvtqq2ph512_mask` 相关的逻辑。
- **L2080 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L2080 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L2081 EN**: Blank line separating nearby declarations or logic blocks.
  **L2081 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2082 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512`.
  **L2082 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512`。
- **L2083 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepi64_ph(__m512i __A) {`.
  **L2083 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepi64_ph(__m512i __A) {`。
- **L2084 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph512_mask(`.
  **L2084 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph512_mask(` 从当前函数返回。
- **L2085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`.
  **L2085 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`。
- **L2086 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2086 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2087 EN**: Closes the current lexical scope or compound statement.
  **L2087 CN**: 结束当前词法作用域或复合语句块。
- **L2088 EN**: Blank line separating nearby declarations or logic blocks.
  **L2088 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2089-2112

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS512
_mm512_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m512i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph512_mask(
      (__v8di)__A, (__v8hf)__W, (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtepi64_ph(__mmask8 __U, __m512i __A) {
  return (__m128h)__builtin_ia32_vcvtqq2ph512_mask(
      (__v8di)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundph_epi64(A, R)                                         \
  ((__m512i)__builtin_ia32_vcvtph2qq512_mask((__v8hf)(A),                      \
                                             (__v8di)_mm512_undefined_epi32(), \
                                             (__mmask8)(-1), (int)(R)))

#define _mm512_mask_cvt_roundph_epi64(W, U, A, R)                              \
  ((__m512i)__builtin_ia32_vcvtph2qq512_mask((__v8hf)(A), (__v8di)(W),         \
                                             (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundph_epi64(U, A, R)                                \
  ((__m512i)__builtin_ia32_vcvtph2qq512_mask(                                  \
````
- **L2089 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512`.
  **L2089 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512`。
- **L2090 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m512i __A) {`.
  **L2090 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepi64_ph(__m128h __W, __mmask8 __U, __m512i __A) {`。
- **L2091 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph512_mask(`.
  **L2091 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph512_mask(` 从当前函数返回。
- **L2092 EN**: Executes a call or declaration centered on `statement`.
  **L2092 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2093 EN**: Closes the current lexical scope or compound statement.
  **L2093 CN**: 结束当前词法作用域或复合语句块。
- **L2094 EN**: Blank line separating nearby declarations or logic blocks.
  **L2094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2095 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512`.
  **L2095 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512`。
- **L2096 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepi64_ph(__mmask8 __U, __m512i __A) {`.
  **L2096 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepi64_ph(__mmask8 __U, __m512i __A) {`。
- **L2097 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtqq2ph512_mask(`.
  **L2097 CN**: 以 `(__m128h)__builtin_ia32_vcvtqq2ph512_mask(` 从当前函数返回。
- **L2098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8di)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L2098 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8di)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L2099 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2099 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2100 EN**: Closes the current lexical scope or compound statement.
  **L2100 CN**: 结束当前词法作用域或复合语句块。
- **L2101 EN**: Blank line separating nearby declarations or logic blocks.
  **L2101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2102 EN**: Defines macro `_mm512_cvt_roundph_epi64(A, R)` for conditional compilation, shorthand, or API generation.
  **L2102 CN**: 定义宏 `_mm512_cvt_roundph_epi64(A, R)`，用于条件编译、简写或 API 生成。
- **L2103 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2qq512_mask`.
  **L2103 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2qq512_mask` 相关的逻辑。
- **L2104 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L2104 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L2105 EN**: Continues the surrounding expression or declaration: `(__mmask8)(-1), (int)(R)))`.
  **L2105 CN**: 继续构造周围的表达式或声明：`(__mmask8)(-1), (int)(R)))`。
- **L2106 EN**: Blank line separating nearby declarations or logic blocks.
  **L2106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2107 EN**: Defines macro `_mm512_mask_cvt_roundph_epi64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2107 CN**: 定义宏 `_mm512_mask_cvt_roundph_epi64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2108 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2qq512_mask`.
  **L2108 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2qq512_mask` 相关的逻辑。
- **L2109 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2109 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2110 EN**: Blank line separating nearby declarations or logic blocks.
  **L2110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2111 EN**: Defines macro `_mm512_maskz_cvt_roundph_epi64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2111 CN**: 定义宏 `_mm512_maskz_cvt_roundph_epi64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2112 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2qq512_mask`.
  **L2112 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2qq512_mask` 相关的逻辑。

### Lines 2113-2136

````c
      (__v8hf)(A), (__v8di)_mm512_setzero_epi32(), (__mmask8)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtph_epi64(__m128h __A) {
  return (__m512i)__builtin_ia32_vcvtph2qq512_mask(
      (__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtph_epi64(__m512i __W, __mmask8 __U, __m128h __A) {
  return (__m512i)__builtin_ia32_vcvtph2qq512_mask(
      (__v8hf)__A, (__v8di)__W, (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {
  return (__m512i)__builtin_ia32_vcvtph2qq512_mask(
      (__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundepu64_ph(A, R)                                         \
  ((__m128h)__builtin_ia32_vcvtuqq2ph512_mask(                                 \
````
- **L2113 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L2113 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L2114 EN**: Blank line separating nearby declarations or logic blocks.
  **L2114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2115 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2115 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2116 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtph_epi64(__m128h __A) {`.
  **L2116 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtph_epi64(__m128h __A) {`。
- **L2117 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2qq512_mask(`.
  **L2117 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2qq512_mask(` 从当前函数返回。
- **L2118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)-1,`.
  **L2118 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)-1,`。
- **L2119 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2119 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2120 EN**: Closes the current lexical scope or compound statement.
  **L2120 CN**: 结束当前词法作用域或复合语句块。
- **L2121 EN**: Blank line separating nearby declarations or logic blocks.
  **L2121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2122 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2122 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2123 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtph_epi64(__m512i __W, __mmask8 __U, __m128h __A) {`.
  **L2123 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtph_epi64(__m512i __W, __mmask8 __U, __m128h __A) {`。
- **L2124 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2qq512_mask(`.
  **L2124 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2qq512_mask(` 从当前函数返回。
- **L2125 EN**: Executes a call or declaration centered on `statement`.
  **L2125 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2126 EN**: Closes the current lexical scope or compound statement.
  **L2126 CN**: 结束当前词法作用域或复合语句块。
- **L2127 EN**: Blank line separating nearby declarations or logic blocks.
  **L2127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2128 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2128 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2129 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {`.
  **L2129 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtph_epi64(__mmask8 __U, __m128h __A) {`。
- **L2130 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2qq512_mask(`.
  **L2130 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2qq512_mask(` 从当前函数返回。
- **L2131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)__U,`.
  **L2131 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)__U,`。
- **L2132 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2132 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2133 EN**: Closes the current lexical scope or compound statement.
  **L2133 CN**: 结束当前词法作用域或复合语句块。
- **L2134 EN**: Blank line separating nearby declarations or logic blocks.
  **L2134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2135 EN**: Defines macro `_mm512_cvt_roundepu64_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L2135 CN**: 定义宏 `_mm512_cvt_roundepu64_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L2136 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtuqq2ph512_mask`.
  **L2136 CN**: 继续与可调用符号 `__builtin_ia32_vcvtuqq2ph512_mask` 相关的逻辑。

### Lines 2137-2160

````c
      (__v8du)(A), (__v8hf)_mm_undefined_ph(), (__mmask8)(-1), (int)(R)))

#define _mm512_mask_cvt_roundepu64_ph(W, U, A, R)                              \
  ((__m128h)__builtin_ia32_vcvtuqq2ph512_mask((__v8du)(A), (__v8hf)(W),        \
                                              (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundepu64_ph(U, A, R)                                \
  ((__m128h)__builtin_ia32_vcvtuqq2ph512_mask(                                 \
      (__v8du)(A), (__v8hf)_mm_setzero_ph(), (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS512
_mm512_cvtepu64_ph(__m512i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph512_mask(
      (__v8du)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS512
_mm512_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m512i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph512_mask(
      (__v8du)__A, (__v8hf)__W, (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS512
````
- **L2137 EN**: Continues logic associated with callable symbol `_mm_undefined_ph`.
  **L2137 CN**: 继续与可调用符号 `_mm_undefined_ph` 相关的逻辑。
- **L2138 EN**: Blank line separating nearby declarations or logic blocks.
  **L2138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2139 EN**: Defines macro `_mm512_mask_cvt_roundepu64_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2139 CN**: 定义宏 `_mm512_mask_cvt_roundepu64_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2140 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtuqq2ph512_mask`.
  **L2140 CN**: 继续与可调用符号 `__builtin_ia32_vcvtuqq2ph512_mask` 相关的逻辑。
- **L2141 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2141 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2142 EN**: Blank line separating nearby declarations or logic blocks.
  **L2142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2143 EN**: Defines macro `_mm512_maskz_cvt_roundepu64_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2143 CN**: 定义宏 `_mm512_maskz_cvt_roundepu64_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2144 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtuqq2ph512_mask`.
  **L2144 CN**: 继续与可调用符号 `__builtin_ia32_vcvtuqq2ph512_mask` 相关的逻辑。
- **L2145 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L2145 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L2146 EN**: Blank line separating nearby declarations or logic blocks.
  **L2146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2147 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512`.
  **L2147 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512`。
- **L2148 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtepu64_ph(__m512i __A) {`.
  **L2148 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtepu64_ph(__m512i __A) {`。
- **L2149 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph512_mask(`.
  **L2149 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph512_mask(` 从当前函数返回。
- **L2150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8du)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`.
  **L2150 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8du)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)-1,`。
- **L2151 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2151 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2152 EN**: Closes the current lexical scope or compound statement.
  **L2152 CN**: 结束当前词法作用域或复合语句块。
- **L2153 EN**: Blank line separating nearby declarations or logic blocks.
  **L2153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2154 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512`.
  **L2154 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512`。
- **L2155 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m512i __A) {`.
  **L2155 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtepu64_ph(__m128h __W, __mmask8 __U, __m512i __A) {`。
- **L2156 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph512_mask(`.
  **L2156 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph512_mask(` 从当前函数返回。
- **L2157 EN**: Executes a call or declaration centered on `statement`.
  **L2157 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2158 EN**: Closes the current lexical scope or compound statement.
  **L2158 CN**: 结束当前词法作用域或复合语句块。
- **L2159 EN**: Blank line separating nearby declarations or logic blocks.
  **L2159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2160 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS512`.
  **L2160 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS512`。

### Lines 2161-2184

````c
_mm512_maskz_cvtepu64_ph(__mmask8 __U, __m512i __A) {
  return (__m128h)__builtin_ia32_vcvtuqq2ph512_mask(
      (__v8du)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvt_roundph_epu64(A, R)                                         \
  ((__m512i)__builtin_ia32_vcvtph2uqq512_mask(                                 \
      (__v8hf)(A), (__v8du)_mm512_undefined_epi32(), (__mmask8)(-1),           \
      (int)(R)))

#define _mm512_mask_cvt_roundph_epu64(W, U, A, R)                              \
  ((__m512i)__builtin_ia32_vcvtph2uqq512_mask((__v8hf)(A), (__v8du)(W),        \
                                              (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvt_roundph_epu64(U, A, R)                                \
  ((__m512i)__builtin_ia32_vcvtph2uqq512_mask(                                 \
      (__v8hf)(A), (__v8du)_mm512_setzero_epi32(), (__mmask8)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvtph_epu64(__m128h __A) {
  return (__m512i)__builtin_ia32_vcvtph2uqq512_mask(
      (__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
````
- **L2161 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtepu64_ph(__mmask8 __U, __m512i __A) {`.
  **L2161 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtepu64_ph(__mmask8 __U, __m512i __A) {`。
- **L2162 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vcvtuqq2ph512_mask(`.
  **L2162 CN**: 以 `(__m128h)__builtin_ia32_vcvtuqq2ph512_mask(` 从当前函数返回。
- **L2163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8du)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L2163 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8du)__A, (__v8hf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L2164 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2164 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2165 EN**: Closes the current lexical scope or compound statement.
  **L2165 CN**: 结束当前词法作用域或复合语句块。
- **L2166 EN**: Blank line separating nearby declarations or logic blocks.
  **L2166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2167 EN**: Defines macro `_mm512_cvt_roundph_epu64(A, R)` for conditional compilation, shorthand, or API generation.
  **L2167 CN**: 定义宏 `_mm512_cvt_roundph_epu64(A, R)`，用于条件编译、简写或 API 生成。
- **L2168 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2uqq512_mask`.
  **L2168 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2uqq512_mask` 相关的逻辑。
- **L2169 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L2169 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L2170 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L2170 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L2171 EN**: Blank line separating nearby declarations or logic blocks.
  **L2171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2172 EN**: Defines macro `_mm512_mask_cvt_roundph_epu64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2172 CN**: 定义宏 `_mm512_mask_cvt_roundph_epu64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2173 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2uqq512_mask`.
  **L2173 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2uqq512_mask` 相关的逻辑。
- **L2174 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2174 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2175 EN**: Blank line separating nearby declarations or logic blocks.
  **L2175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2176 EN**: Defines macro `_mm512_maskz_cvt_roundph_epu64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2176 CN**: 定义宏 `_mm512_maskz_cvt_roundph_epu64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2177 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2uqq512_mask`.
  **L2177 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2uqq512_mask` 相关的逻辑。
- **L2178 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L2178 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L2179 EN**: Blank line separating nearby declarations or logic blocks.
  **L2179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2180 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2180 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2181 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvtph_epu64(__m128h __A) {`.
  **L2181 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvtph_epu64(__m128h __A) {`。
- **L2182 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2uqq512_mask(`.
  **L2182 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2uqq512_mask(` 从当前函数返回。
- **L2183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)-1,`.
  **L2183 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)-1,`。
- **L2184 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2184 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。

### Lines 2185-2208

````c
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvtph_epu64(__m512i __W, __mmask8 __U, __m128h __A) {
  return (__m512i)__builtin_ia32_vcvtph2uqq512_mask(
      (__v8hf)__A, (__v8du)__W, (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {
  return (__m512i)__builtin_ia32_vcvtph2uqq512_mask(
      (__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvtt_roundph_epi64(A, R)                                        \
  ((__m512i)__builtin_ia32_vcvttph2qq512_mask(                                 \
      (__v8hf)(A), (__v8di)_mm512_undefined_epi32(), (__mmask8)(-1),           \
      (int)(R)))

#define _mm512_mask_cvtt_roundph_epi64(W, U, A, R)                             \
  ((__m512i)__builtin_ia32_vcvttph2qq512_mask((__v8hf)(A), (__v8di)(W),        \
                                              (__mmask8)(U), (int)(R)))

````
- **L2185 EN**: Closes the current lexical scope or compound statement.
  **L2185 CN**: 结束当前词法作用域或复合语句块。
- **L2186 EN**: Blank line separating nearby declarations or logic blocks.
  **L2186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2187 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2187 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2188 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtph_epu64(__m512i __W, __mmask8 __U, __m128h __A) {`.
  **L2188 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtph_epu64(__m512i __W, __mmask8 __U, __m128h __A) {`。
- **L2189 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2uqq512_mask(`.
  **L2189 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2uqq512_mask(` 从当前函数返回。
- **L2190 EN**: Executes a call or declaration centered on `statement`.
  **L2190 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2191 EN**: Closes the current lexical scope or compound statement.
  **L2191 CN**: 结束当前词法作用域或复合语句块。
- **L2192 EN**: Blank line separating nearby declarations or logic blocks.
  **L2192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2193 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2193 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2194 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {`.
  **L2194 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtph_epu64(__mmask8 __U, __m128h __A) {`。
- **L2195 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvtph2uqq512_mask(`.
  **L2195 CN**: 以 `(__m512i)__builtin_ia32_vcvtph2uqq512_mask(` 从当前函数返回。
- **L2196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)__U,`.
  **L2196 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)__U,`。
- **L2197 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2197 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2198 EN**: Closes the current lexical scope or compound statement.
  **L2198 CN**: 结束当前词法作用域或复合语句块。
- **L2199 EN**: Blank line separating nearby declarations or logic blocks.
  **L2199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2200 EN**: Defines macro `_mm512_cvtt_roundph_epi64(A, R)` for conditional compilation, shorthand, or API generation.
  **L2200 CN**: 定义宏 `_mm512_cvtt_roundph_epi64(A, R)`，用于条件编译、简写或 API 生成。
- **L2201 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2qq512_mask`.
  **L2201 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2qq512_mask` 相关的逻辑。
- **L2202 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L2202 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L2203 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L2203 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L2204 EN**: Blank line separating nearby declarations or logic blocks.
  **L2204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2205 EN**: Defines macro `_mm512_mask_cvtt_roundph_epi64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2205 CN**: 定义宏 `_mm512_mask_cvtt_roundph_epi64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2206 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2qq512_mask`.
  **L2206 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2qq512_mask` 相关的逻辑。
- **L2207 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2207 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2208 EN**: Blank line separating nearby declarations or logic blocks.
  **L2208 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2209-2232

````c
#define _mm512_maskz_cvtt_roundph_epi64(U, A, R)                               \
  ((__m512i)__builtin_ia32_vcvttph2qq512_mask(                                 \
      (__v8hf)(A), (__v8di)_mm512_setzero_epi32(), (__mmask8)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttph_epi64(__m128h __A) {
  return (__m512i)__builtin_ia32_vcvttph2qq512_mask(
      (__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttph_epi64(__m512i __W, __mmask8 __U, __m128h __A) {
  return (__m512i)__builtin_ia32_vcvttph2qq512_mask(
      (__v8hf)__A, (__v8di)__W, (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {
  return (__m512i)__builtin_ia32_vcvttph2qq512_mask(
      (__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

````
- **L2209 EN**: Defines macro `_mm512_maskz_cvtt_roundph_epi64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2209 CN**: 定义宏 `_mm512_maskz_cvtt_roundph_epi64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2210 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2qq512_mask`.
  **L2210 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2qq512_mask` 相关的逻辑。
- **L2211 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L2211 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L2212 EN**: Blank line separating nearby declarations or logic blocks.
  **L2212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2213 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2213 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2214 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttph_epi64(__m128h __A) {`.
  **L2214 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttph_epi64(__m128h __A) {`。
- **L2215 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2qq512_mask(`.
  **L2215 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2qq512_mask(` 从当前函数返回。
- **L2216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)-1,`.
  **L2216 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)-1,`。
- **L2217 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2217 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2218 EN**: Closes the current lexical scope or compound statement.
  **L2218 CN**: 结束当前词法作用域或复合语句块。
- **L2219 EN**: Blank line separating nearby declarations or logic blocks.
  **L2219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2220 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2220 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2221 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttph_epi64(__m512i __W, __mmask8 __U, __m128h __A) {`.
  **L2221 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttph_epi64(__m512i __W, __mmask8 __U, __m128h __A) {`。
- **L2222 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2qq512_mask(`.
  **L2222 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2qq512_mask(` 从当前函数返回。
- **L2223 EN**: Executes a call or declaration centered on `statement`.
  **L2223 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2224 EN**: Closes the current lexical scope or compound statement.
  **L2224 CN**: 结束当前词法作用域或复合语句块。
- **L2225 EN**: Blank line separating nearby declarations or logic blocks.
  **L2225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2226 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2226 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {`.
  **L2227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttph_epi64(__mmask8 __U, __m128h __A) {`。
- **L2228 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2qq512_mask(`.
  **L2228 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2qq512_mask(` 从当前函数返回。
- **L2229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)__U,`.
  **L2229 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8di)_mm512_setzero_epi32(), (__mmask8)__U,`。
- **L2230 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2230 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2231 EN**: Closes the current lexical scope or compound statement.
  **L2231 CN**: 结束当前词法作用域或复合语句块。
- **L2232 EN**: Blank line separating nearby declarations or logic blocks.
  **L2232 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2233-2256

````c
#define _mm512_cvtt_roundph_epu64(A, R)                                        \
  ((__m512i)__builtin_ia32_vcvttph2uqq512_mask(                                \
      (__v8hf)(A), (__v8du)_mm512_undefined_epi32(), (__mmask8)(-1),           \
      (int)(R)))

#define _mm512_mask_cvtt_roundph_epu64(W, U, A, R)                             \
  ((__m512i)__builtin_ia32_vcvttph2uqq512_mask((__v8hf)(A), (__v8du)(W),       \
                                               (__mmask8)(U), (int)(R)))

#define _mm512_maskz_cvtt_roundph_epu64(U, A, R)                               \
  ((__m512i)__builtin_ia32_vcvttph2uqq512_mask(                                \
      (__v8hf)(A), (__v8du)_mm512_setzero_epi32(), (__mmask8)(U), (int)(R)))

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_cvttph_epu64(__m128h __A) {
  return (__m512i)__builtin_ia32_vcvttph2uqq512_mask(
      (__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_mask_cvttph_epu64(__m512i __W, __mmask8 __U, __m128h __A) {
  return (__m512i)__builtin_ia32_vcvttph2uqq512_mask(
      (__v8hf)__A, (__v8du)__W, (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
````
- **L2233 EN**: Defines macro `_mm512_cvtt_roundph_epu64(A, R)` for conditional compilation, shorthand, or API generation.
  **L2233 CN**: 定义宏 `_mm512_cvtt_roundph_epu64(A, R)`，用于条件编译、简写或 API 生成。
- **L2234 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2uqq512_mask`.
  **L2234 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2uqq512_mask` 相关的逻辑。
- **L2235 EN**: Continues logic associated with callable symbol `_mm512_undefined_epi32`.
  **L2235 CN**: 继续与可调用符号 `_mm512_undefined_epi32` 相关的逻辑。
- **L2236 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L2236 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L2237 EN**: Blank line separating nearby declarations or logic blocks.
  **L2237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2238 EN**: Defines macro `_mm512_mask_cvtt_roundph_epu64(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2238 CN**: 定义宏 `_mm512_mask_cvtt_roundph_epu64(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2239 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2uqq512_mask`.
  **L2239 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2uqq512_mask` 相关的逻辑。
- **L2240 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2240 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2241 EN**: Blank line separating nearby declarations or logic blocks.
  **L2241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2242 EN**: Defines macro `_mm512_maskz_cvtt_roundph_epu64(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2242 CN**: 定义宏 `_mm512_maskz_cvtt_roundph_epu64(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2243 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttph2uqq512_mask`.
  **L2243 CN**: 继续与可调用符号 `__builtin_ia32_vcvttph2uqq512_mask` 相关的逻辑。
- **L2244 EN**: Continues logic associated with callable symbol `_mm512_setzero_epi32`.
  **L2244 CN**: 继续与可调用符号 `_mm512_setzero_epi32` 相关的逻辑。
- **L2245 EN**: Blank line separating nearby declarations or logic blocks.
  **L2245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2246 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2246 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2247 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_cvttph_epu64(__m128h __A) {`.
  **L2247 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_cvttph_epu64(__m128h __A) {`。
- **L2248 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2uqq512_mask(`.
  **L2248 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2uqq512_mask(` 从当前函数返回。
- **L2249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)-1,`.
  **L2249 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)-1,`。
- **L2250 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2250 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2251 EN**: Closes the current lexical scope or compound statement.
  **L2251 CN**: 结束当前词法作用域或复合语句块。
- **L2252 EN**: Blank line separating nearby declarations or logic blocks.
  **L2252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2253 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2253 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2254 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvttph_epu64(__m512i __W, __mmask8 __U, __m128h __A) {`.
  **L2254 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvttph_epu64(__m512i __W, __mmask8 __U, __m128h __A) {`。
- **L2255 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2uqq512_mask(`.
  **L2255 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2uqq512_mask(` 从当前函数返回。
- **L2256 EN**: Executes a call or declaration centered on `statement`.
  **L2256 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 2257-2280

````c
}

static __inline__ __m512i __DEFAULT_FN_ATTRS512
_mm512_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {
  return (__m512i)__builtin_ia32_vcvttph2uqq512_mask(
      (__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_cvt_roundsh_i32(A, R)                                              \
  ((int)__builtin_ia32_vcvtsh2si32((__v8hf)(A), (int)(R)))

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_cvtsh_i32(__m128h __A) {
  return (int)__builtin_ia32_vcvtsh2si32((__v8hf)__A, _MM_FROUND_CUR_DIRECTION);
}

#define _mm_cvt_roundsh_u32(A, R)                                              \
  ((unsigned int)__builtin_ia32_vcvtsh2usi32((__v8hf)(A), (int)(R)))

static __inline__ unsigned int __DEFAULT_FN_ATTRS128
_mm_cvtsh_u32(__m128h __A) {
  return (unsigned int)__builtin_ia32_vcvtsh2usi32((__v8hf)__A,
                                                   _MM_FROUND_CUR_DIRECTION);
}
````
- **L2257 EN**: Closes the current lexical scope or compound statement.
  **L2257 CN**: 结束当前词法作用域或复合语句块。
- **L2258 EN**: Blank line separating nearby declarations or logic blocks.
  **L2258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2259 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512i __DEFAULT_FN_ATTRS512`.
  **L2259 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512i __DEFAULT_FN_ATTRS512`。
- **L2260 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {`.
  **L2260 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvttph_epu64(__mmask8 __U, __m128h __A) {`。
- **L2261 EN**: Returns from the current function with `(__m512i)__builtin_ia32_vcvttph2uqq512_mask(`.
  **L2261 CN**: 以 `(__m512i)__builtin_ia32_vcvttph2uqq512_mask(` 从当前函数返回。
- **L2262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)__U,`.
  **L2262 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hf)__A, (__v8du)_mm512_setzero_epi32(), (__mmask8)__U,`。
- **L2263 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2263 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2264 EN**: Closes the current lexical scope or compound statement.
  **L2264 CN**: 结束当前词法作用域或复合语句块。
- **L2265 EN**: Blank line separating nearby declarations or logic blocks.
  **L2265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2266 EN**: Defines macro `_mm_cvt_roundsh_i32(A, R)` for conditional compilation, shorthand, or API generation.
  **L2266 CN**: 定义宏 `_mm_cvt_roundsh_i32(A, R)`，用于条件编译、简写或 API 生成。
- **L2267 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2si32`.
  **L2267 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2si32` 相关的逻辑。
- **L2268 EN**: Blank line separating nearby declarations or logic blocks.
  **L2268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2269 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_cvtsh_i32(__m128h __A) {`.
  **L2269 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_cvtsh_i32(__m128h __A) {`。
- **L2270 EN**: Returns from the current function with `(int)__builtin_ia32_vcvtsh2si32((__v8hf)__A, _MM_FROUND_CUR_DIRECTION)`.
  **L2270 CN**: 以 `(int)__builtin_ia32_vcvtsh2si32((__v8hf)__A, _MM_FROUND_CUR_DIRECTION)` 从当前函数返回。
- **L2271 EN**: Closes the current lexical scope or compound statement.
  **L2271 CN**: 结束当前词法作用域或复合语句块。
- **L2272 EN**: Blank line separating nearby declarations or logic blocks.
  **L2272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2273 EN**: Defines macro `_mm_cvt_roundsh_u32(A, R)` for conditional compilation, shorthand, or API generation.
  **L2273 CN**: 定义宏 `_mm_cvt_roundsh_u32(A, R)`，用于条件编译、简写或 API 生成。
- **L2274 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2usi32`.
  **L2274 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2usi32` 相关的逻辑。
- **L2275 EN**: Blank line separating nearby declarations or logic blocks.
  **L2275 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2276 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned int __DEFAULT_FN_ATTRS128`.
  **L2276 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned int __DEFAULT_FN_ATTRS128`。
- **L2277 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsh_u32(__m128h __A) {`.
  **L2277 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsh_u32(__m128h __A) {`。
- **L2278 EN**: Returns from the current function with `(unsigned int)__builtin_ia32_vcvtsh2usi32((__v8hf)__A,`.
  **L2278 CN**: 以 `(unsigned int)__builtin_ia32_vcvtsh2usi32((__v8hf)__A,` 从当前函数返回。
- **L2279 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2279 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2280 EN**: Closes the current lexical scope or compound statement.
  **L2280 CN**: 结束当前词法作用域或复合语句块。

### Lines 2281-2304

````c

#ifdef __x86_64__
#define _mm_cvt_roundsh_i64(A, R)                                              \
  ((long long)__builtin_ia32_vcvtsh2si64((__v8hf)(A), (int)(R)))

static __inline__ long long __DEFAULT_FN_ATTRS128 _mm_cvtsh_i64(__m128h __A) {
  return (long long)__builtin_ia32_vcvtsh2si64((__v8hf)__A,
                                               _MM_FROUND_CUR_DIRECTION);
}

#define _mm_cvt_roundsh_u64(A, R)                                              \
  ((unsigned long long)__builtin_ia32_vcvtsh2usi64((__v8hf)(A), (int)(R)))

static __inline__ unsigned long long __DEFAULT_FN_ATTRS128
_mm_cvtsh_u64(__m128h __A) {
  return (unsigned long long)__builtin_ia32_vcvtsh2usi64(
      (__v8hf)__A, _MM_FROUND_CUR_DIRECTION);
}
#endif // __x86_64__

#define _mm_cvt_roundu32_sh(A, B, R)                                           \
  ((__m128h)__builtin_ia32_vcvtusi2sh((__v8hf)(A), (unsigned int)(B), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
````
- **L2281 EN**: Blank line separating nearby declarations or logic blocks.
  **L2281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2282 EN**: Starts a preprocessor conditional block: `#ifdef __x86_64__`.
  **L2282 CN**: 开始一个预处理条件块：`#ifdef __x86_64__`。
- **L2283 EN**: Defines macro `_mm_cvt_roundsh_i64(A, R)` for conditional compilation, shorthand, or API generation.
  **L2283 CN**: 定义宏 `_mm_cvt_roundsh_i64(A, R)`，用于条件编译、简写或 API 生成。
- **L2284 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2si64`.
  **L2284 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2si64` 相关的逻辑。
- **L2285 EN**: Blank line separating nearby declarations or logic blocks.
  **L2285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2286 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ long long __DEFAULT_FN_ATTRS128 _mm_cvtsh_i64(__m128h __A) {`.
  **L2286 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ long long __DEFAULT_FN_ATTRS128 _mm_cvtsh_i64(__m128h __A) {`。
- **L2287 EN**: Returns from the current function with `(long long)__builtin_ia32_vcvtsh2si64((__v8hf)__A,`.
  **L2287 CN**: 以 `(long long)__builtin_ia32_vcvtsh2si64((__v8hf)__A,` 从当前函数返回。
- **L2288 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2288 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2289 EN**: Closes the current lexical scope or compound statement.
  **L2289 CN**: 结束当前词法作用域或复合语句块。
- **L2290 EN**: Blank line separating nearby declarations or logic blocks.
  **L2290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2291 EN**: Defines macro `_mm_cvt_roundsh_u64(A, R)` for conditional compilation, shorthand, or API generation.
  **L2291 CN**: 定义宏 `_mm_cvt_roundsh_u64(A, R)`，用于条件编译、简写或 API 生成。
- **L2292 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsh2usi64`.
  **L2292 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsh2usi64` 相关的逻辑。
- **L2293 EN**: Blank line separating nearby declarations or logic blocks.
  **L2293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2294 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned long long __DEFAULT_FN_ATTRS128`.
  **L2294 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned long long __DEFAULT_FN_ATTRS128`。
- **L2295 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsh_u64(__m128h __A) {`.
  **L2295 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsh_u64(__m128h __A) {`。
- **L2296 EN**: Returns from the current function with `(unsigned long long)__builtin_ia32_vcvtsh2usi64(`.
  **L2296 CN**: 以 `(unsigned long long)__builtin_ia32_vcvtsh2usi64(` 从当前函数返回。
- **L2297 EN**: Executes a call or declaration centered on `statement`.
  **L2297 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2298 EN**: Closes the current lexical scope or compound statement.
  **L2298 CN**: 结束当前词法作用域或复合语句块。
- **L2299 EN**: Closes the current preprocessor conditional block.
  **L2299 CN**: 结束当前预处理条件块。
- **L2300 EN**: Blank line separating nearby declarations or logic blocks.
  **L2300 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2301 EN**: Defines macro `_mm_cvt_roundu32_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L2301 CN**: 定义宏 `_mm_cvt_roundu32_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L2302 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtusi2sh`.
  **L2302 CN**: 继续与可调用符号 `__builtin_ia32_vcvtusi2sh` 相关的逻辑。
- **L2303 EN**: Blank line separating nearby declarations or logic blocks.
  **L2303 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2304 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2304 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。

### Lines 2305-2328

````c
_mm_cvtu32_sh(__m128h __A, unsigned int __B) {
  __A[0] = __B;
  return __A;
}

#ifdef __x86_64__
#define _mm_cvt_roundu64_sh(A, B, R)                                           \
  ((__m128h)__builtin_ia32_vcvtusi642sh((__v8hf)(A), (unsigned long long)(B),  \
                                        (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_cvtu64_sh(__m128h __A, unsigned long long __B) {
  __A[0] = __B;
  return __A;
}
#endif

#define _mm_cvt_roundi32_sh(A, B, R)                                           \
  ((__m128h)__builtin_ia32_vcvtsi2sh((__v8hf)(A), (int)(B), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvti32_sh(__m128h __A,
                                                              int __B) {
  __A[0] = __B;
  return __A;
````
- **L2305 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtu32_sh(__m128h __A, unsigned int __B) {`.
  **L2305 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtu32_sh(__m128h __A, unsigned int __B) {`。
- **L2306 EN**: Adds a standalone statement or declaration: `__A[0] = __B;`.
  **L2306 CN**: 添加一条独立语句或声明：`__A[0] = __B;`。
- **L2307 EN**: Returns from the current function with `__A`.
  **L2307 CN**: 以 `__A` 从当前函数返回。
- **L2308 EN**: Closes the current lexical scope or compound statement.
  **L2308 CN**: 结束当前词法作用域或复合语句块。
- **L2309 EN**: Blank line separating nearby declarations or logic blocks.
  **L2309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2310 EN**: Starts a preprocessor conditional block: `#ifdef __x86_64__`.
  **L2310 CN**: 开始一个预处理条件块：`#ifdef __x86_64__`。
- **L2311 EN**: Defines macro `_mm_cvt_roundu64_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L2311 CN**: 定义宏 `_mm_cvt_roundu64_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L2312 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtusi642sh`.
  **L2312 CN**: 继续与可调用符号 `__builtin_ia32_vcvtusi642sh` 相关的逻辑。
- **L2313 EN**: Continues the surrounding expression or declaration: `(int)(R)))`.
  **L2313 CN**: 继续构造周围的表达式或声明：`(int)(R)))`。
- **L2314 EN**: Blank line separating nearby declarations or logic blocks.
  **L2314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2315 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2315 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2316 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtu64_sh(__m128h __A, unsigned long long __B) {`.
  **L2316 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtu64_sh(__m128h __A, unsigned long long __B) {`。
- **L2317 EN**: Adds a standalone statement or declaration: `__A[0] = __B;`.
  **L2317 CN**: 添加一条独立语句或声明：`__A[0] = __B;`。
- **L2318 EN**: Returns from the current function with `__A`.
  **L2318 CN**: 以 `__A` 从当前函数返回。
- **L2319 EN**: Closes the current lexical scope or compound statement.
  **L2319 CN**: 结束当前词法作用域或复合语句块。
- **L2320 EN**: Closes the current preprocessor conditional block.
  **L2320 CN**: 结束当前预处理条件块。
- **L2321 EN**: Blank line separating nearby declarations or logic blocks.
  **L2321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2322 EN**: Defines macro `_mm_cvt_roundi32_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L2322 CN**: 定义宏 `_mm_cvt_roundi32_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L2323 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsi2sh`.
  **L2323 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsi2sh` 相关的逻辑。
- **L2324 EN**: Blank line separating nearby declarations or logic blocks.
  **L2324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvti32_sh(__m128h __A,`.
  **L2325 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvti32_sh(__m128h __A,`。
- **L2326 EN**: Continues the surrounding expression or declaration: `int __B) {`.
  **L2326 CN**: 继续构造周围的表达式或声明：`int __B) {`。
- **L2327 EN**: Adds a standalone statement or declaration: `__A[0] = __B;`.
  **L2327 CN**: 添加一条独立语句或声明：`__A[0] = __B;`。
- **L2328 EN**: Returns from the current function with `__A`.
  **L2328 CN**: 以 `__A` 从当前函数返回。

### Lines 2329-2352

````c
}

#ifdef __x86_64__
#define _mm_cvt_roundi64_sh(A, B, R)                                           \
  ((__m128h)__builtin_ia32_vcvtsi642sh((__v8hf)(A), (long long)(B), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvti64_sh(__m128h __A,
                                                              long long __B) {
  __A[0] = __B;
  return __A;
}
#endif

#define _mm_cvtt_roundsh_i32(A, R)                                             \
  ((int)__builtin_ia32_vcvttsh2si32((__v8hf)(A), (int)(R)))

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_cvttsh_i32(__m128h __A) {
  return (int)__builtin_ia32_vcvttsh2si32((__v8hf)__A,
                                          _MM_FROUND_CUR_DIRECTION);
}

#ifdef __x86_64__
#define _mm_cvtt_roundsh_i64(A, R)                                             \
  ((long long)__builtin_ia32_vcvttsh2si64((__v8hf)(A), (int)(R)))
````
- **L2329 EN**: Closes the current lexical scope or compound statement.
  **L2329 CN**: 结束当前词法作用域或复合语句块。
- **L2330 EN**: Blank line separating nearby declarations or logic blocks.
  **L2330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2331 EN**: Starts a preprocessor conditional block: `#ifdef __x86_64__`.
  **L2331 CN**: 开始一个预处理条件块：`#ifdef __x86_64__`。
- **L2332 EN**: Defines macro `_mm_cvt_roundi64_sh(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L2332 CN**: 定义宏 `_mm_cvt_roundi64_sh(A, B, R)`，用于条件编译、简写或 API 生成。
- **L2333 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtsi642sh`.
  **L2333 CN**: 继续与可调用符号 `__builtin_ia32_vcvtsi642sh` 相关的逻辑。
- **L2334 EN**: Blank line separating nearby declarations or logic blocks.
  **L2334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvti64_sh(__m128h __A,`.
  **L2335 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_cvti64_sh(__m128h __A,`。
- **L2336 EN**: Continues the surrounding expression or declaration: `long long __B) {`.
  **L2336 CN**: 继续构造周围的表达式或声明：`long long __B) {`。
- **L2337 EN**: Adds a standalone statement or declaration: `__A[0] = __B;`.
  **L2337 CN**: 添加一条独立语句或声明：`__A[0] = __B;`。
- **L2338 EN**: Returns from the current function with `__A`.
  **L2338 CN**: 以 `__A` 从当前函数返回。
- **L2339 EN**: Closes the current lexical scope or compound statement.
  **L2339 CN**: 结束当前词法作用域或复合语句块。
- **L2340 EN**: Closes the current preprocessor conditional block.
  **L2340 CN**: 结束当前预处理条件块。
- **L2341 EN**: Blank line separating nearby declarations or logic blocks.
  **L2341 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2342 EN**: Defines macro `_mm_cvtt_roundsh_i32(A, R)` for conditional compilation, shorthand, or API generation.
  **L2342 CN**: 定义宏 `_mm_cvtt_roundsh_i32(A, R)`，用于条件编译、简写或 API 生成。
- **L2343 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsh2si32`.
  **L2343 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsh2si32` 相关的逻辑。
- **L2344 EN**: Blank line separating nearby declarations or logic blocks.
  **L2344 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2345 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_cvttsh_i32(__m128h __A) {`.
  **L2345 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_cvttsh_i32(__m128h __A) {`。
- **L2346 EN**: Returns from the current function with `(int)__builtin_ia32_vcvttsh2si32((__v8hf)__A,`.
  **L2346 CN**: 以 `(int)__builtin_ia32_vcvttsh2si32((__v8hf)__A,` 从当前函数返回。
- **L2347 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2347 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2348 EN**: Closes the current lexical scope or compound statement.
  **L2348 CN**: 结束当前词法作用域或复合语句块。
- **L2349 EN**: Blank line separating nearby declarations or logic blocks.
  **L2349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2350 EN**: Starts a preprocessor conditional block: `#ifdef __x86_64__`.
  **L2350 CN**: 开始一个预处理条件块：`#ifdef __x86_64__`。
- **L2351 EN**: Defines macro `_mm_cvtt_roundsh_i64(A, R)` for conditional compilation, shorthand, or API generation.
  **L2351 CN**: 定义宏 `_mm_cvtt_roundsh_i64(A, R)`，用于条件编译、简写或 API 生成。
- **L2352 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsh2si64`.
  **L2352 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsh2si64` 相关的逻辑。

### Lines 2353-2376

````c

static __inline__ long long __DEFAULT_FN_ATTRS128 _mm_cvttsh_i64(__m128h __A) {
  return (long long)__builtin_ia32_vcvttsh2si64((__v8hf)__A,
                                                _MM_FROUND_CUR_DIRECTION);
}
#endif

#define _mm_cvtt_roundsh_u32(A, R)                                             \
  ((unsigned int)__builtin_ia32_vcvttsh2usi32((__v8hf)(A), (int)(R)))

static __inline__ unsigned int __DEFAULT_FN_ATTRS128
_mm_cvttsh_u32(__m128h __A) {
  return (unsigned int)__builtin_ia32_vcvttsh2usi32((__v8hf)__A,
                                                    _MM_FROUND_CUR_DIRECTION);
}

#ifdef __x86_64__
#define _mm_cvtt_roundsh_u64(A, R)                                             \
  ((unsigned long long)__builtin_ia32_vcvttsh2usi64((__v8hf)(A), (int)(R)))

static __inline__ unsigned long long __DEFAULT_FN_ATTRS128
_mm_cvttsh_u64(__m128h __A) {
  return (unsigned long long)__builtin_ia32_vcvttsh2usi64(
      (__v8hf)__A, _MM_FROUND_CUR_DIRECTION);
````
- **L2353 EN**: Blank line separating nearby declarations or logic blocks.
  **L2353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2354 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ long long __DEFAULT_FN_ATTRS128 _mm_cvttsh_i64(__m128h __A) {`.
  **L2354 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ long long __DEFAULT_FN_ATTRS128 _mm_cvttsh_i64(__m128h __A) {`。
- **L2355 EN**: Returns from the current function with `(long long)__builtin_ia32_vcvttsh2si64((__v8hf)__A,`.
  **L2355 CN**: 以 `(long long)__builtin_ia32_vcvttsh2si64((__v8hf)__A,` 从当前函数返回。
- **L2356 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2356 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2357 EN**: Closes the current lexical scope or compound statement.
  **L2357 CN**: 结束当前词法作用域或复合语句块。
- **L2358 EN**: Closes the current preprocessor conditional block.
  **L2358 CN**: 结束当前预处理条件块。
- **L2359 EN**: Blank line separating nearby declarations or logic blocks.
  **L2359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2360 EN**: Defines macro `_mm_cvtt_roundsh_u32(A, R)` for conditional compilation, shorthand, or API generation.
  **L2360 CN**: 定义宏 `_mm_cvtt_roundsh_u32(A, R)`，用于条件编译、简写或 API 生成。
- **L2361 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsh2usi32`.
  **L2361 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsh2usi32` 相关的逻辑。
- **L2362 EN**: Blank line separating nearby declarations or logic blocks.
  **L2362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2363 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned int __DEFAULT_FN_ATTRS128`.
  **L2363 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned int __DEFAULT_FN_ATTRS128`。
- **L2364 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttsh_u32(__m128h __A) {`.
  **L2364 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttsh_u32(__m128h __A) {`。
- **L2365 EN**: Returns from the current function with `(unsigned int)__builtin_ia32_vcvttsh2usi32((__v8hf)__A,`.
  **L2365 CN**: 以 `(unsigned int)__builtin_ia32_vcvttsh2usi32((__v8hf)__A,` 从当前函数返回。
- **L2366 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2366 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2367 EN**: Closes the current lexical scope or compound statement.
  **L2367 CN**: 结束当前词法作用域或复合语句块。
- **L2368 EN**: Blank line separating nearby declarations or logic blocks.
  **L2368 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2369 EN**: Starts a preprocessor conditional block: `#ifdef __x86_64__`.
  **L2369 CN**: 开始一个预处理条件块：`#ifdef __x86_64__`。
- **L2370 EN**: Defines macro `_mm_cvtt_roundsh_u64(A, R)` for conditional compilation, shorthand, or API generation.
  **L2370 CN**: 定义宏 `_mm_cvtt_roundsh_u64(A, R)`，用于条件编译、简写或 API 生成。
- **L2371 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsh2usi64`.
  **L2371 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsh2usi64` 相关的逻辑。
- **L2372 EN**: Blank line separating nearby declarations or logic blocks.
  **L2372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2373 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned long long __DEFAULT_FN_ATTRS128`.
  **L2373 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned long long __DEFAULT_FN_ATTRS128`。
- **L2374 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttsh_u64(__m128h __A) {`.
  **L2374 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttsh_u64(__m128h __A) {`。
- **L2375 EN**: Returns from the current function with `(unsigned long long)__builtin_ia32_vcvttsh2usi64(`.
  **L2375 CN**: 以 `(unsigned long long)__builtin_ia32_vcvttsh2usi64(` 从当前函数返回。
- **L2376 EN**: Executes a call or declaration centered on `statement`.
  **L2376 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 2377-2400

````c
}
#endif

#define _mm512_cvtx_roundph_ps(A, R)                                           \
  ((__m512)__builtin_ia32_vcvtph2psx512_mask((__v16hf)(A),                     \
                                             (__v16sf)_mm512_undefined_ps(),   \
                                             (__mmask16)(-1), (int)(R)))

#define _mm512_mask_cvtx_roundph_ps(W, U, A, R)                                \
  ((__m512)__builtin_ia32_vcvtph2psx512_mask((__v16hf)(A), (__v16sf)(W),       \
                                             (__mmask16)(U), (int)(R)))

#define _mm512_maskz_cvtx_roundph_ps(U, A, R)                                  \
  ((__m512)__builtin_ia32_vcvtph2psx512_mask(                                  \
      (__v16hf)(A), (__v16sf)_mm512_setzero_ps(), (__mmask16)(U), (int)(R)))

static __inline__ __m512 __DEFAULT_FN_ATTRS512 _mm512_cvtxph_ps(__m256h __A) {
  return (__m512)__builtin_ia32_vcvtph2psx512_mask(
      (__v16hf)__A, (__v16sf)_mm512_setzero_ps(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_mask_cvtxph_ps(__m512 __W, __mmask16 __U, __m256h __A) {
````
- **L2377 EN**: Closes the current lexical scope or compound statement.
  **L2377 CN**: 结束当前词法作用域或复合语句块。
- **L2378 EN**: Closes the current preprocessor conditional block.
  **L2378 CN**: 结束当前预处理条件块。
- **L2379 EN**: Blank line separating nearby declarations or logic blocks.
  **L2379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2380 EN**: Defines macro `_mm512_cvtx_roundph_ps(A, R)` for conditional compilation, shorthand, or API generation.
  **L2380 CN**: 定义宏 `_mm512_cvtx_roundph_ps(A, R)`，用于条件编译、简写或 API 生成。
- **L2381 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2psx512_mask`.
  **L2381 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2psx512_mask` 相关的逻辑。
- **L2382 EN**: Continues logic associated with callable symbol `_mm512_undefined_ps`.
  **L2382 CN**: 继续与可调用符号 `_mm512_undefined_ps` 相关的逻辑。
- **L2383 EN**: Continues the surrounding expression or declaration: `(__mmask16)(-1), (int)(R)))`.
  **L2383 CN**: 继续构造周围的表达式或声明：`(__mmask16)(-1), (int)(R)))`。
- **L2384 EN**: Blank line separating nearby declarations or logic blocks.
  **L2384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2385 EN**: Defines macro `_mm512_mask_cvtx_roundph_ps(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2385 CN**: 定义宏 `_mm512_mask_cvtx_roundph_ps(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2386 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2psx512_mask`.
  **L2386 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2psx512_mask` 相关的逻辑。
- **L2387 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L2387 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L2388 EN**: Blank line separating nearby declarations or logic blocks.
  **L2388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2389 EN**: Defines macro `_mm512_maskz_cvtx_roundph_ps(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2389 CN**: 定义宏 `_mm512_maskz_cvtx_roundph_ps(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2390 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtph2psx512_mask`.
  **L2390 CN**: 继续与可调用符号 `__builtin_ia32_vcvtph2psx512_mask` 相关的逻辑。
- **L2391 EN**: Continues logic associated with callable symbol `_mm512_setzero_ps`.
  **L2391 CN**: 继续与可调用符号 `_mm512_setzero_ps` 相关的逻辑。
- **L2392 EN**: Blank line separating nearby declarations or logic blocks.
  **L2392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2393 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m512 __DEFAULT_FN_ATTRS512 _mm512_cvtxph_ps(__m256h __A) {`.
  **L2393 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m512 __DEFAULT_FN_ATTRS512 _mm512_cvtxph_ps(__m256h __A) {`。
- **L2394 EN**: Returns from the current function with `(__m512)__builtin_ia32_vcvtph2psx512_mask(`.
  **L2394 CN**: 以 `(__m512)__builtin_ia32_vcvtph2psx512_mask(` 从当前函数返回。
- **L2395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16sf)_mm512_setzero_ps(), (__mmask16)-1,`.
  **L2395 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16sf)_mm512_setzero_ps(), (__mmask16)-1,`。
- **L2396 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2396 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2397 EN**: Closes the current lexical scope or compound statement.
  **L2397 CN**: 结束当前词法作用域或复合语句块。
- **L2398 EN**: Blank line separating nearby declarations or logic blocks.
  **L2398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2399 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L2399 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L2400 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtxph_ps(__m512 __W, __mmask16 __U, __m256h __A) {`.
  **L2400 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtxph_ps(__m512 __W, __mmask16 __U, __m256h __A) {`。

### Lines 2401-2424

````c
  return (__m512)__builtin_ia32_vcvtph2psx512_mask(
      (__v16hf)__A, (__v16sf)__W, (__mmask16)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512 __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtxph_ps(__mmask16 __U, __m256h __A) {
  return (__m512)__builtin_ia32_vcvtph2psx512_mask(
      (__v16hf)__A, (__v16sf)_mm512_setzero_ps(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_cvtx_roundps_ph(A, R)                                           \
  ((__m256h)__builtin_ia32_vcvtps2phx512_mask((__v16sf)(A),                    \
                                              (__v16hf)_mm256_undefined_ph(),  \
                                              (__mmask16)(-1), (int)(R)))

#define _mm512_mask_cvtx_roundps_ph(W, U, A, R)                                \
  ((__m256h)__builtin_ia32_vcvtps2phx512_mask((__v16sf)(A), (__v16hf)(W),      \
                                              (__mmask16)(U), (int)(R)))

#define _mm512_maskz_cvtx_roundps_ph(U, A, R)                                  \
  ((__m256h)__builtin_ia32_vcvtps2phx512_mask(                                 \
      (__v16sf)(A), (__v16hf)_mm256_setzero_ph(), (__mmask16)(U), (int)(R)))

````
- **L2401 EN**: Returns from the current function with `(__m512)__builtin_ia32_vcvtph2psx512_mask(`.
  **L2401 CN**: 以 `(__m512)__builtin_ia32_vcvtph2psx512_mask(` 从当前函数返回。
- **L2402 EN**: Executes a call or declaration centered on `statement`.
  **L2402 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2403 EN**: Closes the current lexical scope or compound statement.
  **L2403 CN**: 结束当前词法作用域或复合语句块。
- **L2404 EN**: Blank line separating nearby declarations or logic blocks.
  **L2404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2405 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512 __DEFAULT_FN_ATTRS512`.
  **L2405 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512 __DEFAULT_FN_ATTRS512`。
- **L2406 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtxph_ps(__mmask16 __U, __m256h __A) {`.
  **L2406 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtxph_ps(__mmask16 __U, __m256h __A) {`。
- **L2407 EN**: Returns from the current function with `(__m512)__builtin_ia32_vcvtph2psx512_mask(`.
  **L2407 CN**: 以 `(__m512)__builtin_ia32_vcvtph2psx512_mask(` 从当前函数返回。
- **L2408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hf)__A, (__v16sf)_mm512_setzero_ps(), (__mmask16)__U,`.
  **L2408 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hf)__A, (__v16sf)_mm512_setzero_ps(), (__mmask16)__U,`。
- **L2409 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2409 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2410 EN**: Closes the current lexical scope or compound statement.
  **L2410 CN**: 结束当前词法作用域或复合语句块。
- **L2411 EN**: Blank line separating nearby declarations or logic blocks.
  **L2411 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2412 EN**: Defines macro `_mm512_cvtx_roundps_ph(A, R)` for conditional compilation, shorthand, or API generation.
  **L2412 CN**: 定义宏 `_mm512_cvtx_roundps_ph(A, R)`，用于条件编译、简写或 API 生成。
- **L2413 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtps2phx512_mask`.
  **L2413 CN**: 继续与可调用符号 `__builtin_ia32_vcvtps2phx512_mask` 相关的逻辑。
- **L2414 EN**: Continues logic associated with callable symbol `_mm256_undefined_ph`.
  **L2414 CN**: 继续与可调用符号 `_mm256_undefined_ph` 相关的逻辑。
- **L2415 EN**: Continues the surrounding expression or declaration: `(__mmask16)(-1), (int)(R)))`.
  **L2415 CN**: 继续构造周围的表达式或声明：`(__mmask16)(-1), (int)(R)))`。
- **L2416 EN**: Blank line separating nearby declarations or logic blocks.
  **L2416 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2417 EN**: Defines macro `_mm512_mask_cvtx_roundps_ph(W, U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2417 CN**: 定义宏 `_mm512_mask_cvtx_roundps_ph(W, U, A, R)`，用于条件编译、简写或 API 生成。
- **L2418 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtps2phx512_mask`.
  **L2418 CN**: 继续与可调用符号 `__builtin_ia32_vcvtps2phx512_mask` 相关的逻辑。
- **L2419 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L2419 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L2420 EN**: Blank line separating nearby declarations or logic blocks.
  **L2420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2421 EN**: Defines macro `_mm512_maskz_cvtx_roundps_ph(U, A, R)` for conditional compilation, shorthand, or API generation.
  **L2421 CN**: 定义宏 `_mm512_maskz_cvtx_roundps_ph(U, A, R)`，用于条件编译、简写或 API 生成。
- **L2422 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvtps2phx512_mask`.
  **L2422 CN**: 继续与可调用符号 `__builtin_ia32_vcvtps2phx512_mask` 相关的逻辑。
- **L2423 EN**: Continues logic associated with callable symbol `_mm256_setzero_ph`.
  **L2423 CN**: 继续与可调用符号 `_mm256_setzero_ph` 相关的逻辑。
- **L2424 EN**: Blank line separating nearby declarations or logic blocks.
  **L2424 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2425-2448

````c
static __inline__ __m256h __DEFAULT_FN_ATTRS512 _mm512_cvtxps_ph(__m512 __A) {
  return (__m256h)__builtin_ia32_vcvtps2phx512_mask(
      (__v16sf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS512
_mm512_mask_cvtxps_ph(__m256h __W, __mmask16 __U, __m512 __A) {
  return (__m256h)__builtin_ia32_vcvtps2phx512_mask(
      (__v16sf)__A, (__v16hf)__W, (__mmask16)__U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m256h __DEFAULT_FN_ATTRS512
_mm512_maskz_cvtxps_ph(__mmask16 __U, __m512 __A) {
  return (__m256h)__builtin_ia32_vcvtps2phx512_mask(
      (__v16sf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_fmadd_round_ph(A, B, C, R)                                      \
  ((__m512h)__builtin_ia32_vfmaddph512_mask(                                   \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)-1, (int)(R)))

````
- **L2425 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256h __DEFAULT_FN_ATTRS512 _mm512_cvtxps_ph(__m512 __A) {`.
  **L2425 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256h __DEFAULT_FN_ATTRS512 _mm512_cvtxps_ph(__m512 __A) {`。
- **L2426 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtps2phx512_mask(`.
  **L2426 CN**: 以 `(__m256h)__builtin_ia32_vcvtps2phx512_mask(` 从当前函数返回。
- **L2427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,`.
  **L2427 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)-1,`。
- **L2428 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2428 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2429 EN**: Closes the current lexical scope or compound statement.
  **L2429 CN**: 结束当前词法作用域或复合语句块。
- **L2430 EN**: Blank line separating nearby declarations or logic blocks.
  **L2430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2431 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512`.
  **L2431 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512`。
- **L2432 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_cvtxps_ph(__m256h __W, __mmask16 __U, __m512 __A) {`.
  **L2432 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_cvtxps_ph(__m256h __W, __mmask16 __U, __m512 __A) {`。
- **L2433 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtps2phx512_mask(`.
  **L2433 CN**: 以 `(__m256h)__builtin_ia32_vcvtps2phx512_mask(` 从当前函数返回。
- **L2434 EN**: Executes a call or declaration centered on `statement`.
  **L2434 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2435 EN**: Closes the current lexical scope or compound statement.
  **L2435 CN**: 结束当前词法作用域或复合语句块。
- **L2436 EN**: Blank line separating nearby declarations or logic blocks.
  **L2436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2437 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256h __DEFAULT_FN_ATTRS512`.
  **L2437 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256h __DEFAULT_FN_ATTRS512`。
- **L2438 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_cvtxps_ph(__mmask16 __U, __m512 __A) {`.
  **L2438 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_cvtxps_ph(__mmask16 __U, __m512 __A) {`。
- **L2439 EN**: Returns from the current function with `(__m256h)__builtin_ia32_vcvtps2phx512_mask(`.
  **L2439 CN**: 以 `(__m256h)__builtin_ia32_vcvtps2phx512_mask(` 从当前函数返回。
- **L2440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,`.
  **L2440 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16hf)_mm256_setzero_ph(), (__mmask16)__U,`。
- **L2441 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2441 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2442 EN**: Closes the current lexical scope or compound statement.
  **L2442 CN**: 结束当前词法作用域或复合语句块。
- **L2443 EN**: Blank line separating nearby declarations or logic blocks.
  **L2443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2444 EN**: Defines macro `_mm512_fmadd_round_ph(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2444 CN**: 定义宏 `_mm512_fmadd_round_ph(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2445 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask`.
  **L2445 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask` 相关的逻辑。
- **L2446 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2446 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2447 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L2447 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L2448 EN**: Blank line separating nearby declarations or logic blocks.
  **L2448 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2449-2472

````c
#define _mm512_mask_fmadd_round_ph(A, U, B, C, R)                              \
  ((__m512h)__builtin_ia32_vfmaddph512_mask(                                   \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)(U), (int)(R)))

#define _mm512_mask3_fmadd_round_ph(A, B, C, U, R)                             \
  ((__m512h)__builtin_ia32_vfmaddph512_mask3(                                  \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)(U), (int)(R)))

#define _mm512_maskz_fmadd_round_ph(U, A, B, C, R)                             \
  ((__m512h)__builtin_ia32_vfmaddph512_maskz(                                  \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)(U), (int)(R)))

#define _mm512_fmsub_round_ph(A, B, C, R)                                      \
  ((__m512h)__builtin_ia32_vfmaddph512_mask(                                   \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \
      (__mmask32)-1, (int)(R)))

#define _mm512_mask_fmsub_round_ph(A, U, B, C, R)                              \
  ((__m512h)__builtin_ia32_vfmaddph512_mask(                                   \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \
      (__mmask32)(U), (int)(R)))
````
- **L2449 EN**: Defines macro `_mm512_mask_fmadd_round_ph(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2449 CN**: 定义宏 `_mm512_mask_fmadd_round_ph(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2450 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask`.
  **L2450 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask` 相关的逻辑。
- **L2451 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2451 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2452 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2452 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2453 EN**: Blank line separating nearby declarations or logic blocks.
  **L2453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2454 EN**: Defines macro `_mm512_mask3_fmadd_round_ph(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L2454 CN**: 定义宏 `_mm512_mask3_fmadd_round_ph(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L2455 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask3`.
  **L2455 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask3` 相关的逻辑。
- **L2456 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2456 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2457 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2457 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2458 EN**: Blank line separating nearby declarations or logic blocks.
  **L2458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2459 EN**: Defines macro `_mm512_maskz_fmadd_round_ph(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2459 CN**: 定义宏 `_mm512_maskz_fmadd_round_ph(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2460 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_maskz`.
  **L2460 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_maskz` 相关的逻辑。
- **L2461 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2461 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2462 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2462 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2463 EN**: Blank line separating nearby declarations or logic blocks.
  **L2463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2464 EN**: Defines macro `_mm512_fmsub_round_ph(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2464 CN**: 定义宏 `_mm512_fmsub_round_ph(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2465 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask`.
  **L2465 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask` 相关的逻辑。
- **L2466 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`.
  **L2466 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`。
- **L2467 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L2467 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L2468 EN**: Blank line separating nearby declarations or logic blocks.
  **L2468 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2469 EN**: Defines macro `_mm512_mask_fmsub_round_ph(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2469 CN**: 定义宏 `_mm512_mask_fmsub_round_ph(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2470 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask`.
  **L2470 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask` 相关的逻辑。
- **L2471 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`.
  **L2471 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`。
- **L2472 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2472 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。

### Lines 2473-2496

````c

#define _mm512_maskz_fmsub_round_ph(U, A, B, C, R)                             \
  ((__m512h)__builtin_ia32_vfmaddph512_maskz(                                  \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \
      (__mmask32)(U), (int)(R)))

#define _mm512_fnmadd_round_ph(A, B, C, R)                                     \
  ((__m512h)__builtin_ia32_vfmaddph512_mask(                                   \
      (__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \
      (__mmask32)-1, (int)(R)))

#define _mm512_mask3_fnmadd_round_ph(A, B, C, U, R)                            \
  ((__m512h)__builtin_ia32_vfmaddph512_mask3(                                  \
      -(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \
      (__mmask32)(U), (int)(R)))

#define _mm512_maskz_fnmadd_round_ph(U, A, B, C, R)                            \
  ((__m512h)__builtin_ia32_vfmaddph512_maskz(                                  \
      -(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \
      (__mmask32)(U), (int)(R)))

#define _mm512_fnmsub_round_ph(A, B, C, R)                                     \
  ((__m512h)__builtin_ia32_vfmaddph512_mask(                                   \
      (__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \
````
- **L2473 EN**: Blank line separating nearby declarations or logic blocks.
  **L2473 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2474 EN**: Defines macro `_mm512_maskz_fmsub_round_ph(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2474 CN**: 定义宏 `_mm512_maskz_fmsub_round_ph(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2475 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_maskz`.
  **L2475 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_maskz` 相关的逻辑。
- **L2476 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`.
  **L2476 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`。
- **L2477 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2477 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2478 EN**: Blank line separating nearby declarations or logic blocks.
  **L2478 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2479 EN**: Defines macro `_mm512_fnmadd_round_ph(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2479 CN**: 定义宏 `_mm512_fnmadd_round_ph(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2480 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask`.
  **L2480 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask` 相关的逻辑。
- **L2481 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`.
  **L2481 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`。
- **L2482 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L2482 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L2483 EN**: Blank line separating nearby declarations or logic blocks.
  **L2483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2484 EN**: Defines macro `_mm512_mask3_fnmadd_round_ph(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L2484 CN**: 定义宏 `_mm512_mask3_fnmadd_round_ph(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L2485 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask3`.
  **L2485 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask3` 相关的逻辑。
- **L2486 EN**: Continues the surrounding expression or declaration: `-(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`.
  **L2486 CN**: 继续构造周围的表达式或声明：`-(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`。
- **L2487 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2487 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2488 EN**: Blank line separating nearby declarations or logic blocks.
  **L2488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2489 EN**: Defines macro `_mm512_maskz_fnmadd_round_ph(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2489 CN**: 定义宏 `_mm512_maskz_fnmadd_round_ph(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2490 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_maskz`.
  **L2490 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_maskz` 相关的逻辑。
- **L2491 EN**: Continues the surrounding expression or declaration: `-(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`.
  **L2491 CN**: 继续构造周围的表达式或声明：`-(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`。
- **L2492 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2492 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2493 EN**: Blank line separating nearby declarations or logic blocks.
  **L2493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2494 EN**: Defines macro `_mm512_fnmsub_round_ph(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2494 CN**: 定义宏 `_mm512_fnmsub_round_ph(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2495 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask`.
  **L2495 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask` 相关的逻辑。
- **L2496 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \`.
  **L2496 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \`。

### Lines 2497-2520

````c
      (__mmask32)-1, (int)(R)))

#define _mm512_maskz_fnmsub_round_ph(U, A, B, C, R)                            \
  ((__m512h)__builtin_ia32_vfmaddph512_maskz(                                  \
      -(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \
      (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmadd_ph(__m512h __A,
                                                                __m512h __B,
                                                                __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,
                                                  (__v32hf)__C, (__mmask32)-1,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fmadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,
                                                  (__v32hf)__C, (__mmask32)__U,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask3_fmadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {
````
- **L2497 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L2497 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L2498 EN**: Blank line separating nearby declarations or logic blocks.
  **L2498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2499 EN**: Defines macro `_mm512_maskz_fnmsub_round_ph(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2499 CN**: 定义宏 `_mm512_maskz_fnmsub_round_ph(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2500 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_maskz`.
  **L2500 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_maskz` 相关的逻辑。
- **L2501 EN**: Continues the surrounding expression or declaration: `-(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \`.
  **L2501 CN**: 继续构造周围的表达式或声明：`-(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \`。
- **L2502 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2502 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2503 EN**: Blank line separating nearby declarations or logic blocks.
  **L2503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmadd_ph(__m512h __A,`.
  **L2504 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmadd_ph(__m512h __A,`。
- **L2505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m512h __B,`.
  **L2505 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m512h __B,`。
- **L2506 EN**: Continues the surrounding expression or declaration: `__m512h __C) {`.
  **L2506 CN**: 继续构造周围的表达式或声明：`__m512h __C) {`。
- **L2507 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,`.
  **L2507 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)-1,`.
  **L2508 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)-1,`。
- **L2509 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2509 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2510 EN**: Closes the current lexical scope or compound statement.
  **L2510 CN**: 结束当前词法作用域或复合语句块。
- **L2511 EN**: Blank line separating nearby declarations or logic blocks.
  **L2511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2512 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2512 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2513 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fmadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`.
  **L2513 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fmadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`。
- **L2514 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,`.
  **L2514 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)__U,`.
  **L2515 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)__U,`。
- **L2516 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2516 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2517 EN**: Closes the current lexical scope or compound statement.
  **L2517 CN**: 结束当前词法作用域或复合语句块。
- **L2518 EN**: Blank line separating nearby declarations or logic blocks.
  **L2518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2519 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2519 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2520 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask3_fmadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`.
  **L2520 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask3_fmadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`。

### Lines 2521-2544

````c
  return (__m512h)__builtin_ia32_vfmaddph512_mask3((__v32hf)__A, (__v32hf)__B,
                                                   (__v32hf)__C, (__mmask32)__U,
                                                   _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fmadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_maskz((__v32hf)__A, (__v32hf)__B,
                                                   (__v32hf)__C, (__mmask32)__U,
                                                   _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmsub_ph(__m512h __A,
                                                                __m512h __B,
                                                                __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,
                                                  -(__v32hf)__C, (__mmask32)-1,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fmsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,
                                                  -(__v32hf)__C, (__mmask32)__U,
````
- **L2521 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask3((__v32hf)__A, (__v32hf)__B,`.
  **L2521 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask3((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)__U,`.
  **L2522 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)__U,`。
- **L2523 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2523 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2524 EN**: Closes the current lexical scope or compound statement.
  **L2524 CN**: 结束当前词法作用域或复合语句块。
- **L2525 EN**: Blank line separating nearby declarations or logic blocks.
  **L2525 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2526 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2526 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2527 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fmadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`.
  **L2527 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fmadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`。
- **L2528 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_maskz((__v32hf)__A, (__v32hf)__B,`.
  **L2528 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_maskz((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)__U,`.
  **L2529 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)__U,`。
- **L2530 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2530 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2531 EN**: Closes the current lexical scope or compound statement.
  **L2531 CN**: 结束当前词法作用域或复合语句块。
- **L2532 EN**: Blank line separating nearby declarations or logic blocks.
  **L2532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmsub_ph(__m512h __A,`.
  **L2533 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmsub_ph(__m512h __A,`。
- **L2534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m512h __B,`.
  **L2534 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m512h __B,`。
- **L2535 EN**: Continues the surrounding expression or declaration: `__m512h __C) {`.
  **L2535 CN**: 继续构造周围的表达式或声明：`__m512h __C) {`。
- **L2536 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,`.
  **L2536 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-(__v32hf)__C, (__mmask32)-1,`.
  **L2537 CN**: 继续一个多行参数列表、初始化器或聚合项：`-(__v32hf)__C, (__mmask32)-1,`。
- **L2538 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2538 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2539 EN**: Closes the current lexical scope or compound statement.
  **L2539 CN**: 结束当前词法作用域或复合语句块。
- **L2540 EN**: Blank line separating nearby declarations or logic blocks.
  **L2540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2541 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2541 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2542 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fmsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`.
  **L2542 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fmsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`。
- **L2543 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,`.
  **L2543 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-(__v32hf)__C, (__mmask32)__U,`.
  **L2544 CN**: 继续一个多行参数列表、初始化器或聚合项：`-(__v32hf)__C, (__mmask32)__U,`。

### Lines 2545-2568

````c
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fmsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_maskz(
      (__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fnmadd_ph(__m512h __A,
                                                                 __m512h __B,
                                                                 __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,
                                                  (__v32hf)__C, (__mmask32)-1,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask3_fnmadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask3(-(__v32hf)__A, (__v32hf)__B,
                                                   (__v32hf)__C, (__mmask32)__U,
                                                   _MM_FROUND_CUR_DIRECTION);
}
````
- **L2545 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2545 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2546 EN**: Closes the current lexical scope or compound statement.
  **L2546 CN**: 结束当前词法作用域或复合语句块。
- **L2547 EN**: Blank line separating nearby declarations or logic blocks.
  **L2547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2548 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2548 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2549 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fmsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`.
  **L2549 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fmsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`。
- **L2550 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_maskz(`.
  **L2550 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_maskz(` 从当前函数返回。
- **L2551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,`.
  **L2551 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,`。
- **L2552 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2552 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2553 EN**: Closes the current lexical scope or compound statement.
  **L2553 CN**: 结束当前词法作用域或复合语句块。
- **L2554 EN**: Blank line separating nearby declarations or logic blocks.
  **L2554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fnmadd_ph(__m512h __A,`.
  **L2555 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fnmadd_ph(__m512h __A,`。
- **L2556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m512h __B,`.
  **L2556 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m512h __B,`。
- **L2557 EN**: Continues the surrounding expression or declaration: `__m512h __C) {`.
  **L2557 CN**: 继续构造周围的表达式或声明：`__m512h __C) {`。
- **L2558 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,`.
  **L2558 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,` 从当前函数返回。
- **L2559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)-1,`.
  **L2559 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)-1,`。
- **L2560 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2560 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2561 EN**: Closes the current lexical scope or compound statement.
  **L2561 CN**: 结束当前词法作用域或复合语句块。
- **L2562 EN**: Blank line separating nearby declarations or logic blocks.
  **L2562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2563 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2563 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2564 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask3_fnmadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`.
  **L2564 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask3_fnmadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`。
- **L2565 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask3(-(__v32hf)__A, (__v32hf)__B,`.
  **L2565 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask3(-(__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)__U,`.
  **L2566 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)__U,`。
- **L2567 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2567 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2568 EN**: Closes the current lexical scope or compound statement.
  **L2568 CN**: 结束当前词法作用域或复合语句块。

### Lines 2569-2592

````c

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fnmadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_maskz(-(__v32hf)__A, (__v32hf)__B,
                                                   (__v32hf)__C, (__mmask32)__U,
                                                   _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fnmsub_ph(__m512h __A,
                                                                 __m512h __B,
                                                                 __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,
                                                  -(__v32hf)__C, (__mmask32)-1,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fnmsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_maskz(
      -(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_fmaddsub_round_ph(A, B, C, R)                                   \
````
- **L2569 EN**: Blank line separating nearby declarations or logic blocks.
  **L2569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2570 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2570 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2571 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fnmadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`.
  **L2571 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fnmadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`。
- **L2572 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_maskz(-(__v32hf)__A, (__v32hf)__B,`.
  **L2572 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_maskz(-(__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)__U,`.
  **L2573 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)__U,`。
- **L2574 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2574 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2575 EN**: Closes the current lexical scope or compound statement.
  **L2575 CN**: 结束当前词法作用域或复合语句块。
- **L2576 EN**: Blank line separating nearby declarations or logic blocks.
  **L2576 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fnmsub_ph(__m512h __A,`.
  **L2577 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fnmsub_ph(__m512h __A,`。
- **L2578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m512h __B,`.
  **L2578 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m512h __B,`。
- **L2579 EN**: Continues the surrounding expression or declaration: `__m512h __C) {`.
  **L2579 CN**: 继续构造周围的表达式或声明：`__m512h __C) {`。
- **L2580 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,`.
  **L2580 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,` 从当前函数返回。
- **L2581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-(__v32hf)__C, (__mmask32)-1,`.
  **L2581 CN**: 继续一个多行参数列表、初始化器或聚合项：`-(__v32hf)__C, (__mmask32)-1,`。
- **L2582 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2582 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2583 EN**: Closes the current lexical scope or compound statement.
  **L2583 CN**: 结束当前词法作用域或复合语句块。
- **L2584 EN**: Blank line separating nearby declarations or logic blocks.
  **L2584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2585 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2585 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2586 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fnmsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`.
  **L2586 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fnmsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`。
- **L2587 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_maskz(`.
  **L2587 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_maskz(` 从当前函数返回。
- **L2588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,`.
  **L2588 CN**: 继续一个多行参数列表、初始化器或聚合项：`-(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,`。
- **L2589 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2589 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2590 EN**: Closes the current lexical scope or compound statement.
  **L2590 CN**: 结束当前词法作用域或复合语句块。
- **L2591 EN**: Blank line separating nearby declarations or logic blocks.
  **L2591 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2592 EN**: Defines macro `_mm512_fmaddsub_round_ph(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2592 CN**: 定义宏 `_mm512_fmaddsub_round_ph(A, B, C, R)`，用于条件编译、简写或 API 生成。

### Lines 2593-2616

````c
  ((__m512h)__builtin_ia32_vfmaddsubph512_mask(                                \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)-1, (int)(R)))

#define _mm512_mask_fmaddsub_round_ph(A, U, B, C, R)                           \
  ((__m512h)__builtin_ia32_vfmaddsubph512_mask(                                \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)(U), (int)(R)))

#define _mm512_mask3_fmaddsub_round_ph(A, B, C, U, R)                          \
  ((__m512h)__builtin_ia32_vfmaddsubph512_mask3(                               \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)(U), (int)(R)))

#define _mm512_maskz_fmaddsub_round_ph(U, A, B, C, R)                          \
  ((__m512h)__builtin_ia32_vfmaddsubph512_maskz(                               \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)(U), (int)(R)))

#define _mm512_fmsubadd_round_ph(A, B, C, R)                                   \
  ((__m512h)__builtin_ia32_vfmaddsubph512_mask(                                \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \
      (__mmask32)-1, (int)(R)))

````
- **L2593 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsubph512_mask`.
  **L2593 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsubph512_mask` 相关的逻辑。
- **L2594 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2594 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2595 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L2595 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L2596 EN**: Blank line separating nearby declarations or logic blocks.
  **L2596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2597 EN**: Defines macro `_mm512_mask_fmaddsub_round_ph(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2597 CN**: 定义宏 `_mm512_mask_fmaddsub_round_ph(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2598 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsubph512_mask`.
  **L2598 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsubph512_mask` 相关的逻辑。
- **L2599 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2599 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2600 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2600 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2601 EN**: Blank line separating nearby declarations or logic blocks.
  **L2601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2602 EN**: Defines macro `_mm512_mask3_fmaddsub_round_ph(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L2602 CN**: 定义宏 `_mm512_mask3_fmaddsub_round_ph(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L2603 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsubph512_mask3`.
  **L2603 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsubph512_mask3` 相关的逻辑。
- **L2604 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2604 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2605 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2605 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2606 EN**: Blank line separating nearby declarations or logic blocks.
  **L2606 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2607 EN**: Defines macro `_mm512_maskz_fmaddsub_round_ph(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2607 CN**: 定义宏 `_mm512_maskz_fmaddsub_round_ph(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2608 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsubph512_maskz`.
  **L2608 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsubph512_maskz` 相关的逻辑。
- **L2609 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2609 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2610 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2610 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2611 EN**: Blank line separating nearby declarations or logic blocks.
  **L2611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2612 EN**: Defines macro `_mm512_fmsubadd_round_ph(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2612 CN**: 定义宏 `_mm512_fmsubadd_round_ph(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2613 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsubph512_mask`.
  **L2613 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsubph512_mask` 相关的逻辑。
- **L2614 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`.
  **L2614 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`。
- **L2615 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1, (int)(R)))`.
  **L2615 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1, (int)(R)))`。
- **L2616 EN**: Blank line separating nearby declarations or logic blocks.
  **L2616 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2617-2640

````c
#define _mm512_mask_fmsubadd_round_ph(A, U, B, C, R)                           \
  ((__m512h)__builtin_ia32_vfmaddsubph512_mask(                                \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \
      (__mmask32)(U), (int)(R)))

#define _mm512_maskz_fmsubadd_round_ph(U, A, B, C, R)                          \
  ((__m512h)__builtin_ia32_vfmaddsubph512_maskz(                               \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \
      (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_fmaddsub_ph(__m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddsubph512_mask(
      (__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fmaddsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddsubph512_mask(
      (__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

````
- **L2617 EN**: Defines macro `_mm512_mask_fmsubadd_round_ph(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2617 CN**: 定义宏 `_mm512_mask_fmsubadd_round_ph(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2618 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsubph512_mask`.
  **L2618 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsubph512_mask` 相关的逻辑。
- **L2619 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`.
  **L2619 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`。
- **L2620 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2620 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2621 EN**: Blank line separating nearby declarations or logic blocks.
  **L2621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2622 EN**: Defines macro `_mm512_maskz_fmsubadd_round_ph(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2622 CN**: 定义宏 `_mm512_maskz_fmsubadd_round_ph(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2623 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsubph512_maskz`.
  **L2623 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsubph512_maskz` 相关的逻辑。
- **L2624 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`.
  **L2624 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),    \`。
- **L2625 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2625 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2626 EN**: Blank line separating nearby declarations or logic blocks.
  **L2626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2627 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2627 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2628 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_fmaddsub_ph(__m512h __A, __m512h __B, __m512h __C) {`.
  **L2628 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_fmaddsub_ph(__m512h __A, __m512h __B, __m512h __C) {`。
- **L2629 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddsubph512_mask(`.
  **L2629 CN**: 以 `(__m512h)__builtin_ia32_vfmaddsubph512_mask(` 从当前函数返回。
- **L2630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)-1,`.
  **L2630 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)-1,`。
- **L2631 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2631 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2632 EN**: Closes the current lexical scope or compound statement.
  **L2632 CN**: 结束当前词法作用域或复合语句块。
- **L2633 EN**: Blank line separating nearby declarations or logic blocks.
  **L2633 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2634 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2634 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2635 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fmaddsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`.
  **L2635 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fmaddsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`。
- **L2636 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddsubph512_mask(`.
  **L2636 CN**: 以 `(__m512h)__builtin_ia32_vfmaddsubph512_mask(` 从当前函数返回。
- **L2637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,`.
  **L2637 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,`。
- **L2638 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2638 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2639 EN**: Closes the current lexical scope or compound statement.
  **L2639 CN**: 结束当前词法作用域或复合语句块。
- **L2640 EN**: Blank line separating nearby declarations or logic blocks.
  **L2640 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2641-2664

````c
static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask3_fmaddsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {
  return (__m512h)__builtin_ia32_vfmaddsubph512_mask3(
      (__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fmaddsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddsubph512_maskz(
      (__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_fmsubadd_ph(__m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddsubph512_mask(
      (__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fmsubadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddsubph512_mask(
````
- **L2641 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2641 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2642 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask3_fmaddsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`.
  **L2642 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask3_fmaddsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`。
- **L2643 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddsubph512_mask3(`.
  **L2643 CN**: 以 `(__m512h)__builtin_ia32_vfmaddsubph512_mask3(` 从当前函数返回。
- **L2644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,`.
  **L2644 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,`。
- **L2645 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2645 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2646 EN**: Closes the current lexical scope or compound statement.
  **L2646 CN**: 结束当前词法作用域或复合语句块。
- **L2647 EN**: Blank line separating nearby declarations or logic blocks.
  **L2647 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2648 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2648 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2649 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fmaddsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`.
  **L2649 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fmaddsub_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`。
- **L2650 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddsubph512_maskz(`.
  **L2650 CN**: 以 `(__m512h)__builtin_ia32_vfmaddsubph512_maskz(` 从当前函数返回。
- **L2651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,`.
  **L2651 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,`。
- **L2652 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2652 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2653 EN**: Closes the current lexical scope or compound statement.
  **L2653 CN**: 结束当前词法作用域或复合语句块。
- **L2654 EN**: Blank line separating nearby declarations or logic blocks.
  **L2654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2655 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2655 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2656 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_fmsubadd_ph(__m512h __A, __m512h __B, __m512h __C) {`.
  **L2656 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_fmsubadd_ph(__m512h __A, __m512h __B, __m512h __C) {`。
- **L2657 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddsubph512_mask(`.
  **L2657 CN**: 以 `(__m512h)__builtin_ia32_vfmaddsubph512_mask(` 从当前函数返回。
- **L2658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)-1,`.
  **L2658 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)-1,`。
- **L2659 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2659 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2660 EN**: Closes the current lexical scope or compound statement.
  **L2660 CN**: 结束当前词法作用域或复合语句块。
- **L2661 EN**: Blank line separating nearby declarations or logic blocks.
  **L2661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2662 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2662 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2663 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fmsubadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`.
  **L2663 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fmsubadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`。
- **L2664 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddsubph512_mask(`.
  **L2664 CN**: 以 `(__m512h)__builtin_ia32_vfmaddsubph512_mask(` 从当前函数返回。

### Lines 2665-2688

````c
      (__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fmsubadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddsubph512_maskz(
      (__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_mask3_fmsub_round_ph(A, B, C, U, R)                             \
  ((__m512h)__builtin_ia32_vfmsubph512_mask3(                                  \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask3_fmsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {
  return (__m512h)__builtin_ia32_vfmsubph512_mask3((__v32hf)__A, (__v32hf)__B,
                                                   (__v32hf)__C, (__mmask32)__U,
                                                   _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_mask3_fmsubadd_round_ph(A, B, C, U, R)                          \
````
- **L2665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,`.
  **L2665 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,`。
- **L2666 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2666 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2667 EN**: Closes the current lexical scope or compound statement.
  **L2667 CN**: 结束当前词法作用域或复合语句块。
- **L2668 EN**: Blank line separating nearby declarations or logic blocks.
  **L2668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2669 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2669 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2670 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fmsubadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`.
  **L2670 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fmsubadd_ph(__mmask32 __U, __m512h __A, __m512h __B, __m512h __C) {`。
- **L2671 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddsubph512_maskz(`.
  **L2671 CN**: 以 `(__m512h)__builtin_ia32_vfmaddsubph512_maskz(` 从当前函数返回。
- **L2672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,`.
  **L2672 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, -(__v32hf)__C, (__mmask32)__U,`。
- **L2673 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2673 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2674 EN**: Closes the current lexical scope or compound statement.
  **L2674 CN**: 结束当前词法作用域或复合语句块。
- **L2675 EN**: Blank line separating nearby declarations or logic blocks.
  **L2675 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2676 EN**: Defines macro `_mm512_mask3_fmsub_round_ph(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L2676 CN**: 定义宏 `_mm512_mask3_fmsub_round_ph(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L2677 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmsubph512_mask3`.
  **L2677 CN**: 继续与可调用符号 `__builtin_ia32_vfmsubph512_mask3` 相关的逻辑。
- **L2678 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2678 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2679 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2679 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2680 EN**: Blank line separating nearby declarations or logic blocks.
  **L2680 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2681 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2681 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2682 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask3_fmsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`.
  **L2682 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask3_fmsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`。
- **L2683 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmsubph512_mask3((__v32hf)__A, (__v32hf)__B,`.
  **L2683 CN**: 以 `(__m512h)__builtin_ia32_vfmsubph512_mask3((__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)__U,`.
  **L2684 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)__U,`。
- **L2685 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2685 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2686 EN**: Closes the current lexical scope or compound statement.
  **L2686 CN**: 结束当前词法作用域或复合语句块。
- **L2687 EN**: Blank line separating nearby declarations or logic blocks.
  **L2687 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2688 EN**: Defines macro `_mm512_mask3_fmsubadd_round_ph(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L2688 CN**: 定义宏 `_mm512_mask3_fmsubadd_round_ph(A, B, C, U, R)`，用于条件编译、简写或 API 生成。

### Lines 2689-2712

````c
  ((__m512h)__builtin_ia32_vfmsubaddph512_mask3(                               \
      (__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \
      (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask3_fmsubadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {
  return (__m512h)__builtin_ia32_vfmsubaddph512_mask3(
      (__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_mask_fnmadd_round_ph(A, U, B, C, R)                             \
  ((__m512h)__builtin_ia32_vfmaddph512_mask(                                   \
      (__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \
      (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fnmadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,
                                                  (__v32hf)__C, (__mmask32)__U,
                                                  _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_mask_fnmsub_round_ph(A, U, B, C, R)                             \
````
- **L2689 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmsubaddph512_mask3`.
  **L2689 CN**: 继续与可调用符号 `__builtin_ia32_vfmsubaddph512_mask3` 相关的逻辑。
- **L2690 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`.
  **L2690 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),     \`。
- **L2691 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2691 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2692 EN**: Blank line separating nearby declarations or logic blocks.
  **L2692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2693 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2693 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2694 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask3_fmsubadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`.
  **L2694 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask3_fmsubadd_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`。
- **L2695 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmsubaddph512_mask3(`.
  **L2695 CN**: 以 `(__m512h)__builtin_ia32_vfmsubaddph512_mask3(` 从当前函数返回。
- **L2696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,`.
  **L2696 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__A, (__v32hf)__B, (__v32hf)__C, (__mmask32)__U,`。
- **L2697 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2697 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2698 EN**: Closes the current lexical scope or compound statement.
  **L2698 CN**: 结束当前词法作用域或复合语句块。
- **L2699 EN**: Blank line separating nearby declarations or logic blocks.
  **L2699 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2700 EN**: Defines macro `_mm512_mask_fnmadd_round_ph(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2700 CN**: 定义宏 `_mm512_mask_fnmadd_round_ph(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2701 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask`.
  **L2701 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask` 相关的逻辑。
- **L2702 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`.
  **L2702 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`。
- **L2703 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2703 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2704 EN**: Blank line separating nearby declarations or logic blocks.
  **L2704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2705 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2705 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2706 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fnmadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`.
  **L2706 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fnmadd_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`。
- **L2707 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,`.
  **L2707 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,` 从当前函数返回。
- **L2708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)__U,`.
  **L2708 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)__U,`。
- **L2709 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2709 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2710 EN**: Closes the current lexical scope or compound statement.
  **L2710 CN**: 结束当前词法作用域或复合语句块。
- **L2711 EN**: Blank line separating nearby declarations or logic blocks.
  **L2711 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2712 EN**: Defines macro `_mm512_mask_fnmsub_round_ph(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2712 CN**: 定义宏 `_mm512_mask_fnmsub_round_ph(A, U, B, C, R)`，用于条件编译、简写或 API 生成。

### Lines 2713-2736

````c
  ((__m512h)__builtin_ia32_vfmaddph512_mask(                                   \
      (__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \
      (__mmask32)(U), (int)(R)))

#define _mm512_mask3_fnmsub_round_ph(A, B, C, U, R)                            \
  ((__m512h)__builtin_ia32_vfmsubph512_mask3(                                  \
      -(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \
      (__mmask32)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fnmsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,
                                                  -(__v32hf)__C, (__mmask32)__U,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask3_fnmsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {
  return (__m512h)__builtin_ia32_vfmsubph512_mask3(-(__v32hf)__A, (__v32hf)__B,
                                                   (__v32hf)__C, (__mmask32)__U,
                                                   _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_sh(__m128h __W,
````
- **L2713 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddph512_mask`.
  **L2713 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddph512_mask` 相关的逻辑。
- **L2714 EN**: Continues the surrounding expression or declaration: `(__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \`.
  **L2714 CN**: 继续构造周围的表达式或声明：`(__v32hf)(__m512h)(A), -(__v32hf)(__m512h)(B), -(__v32hf)(__m512h)(C),   \`。
- **L2715 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2715 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2716 EN**: Blank line separating nearby declarations or logic blocks.
  **L2716 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2717 EN**: Defines macro `_mm512_mask3_fnmsub_round_ph(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L2717 CN**: 定义宏 `_mm512_mask3_fnmsub_round_ph(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L2718 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmsubph512_mask3`.
  **L2718 CN**: 继续与可调用符号 `__builtin_ia32_vfmsubph512_mask3` 相关的逻辑。
- **L2719 EN**: Continues the surrounding expression or declaration: `-(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`.
  **L2719 CN**: 继续构造周围的表达式或声明：`-(__v32hf)(__m512h)(A), (__v32hf)(__m512h)(B), (__v32hf)(__m512h)(C),    \`。
- **L2720 EN**: Continues the surrounding expression or declaration: `(__mmask32)(U), (int)(R)))`.
  **L2720 CN**: 继续构造周围的表达式或声明：`(__mmask32)(U), (int)(R)))`。
- **L2721 EN**: Blank line separating nearby declarations or logic blocks.
  **L2721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2722 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2722 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2723 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fnmsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`.
  **L2723 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fnmsub_ph(__m512h __A, __mmask32 __U, __m512h __B, __m512h __C) {`。
- **L2724 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,`.
  **L2724 CN**: 以 `(__m512h)__builtin_ia32_vfmaddph512_mask((__v32hf)__A, -(__v32hf)__B,` 从当前函数返回。
- **L2725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-(__v32hf)__C, (__mmask32)__U,`.
  **L2725 CN**: 继续一个多行参数列表、初始化器或聚合项：`-(__v32hf)__C, (__mmask32)__U,`。
- **L2726 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2726 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2727 EN**: Closes the current lexical scope or compound statement.
  **L2727 CN**: 结束当前词法作用域或复合语句块。
- **L2728 EN**: Blank line separating nearby declarations or logic blocks.
  **L2728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2729 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L2729 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L2730 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask3_fnmsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`.
  **L2730 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask3_fnmsub_ph(__m512h __A, __m512h __B, __m512h __C, __mmask32 __U) {`。
- **L2731 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmsubph512_mask3(-(__v32hf)__A, (__v32hf)__B,`.
  **L2731 CN**: 以 `(__m512h)__builtin_ia32_vfmsubph512_mask3(-(__v32hf)__A, (__v32hf)__B,` 从当前函数返回。
- **L2732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32hf)__C, (__mmask32)__U,`.
  **L2732 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32hf)__C, (__mmask32)__U,`。
- **L2733 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2733 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2734 EN**: Closes the current lexical scope or compound statement.
  **L2734 CN**: 结束当前词法作用域或复合语句块。
- **L2735 EN**: Blank line separating nearby declarations or logic blocks.
  **L2735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_sh(__m128h __W,`.
  **L2736 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_sh(__m128h __W,`。

### Lines 2737-2760

````c
                                                             __m128h __A,
                                                             __m128h __B) {
  return __builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A, (__v8hf)__B,
                                       (__mmask8)-1, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmadd_sh(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A,
                                                                  __m128h __B) {
  return __builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A, (__v8hf)__B,
                                       (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

#define _mm_fmadd_round_sh(A, B, C, R)                                         \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask(                                     \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),        \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_fmadd_round_sh(W, U, A, B, R)                                 \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask(                                     \
      (__v8hf)(__m128h)(W), (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B),        \
      (__mmask8)(U), (int)(R)))

````
- **L2737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L2737 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L2738 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L2738 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L2739 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A, (__v8hf)__B,`.
  **L2739 CN**: 以 `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L2740 EN**: Executes a call or declaration centered on `statement`.
  **L2740 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2741 EN**: Closes the current lexical scope or compound statement.
  **L2741 CN**: 结束当前词法作用域或复合语句块。
- **L2742 EN**: Blank line separating nearby declarations or logic blocks.
  **L2742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmadd_sh(__m128h __W,`.
  **L2743 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmadd_sh(__m128h __W,`。
- **L2744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L2744 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L2745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L2745 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L2746 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L2746 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L2747 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A, (__v8hf)__B,`.
  **L2747 CN**: 以 `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L2748 EN**: Executes a call or declaration centered on `statement`.
  **L2748 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2749 EN**: Closes the current lexical scope or compound statement.
  **L2749 CN**: 结束当前词法作用域或复合语句块。
- **L2750 EN**: Blank line separating nearby declarations or logic blocks.
  **L2750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2751 EN**: Defines macro `_mm_fmadd_round_sh(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2751 CN**: 定义宏 `_mm_fmadd_round_sh(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2752 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask`.
  **L2752 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask` 相关的逻辑。
- **L2753 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),        \`.
  **L2753 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),        \`。
- **L2754 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L2754 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L2755 EN**: Blank line separating nearby declarations or logic blocks.
  **L2755 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2756 EN**: Defines macro `_mm_mask_fmadd_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L2756 CN**: 定义宏 `_mm_mask_fmadd_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L2757 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask`.
  **L2757 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask` 相关的逻辑。
- **L2758 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B),        \`.
  **L2758 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B),        \`。
- **L2759 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2759 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2760 EN**: Blank line separating nearby declarations or logic blocks.
  **L2760 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2761-2784

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmadd_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return __builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, (__v8hf)__B, (__v8hf)__C,
                                        (__mmask8)__U,
                                        _MM_FROUND_CUR_DIRECTION);
}

#define _mm_maskz_fmadd_round_sh(U, A, B, C, R)                                \
  ((__m128h)__builtin_ia32_vfmaddsh3_maskz(                                    \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),        \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fmadd_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {
  return __builtin_ia32_vfmaddsh3_mask3((__v8hf)__W, (__v8hf)__X, (__v8hf)__Y,
                                        (__mmask8)__U,
                                        _MM_FROUND_CUR_DIRECTION);
}

#define _mm_mask3_fmadd_round_sh(W, X, Y, U, R)                                \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask3(                                    \
      (__v8hf)(__m128h)(W), (__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),        \
      (__mmask8)(U), (int)(R)))

````
- **L2761 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2761 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2762 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmadd_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L2762 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmadd_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L2763 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, (__v8hf)__B, (__v8hf)__C,`.
  **L2763 CN**: 以 `__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, (__v8hf)__B, (__v8hf)__C,` 从当前函数返回。
- **L2764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L2764 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L2765 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2765 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2766 EN**: Closes the current lexical scope or compound statement.
  **L2766 CN**: 结束当前词法作用域或复合语句块。
- **L2767 EN**: Blank line separating nearby declarations or logic blocks.
  **L2767 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2768 EN**: Defines macro `_mm_maskz_fmadd_round_sh(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2768 CN**: 定义宏 `_mm_maskz_fmadd_round_sh(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2769 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_maskz`.
  **L2769 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_maskz` 相关的逻辑。
- **L2770 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),        \`.
  **L2770 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),        \`。
- **L2771 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2771 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2772 EN**: Blank line separating nearby declarations or logic blocks.
  **L2772 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2773 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2773 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2774 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmadd_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {`.
  **L2774 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmadd_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {`。
- **L2775 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_mask3((__v8hf)__W, (__v8hf)__X, (__v8hf)__Y,`.
  **L2775 CN**: 以 `__builtin_ia32_vfmaddsh3_mask3((__v8hf)__W, (__v8hf)__X, (__v8hf)__Y,` 从当前函数返回。
- **L2776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L2776 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L2777 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2777 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2778 EN**: Closes the current lexical scope or compound statement.
  **L2778 CN**: 结束当前词法作用域或复合语句块。
- **L2779 EN**: Blank line separating nearby declarations or logic blocks.
  **L2779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2780 EN**: Defines macro `_mm_mask3_fmadd_round_sh(W, X, Y, U, R)` for conditional compilation, shorthand, or API generation.
  **L2780 CN**: 定义宏 `_mm_mask3_fmadd_round_sh(W, X, Y, U, R)`，用于条件编译、简写或 API 生成。
- **L2781 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask3`.
  **L2781 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask3` 相关的逻辑。
- **L2782 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), (__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),        \`.
  **L2782 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), (__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),        \`。
- **L2783 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2783 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2784 EN**: Blank line separating nearby declarations or logic blocks.
  **L2784 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2785-2808

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsub_sh(__m128h __W,
                                                             __m128h __A,
                                                             __m128h __B) {
  return (__m128h)__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A,
                                                -(__v8hf)__B, (__mmask8)-1,
                                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmsub_sh(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A,
                                                                  __m128h __B) {
  return (__m128h)__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A,
                                                -(__v8hf)__B, (__mmask8)__U,
                                                _MM_FROUND_CUR_DIRECTION);
}

#define _mm_fmsub_round_sh(A, B, C, R)                                         \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask(                                     \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),       \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_fmsub_round_sh(W, U, A, B, R)                                 \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask(                                     \
````
- **L2785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsub_sh(__m128h __W,`.
  **L2785 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmsub_sh(__m128h __W,`。
- **L2786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L2786 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L2787 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L2787 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L2788 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A,`.
  **L2788 CN**: 以 `(__m128h)__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A,` 从当前函数返回。
- **L2789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-(__v8hf)__B, (__mmask8)-1,`.
  **L2789 CN**: 继续一个多行参数列表、初始化器或聚合项：`-(__v8hf)__B, (__mmask8)-1,`。
- **L2790 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2790 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2791 EN**: Closes the current lexical scope or compound statement.
  **L2791 CN**: 结束当前词法作用域或复合语句块。
- **L2792 EN**: Blank line separating nearby declarations or logic blocks.
  **L2792 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmsub_sh(__m128h __W,`.
  **L2793 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmsub_sh(__m128h __W,`。
- **L2794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L2794 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L2795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L2795 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L2796 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L2796 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L2797 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A,`.
  **L2797 CN**: 以 `(__m128h)__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, (__v8hf)__A,` 从当前函数返回。
- **L2798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-(__v8hf)__B, (__mmask8)__U,`.
  **L2798 CN**: 继续一个多行参数列表、初始化器或聚合项：`-(__v8hf)__B, (__mmask8)__U,`。
- **L2799 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2799 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2800 EN**: Closes the current lexical scope or compound statement.
  **L2800 CN**: 结束当前词法作用域或复合语句块。
- **L2801 EN**: Blank line separating nearby declarations or logic blocks.
  **L2801 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2802 EN**: Defines macro `_mm_fmsub_round_sh(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2802 CN**: 定义宏 `_mm_fmsub_round_sh(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2803 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask`.
  **L2803 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask` 相关的逻辑。
- **L2804 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),       \`.
  **L2804 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),       \`。
- **L2805 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L2805 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L2806 EN**: Blank line separating nearby declarations or logic blocks.
  **L2806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2807 EN**: Defines macro `_mm_mask_fmsub_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L2807 CN**: 定义宏 `_mm_mask_fmsub_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L2808 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask`.
  **L2808 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask` 相关的逻辑。

### Lines 2809-2832

````c
      (__v8hf)(__m128h)(W), (__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B),       \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmsub_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, (__v8hf)__B,
                                                 -(__v8hf)__C, (__mmask8)__U,
                                                 _MM_FROUND_CUR_DIRECTION);
}

#define _mm_maskz_fmsub_round_sh(U, A, B, C, R)                                \
  ((__m128h)__builtin_ia32_vfmaddsh3_maskz(                                    \
      (__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),       \
      (__mmask8)(U), (int)R))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fmsub_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {
  return __builtin_ia32_vfmsubsh3_mask3((__v8hf)__W, (__v8hf)__X, (__v8hf)__Y,
                                        (__mmask8)__U,
                                        _MM_FROUND_CUR_DIRECTION);
}

#define _mm_mask3_fmsub_round_sh(W, X, Y, U, R)                                \
  ((__m128h)__builtin_ia32_vfmsubsh3_mask3(                                    \
````
- **L2809 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), (__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B),       \`.
  **L2809 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), (__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B),       \`。
- **L2810 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2810 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2811 EN**: Blank line separating nearby declarations or logic blocks.
  **L2811 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2812 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2812 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2813 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmsub_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L2813 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmsub_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L2814 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, (__v8hf)__B,`.
  **L2814 CN**: 以 `(__m128h)__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L2815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `-(__v8hf)__C, (__mmask8)__U,`.
  **L2815 CN**: 继续一个多行参数列表、初始化器或聚合项：`-(__v8hf)__C, (__mmask8)__U,`。
- **L2816 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2816 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2817 EN**: Closes the current lexical scope or compound statement.
  **L2817 CN**: 结束当前词法作用域或复合语句块。
- **L2818 EN**: Blank line separating nearby declarations or logic blocks.
  **L2818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2819 EN**: Defines macro `_mm_maskz_fmsub_round_sh(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2819 CN**: 定义宏 `_mm_maskz_fmsub_round_sh(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2820 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_maskz`.
  **L2820 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_maskz` 相关的逻辑。
- **L2821 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),       \`.
  **L2821 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),       \`。
- **L2822 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)R))`.
  **L2822 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)R))`。
- **L2823 EN**: Blank line separating nearby declarations or logic blocks.
  **L2823 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2824 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2824 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2825 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmsub_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {`.
  **L2825 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmsub_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {`。
- **L2826 EN**: Returns from the current function with `__builtin_ia32_vfmsubsh3_mask3((__v8hf)__W, (__v8hf)__X, (__v8hf)__Y,`.
  **L2826 CN**: 以 `__builtin_ia32_vfmsubsh3_mask3((__v8hf)__W, (__v8hf)__X, (__v8hf)__Y,` 从当前函数返回。
- **L2827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L2827 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L2828 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2828 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2829 EN**: Closes the current lexical scope or compound statement.
  **L2829 CN**: 结束当前词法作用域或复合语句块。
- **L2830 EN**: Blank line separating nearby declarations or logic blocks.
  **L2830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2831 EN**: Defines macro `_mm_mask3_fmsub_round_sh(W, X, Y, U, R)` for conditional compilation, shorthand, or API generation.
  **L2831 CN**: 定义宏 `_mm_mask3_fmsub_round_sh(W, X, Y, U, R)`，用于条件编译、简写或 API 生成。
- **L2832 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmsubsh3_mask3`.
  **L2832 CN**: 继续与可调用符号 `__builtin_ia32_vfmsubsh3_mask3` 相关的逻辑。

### Lines 2833-2856

````c
      (__v8hf)(__m128h)(W), (__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),        \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmadd_sh(__m128h __W,
                                                              __m128h __A,
                                                              __m128h __B) {
  return __builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, (__v8hf)__B,
                                       (__mmask8)-1, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fnmadd_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return __builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, (__v8hf)__B,
                                       (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

#define _mm_fnmadd_round_sh(A, B, C, R)                                        \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask(                                     \
      (__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),       \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_fnmadd_round_sh(W, U, A, B, R)                                \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask(                                     \
      (__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B),       \
````
- **L2833 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), (__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),        \`.
  **L2833 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), (__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),        \`。
- **L2834 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2834 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2835 EN**: Blank line separating nearby declarations or logic blocks.
  **L2835 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmadd_sh(__m128h __W,`.
  **L2836 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmadd_sh(__m128h __W,`。
- **L2837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L2837 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L2838 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L2838 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L2839 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, (__v8hf)__B,`.
  **L2839 CN**: 以 `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L2840 EN**: Executes a call or declaration centered on `statement`.
  **L2840 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2841 EN**: Closes the current lexical scope or compound statement.
  **L2841 CN**: 结束当前词法作用域或复合语句块。
- **L2842 EN**: Blank line separating nearby declarations or logic blocks.
  **L2842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2843 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2843 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2844 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fnmadd_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L2844 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fnmadd_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L2845 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, (__v8hf)__B,`.
  **L2845 CN**: 以 `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, (__v8hf)__B,` 从当前函数返回。
- **L2846 EN**: Executes a call or declaration centered on `statement`.
  **L2846 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2847 EN**: Closes the current lexical scope or compound statement.
  **L2847 CN**: 结束当前词法作用域或复合语句块。
- **L2848 EN**: Blank line separating nearby declarations or logic blocks.
  **L2848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2849 EN**: Defines macro `_mm_fnmadd_round_sh(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2849 CN**: 定义宏 `_mm_fnmadd_round_sh(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2850 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask`.
  **L2850 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask` 相关的逻辑。
- **L2851 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),       \`.
  **L2851 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),       \`。
- **L2852 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L2852 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L2853 EN**: Blank line separating nearby declarations or logic blocks.
  **L2853 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2854 EN**: Defines macro `_mm_mask_fnmadd_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L2854 CN**: 定义宏 `_mm_mask_fnmadd_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L2855 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask`.
  **L2855 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask` 相关的逻辑。
- **L2856 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B),       \`.
  **L2856 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(A), (__v8hf)(__m128h)(B),       \`。

### Lines 2857-2880

````c
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fnmadd_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return __builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, -(__v8hf)__B, (__v8hf)__C,
                                        (__mmask8)__U,
                                        _MM_FROUND_CUR_DIRECTION);
}

#define _mm_maskz_fnmadd_round_sh(U, A, B, C, R)                               \
  ((__m128h)__builtin_ia32_vfmaddsh3_maskz(                                    \
      (__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),       \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fnmadd_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {
  return __builtin_ia32_vfmaddsh3_mask3((__v8hf)__W, -(__v8hf)__X, (__v8hf)__Y,
                                        (__mmask8)__U,
                                        _MM_FROUND_CUR_DIRECTION);
}

#define _mm_mask3_fnmadd_round_sh(W, X, Y, U, R)                               \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask3(                                    \
      (__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),       \
````
- **L2857 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2857 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2858 EN**: Blank line separating nearby declarations or logic blocks.
  **L2858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2859 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2859 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2860 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fnmadd_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L2860 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fnmadd_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L2861 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, -(__v8hf)__B, (__v8hf)__C,`.
  **L2861 CN**: 以 `__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, -(__v8hf)__B, (__v8hf)__C,` 从当前函数返回。
- **L2862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L2862 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L2863 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2863 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2864 EN**: Closes the current lexical scope or compound statement.
  **L2864 CN**: 结束当前词法作用域或复合语句块。
- **L2865 EN**: Blank line separating nearby declarations or logic blocks.
  **L2865 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2866 EN**: Defines macro `_mm_maskz_fnmadd_round_sh(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2866 CN**: 定义宏 `_mm_maskz_fnmadd_round_sh(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2867 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_maskz`.
  **L2867 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_maskz` 相关的逻辑。
- **L2868 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),       \`.
  **L2868 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), (__v8hf)(__m128h)(C),       \`。
- **L2869 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2869 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2870 EN**: Blank line separating nearby declarations or logic blocks.
  **L2870 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2871 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2871 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2872 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fnmadd_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {`.
  **L2872 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fnmadd_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {`。
- **L2873 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_mask3((__v8hf)__W, -(__v8hf)__X, (__v8hf)__Y,`.
  **L2873 CN**: 以 `__builtin_ia32_vfmaddsh3_mask3((__v8hf)__W, -(__v8hf)__X, (__v8hf)__Y,` 从当前函数返回。
- **L2874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L2874 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L2875 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2875 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2876 EN**: Closes the current lexical scope or compound statement.
  **L2876 CN**: 结束当前词法作用域或复合语句块。
- **L2877 EN**: Blank line separating nearby declarations or logic blocks.
  **L2877 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2878 EN**: Defines macro `_mm_mask3_fnmadd_round_sh(W, X, Y, U, R)` for conditional compilation, shorthand, or API generation.
  **L2878 CN**: 定义宏 `_mm_mask3_fnmadd_round_sh(W, X, Y, U, R)`，用于条件编译、简写或 API 生成。
- **L2879 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask3`.
  **L2879 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask3` 相关的逻辑。
- **L2880 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),       \`.
  **L2880 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),       \`。

### Lines 2881-2904

````c
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmsub_sh(__m128h __W,
                                                              __m128h __A,
                                                              __m128h __B) {
  return __builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, -(__v8hf)__B,
                                       (__mmask8)-1, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fnmsub_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return __builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, -(__v8hf)__B,
                                       (__mmask8)__U, _MM_FROUND_CUR_DIRECTION);
}

#define _mm_fnmsub_round_sh(A, B, C, R)                                        \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask(                                     \
      (__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),      \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_fnmsub_round_sh(W, U, A, B, R)                                \
  ((__m128h)__builtin_ia32_vfmaddsh3_mask(                                     \
      (__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B),      \
      (__mmask8)(U), (int)(R)))
````
- **L2881 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2881 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2882 EN**: Blank line separating nearby declarations or logic blocks.
  **L2882 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmsub_sh(__m128h __W,`.
  **L2883 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fnmsub_sh(__m128h __W,`。
- **L2884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L2884 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L2885 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L2885 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L2886 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, -(__v8hf)__B,`.
  **L2886 CN**: 以 `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, -(__v8hf)__B,` 从当前函数返回。
- **L2887 EN**: Executes a call or declaration centered on `statement`.
  **L2887 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2888 EN**: Closes the current lexical scope or compound statement.
  **L2888 CN**: 结束当前词法作用域或复合语句块。
- **L2889 EN**: Blank line separating nearby declarations or logic blocks.
  **L2889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2890 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2890 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2891 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fnmsub_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L2891 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fnmsub_sh(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L2892 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, -(__v8hf)__B,`.
  **L2892 CN**: 以 `__builtin_ia32_vfmaddsh3_mask((__v8hf)__W, -(__v8hf)__A, -(__v8hf)__B,` 从当前函数返回。
- **L2893 EN**: Executes a call or declaration centered on `statement`.
  **L2893 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2894 EN**: Closes the current lexical scope or compound statement.
  **L2894 CN**: 结束当前词法作用域或复合语句块。
- **L2895 EN**: Blank line separating nearby declarations or logic blocks.
  **L2895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2896 EN**: Defines macro `_mm_fnmsub_round_sh(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2896 CN**: 定义宏 `_mm_fnmsub_round_sh(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2897 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask`.
  **L2897 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask` 相关的逻辑。
- **L2898 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),      \`.
  **L2898 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),      \`。
- **L2899 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L2899 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L2900 EN**: Blank line separating nearby declarations or logic blocks.
  **L2900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2901 EN**: Defines macro `_mm_mask_fnmsub_round_sh(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L2901 CN**: 定义宏 `_mm_mask_fnmsub_round_sh(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L2902 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_mask`.
  **L2902 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_mask` 相关的逻辑。
- **L2903 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B),      \`.
  **L2903 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B),      \`。
- **L2904 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2904 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。

### Lines 2905-2928

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fnmsub_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return __builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C,
                                        (__mmask8)__U,
                                        _MM_FROUND_CUR_DIRECTION);
}

#define _mm_maskz_fnmsub_round_sh(U, A, B, C, R)                               \
  ((__m128h)__builtin_ia32_vfmaddsh3_maskz(                                    \
      (__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),      \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fnmsub_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {
  return __builtin_ia32_vfmsubsh3_mask3((__v8hf)__W, -(__v8hf)__X, (__v8hf)__Y,
                                        (__mmask8)__U,
                                        _MM_FROUND_CUR_DIRECTION);
}

#define _mm_mask3_fnmsub_round_sh(W, X, Y, U, R)                               \
  ((__m128h)__builtin_ia32_vfmsubsh3_mask3(                                    \
      (__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),       \
      (__mmask8)(U), (int)(R)))
````
- **L2905 EN**: Blank line separating nearby declarations or logic blocks.
  **L2905 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2906 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2906 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2907 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fnmsub_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L2907 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fnmsub_sh(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L2908 EN**: Returns from the current function with `__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C,`.
  **L2908 CN**: 以 `__builtin_ia32_vfmaddsh3_maskz((__v8hf)__A, -(__v8hf)__B, -(__v8hf)__C,` 从当前函数返回。
- **L2909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L2909 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L2910 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2910 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2911 EN**: Closes the current lexical scope or compound statement.
  **L2911 CN**: 结束当前词法作用域或复合语句块。
- **L2912 EN**: Blank line separating nearby declarations or logic blocks.
  **L2912 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2913 EN**: Defines macro `_mm_maskz_fnmsub_round_sh(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2913 CN**: 定义宏 `_mm_maskz_fnmsub_round_sh(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2914 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddsh3_maskz`.
  **L2914 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddsh3_maskz` 相关的逻辑。
- **L2915 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),      \`.
  **L2915 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(A), -(__v8hf)(__m128h)(B), -(__v8hf)(__m128h)(C),      \`。
- **L2916 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2916 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2917 EN**: Blank line separating nearby declarations or logic blocks.
  **L2917 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2918 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2918 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2919 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fnmsub_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {`.
  **L2919 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fnmsub_sh(__m128h __W, __m128h __X, __m128h __Y, __mmask8 __U) {`。
- **L2920 EN**: Returns from the current function with `__builtin_ia32_vfmsubsh3_mask3((__v8hf)__W, -(__v8hf)__X, (__v8hf)__Y,`.
  **L2920 CN**: 以 `__builtin_ia32_vfmsubsh3_mask3((__v8hf)__W, -(__v8hf)__X, (__v8hf)__Y,` 从当前函数返回。
- **L2921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U,`.
  **L2921 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U,`。
- **L2922 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2922 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2923 EN**: Closes the current lexical scope or compound statement.
  **L2923 CN**: 结束当前词法作用域或复合语句块。
- **L2924 EN**: Blank line separating nearby declarations or logic blocks.
  **L2924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2925 EN**: Defines macro `_mm_mask3_fnmsub_round_sh(W, X, Y, U, R)` for conditional compilation, shorthand, or API generation.
  **L2925 CN**: 定义宏 `_mm_mask3_fnmsub_round_sh(W, X, Y, U, R)`，用于条件编译、简写或 API 生成。
- **L2926 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmsubsh3_mask3`.
  **L2926 CN**: 继续与可调用符号 `__builtin_ia32_vfmsubsh3_mask3` 相关的逻辑。
- **L2927 EN**: Continues the surrounding expression or declaration: `(__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),       \`.
  **L2927 CN**: 继续构造周围的表达式或声明：`(__v8hf)(__m128h)(W), -(__v8hf)(__m128h)(X), (__v8hf)(__m128h)(Y),       \`。
- **L2928 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2928 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。

### Lines 2929-2952

````c

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmadd_sch(__m128h __A,
                                                               __m128h __B,
                                                               __m128h __C) {
  return (__m128h)__builtin_ia32_vfcmaddcsh_mask((__v4sf)__A, (__v4sf)__B,
                                                 (__v4sf)__C, (__mmask8)-1,
                                                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fcmadd_sch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_vfcmaddcsh_round_mask(
      (__v4sf)__A, (__v4sf)(__B), (__v4sf)(__C), __U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fcmadd_sch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_vfcmaddcsh_maskz((__v4sf)__A, (__v4sf)__B,
                                                  (__v4sf)__C, (__mmask8)__U,
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fcmadd_sch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
````
- **L2929 EN**: Blank line separating nearby declarations or logic blocks.
  **L2929 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmadd_sch(__m128h __A,`.
  **L2930 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmadd_sch(__m128h __A,`。
- **L2931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L2931 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L2932 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L2932 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L2933 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmaddcsh_mask((__v4sf)__A, (__v4sf)__B,`.
  **L2933 CN**: 以 `(__m128h)__builtin_ia32_vfcmaddcsh_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L2934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__C, (__mmask8)-1,`.
  **L2934 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__C, (__mmask8)-1,`。
- **L2935 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2935 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2936 EN**: Closes the current lexical scope or compound statement.
  **L2936 CN**: 结束当前词法作用域或复合语句块。
- **L2937 EN**: Blank line separating nearby declarations or logic blocks.
  **L2937 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2938 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2938 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2939 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fcmadd_sch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`.
  **L2939 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fcmadd_sch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`。
- **L2940 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmaddcsh_round_mask(`.
  **L2940 CN**: 以 `(__m128h)__builtin_ia32_vfcmaddcsh_round_mask(` 从当前函数返回。
- **L2941 EN**: Executes a call or declaration centered on `statement`.
  **L2941 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2942 EN**: Closes the current lexical scope or compound statement.
  **L2942 CN**: 结束当前词法作用域或复合语句块。
- **L2943 EN**: Blank line separating nearby declarations or logic blocks.
  **L2943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2944 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2944 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2945 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fcmadd_sch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L2945 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fcmadd_sch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L2946 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmaddcsh_maskz((__v4sf)__A, (__v4sf)__B,`.
  **L2946 CN**: 以 `(__m128h)__builtin_ia32_vfcmaddcsh_maskz((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L2947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__C, (__mmask8)__U,`.
  **L2947 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__C, (__mmask8)__U,`。
- **L2948 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2948 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2949 EN**: Closes the current lexical scope or compound statement.
  **L2949 CN**: 结束当前词法作用域或复合语句块。
- **L2950 EN**: Blank line separating nearby declarations or logic blocks.
  **L2950 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2951 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2951 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fcmadd_sch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L2952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fcmadd_sch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。

### Lines 2953-2976

````c
  return (__m128h)__builtin_ia32_vfcmaddcsh_round_mask3(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)__C, __U, _MM_FROUND_CUR_DIRECTION);
}

#define _mm_fcmadd_round_sch(A, B, C, R)                                       \
  ((__m128h)__builtin_ia32_vfcmaddcsh_mask(                                    \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_fcmadd_round_sch(A, U, B, C, R)                               \
  ((__m128h)__builtin_ia32_vfcmaddcsh_round_mask(                              \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_fcmadd_round_sch(U, A, B, C, R)                              \
  ((__m128h)__builtin_ia32_vfcmaddcsh_maskz(                                   \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \
      (__mmask8)(U), (int)(R)))

#define _mm_mask3_fcmadd_round_sch(A, B, C, U, R)                              \
  ((__m128h)__builtin_ia32_vfcmaddcsh_round_mask3(                             \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \
      (__mmask8)(U), (int)(R)))

````
- **L2953 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmaddcsh_round_mask3(`.
  **L2953 CN**: 以 `(__m128h)__builtin_ia32_vfcmaddcsh_round_mask3(` 从当前函数返回。
- **L2954 EN**: Executes a call or declaration centered on `statement`.
  **L2954 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2955 EN**: Closes the current lexical scope or compound statement.
  **L2955 CN**: 结束当前词法作用域或复合语句块。
- **L2956 EN**: Blank line separating nearby declarations or logic blocks.
  **L2956 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2957 EN**: Defines macro `_mm_fcmadd_round_sch(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2957 CN**: 定义宏 `_mm_fcmadd_round_sch(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2958 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmaddcsh_mask`.
  **L2958 CN**: 继续与可调用符号 `__builtin_ia32_vfcmaddcsh_mask` 相关的逻辑。
- **L2959 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`.
  **L2959 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`。
- **L2960 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L2960 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L2961 EN**: Blank line separating nearby declarations or logic blocks.
  **L2961 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2962 EN**: Defines macro `_mm_mask_fcmadd_round_sch(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2962 CN**: 定义宏 `_mm_mask_fcmadd_round_sch(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2963 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmaddcsh_round_mask`.
  **L2963 CN**: 继续与可调用符号 `__builtin_ia32_vfcmaddcsh_round_mask` 相关的逻辑。
- **L2964 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`.
  **L2964 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`。
- **L2965 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2965 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2966 EN**: Blank line separating nearby declarations or logic blocks.
  **L2966 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2967 EN**: Defines macro `_mm_maskz_fcmadd_round_sch(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L2967 CN**: 定义宏 `_mm_maskz_fcmadd_round_sch(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L2968 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmaddcsh_maskz`.
  **L2968 CN**: 继续与可调用符号 `__builtin_ia32_vfcmaddcsh_maskz` 相关的逻辑。
- **L2969 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`.
  **L2969 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`。
- **L2970 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2970 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2971 EN**: Blank line separating nearby declarations or logic blocks.
  **L2971 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2972 EN**: Defines macro `_mm_mask3_fcmadd_round_sch(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L2972 CN**: 定义宏 `_mm_mask3_fcmadd_round_sch(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L2973 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmaddcsh_round_mask3`.
  **L2973 CN**: 继续与可调用符号 `__builtin_ia32_vfcmaddcsh_round_mask3` 相关的逻辑。
- **L2974 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`.
  **L2974 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`。
- **L2975 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L2975 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L2976 EN**: Blank line separating nearby declarations or logic blocks.
  **L2976 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2977-3000

````c
static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_sch(__m128h __A,
                                                              __m128h __B,
                                                              __m128h __C) {
  return (__m128h)__builtin_ia32_vfmaddcsh_mask((__v4sf)__A, (__v4sf)__B,
                                                (__v4sf)__C, (__mmask8)-1,
                                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fmadd_sch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_vfmaddcsh_round_mask(
      (__v4sf)__A, (__v4sf)(__B), (__v4sf)(__C), __U, _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmadd_sch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {
  return (__m128h)__builtin_ia32_vfmaddcsh_maskz((__v4sf)__A, (__v4sf)__B,
                                                 (__v4sf)__C, (__mmask8)__U,
                                                 _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask3_fmadd_sch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {
  return (__m128h)__builtin_ia32_vfmaddcsh_round_mask3(
````
- **L2977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_sch(__m128h __A,`.
  **L2977 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmadd_sch(__m128h __A,`。
- **L2978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __B,`.
  **L2978 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __B,`。
- **L2979 EN**: Continues the surrounding expression or declaration: `__m128h __C) {`.
  **L2979 CN**: 继续构造周围的表达式或声明：`__m128h __C) {`。
- **L2980 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddcsh_mask((__v4sf)__A, (__v4sf)__B,`.
  **L2980 CN**: 以 `(__m128h)__builtin_ia32_vfmaddcsh_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L2981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__C, (__mmask8)-1,`.
  **L2981 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__C, (__mmask8)-1,`。
- **L2982 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2982 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2983 EN**: Closes the current lexical scope or compound statement.
  **L2983 CN**: 结束当前词法作用域或复合语句块。
- **L2984 EN**: Blank line separating nearby declarations or logic blocks.
  **L2984 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2985 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2985 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2986 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fmadd_sch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`.
  **L2986 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fmadd_sch(__m128h __A, __mmask8 __U, __m128h __B, __m128h __C) {`。
- **L2987 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddcsh_round_mask(`.
  **L2987 CN**: 以 `(__m128h)__builtin_ia32_vfmaddcsh_round_mask(` 从当前函数返回。
- **L2988 EN**: Executes a call or declaration centered on `statement`.
  **L2988 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2989 EN**: Closes the current lexical scope or compound statement.
  **L2989 CN**: 结束当前词法作用域或复合语句块。
- **L2990 EN**: Blank line separating nearby declarations or logic blocks.
  **L2990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2991 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2991 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2992 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmadd_sch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`.
  **L2992 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmadd_sch(__mmask8 __U, __m128h __A, __m128h __B, __m128h __C) {`。
- **L2993 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddcsh_maskz((__v4sf)__A, (__v4sf)__B,`.
  **L2993 CN**: 以 `(__m128h)__builtin_ia32_vfmaddcsh_maskz((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L2994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__C, (__mmask8)__U,`.
  **L2994 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__C, (__mmask8)__U,`。
- **L2995 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L2995 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L2996 EN**: Closes the current lexical scope or compound statement.
  **L2996 CN**: 结束当前词法作用域或复合语句块。
- **L2997 EN**: Blank line separating nearby declarations or logic blocks.
  **L2997 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2998 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L2998 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L2999 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmadd_sch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`.
  **L2999 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmadd_sch(__m128h __A, __m128h __B, __m128h __C, __mmask8 __U) {`。
- **L3000 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmaddcsh_round_mask3(`.
  **L3000 CN**: 以 `(__m128h)__builtin_ia32_vfmaddcsh_round_mask3(` 从当前函数返回。

### Lines 3001-3024

````c
      (__v4sf)__A, (__v4sf)__B, (__v4sf)__C, __U, _MM_FROUND_CUR_DIRECTION);
}

#define _mm_fmadd_round_sch(A, B, C, R)                                        \
  ((__m128h)__builtin_ia32_vfmaddcsh_mask(                                     \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \
      (__mmask8)-1, (int)(R)))

#define _mm_mask_fmadd_round_sch(A, U, B, C, R)                                \
  ((__m128h)__builtin_ia32_vfmaddcsh_round_mask(                               \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_fmadd_round_sch(U, A, B, C, R)                               \
  ((__m128h)__builtin_ia32_vfmaddcsh_maskz(                                    \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \
      (__mmask8)(U), (int)(R)))

#define _mm_mask3_fmadd_round_sch(A, B, C, U, R)                               \
  ((__m128h)__builtin_ia32_vfmaddcsh_round_mask3(                              \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \
      (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmul_sch(__m128h __A,
````
- **L3001 EN**: Executes a call or declaration centered on `statement`.
  **L3001 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3002 EN**: Closes the current lexical scope or compound statement.
  **L3002 CN**: 结束当前词法作用域或复合语句块。
- **L3003 EN**: Blank line separating nearby declarations or logic blocks.
  **L3003 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3004 EN**: Defines macro `_mm_fmadd_round_sch(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3004 CN**: 定义宏 `_mm_fmadd_round_sch(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3005 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddcsh_mask`.
  **L3005 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddcsh_mask` 相关的逻辑。
- **L3006 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`.
  **L3006 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`。
- **L3007 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1, (int)(R)))`.
  **L3007 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1, (int)(R)))`。
- **L3008 EN**: Blank line separating nearby declarations or logic blocks.
  **L3008 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3009 EN**: Defines macro `_mm_mask_fmadd_round_sch(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3009 CN**: 定义宏 `_mm_mask_fmadd_round_sch(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3010 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddcsh_round_mask`.
  **L3010 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddcsh_round_mask` 相关的逻辑。
- **L3011 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`.
  **L3011 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`。
- **L3012 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L3012 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L3013 EN**: Blank line separating nearby declarations or logic blocks.
  **L3013 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3014 EN**: Defines macro `_mm_maskz_fmadd_round_sch(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3014 CN**: 定义宏 `_mm_maskz_fmadd_round_sch(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3015 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddcsh_maskz`.
  **L3015 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddcsh_maskz` 相关的逻辑。
- **L3016 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`.
  **L3016 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`。
- **L3017 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L3017 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L3018 EN**: Blank line separating nearby declarations or logic blocks.
  **L3018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3019 EN**: Defines macro `_mm_mask3_fmadd_round_sch(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L3019 CN**: 定义宏 `_mm_mask3_fmadd_round_sch(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L3020 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddcsh_round_mask3`.
  **L3020 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddcsh_round_mask3` 相关的逻辑。
- **L3021 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`.
  **L3021 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(C),        \`。
- **L3022 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L3022 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L3023 EN**: Blank line separating nearby declarations or logic blocks.
  **L3023 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmul_sch(__m128h __A,`.
  **L3024 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fcmul_sch(__m128h __A,`。

### Lines 3025-3048

````c
                                                              __m128h __B) {
  return (__m128h)__builtin_ia32_vfcmulcsh_mask(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_undefined_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_mask_fcmul_sch(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_vfcmulcsh_mask((__v4sf)__A, (__v4sf)__B,
                                                (__v4sf)__W, (__mmask8)__U,
                                                _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fcmul_sch(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_vfcmulcsh_mask(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_fcmul_round_sch(A, B, R)                                           \
  ((__m128h)__builtin_ia32_vfcmulcsh_mask(                                     \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \
      (__v4sf)(__m128h)_mm_undefined_ph(), (__mmask8)-1, (int)(R)))
````
- **L3025 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L3025 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L3026 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmulcsh_mask(`.
  **L3026 CN**: 以 `(__m128h)__builtin_ia32_vfcmulcsh_mask(` 从当前函数返回。
- **L3027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_undefined_ph(), (__mmask8)-1,`.
  **L3027 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_undefined_ph(), (__mmask8)-1,`。
- **L3028 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3028 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3029 EN**: Closes the current lexical scope or compound statement.
  **L3029 CN**: 结束当前词法作用域或复合语句块。
- **L3030 EN**: Blank line separating nearby declarations or logic blocks.
  **L3030 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3031 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L3031 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L3032 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fcmul_sch(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`.
  **L3032 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fcmul_sch(__m128h __W, __mmask8 __U, __m128h __A, __m128h __B) {`。
- **L3033 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmulcsh_mask((__v4sf)__A, (__v4sf)__B,`.
  **L3033 CN**: 以 `(__m128h)__builtin_ia32_vfcmulcsh_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L3034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__W, (__mmask8)__U,`.
  **L3034 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__W, (__mmask8)__U,`。
- **L3035 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3035 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3036 EN**: Closes the current lexical scope or compound statement.
  **L3036 CN**: 结束当前词法作用域或复合语句块。
- **L3037 EN**: Blank line separating nearby declarations or logic blocks.
  **L3037 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3038 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L3038 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L3039 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fcmul_sch(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L3039 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fcmul_sch(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L3040 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfcmulcsh_mask(`.
  **L3040 CN**: 以 `(__m128h)__builtin_ia32_vfcmulcsh_mask(` 从当前函数返回。
- **L3041 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L3041 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L3042 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3042 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3043 EN**: Closes the current lexical scope or compound statement.
  **L3043 CN**: 结束当前词法作用域或复合语句块。
- **L3044 EN**: Blank line separating nearby declarations or logic blocks.
  **L3044 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3045 EN**: Defines macro `_mm_fcmul_round_sch(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3045 CN**: 定义宏 `_mm_fcmul_round_sch(A, B, R)`，用于条件编译、简写或 API 生成。
- **L3046 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmulcsh_mask`.
  **L3046 CN**: 继续与可调用符号 `__builtin_ia32_vfcmulcsh_mask` 相关的逻辑。
- **L3047 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \`.
  **L3047 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \`。
- **L3048 EN**: Continues logic associated with callable symbol `_mm_undefined_ph`.
  **L3048 CN**: 继续与可调用符号 `_mm_undefined_ph` 相关的逻辑。

### Lines 3049-3072

````c

#define _mm_mask_fcmul_round_sch(W, U, A, B, R)                                \
  ((__m128h)__builtin_ia32_vfcmulcsh_mask(                                     \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_fcmul_round_sch(U, A, B, R)                                  \
  ((__m128h)__builtin_ia32_vfcmulcsh_mask(                                     \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \
      (__v4sf)(__m128h)_mm_setzero_ph(), (__mmask8)(U), (int)(R)))

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmul_sch(__m128h __A,
                                                             __m128h __B) {
  return (__m128h)__builtin_ia32_vfmulcsh_mask(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_undefined_ph(), (__mmask8)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmul_sch(__m128h __W,
                                                                  __mmask8 __U,
                                                                  __m128h __A,
                                                                  __m128h __B) {
  return (__m128h)__builtin_ia32_vfmulcsh_mask((__v4sf)__A, (__v4sf)__B,
                                               (__v4sf)__W, (__mmask8)__U,
````
- **L3049 EN**: Blank line separating nearby declarations or logic blocks.
  **L3049 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3050 EN**: Defines macro `_mm_mask_fcmul_round_sch(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3050 CN**: 定义宏 `_mm_mask_fcmul_round_sch(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3051 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmulcsh_mask`.
  **L3051 CN**: 继续与可调用符号 `__builtin_ia32_vfcmulcsh_mask` 相关的逻辑。
- **L3052 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(W),        \`.
  **L3052 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(W),        \`。
- **L3053 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L3053 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L3054 EN**: Blank line separating nearby declarations or logic blocks.
  **L3054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3055 EN**: Defines macro `_mm_maskz_fcmul_round_sch(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3055 CN**: 定义宏 `_mm_maskz_fcmul_round_sch(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3056 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmulcsh_mask`.
  **L3056 CN**: 继续与可调用符号 `__builtin_ia32_vfcmulcsh_mask` 相关的逻辑。
- **L3057 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \`.
  **L3057 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \`。
- **L3058 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L3058 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。
- **L3059 EN**: Blank line separating nearby declarations or logic blocks.
  **L3059 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmul_sch(__m128h __A,`.
  **L3060 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_fmul_sch(__m128h __A,`。
- **L3061 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L3061 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L3062 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmulcsh_mask(`.
  **L3062 CN**: 以 `(__m128h)__builtin_ia32_vfmulcsh_mask(` 从当前函数返回。
- **L3063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_undefined_ph(), (__mmask8)-1,`.
  **L3063 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_undefined_ph(), (__mmask8)-1,`。
- **L3064 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3064 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3065 EN**: Closes the current lexical scope or compound statement.
  **L3065 CN**: 结束当前词法作用域或复合语句块。
- **L3066 EN**: Blank line separating nearby declarations or logic blocks.
  **L3066 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmul_sch(__m128h __W,`.
  **L3067 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128h __DEFAULT_FN_ATTRS128 _mm_mask_fmul_sch(__m128h __W,`。
- **L3068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L3068 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L3069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128h __A,`.
  **L3069 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128h __A,`。
- **L3070 EN**: Continues the surrounding expression or declaration: `__m128h __B) {`.
  **L3070 CN**: 继续构造周围的表达式或声明：`__m128h __B) {`。
- **L3071 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmulcsh_mask((__v4sf)__A, (__v4sf)__B,`.
  **L3071 CN**: 以 `(__m128h)__builtin_ia32_vfmulcsh_mask((__v4sf)__A, (__v4sf)__B,` 从当前函数返回。
- **L3072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__W, (__mmask8)__U,`.
  **L3072 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__W, (__mmask8)__U,`。

### Lines 3073-3096

````c
                                               _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m128h __DEFAULT_FN_ATTRS128
_mm_maskz_fmul_sch(__mmask8 __U, __m128h __A, __m128h __B) {
  return (__m128h)__builtin_ia32_vfmulcsh_mask(
      (__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_setzero_ph(), (__mmask8)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm_fmul_round_sch(A, B, R)                                            \
  ((__m128h)__builtin_ia32_vfmulcsh_mask(                                      \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \
      (__v4sf)(__m128h)_mm_undefined_ph(), (__mmask8)-1, (int)(R)))

#define _mm_mask_fmul_round_sch(W, U, A, B, R)                                 \
  ((__m128h)__builtin_ia32_vfmulcsh_mask(                                      \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(W),        \
      (__mmask8)(U), (int)(R)))

#define _mm_maskz_fmul_round_sch(U, A, B, R)                                   \
  ((__m128h)__builtin_ia32_vfmulcsh_mask(                                      \
      (__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \
      (__v4sf)(__m128h)_mm_setzero_ph(), (__mmask8)(U), (int)(R)))
````
- **L3073 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3073 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3074 EN**: Closes the current lexical scope or compound statement.
  **L3074 CN**: 结束当前词法作用域或复合语句块。
- **L3075 EN**: Blank line separating nearby declarations or logic blocks.
  **L3075 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3076 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128h __DEFAULT_FN_ATTRS128`.
  **L3076 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128h __DEFAULT_FN_ATTRS128`。
- **L3077 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmul_sch(__mmask8 __U, __m128h __A, __m128h __B) {`.
  **L3077 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmul_sch(__mmask8 __U, __m128h __A, __m128h __B) {`。
- **L3078 EN**: Returns from the current function with `(__m128h)__builtin_ia32_vfmulcsh_mask(`.
  **L3078 CN**: 以 `(__m128h)__builtin_ia32_vfmulcsh_mask(` 从当前函数返回。
- **L3079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_setzero_ph(), (__mmask8)__U,`.
  **L3079 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)__A, (__v4sf)__B, (__v4sf)_mm_setzero_ph(), (__mmask8)__U,`。
- **L3080 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3080 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3081 EN**: Closes the current lexical scope or compound statement.
  **L3081 CN**: 结束当前词法作用域或复合语句块。
- **L3082 EN**: Blank line separating nearby declarations or logic blocks.
  **L3082 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3083 EN**: Defines macro `_mm_fmul_round_sch(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3083 CN**: 定义宏 `_mm_fmul_round_sch(A, B, R)`，用于条件编译、简写或 API 生成。
- **L3084 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmulcsh_mask`.
  **L3084 CN**: 继续与可调用符号 `__builtin_ia32_vfmulcsh_mask` 相关的逻辑。
- **L3085 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \`.
  **L3085 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \`。
- **L3086 EN**: Continues logic associated with callable symbol `_mm_undefined_ph`.
  **L3086 CN**: 继续与可调用符号 `_mm_undefined_ph` 相关的逻辑。
- **L3087 EN**: Blank line separating nearby declarations or logic blocks.
  **L3087 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3088 EN**: Defines macro `_mm_mask_fmul_round_sch(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3088 CN**: 定义宏 `_mm_mask_fmul_round_sch(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3089 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmulcsh_mask`.
  **L3089 CN**: 继续与可调用符号 `__builtin_ia32_vfmulcsh_mask` 相关的逻辑。
- **L3090 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(W),        \`.
  **L3090 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B), (__v4sf)(__m128h)(W),        \`。
- **L3091 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U), (int)(R)))`.
  **L3091 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U), (int)(R)))`。
- **L3092 EN**: Blank line separating nearby declarations or logic blocks.
  **L3092 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3093 EN**: Defines macro `_mm_maskz_fmul_round_sch(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3093 CN**: 定义宏 `_mm_maskz_fmul_round_sch(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3094 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmulcsh_mask`.
  **L3094 CN**: 继续与可调用符号 `__builtin_ia32_vfmulcsh_mask` 相关的逻辑。
- **L3095 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \`.
  **L3095 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128h)(A), (__v4sf)(__m128h)(B),                              \`。
- **L3096 EN**: Continues logic associated with callable symbol `_mm_setzero_ph`.
  **L3096 CN**: 继续与可调用符号 `_mm_setzero_ph` 相关的逻辑。

### Lines 3097-3120

````c

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fcmul_pch(__m512h __A,
                                                                 __m512h __B) {
  return (__m512h)__builtin_ia32_vfcmulcph512_mask(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_undefined_ph(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fcmul_pch(__m512h __W, __mmask16 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_vfcmulcph512_mask((__v16sf)__A, (__v16sf)__B,
                                                   (__v16sf)__W, (__mmask16)__U,
                                                   _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fcmul_pch(__mmask16 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_vfcmulcph512_mask(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_setzero_ph(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_fcmul_round_pch(A, B, R)                                        \
  ((__m512h)__builtin_ia32_vfcmulcph512_mask(                                  \
````
- **L3097 EN**: Blank line separating nearby declarations or logic blocks.
  **L3097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fcmul_pch(__m512h __A,`.
  **L3098 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fcmul_pch(__m512h __A,`。
- **L3099 EN**: Continues the surrounding expression or declaration: `__m512h __B) {`.
  **L3099 CN**: 继续构造周围的表达式或声明：`__m512h __B) {`。
- **L3100 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfcmulcph512_mask(`.
  **L3100 CN**: 以 `(__m512h)__builtin_ia32_vfcmulcph512_mask(` 从当前函数返回。
- **L3101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_undefined_ph(), (__mmask16)-1,`.
  **L3101 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_undefined_ph(), (__mmask16)-1,`。
- **L3102 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3102 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3103 EN**: Closes the current lexical scope or compound statement.
  **L3103 CN**: 结束当前词法作用域或复合语句块。
- **L3104 EN**: Blank line separating nearby declarations or logic blocks.
  **L3104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3105 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3105 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3106 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fcmul_pch(__m512h __W, __mmask16 __U, __m512h __A, __m512h __B) {`.
  **L3106 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fcmul_pch(__m512h __W, __mmask16 __U, __m512h __A, __m512h __B) {`。
- **L3107 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfcmulcph512_mask((__v16sf)__A, (__v16sf)__B,`.
  **L3107 CN**: 以 `(__m512h)__builtin_ia32_vfcmulcph512_mask((__v16sf)__A, (__v16sf)__B,` 从当前函数返回。
- **L3108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__W, (__mmask16)__U,`.
  **L3108 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__W, (__mmask16)__U,`。
- **L3109 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3109 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3110 EN**: Closes the current lexical scope or compound statement.
  **L3110 CN**: 结束当前词法作用域或复合语句块。
- **L3111 EN**: Blank line separating nearby declarations or logic blocks.
  **L3111 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3112 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3112 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3113 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fcmul_pch(__mmask16 __U, __m512h __A, __m512h __B) {`.
  **L3113 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fcmul_pch(__mmask16 __U, __m512h __A, __m512h __B) {`。
- **L3114 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfcmulcph512_mask(`.
  **L3114 CN**: 以 `(__m512h)__builtin_ia32_vfcmulcph512_mask(` 从当前函数返回。
- **L3115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_setzero_ph(), (__mmask16)__U,`.
  **L3115 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_setzero_ph(), (__mmask16)__U,`。
- **L3116 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3116 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3117 EN**: Closes the current lexical scope or compound statement.
  **L3117 CN**: 结束当前词法作用域或复合语句块。
- **L3118 EN**: Blank line separating nearby declarations or logic blocks.
  **L3118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3119 EN**: Defines macro `_mm512_fcmul_round_pch(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3119 CN**: 定义宏 `_mm512_fcmul_round_pch(A, B, R)`，用于条件编译、简写或 API 生成。
- **L3120 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmulcph512_mask`.
  **L3120 CN**: 继续与可调用符号 `__builtin_ia32_vfcmulcph512_mask` 相关的逻辑。

### Lines 3121-3144

````c
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \
      (__v16sf)(__m512h)_mm512_undefined_ph(), (__mmask16)-1, (int)(R)))

#define _mm512_mask_fcmul_round_pch(W, U, A, B, R)                             \
  ((__m512h)__builtin_ia32_vfcmulcph512_mask(                                  \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(W),     \
      (__mmask16)(U), (int)(R)))

#define _mm512_maskz_fcmul_round_pch(U, A, B, R)                               \
  ((__m512h)__builtin_ia32_vfcmulcph512_mask(                                  \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \
      (__v16sf)(__m512h)_mm512_setzero_ph(), (__mmask16)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmul_pch(__m512h __A,
                                                                __m512h __B) {
  return (__m512h)__builtin_ia32_vfmulcph512_mask(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_undefined_ph(), (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fmul_pch(__m512h __W, __mmask16 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_vfmulcph512_mask((__v16sf)__A, (__v16sf)__B,
                                                  (__v16sf)__W, (__mmask16)__U,
````
- **L3121 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \`.
  **L3121 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \`。
- **L3122 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L3122 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L3123 EN**: Blank line separating nearby declarations or logic blocks.
  **L3123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3124 EN**: Defines macro `_mm512_mask_fcmul_round_pch(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3124 CN**: 定义宏 `_mm512_mask_fcmul_round_pch(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3125 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmulcph512_mask`.
  **L3125 CN**: 继续与可调用符号 `__builtin_ia32_vfcmulcph512_mask` 相关的逻辑。
- **L3126 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(W),     \`.
  **L3126 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(W),     \`。
- **L3127 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L3127 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L3128 EN**: Blank line separating nearby declarations or logic blocks.
  **L3128 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3129 EN**: Defines macro `_mm512_maskz_fcmul_round_pch(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3129 CN**: 定义宏 `_mm512_maskz_fcmul_round_pch(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3130 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmulcph512_mask`.
  **L3130 CN**: 继续与可调用符号 `__builtin_ia32_vfcmulcph512_mask` 相关的逻辑。
- **L3131 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \`.
  **L3131 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \`。
- **L3132 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L3132 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。
- **L3133 EN**: Blank line separating nearby declarations or logic blocks.
  **L3133 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmul_pch(__m512h __A,`.
  **L3134 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmul_pch(__m512h __A,`。
- **L3135 EN**: Continues the surrounding expression or declaration: `__m512h __B) {`.
  **L3135 CN**: 继续构造周围的表达式或声明：`__m512h __B) {`。
- **L3136 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmulcph512_mask(`.
  **L3136 CN**: 以 `(__m512h)__builtin_ia32_vfmulcph512_mask(` 从当前函数返回。
- **L3137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_undefined_ph(), (__mmask16)-1,`.
  **L3137 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_undefined_ph(), (__mmask16)-1,`。
- **L3138 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3138 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3139 EN**: Closes the current lexical scope or compound statement.
  **L3139 CN**: 结束当前词法作用域或复合语句块。
- **L3140 EN**: Blank line separating nearby declarations or logic blocks.
  **L3140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3141 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3141 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fmul_pch(__m512h __W, __mmask16 __U, __m512h __A, __m512h __B) {`.
  **L3142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fmul_pch(__m512h __W, __mmask16 __U, __m512h __A, __m512h __B) {`。
- **L3143 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmulcph512_mask((__v16sf)__A, (__v16sf)__B,`.
  **L3143 CN**: 以 `(__m512h)__builtin_ia32_vfmulcph512_mask((__v16sf)__A, (__v16sf)__B,` 从当前函数返回。
- **L3144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__W, (__mmask16)__U,`.
  **L3144 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__W, (__mmask16)__U,`。

### Lines 3145-3168

````c
                                                  _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fmul_pch(__mmask16 __U, __m512h __A, __m512h __B) {
  return (__m512h)__builtin_ia32_vfmulcph512_mask(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_setzero_ph(), (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_fmul_round_pch(A, B, R)                                         \
  ((__m512h)__builtin_ia32_vfmulcph512_mask(                                   \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \
      (__v16sf)(__m512h)_mm512_undefined_ph(), (__mmask16)-1, (int)(R)))

#define _mm512_mask_fmul_round_pch(W, U, A, B, R)                              \
  ((__m512h)__builtin_ia32_vfmulcph512_mask(                                   \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(W),     \
      (__mmask16)(U), (int)(R)))

#define _mm512_maskz_fmul_round_pch(U, A, B, R)                                \
  ((__m512h)__builtin_ia32_vfmulcph512_mask(                                   \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \
      (__v16sf)(__m512h)_mm512_setzero_ph(), (__mmask16)(U), (int)(R)))
````
- **L3145 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3145 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3146 EN**: Closes the current lexical scope or compound statement.
  **L3146 CN**: 结束当前词法作用域或复合语句块。
- **L3147 EN**: Blank line separating nearby declarations or logic blocks.
  **L3147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3148 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3148 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fmul_pch(__mmask16 __U, __m512h __A, __m512h __B) {`.
  **L3149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fmul_pch(__mmask16 __U, __m512h __A, __m512h __B) {`。
- **L3150 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmulcph512_mask(`.
  **L3150 CN**: 以 `(__m512h)__builtin_ia32_vfmulcph512_mask(` 从当前函数返回。
- **L3151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_setzero_ph(), (__mmask16)__U,`.
  **L3151 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)_mm512_setzero_ph(), (__mmask16)__U,`。
- **L3152 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3152 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3153 EN**: Closes the current lexical scope or compound statement.
  **L3153 CN**: 结束当前词法作用域或复合语句块。
- **L3154 EN**: Blank line separating nearby declarations or logic blocks.
  **L3154 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3155 EN**: Defines macro `_mm512_fmul_round_pch(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3155 CN**: 定义宏 `_mm512_fmul_round_pch(A, B, R)`，用于条件编译、简写或 API 生成。
- **L3156 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmulcph512_mask`.
  **L3156 CN**: 继续与可调用符号 `__builtin_ia32_vfmulcph512_mask` 相关的逻辑。
- **L3157 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \`.
  **L3157 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \`。
- **L3158 EN**: Continues logic associated with callable symbol `_mm512_undefined_ph`.
  **L3158 CN**: 继续与可调用符号 `_mm512_undefined_ph` 相关的逻辑。
- **L3159 EN**: Blank line separating nearby declarations or logic blocks.
  **L3159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3160 EN**: Defines macro `_mm512_mask_fmul_round_pch(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3160 CN**: 定义宏 `_mm512_mask_fmul_round_pch(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3161 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmulcph512_mask`.
  **L3161 CN**: 继续与可调用符号 `__builtin_ia32_vfmulcph512_mask` 相关的逻辑。
- **L3162 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(W),     \`.
  **L3162 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(W),     \`。
- **L3163 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L3163 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L3164 EN**: Blank line separating nearby declarations or logic blocks.
  **L3164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3165 EN**: Defines macro `_mm512_maskz_fmul_round_pch(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3165 CN**: 定义宏 `_mm512_maskz_fmul_round_pch(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3166 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmulcph512_mask`.
  **L3166 CN**: 继续与可调用符号 `__builtin_ia32_vfmulcph512_mask` 相关的逻辑。
- **L3167 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \`.
  **L3167 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B),                            \`。
- **L3168 EN**: Continues logic associated with callable symbol `_mm512_setzero_ph`.
  **L3168 CN**: 继续与可调用符号 `_mm512_setzero_ph` 相关的逻辑。

### Lines 3169-3192

````c

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fcmadd_pch(__m512h __A,
                                                                  __m512h __B,
                                                                  __m512h __C) {
  return (__m512h)__builtin_ia32_vfcmaddcph512_mask3(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)-1,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fcmadd_pch(__m512h __A, __mmask16 __U, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfcmaddcph512_mask(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask3_fcmadd_pch(__m512h __A, __m512h __B, __m512h __C, __mmask16 __U) {
  return (__m512h)__builtin_ia32_vfcmaddcph512_mask3(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
````
- **L3169 EN**: Blank line separating nearby declarations or logic blocks.
  **L3169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fcmadd_pch(__m512h __A,`.
  **L3170 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fcmadd_pch(__m512h __A,`。
- **L3171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m512h __B,`.
  **L3171 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m512h __B,`。
- **L3172 EN**: Continues the surrounding expression or declaration: `__m512h __C) {`.
  **L3172 CN**: 继续构造周围的表达式或声明：`__m512h __C) {`。
- **L3173 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfcmaddcph512_mask3(`.
  **L3173 CN**: 以 `(__m512h)__builtin_ia32_vfcmaddcph512_mask3(` 从当前函数返回。
- **L3174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)-1,`.
  **L3174 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)-1,`。
- **L3175 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3175 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3176 EN**: Closes the current lexical scope or compound statement.
  **L3176 CN**: 结束当前词法作用域或复合语句块。
- **L3177 EN**: Blank line separating nearby declarations or logic blocks.
  **L3177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3178 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3178 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3179 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fcmadd_pch(__m512h __A, __mmask16 __U, __m512h __B, __m512h __C) {`.
  **L3179 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fcmadd_pch(__m512h __A, __mmask16 __U, __m512h __B, __m512h __C) {`。
- **L3180 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfcmaddcph512_mask(`.
  **L3180 CN**: 以 `(__m512h)__builtin_ia32_vfcmaddcph512_mask(` 从当前函数返回。
- **L3181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`.
  **L3181 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`。
- **L3182 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3182 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3183 EN**: Closes the current lexical scope or compound statement.
  **L3183 CN**: 结束当前词法作用域或复合语句块。
- **L3184 EN**: Blank line separating nearby declarations or logic blocks.
  **L3184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3185 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3185 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3186 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask3_fcmadd_pch(__m512h __A, __m512h __B, __m512h __C, __mmask16 __U) {`.
  **L3186 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask3_fcmadd_pch(__m512h __A, __m512h __B, __m512h __C, __mmask16 __U) {`。
- **L3187 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfcmaddcph512_mask3(`.
  **L3187 CN**: 以 `(__m512h)__builtin_ia32_vfcmaddcph512_mask3(` 从当前函数返回。
- **L3188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`.
  **L3188 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`。
- **L3189 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3189 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3190 EN**: Closes the current lexical scope or compound statement.
  **L3190 CN**: 结束当前词法作用域或复合语句块。
- **L3191 EN**: Blank line separating nearby declarations or logic blocks.
  **L3191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3192 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3192 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。

### Lines 3193-3216

````c
_mm512_maskz_fcmadd_pch(__mmask16 __U, __m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfcmaddcph512_maskz(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_fcmadd_round_pch(A, B, C, R)                                    \
  ((__m512h)__builtin_ia32_vfcmaddcph512_mask3(                                \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \
      (__mmask16)-1, (int)(R)))

#define _mm512_mask_fcmadd_round_pch(A, U, B, C, R)                            \
  ((__m512h)__builtin_ia32_vfcmaddcph512_mask(                                 \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \
      (__mmask16)(U), (int)(R)))

#define _mm512_mask3_fcmadd_round_pch(A, B, C, U, R)                           \
  ((__m512h)__builtin_ia32_vfcmaddcph512_mask3(                                \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \
      (__mmask16)(U), (int)(R)))

#define _mm512_maskz_fcmadd_round_pch(U, A, B, C, R)                           \
  ((__m512h)__builtin_ia32_vfcmaddcph512_maskz(                                \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \
````
- **L3193 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fcmadd_pch(__mmask16 __U, __m512h __A, __m512h __B, __m512h __C) {`.
  **L3193 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fcmadd_pch(__mmask16 __U, __m512h __A, __m512h __B, __m512h __C) {`。
- **L3194 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfcmaddcph512_maskz(`.
  **L3194 CN**: 以 `(__m512h)__builtin_ia32_vfcmaddcph512_maskz(` 从当前函数返回。
- **L3195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`.
  **L3195 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`。
- **L3196 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3196 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3197 EN**: Closes the current lexical scope or compound statement.
  **L3197 CN**: 结束当前词法作用域或复合语句块。
- **L3198 EN**: Blank line separating nearby declarations or logic blocks.
  **L3198 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3199 EN**: Defines macro `_mm512_fcmadd_round_pch(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3199 CN**: 定义宏 `_mm512_fcmadd_round_pch(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3200 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmaddcph512_mask3`.
  **L3200 CN**: 继续与可调用符号 `__builtin_ia32_vfcmaddcph512_mask3` 相关的逻辑。
- **L3201 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`.
  **L3201 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`。
- **L3202 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1, (int)(R)))`.
  **L3202 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1, (int)(R)))`。
- **L3203 EN**: Blank line separating nearby declarations or logic blocks.
  **L3203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3204 EN**: Defines macro `_mm512_mask_fcmadd_round_pch(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3204 CN**: 定义宏 `_mm512_mask_fcmadd_round_pch(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3205 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmaddcph512_mask`.
  **L3205 CN**: 继续与可调用符号 `__builtin_ia32_vfcmaddcph512_mask` 相关的逻辑。
- **L3206 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`.
  **L3206 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`。
- **L3207 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L3207 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L3208 EN**: Blank line separating nearby declarations or logic blocks.
  **L3208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3209 EN**: Defines macro `_mm512_mask3_fcmadd_round_pch(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L3209 CN**: 定义宏 `_mm512_mask3_fcmadd_round_pch(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L3210 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmaddcph512_mask3`.
  **L3210 CN**: 继续与可调用符号 `__builtin_ia32_vfcmaddcph512_mask3` 相关的逻辑。
- **L3211 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`.
  **L3211 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`。
- **L3212 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L3212 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L3213 EN**: Blank line separating nearby declarations or logic blocks.
  **L3213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3214 EN**: Defines macro `_mm512_maskz_fcmadd_round_pch(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3214 CN**: 定义宏 `_mm512_maskz_fcmadd_round_pch(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3215 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfcmaddcph512_maskz`.
  **L3215 CN**: 继续与可调用符号 `__builtin_ia32_vfcmaddcph512_maskz` 相关的逻辑。
- **L3216 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`.
  **L3216 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`。

### Lines 3217-3240

````c
      (__mmask16)(U), (int)(R)))

static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmadd_pch(__m512h __A,
                                                                 __m512h __B,
                                                                 __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddcph512_mask3((__v16sf)__A, (__v16sf)__B,
                                                    (__v16sf)__C, (__mmask16)-1,
                                                    _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask_fmadd_pch(__m512h __A, __mmask16 __U, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddcph512_mask((__v16sf)__A, (__v16sf)__B,
                                                   (__v16sf)__C, (__mmask16)__U,
                                                   _MM_FROUND_CUR_DIRECTION);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_mask3_fmadd_pch(__m512h __A, __m512h __B, __m512h __C, __mmask16 __U) {
  return (__m512h)__builtin_ia32_vfmaddcph512_mask3(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

````
- **L3217 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L3217 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L3218 EN**: Blank line separating nearby declarations or logic blocks.
  **L3218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmadd_pch(__m512h __A,`.
  **L3219 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m512h __DEFAULT_FN_ATTRS512 _mm512_fmadd_pch(__m512h __A,`。
- **L3220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m512h __B,`.
  **L3220 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m512h __B,`。
- **L3221 EN**: Continues the surrounding expression or declaration: `__m512h __C) {`.
  **L3221 CN**: 继续构造周围的表达式或声明：`__m512h __C) {`。
- **L3222 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddcph512_mask3((__v16sf)__A, (__v16sf)__B,`.
  **L3222 CN**: 以 `(__m512h)__builtin_ia32_vfmaddcph512_mask3((__v16sf)__A, (__v16sf)__B,` 从当前函数返回。
- **L3223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__C, (__mmask16)-1,`.
  **L3223 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__C, (__mmask16)-1,`。
- **L3224 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3224 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3225 EN**: Closes the current lexical scope or compound statement.
  **L3225 CN**: 结束当前词法作用域或复合语句块。
- **L3226 EN**: Blank line separating nearby declarations or logic blocks.
  **L3226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3227 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3227 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3228 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_fmadd_pch(__m512h __A, __mmask16 __U, __m512h __B, __m512h __C) {`.
  **L3228 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_fmadd_pch(__m512h __A, __mmask16 __U, __m512h __B, __m512h __C) {`。
- **L3229 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddcph512_mask((__v16sf)__A, (__v16sf)__B,`.
  **L3229 CN**: 以 `(__m512h)__builtin_ia32_vfmaddcph512_mask((__v16sf)__A, (__v16sf)__B,` 从当前函数返回。
- **L3230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__C, (__mmask16)__U,`.
  **L3230 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__C, (__mmask16)__U,`。
- **L3231 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3231 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3232 EN**: Closes the current lexical scope or compound statement.
  **L3232 CN**: 结束当前词法作用域或复合语句块。
- **L3233 EN**: Blank line separating nearby declarations or logic blocks.
  **L3233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3234 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3234 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3235 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask3_fmadd_pch(__m512h __A, __m512h __B, __m512h __C, __mmask16 __U) {`.
  **L3235 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask3_fmadd_pch(__m512h __A, __m512h __B, __m512h __C, __mmask16 __U) {`。
- **L3236 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddcph512_mask3(`.
  **L3236 CN**: 以 `(__m512h)__builtin_ia32_vfmaddcph512_mask3(` 从当前函数返回。
- **L3237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`.
  **L3237 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`。
- **L3238 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3238 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3239 EN**: Closes the current lexical scope or compound statement.
  **L3239 CN**: 结束当前词法作用域或复合语句块。
- **L3240 EN**: Blank line separating nearby declarations or logic blocks.
  **L3240 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 3241-3264

````c
static __inline__ __m512h __DEFAULT_FN_ATTRS512
_mm512_maskz_fmadd_pch(__mmask16 __U, __m512h __A, __m512h __B, __m512h __C) {
  return (__m512h)__builtin_ia32_vfmaddcph512_maskz(
      (__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,
      _MM_FROUND_CUR_DIRECTION);
}

#define _mm512_fmadd_round_pch(A, B, C, R)                                     \
  ((__m512h)__builtin_ia32_vfmaddcph512_mask3(                                 \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \
      (__mmask16)-1, (int)(R)))

#define _mm512_mask_fmadd_round_pch(A, U, B, C, R)                             \
  ((__m512h)__builtin_ia32_vfmaddcph512_mask(                                  \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \
      (__mmask16)(U), (int)(R)))

#define _mm512_mask3_fmadd_round_pch(A, B, C, U, R)                            \
  ((__m512h)__builtin_ia32_vfmaddcph512_mask3(                                 \
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \
      (__mmask16)(U), (int)(R)))

#define _mm512_maskz_fmadd_round_pch(U, A, B, C, R)                            \
  ((__m512h)__builtin_ia32_vfmaddcph512_maskz(                                 \
````
- **L3241 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512`.
  **L3241 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512`。
- **L3242 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_maskz_fmadd_pch(__mmask16 __U, __m512h __A, __m512h __B, __m512h __C) {`.
  **L3242 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_maskz_fmadd_pch(__mmask16 __U, __m512h __A, __m512h __B, __m512h __C) {`。
- **L3243 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vfmaddcph512_maskz(`.
  **L3243 CN**: 以 `(__m512h)__builtin_ia32_vfmaddcph512_maskz(` 从当前函数返回。
- **L3244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`.
  **L3244 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16sf)__A, (__v16sf)__B, (__v16sf)__C, (__mmask16)__U,`。
- **L3245 EN**: Adds a standalone statement or declaration: `_MM_FROUND_CUR_DIRECTION);`.
  **L3245 CN**: 添加一条独立语句或声明：`_MM_FROUND_CUR_DIRECTION);`。
- **L3246 EN**: Closes the current lexical scope or compound statement.
  **L3246 CN**: 结束当前词法作用域或复合语句块。
- **L3247 EN**: Blank line separating nearby declarations or logic blocks.
  **L3247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3248 EN**: Defines macro `_mm512_fmadd_round_pch(A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3248 CN**: 定义宏 `_mm512_fmadd_round_pch(A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3249 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddcph512_mask3`.
  **L3249 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddcph512_mask3` 相关的逻辑。
- **L3250 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`.
  **L3250 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`。
- **L3251 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1, (int)(R)))`.
  **L3251 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1, (int)(R)))`。
- **L3252 EN**: Blank line separating nearby declarations or logic blocks.
  **L3252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3253 EN**: Defines macro `_mm512_mask_fmadd_round_pch(A, U, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3253 CN**: 定义宏 `_mm512_mask_fmadd_round_pch(A, U, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3254 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddcph512_mask`.
  **L3254 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddcph512_mask` 相关的逻辑。
- **L3255 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`.
  **L3255 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`。
- **L3256 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L3256 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L3257 EN**: Blank line separating nearby declarations or logic blocks.
  **L3257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3258 EN**: Defines macro `_mm512_mask3_fmadd_round_pch(A, B, C, U, R)` for conditional compilation, shorthand, or API generation.
  **L3258 CN**: 定义宏 `_mm512_mask3_fmadd_round_pch(A, B, C, U, R)`，用于条件编译、简写或 API 生成。
- **L3259 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddcph512_mask3`.
  **L3259 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddcph512_mask3` 相关的逻辑。
- **L3260 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`.
  **L3260 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`。
- **L3261 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L3261 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L3262 EN**: Blank line separating nearby declarations or logic blocks.
  **L3262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3263 EN**: Defines macro `_mm512_maskz_fmadd_round_pch(U, A, B, C, R)` for conditional compilation, shorthand, or API generation.
  **L3263 CN**: 定义宏 `_mm512_maskz_fmadd_round_pch(U, A, B, C, R)`，用于条件编译、简写或 API 生成。
- **L3264 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfmaddcph512_maskz`.
  **L3264 CN**: 继续与可调用符号 `__builtin_ia32_vfmaddcph512_maskz` 相关的逻辑。

### Lines 3265-3288

````c
      (__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \
      (__mmask16)(U), (int)(R)))

static __inline__ _Float16 __DEFAULT_FN_ATTRS512
_mm512_reduce_add_ph(__m512h __W) {
  return __builtin_ia32_reduce_fadd_ph512(-0.0f16, __W);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS512
_mm512_reduce_mul_ph(__m512h __W) {
  return __builtin_ia32_reduce_fmul_ph512(1.0f16, __W);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS512
_mm512_reduce_max_ph(__m512h __V) {
  return __builtin_ia32_reduce_fmax_ph512(__V);
}

static __inline__ _Float16 __DEFAULT_FN_ATTRS512
_mm512_reduce_min_ph(__m512h __V) {
  return __builtin_ia32_reduce_fmin_ph512(__V);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
````
- **L3265 EN**: Continues the surrounding expression or declaration: `(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`.
  **L3265 CN**: 继续构造周围的表达式或声明：`(__v16sf)(__m512h)(A), (__v16sf)(__m512h)(B), (__v16sf)(__m512h)(C),     \`。
- **L3266 EN**: Continues the surrounding expression or declaration: `(__mmask16)(U), (int)(R)))`.
  **L3266 CN**: 继续构造周围的表达式或声明：`(__mmask16)(U), (int)(R)))`。
- **L3267 EN**: Blank line separating nearby declarations or logic blocks.
  **L3267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3268 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS512`.
  **L3268 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS512`。
- **L3269 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_reduce_add_ph(__m512h __W) {`.
  **L3269 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_reduce_add_ph(__m512h __W) {`。
- **L3270 EN**: Returns from the current function with `__builtin_ia32_reduce_fadd_ph512(-0.0f16, __W)`.
  **L3270 CN**: 以 `__builtin_ia32_reduce_fadd_ph512(-0.0f16, __W)` 从当前函数返回。
- **L3271 EN**: Closes the current lexical scope or compound statement.
  **L3271 CN**: 结束当前词法作用域或复合语句块。
- **L3272 EN**: Blank line separating nearby declarations or logic blocks.
  **L3272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3273 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS512`.
  **L3273 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS512`。
- **L3274 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_reduce_mul_ph(__m512h __W) {`.
  **L3274 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_reduce_mul_ph(__m512h __W) {`。
- **L3275 EN**: Returns from the current function with `__builtin_ia32_reduce_fmul_ph512(1.0f16, __W)`.
  **L3275 CN**: 以 `__builtin_ia32_reduce_fmul_ph512(1.0f16, __W)` 从当前函数返回。
- **L3276 EN**: Closes the current lexical scope or compound statement.
  **L3276 CN**: 结束当前词法作用域或复合语句块。
- **L3277 EN**: Blank line separating nearby declarations or logic blocks.
  **L3277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3278 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS512`.
  **L3278 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS512`。
- **L3279 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_reduce_max_ph(__m512h __V) {`.
  **L3279 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_reduce_max_ph(__m512h __V) {`。
- **L3280 EN**: Returns from the current function with `__builtin_ia32_reduce_fmax_ph512(__V)`.
  **L3280 CN**: 以 `__builtin_ia32_reduce_fmax_ph512(__V)` 从当前函数返回。
- **L3281 EN**: Closes the current lexical scope or compound statement.
  **L3281 CN**: 结束当前词法作用域或复合语句块。
- **L3282 EN**: Blank line separating nearby declarations or logic blocks.
  **L3282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3283 EN**: Continues the surrounding expression or declaration: `static __inline__ _Float16 __DEFAULT_FN_ATTRS512`.
  **L3283 CN**: 继续构造周围的表达式或声明：`static __inline__ _Float16 __DEFAULT_FN_ATTRS512`。
- **L3284 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_reduce_min_ph(__m512h __V) {`.
  **L3284 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_reduce_min_ph(__m512h __V) {`。
- **L3285 EN**: Returns from the current function with `__builtin_ia32_reduce_fmin_ph512(__V)`.
  **L3285 CN**: 以 `__builtin_ia32_reduce_fmin_ph512(__V)` 从当前函数返回。
- **L3286 EN**: Closes the current lexical scope or compound statement.
  **L3286 CN**: 结束当前词法作用域或复合语句块。
- **L3287 EN**: Blank line separating nearby declarations or logic blocks.
  **L3287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3288 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L3288 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。

### Lines 3289-3312

````c
_mm512_mask_blend_ph(__mmask32 __U, __m512h __A, __m512h __W) {
  return (__m512h)__builtin_ia32_selectph_512((__mmask32)__U, (__v32hf)__W,
                                              (__v32hf)__A);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_permutex2var_ph(__m512h __A, __m512i __I, __m512h __B) {
  return (__m512h)__builtin_ia32_vpermi2varhi512((__v32hi)__A, (__v32hi)__I,
                                                 (__v32hi)__B);
}

static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR
_mm512_permutexvar_ph(__m512i __A, __m512h __B) {
  return (__m512h)__builtin_ia32_permvarhi512((__v32hi)__B, (__v32hi)__A);
}

// intrinsics below are alias for f*mul_*ch
#define _mm512_mul_pch(A, B) _mm512_fmul_pch(A, B)
#define _mm512_mask_mul_pch(W, U, A, B) _mm512_mask_fmul_pch(W, U, A, B)
#define _mm512_maskz_mul_pch(U, A, B) _mm512_maskz_fmul_pch(U, A, B)
#define _mm512_mul_round_pch(A, B, R) _mm512_fmul_round_pch(A, B, R)
#define _mm512_mask_mul_round_pch(W, U, A, B, R)                               \
  _mm512_mask_fmul_round_pch(W, U, A, B, R)
#define _mm512_maskz_mul_round_pch(U, A, B, R)                                 \
````
- **L3289 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_mask_blend_ph(__mmask32 __U, __m512h __A, __m512h __W) {`.
  **L3289 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_mask_blend_ph(__mmask32 __U, __m512h __A, __m512h __W) {`。
- **L3290 EN**: Returns from the current function with `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U, (__v32hf)__W,`.
  **L3290 CN**: 以 `(__m512h)__builtin_ia32_selectph_512((__mmask32)__U, (__v32hf)__W,` 从当前函数返回。
- **L3291 EN**: Executes a call or declaration centered on `statement`.
  **L3291 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3292 EN**: Closes the current lexical scope or compound statement.
  **L3292 CN**: 结束当前词法作用域或复合语句块。
- **L3293 EN**: Blank line separating nearby declarations or logic blocks.
  **L3293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3294 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L3294 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L3295 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_permutex2var_ph(__m512h __A, __m512i __I, __m512h __B) {`.
  **L3295 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_permutex2var_ph(__m512h __A, __m512i __I, __m512h __B) {`。
- **L3296 EN**: Returns from the current function with `(__m512h)__builtin_ia32_vpermi2varhi512((__v32hi)__A, (__v32hi)__I,`.
  **L3296 CN**: 以 `(__m512h)__builtin_ia32_vpermi2varhi512((__v32hi)__A, (__v32hi)__I,` 从当前函数返回。
- **L3297 EN**: Executes a call or declaration centered on `statement`.
  **L3297 CN**: 执行以 `statement` 为核心的调用或声明。
- **L3298 EN**: Closes the current lexical scope or compound statement.
  **L3298 CN**: 结束当前词法作用域或复合语句块。
- **L3299 EN**: Blank line separating nearby declarations or logic blocks.
  **L3299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3300 EN**: Continues the surrounding expression or declaration: `static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L3300 CN**: 继续构造周围的表达式或声明：`static __inline__ __m512h __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L3301 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm512_permutexvar_ph(__m512i __A, __m512h __B) {`.
  **L3301 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm512_permutexvar_ph(__m512i __A, __m512h __B) {`。
- **L3302 EN**: Returns from the current function with `(__m512h)__builtin_ia32_permvarhi512((__v32hi)__B, (__v32hi)__A)`.
  **L3302 CN**: 以 `(__m512h)__builtin_ia32_permvarhi512((__v32hi)__B, (__v32hi)__A)` 从当前函数返回。
- **L3303 EN**: Closes the current lexical scope or compound statement.
  **L3303 CN**: 结束当前词法作用域或复合语句块。
- **L3304 EN**: Blank line separating nearby declarations or logic blocks.
  **L3304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3305 EN**: Comment explains nearby logic, constraints, or intent: `intrinsics below are alias for f*mul_*ch`.
  **L3305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`intrinsics below are alias for f*mul_*ch`。
- **L3306 EN**: Defines macro `_mm512_mul_pch(A, B)` for conditional compilation, shorthand, or API generation.
  **L3306 CN**: 定义宏 `_mm512_mul_pch(A, B)`，用于条件编译、简写或 API 生成。
- **L3307 EN**: Defines macro `_mm512_mask_mul_pch(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L3307 CN**: 定义宏 `_mm512_mask_mul_pch(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L3308 EN**: Defines macro `_mm512_maskz_mul_pch(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L3308 CN**: 定义宏 `_mm512_maskz_mul_pch(U, A, B)`，用于条件编译、简写或 API 生成。
- **L3309 EN**: Defines macro `_mm512_mul_round_pch(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3309 CN**: 定义宏 `_mm512_mul_round_pch(A, B, R)`，用于条件编译、简写或 API 生成。
- **L3310 EN**: Defines macro `_mm512_mask_mul_round_pch(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3310 CN**: 定义宏 `_mm512_mask_mul_round_pch(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3311 EN**: Continues logic associated with callable symbol `_mm512_mask_fmul_round_pch`.
  **L3311 CN**: 继续与可调用符号 `_mm512_mask_fmul_round_pch` 相关的逻辑。
- **L3312 EN**: Defines macro `_mm512_maskz_mul_round_pch(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3312 CN**: 定义宏 `_mm512_maskz_mul_round_pch(U, A, B, R)`，用于条件编译、简写或 API 生成。

### Lines 3313-3336

````c
  _mm512_maskz_fmul_round_pch(U, A, B, R)

#define _mm512_cmul_pch(A, B) _mm512_fcmul_pch(A, B)
#define _mm512_mask_cmul_pch(W, U, A, B) _mm512_mask_fcmul_pch(W, U, A, B)
#define _mm512_maskz_cmul_pch(U, A, B) _mm512_maskz_fcmul_pch(U, A, B)
#define _mm512_cmul_round_pch(A, B, R) _mm512_fcmul_round_pch(A, B, R)
#define _mm512_mask_cmul_round_pch(W, U, A, B, R)                              \
  _mm512_mask_fcmul_round_pch(W, U, A, B, R)
#define _mm512_maskz_cmul_round_pch(U, A, B, R)                                \
  _mm512_maskz_fcmul_round_pch(U, A, B, R)

#define _mm_mul_sch(A, B) _mm_fmul_sch(A, B)
#define _mm_mask_mul_sch(W, U, A, B) _mm_mask_fmul_sch(W, U, A, B)
#define _mm_maskz_mul_sch(U, A, B) _mm_maskz_fmul_sch(U, A, B)
#define _mm_mul_round_sch(A, B, R) _mm_fmul_round_sch(A, B, R)
#define _mm_mask_mul_round_sch(W, U, A, B, R)                                  \
  _mm_mask_fmul_round_sch(W, U, A, B, R)
#define _mm_maskz_mul_round_sch(U, A, B, R) _mm_maskz_fmul_round_sch(U, A, B, R)

#define _mm_cmul_sch(A, B) _mm_fcmul_sch(A, B)
#define _mm_mask_cmul_sch(W, U, A, B) _mm_mask_fcmul_sch(W, U, A, B)
#define _mm_maskz_cmul_sch(U, A, B) _mm_maskz_fcmul_sch(U, A, B)
#define _mm_cmul_round_sch(A, B, R) _mm_fcmul_round_sch(A, B, R)
#define _mm_mask_cmul_round_sch(W, U, A, B, R)                                 \
````
- **L3313 EN**: Continues logic associated with callable symbol `_mm512_maskz_fmul_round_pch`.
  **L3313 CN**: 继续与可调用符号 `_mm512_maskz_fmul_round_pch` 相关的逻辑。
- **L3314 EN**: Blank line separating nearby declarations or logic blocks.
  **L3314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3315 EN**: Defines macro `_mm512_cmul_pch(A, B)` for conditional compilation, shorthand, or API generation.
  **L3315 CN**: 定义宏 `_mm512_cmul_pch(A, B)`，用于条件编译、简写或 API 生成。
- **L3316 EN**: Defines macro `_mm512_mask_cmul_pch(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L3316 CN**: 定义宏 `_mm512_mask_cmul_pch(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L3317 EN**: Defines macro `_mm512_maskz_cmul_pch(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L3317 CN**: 定义宏 `_mm512_maskz_cmul_pch(U, A, B)`，用于条件编译、简写或 API 生成。
- **L3318 EN**: Defines macro `_mm512_cmul_round_pch(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3318 CN**: 定义宏 `_mm512_cmul_round_pch(A, B, R)`，用于条件编译、简写或 API 生成。
- **L3319 EN**: Defines macro `_mm512_mask_cmul_round_pch(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3319 CN**: 定义宏 `_mm512_mask_cmul_round_pch(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3320 EN**: Continues logic associated with callable symbol `_mm512_mask_fcmul_round_pch`.
  **L3320 CN**: 继续与可调用符号 `_mm512_mask_fcmul_round_pch` 相关的逻辑。
- **L3321 EN**: Defines macro `_mm512_maskz_cmul_round_pch(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3321 CN**: 定义宏 `_mm512_maskz_cmul_round_pch(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3322 EN**: Continues logic associated with callable symbol `_mm512_maskz_fcmul_round_pch`.
  **L3322 CN**: 继续与可调用符号 `_mm512_maskz_fcmul_round_pch` 相关的逻辑。
- **L3323 EN**: Blank line separating nearby declarations or logic blocks.
  **L3323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3324 EN**: Defines macro `_mm_mul_sch(A, B)` for conditional compilation, shorthand, or API generation.
  **L3324 CN**: 定义宏 `_mm_mul_sch(A, B)`，用于条件编译、简写或 API 生成。
- **L3325 EN**: Defines macro `_mm_mask_mul_sch(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L3325 CN**: 定义宏 `_mm_mask_mul_sch(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L3326 EN**: Defines macro `_mm_maskz_mul_sch(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L3326 CN**: 定义宏 `_mm_maskz_mul_sch(U, A, B)`，用于条件编译、简写或 API 生成。
- **L3327 EN**: Defines macro `_mm_mul_round_sch(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3327 CN**: 定义宏 `_mm_mul_round_sch(A, B, R)`，用于条件编译、简写或 API 生成。
- **L3328 EN**: Defines macro `_mm_mask_mul_round_sch(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3328 CN**: 定义宏 `_mm_mask_mul_round_sch(W, U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3329 EN**: Continues logic associated with callable symbol `_mm_mask_fmul_round_sch`.
  **L3329 CN**: 继续与可调用符号 `_mm_mask_fmul_round_sch` 相关的逻辑。
- **L3330 EN**: Defines macro `_mm_maskz_mul_round_sch(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3330 CN**: 定义宏 `_mm_maskz_mul_round_sch(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3331 EN**: Blank line separating nearby declarations or logic blocks.
  **L3331 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3332 EN**: Defines macro `_mm_cmul_sch(A, B)` for conditional compilation, shorthand, or API generation.
  **L3332 CN**: 定义宏 `_mm_cmul_sch(A, B)`，用于条件编译、简写或 API 生成。
- **L3333 EN**: Defines macro `_mm_mask_cmul_sch(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L3333 CN**: 定义宏 `_mm_mask_cmul_sch(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L3334 EN**: Defines macro `_mm_maskz_cmul_sch(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L3334 CN**: 定义宏 `_mm_maskz_cmul_sch(U, A, B)`，用于条件编译、简写或 API 生成。
- **L3335 EN**: Defines macro `_mm_cmul_round_sch(A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3335 CN**: 定义宏 `_mm_cmul_round_sch(A, B, R)`，用于条件编译、简写或 API 生成。
- **L3336 EN**: Defines macro `_mm_mask_cmul_round_sch(W, U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3336 CN**: 定义宏 `_mm_mask_cmul_round_sch(W, U, A, B, R)`，用于条件编译、简写或 API 生成。

### Lines 3337-3349

````c
  _mm_mask_fcmul_round_sch(W, U, A, B, R)
#define _mm_maskz_cmul_round_sch(U, A, B, R)                                   \
  _mm_maskz_fcmul_round_sch(U, A, B, R)

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256
#undef __DEFAULT_FN_ATTRS512
#undef __DEFAULT_FN_ATTRS128_CONSTEXPR
#undef __DEFAULT_FN_ATTRS256_CONSTEXPR
#undef __DEFAULT_FN_ATTRS512_CONSTEXPR

#endif
#endif
````
- **L3337 EN**: Continues logic associated with callable symbol `_mm_mask_fcmul_round_sch`.
  **L3337 CN**: 继续与可调用符号 `_mm_mask_fcmul_round_sch` 相关的逻辑。
- **L3338 EN**: Defines macro `_mm_maskz_cmul_round_sch(U, A, B, R)` for conditional compilation, shorthand, or API generation.
  **L3338 CN**: 定义宏 `_mm_maskz_cmul_round_sch(U, A, B, R)`，用于条件编译、简写或 API 生成。
- **L3339 EN**: Continues logic associated with callable symbol `_mm_maskz_fcmul_round_sch`.
  **L3339 CN**: 继续与可调用符号 `_mm_maskz_fcmul_round_sch` 相关的逻辑。
- **L3340 EN**: Blank line separating nearby declarations or logic blocks.
  **L3340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3341 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L3341 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L3342 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L3342 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。
- **L3343 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS512`.
  **L3343 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS512`。
- **L3344 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L3344 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L3345 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3345 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3346 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS512_CONSTEXPR`.
  **L3346 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS512_CONSTEXPR`。
- **L3347 EN**: Blank line separating nearby declarations or logic blocks.
  **L3347 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3348 EN**: Closes the current preprocessor conditional block.
  **L3348 CN**: 结束当前预处理条件块。
- **L3349 EN**: Closes the current preprocessor conditional block.
  **L3349 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **x86 SIMD or system intrinsics / x86 SIMD 或系统 intrinsic 接口**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Inline wrapper functions / 内联包装函数**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Structured data declarations / 结构化数据声明**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__SSE2__`, `__AVX512FP16INTRIN_H`, `__cplusplus`, `__x86_64__`
- **External builtins / 外部 builtin**: `__builtin_ia32_undef256`, `__builtin_ia32_undef128`, `__builtin_ia32_undef512`, `__builtin_bit_cast`, `__builtin_shufflevector`, `__builtin_nondeterministic_value`, `__builtin_ia32_vcomish`, `__builtin_ia32_selectph_512`, `__builtin_ia32_addph512`, `__builtin_ia32_subph512`, `__builtin_ia32_mulph512`, `__builtin_ia32_divph512`
