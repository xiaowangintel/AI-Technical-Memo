# avx10_2bf16intrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx10_2bf16intrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: AVX10-BF16 intrinsics.
- **Purpose (CN)**: 提供 AVX10-BF16 intrinsic 接口。
- **Line Count / 行数**: 1094

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===-------------- avx10_2bf16intrin.h - AVX10-BF16 intrinsics ------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */
#ifndef __IMMINTRIN_H
#error                                                                         \
    "Never use <avx10_2bf16intrin.h> directly; include <immintrin.h> instead."
#endif

#ifdef __SSE2__

#ifndef __AVX10_2BF16INTRIN_H
#define __AVX10_2BF16INTRIN_H

typedef __bf16 __m128bh_u __attribute__((__vector_size__(16), __aligned__(1)));
typedef __bf16 __m256bh_u __attribute__((__vector_size__(32), __aligned__(1)));

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS256                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \
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
- **L10 EN**: Emits a compilation error for an unsupported configuration: `#error                                                                         \`.
  **L10 CN**: 为不受支持的配置触发编译错误：`#error                                                                         \`。
- **L11 EN**: Continues the surrounding expression or declaration: `"Never use <avx10_2bf16intrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 继续构造周围的表达式或声明：`"Never use <avx10_2bf16intrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifdef __SSE2__`.
  **L14 CN**: 开始一个预处理条件块：`#ifdef __SSE2__`。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Starts a preprocessor conditional block: `#ifndef __AVX10_2BF16INTRIN_H`.
  **L16 CN**: 开始一个预处理条件块：`#ifndef __AVX10_2BF16INTRIN_H`。
- **L17 EN**: Defines macro `__AVX10_2BF16INTRIN_H` for conditional compilation, shorthand, or API generation.
  **L17 CN**: 定义宏 `__AVX10_2BF16INTRIN_H`，用于条件编译、简写或 API 生成。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Introduces an alias or helper declaration: `typedef __bf16 __m128bh_u __attribute__((__vector_size__(16), __aligned__(1)));`.
  **L19 CN**: 引入一条别名或辅助声明：`typedef __bf16 __m128bh_u __attribute__((__vector_size__(16), __aligned__(1)));`。
- **L20 EN**: Introduces an alias or helper declaration: `typedef __bf16 __m256bh_u __attribute__((__vector_size__(32), __aligned__(1)));`.
  **L20 CN**: 引入一条别名或辅助声明：`typedef __bf16 __m256bh_u __attribute__((__vector_size__(32), __aligned__(1)));`。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L22 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L22 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L23 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L23 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L24 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`.
  **L24 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`。

### Lines 25-48

````c
                 __min_vector_width__(256)))
#define __DEFAULT_FN_ATTRS128                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \
                 __min_vector_width__(128)))

#if defined(__cplusplus) && (__cplusplus >= 201103L)
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128 constexpr
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256 constexpr
#else
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256
#endif

static __inline __m256bh __DEFAULT_FN_ATTRS256 _mm256_setzero_pbh(void) {
  return __builtin_bit_cast(__m256bh, _mm256_setzero_ps());
}

static __inline __m128bh __DEFAULT_FN_ATTRS128 _mm_setzero_pbh(void) {
  return __builtin_bit_cast(__m128bh, _mm_setzero_ps());
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_castbf16_ps(__m128bh __a) {
  return (__m128)__a;
}
````
- **L25 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L25 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L26 EN**: Defines macro `__DEFAULT_FN_ATTRS128` for conditional compilation, shorthand, or API generation.
  **L26 CN**: 定义宏 `__DEFAULT_FN_ATTRS128`，用于条件编译、简写或 API 生成。
- **L27 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`.
  **L27 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`。
- **L28 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L28 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L30 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L30 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L31 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L31 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L32 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L32 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L33 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L33 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L34 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L34 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L35 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L35 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L36 EN**: Closes the current preprocessor conditional block.
  **L36 CN**: 结束当前预处理条件块。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline __m256bh __DEFAULT_FN_ATTRS256 _mm256_setzero_pbh(void) {`.
  **L38 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline __m256bh __DEFAULT_FN_ATTRS256 _mm256_setzero_pbh(void) {`。
- **L39 EN**: Returns from the current function with `__builtin_bit_cast(__m256bh, _mm256_setzero_ps())`.
  **L39 CN**: 以 `__builtin_bit_cast(__m256bh, _mm256_setzero_ps())` 从当前函数返回。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline __m128bh __DEFAULT_FN_ATTRS128 _mm_setzero_pbh(void) {`.
  **L42 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline __m128bh __DEFAULT_FN_ATTRS128 _mm_setzero_pbh(void) {`。
- **L43 EN**: Returns from the current function with `__builtin_bit_cast(__m128bh, _mm_setzero_ps())`.
  **L43 CN**: 以 `__builtin_bit_cast(__m128bh, _mm_setzero_ps())` 从当前函数返回。
- **L44 EN**: Closes the current lexical scope or compound statement.
  **L44 CN**: 结束当前词法作用域或复合语句块。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L46 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_castbf16_ps(__m128bh __a) {`.
  **L46 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128 __DEFAULT_FN_ATTRS128 _mm_castbf16_ps(__m128bh __a) {`。
- **L47 EN**: Returns from the current function with `(__m128)__a`.
  **L47 CN**: 以 `(__m128)__a` 从当前函数返回。
- **L48 EN**: Closes the current lexical scope or compound statement.
  **L48 CN**: 结束当前词法作用域或复合语句块。

### Lines 49-72

````c

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_castbf16_ps(__m256bh __a) {
  return (__m256)__a;
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_castbf16_pd(__m256bh __a) {
  return (__m256d)__a;
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_castbf16_pd(__m128bh __a) {
  return (__m128d)__a;
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_castbf16_si128(__m128bh __a) {
  return (__m128i)__a;
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_castbf16_si256(__m256bh __a) {
  return (__m256i)__a;
}
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L50 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L50 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L51 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castbf16_ps(__m256bh __a) {`.
  **L51 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castbf16_ps(__m256bh __a) {`。
- **L52 EN**: Returns from the current function with `(__m256)__a`.
  **L52 CN**: 以 `(__m256)__a` 从当前函数返回。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L55 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L55 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L56 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castbf16_pd(__m256bh __a) {`.
  **L56 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castbf16_pd(__m256bh __a) {`。
- **L57 EN**: Returns from the current function with `(__m256d)__a`.
  **L57 CN**: 以 `(__m256d)__a` 从当前函数返回。
- **L58 EN**: Closes the current lexical scope or compound statement.
  **L58 CN**: 结束当前词法作用域或复合语句块。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L60 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_castbf16_pd(__m128bh __a) {`.
  **L60 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128d __DEFAULT_FN_ATTRS128 _mm_castbf16_pd(__m128bh __a) {`。
- **L61 EN**: Returns from the current function with `(__m128d)__a`.
  **L61 CN**: 以 `(__m128d)__a` 从当前函数返回。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L64 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L65 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_castbf16_si128(__m128bh __a) {`.
  **L65 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_castbf16_si128(__m128bh __a) {`。
- **L66 EN**: Returns from the current function with `(__m128i)__a`.
  **L66 CN**: 以 `(__m128i)__a` 从当前函数返回。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L69 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L69 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L70 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castbf16_si256(__m256bh __a) {`.
  **L70 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castbf16_si256(__m256bh __a) {`。
- **L71 EN**: Returns from the current function with `(__m256i)__a`.
  **L71 CN**: 以 `(__m256i)__a` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-96

````c

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_castps_pbh(__m128 __a) {
  return (__m128bh)__a;
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_castps_pbh(__m256 __a) {
  return (__m256bh)__a;
}

static __inline__ __bf16 __DEFAULT_FN_ATTRS128 _mm_cvtsbh_bf16(__m128bh __a) {
  return __a[0];
}

static __inline__ __bf16 __DEFAULT_FN_ATTRS256
_mm256_cvtsbh_bf16(__m256bh __a) {
  return __a[0];
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_castpd_pbh(__m128d __a) {
  return (__m128bh)__a;
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_castpd_pbh(__m256d __a) {
````
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L74 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_castps_pbh(__m128 __a) {`.
  **L74 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_castps_pbh(__m128 __a) {`。
- **L75 EN**: Returns from the current function with `(__m128bh)__a`.
  **L75 CN**: 以 `(__m128bh)__a` 从当前函数返回。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L78 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_castps_pbh(__m256 __a) {`.
  **L78 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_castps_pbh(__m256 __a) {`。
- **L79 EN**: Returns from the current function with `(__m256bh)__a`.
  **L79 CN**: 以 `(__m256bh)__a` 从当前函数返回。
- **L80 EN**: Closes the current lexical scope or compound statement.
  **L80 CN**: 结束当前词法作用域或复合语句块。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L82 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __bf16 __DEFAULT_FN_ATTRS128 _mm_cvtsbh_bf16(__m128bh __a) {`.
  **L82 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __bf16 __DEFAULT_FN_ATTRS128 _mm_cvtsbh_bf16(__m128bh __a) {`。
- **L83 EN**: Returns from the current function with `__a[0]`.
  **L83 CN**: 以 `__a[0]` 从当前函数返回。
- **L84 EN**: Closes the current lexical scope or compound statement.
  **L84 CN**: 结束当前词法作用域或复合语句块。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L86 EN**: Continues the surrounding expression or declaration: `static __inline__ __bf16 __DEFAULT_FN_ATTRS256`.
  **L86 CN**: 继续构造周围的表达式或声明：`static __inline__ __bf16 __DEFAULT_FN_ATTRS256`。
- **L87 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtsbh_bf16(__m256bh __a) {`.
  **L87 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtsbh_bf16(__m256bh __a) {`。
- **L88 EN**: Returns from the current function with `__a[0]`.
  **L88 CN**: 以 `__a[0]` 从当前函数返回。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L91 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_castpd_pbh(__m128d __a) {`.
  **L91 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_castpd_pbh(__m128d __a) {`。
- **L92 EN**: Returns from the current function with `(__m128bh)__a`.
  **L92 CN**: 以 `(__m128bh)__a` 从当前函数返回。
- **L93 EN**: Closes the current lexical scope or compound statement.
  **L93 CN**: 结束当前词法作用域或复合语句块。
- **L94 EN**: Blank line separating nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L95 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L95 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L96 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castpd_pbh(__m256d __a) {`.
  **L96 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castpd_pbh(__m256d __a) {`。

### Lines 97-120

````c
  return (__m256bh)__a;
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_castsi128_pbh(__m128i __a) {
  return (__m128bh)__a;
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_castsi256_pbh(__m256i __a) {
  return (__m256bh)__a;
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS256
_mm256_castbf16256_pbh128(__m256bh __a) {
  return __builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_castbf16128_pbh256(__m128bh __a) {
  return __builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7, -1, -1, -1,
                                 -1, -1, -1, -1, -1);
}

````
- **L97 EN**: Returns from the current function with `(__m256bh)__a`.
  **L97 CN**: 以 `(__m256bh)__a` 从当前函数返回。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L100 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L100 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L101 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_castsi128_pbh(__m128i __a) {`.
  **L101 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_castsi128_pbh(__m128i __a) {`。
- **L102 EN**: Returns from the current function with `(__m128bh)__a`.
  **L102 CN**: 以 `(__m128bh)__a` 从当前函数返回。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L105 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L106 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castsi256_pbh(__m256i __a) {`.
  **L106 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castsi256_pbh(__m256i __a) {`。
- **L107 EN**: Returns from the current function with `(__m256bh)__a`.
  **L107 CN**: 以 `(__m256bh)__a` 从当前函数返回。
- **L108 EN**: Closes the current lexical scope or compound statement.
  **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Blank line separating nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L110 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS256`.
  **L110 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS256`。
- **L111 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castbf16256_pbh128(__m256bh __a) {`.
  **L111 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castbf16256_pbh128(__m256bh __a) {`。
- **L112 EN**: Returns from the current function with `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7)`.
  **L112 CN**: 以 `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7)` 从当前函数返回。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L115 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L116 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_castbf16128_pbh256(__m128bh __a) {`.
  **L116 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_castbf16128_pbh256(__m128bh __a) {`。
- **L117 EN**: Returns from the current function with `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7, -1, -1, -1,`.
  **L117 CN**: 以 `__builtin_shufflevector(__a, __a, 0, 1, 2, 3, 4, 5, 6, 7, -1, -1, -1,` 从当前函数返回。
- **L118 EN**: Adds a standalone statement or declaration: `-1, -1, -1, -1, -1);`.
  **L118 CN**: 添加一条独立语句或声明：`-1, -1, -1, -1, -1);`。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 121-144

````c
static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_zextbf16128_pbh256(__m128bh __a) {
  return __builtin_shufflevector(__a, (__v8bf)_mm_setzero_pbh(), 0, 1, 2, 3, 4,
                                 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_undefined_pbh(void) {
  return (__m256bh)__builtin_ia32_undef256();
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_load_sbh(void const *__dp) {
  __m128bh src = (__v8bf)_mm_setzero_pbh();
  return (__m128bh)__builtin_ia32_loadsbf16128_mask((const __v8bf *)__dp, src,
                                                    1);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_load_sbh(__m128bh __W, __mmask8 __U, const void *__A) {
  __m128bh src = (__v8bf)__builtin_shufflevector(
      (__v8bf)__W, (__v8bf)_mm_setzero_pbh(), 0, 8, 8, 8, 8, 8, 8, 8);

  return (__m128bh)__builtin_ia32_loadsbf16128_mask((const __v8bf *)__A, src,
                                                    __U & 1);
````
- **L121 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L121 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L122 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_zextbf16128_pbh256(__m128bh __a) {`.
  **L122 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_zextbf16128_pbh256(__m128bh __a) {`。
- **L123 EN**: Returns from the current function with `__builtin_shufflevector(__a, (__v8bf)_mm_setzero_pbh(), 0, 1, 2, 3, 4,`.
  **L123 CN**: 以 `__builtin_shufflevector(__a, (__v8bf)_mm_setzero_pbh(), 0, 1, 2, 3, 4,` 从当前函数返回。
- **L124 EN**: Adds a standalone statement or declaration: `5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);`.
  **L124 CN**: 添加一条独立语句或声明：`5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);`。
- **L125 EN**: Closes the current lexical scope or compound statement.
  **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L127 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_undefined_pbh(void) {`.
  **L127 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_undefined_pbh(void) {`。
- **L128 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_undef256()`.
  **L128 CN**: 以 `(__m256bh)__builtin_ia32_undef256()` 从当前函数返回。
- **L129 EN**: Closes the current lexical scope or compound statement.
  **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L131 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L131 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L132 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_load_sbh(void const *__dp) {`.
  **L132 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_load_sbh(void const *__dp) {`。
- **L133 EN**: Initializes variable `src` from the expression on the right-hand side.
  **L133 CN**: 使用右侧表达式初始化变量 `src`。
- **L134 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_loadsbf16128_mask((const __v8bf *)__dp, src,`.
  **L134 CN**: 以 `(__m128bh)__builtin_ia32_loadsbf16128_mask((const __v8bf *)__dp, src,` 从当前函数返回。
- **L135 EN**: Adds a standalone statement or declaration: `1);`.
  **L135 CN**: 添加一条独立语句或声明：`1);`。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L138 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L139 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_load_sbh(__m128bh __W, __mmask8 __U, const void *__A) {`.
  **L139 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_load_sbh(__m128bh __W, __mmask8 __U, const void *__A) {`。
- **L140 EN**: Continues logic associated with callable symbol `__builtin_shufflevector`.
  **L140 CN**: 继续与可调用符号 `__builtin_shufflevector` 相关的逻辑。
- **L141 EN**: Executes a call or declaration centered on `statement`.
  **L141 CN**: 执行以 `statement` 为核心的调用或声明。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L143 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_loadsbf16128_mask((const __v8bf *)__A, src,`.
  **L143 CN**: 以 `(__m128bh)__builtin_ia32_loadsbf16128_mask((const __v8bf *)__A, src,` 从当前函数返回。
- **L144 EN**: Adds a standalone statement or declaration: `__U & 1);`.
  **L144 CN**: 添加一条独立语句或声明：`__U & 1);`。

### Lines 145-168

````c
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_load_sbh(__mmask8 __U, const void *__A) {
  return (__m128bh)__builtin_ia32_loadsbf16128_mask(
      (const __v8bf *)__A, (__v8bf)_mm_setzero_pbh(), __U & 1);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_load_pbh(void const *__p) {
  return *(const __m256bh *)__p;
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_load_pbh(void const *__p) {
  return *(const __m128bh *)__p;
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_loadu_pbh(void const *__p) {
  struct __loadu_pbh {
    __m256bh_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_pbh *)__p)->__v;
}
````
- **L145 EN**: Closes the current lexical scope or compound statement.
  **L145 CN**: 结束当前词法作用域或复合语句块。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L147 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L147 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L148 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_load_sbh(__mmask8 __U, const void *__A) {`.
  **L148 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_load_sbh(__mmask8 __U, const void *__A) {`。
- **L149 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_loadsbf16128_mask(`.
  **L149 CN**: 以 `(__m128bh)__builtin_ia32_loadsbf16128_mask(` 从当前函数返回。
- **L150 EN**: Executes a call or declaration centered on `statement`.
  **L150 CN**: 执行以 `statement` 为核心的调用或声明。
- **L151 EN**: Closes the current lexical scope or compound statement.
  **L151 CN**: 结束当前词法作用域或复合语句块。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L153 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L153 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L154 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_load_pbh(void const *__p) {`.
  **L154 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_load_pbh(void const *__p) {`。
- **L155 EN**: Returns from the current function with `*(const __m256bh *)__p`.
  **L155 CN**: 以 `*(const __m256bh *)__p` 从当前函数返回。
- **L156 EN**: Closes the current lexical scope or compound statement.
  **L156 CN**: 结束当前词法作用域或复合语句块。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L158 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_load_pbh(void const *__p) {`.
  **L158 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_load_pbh(void const *__p) {`。
- **L159 EN**: Returns from the current function with `*(const __m128bh *)__p`.
  **L159 CN**: 以 `*(const __m128bh *)__p` 从当前函数返回。
- **L160 EN**: Closes the current lexical scope or compound statement.
  **L160 CN**: 结束当前词法作用域或复合语句块。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L162 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_loadu_pbh(void const *__p) {`.
  **L163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_loadu_pbh(void const *__p) {`。
- **L164 EN**: Declares struct `__loadu_pbh`.
  **L164 CN**: 声明 struct `__loadu_pbh`。
- **L165 EN**: Adds a standalone statement or declaration: `__m256bh_u __v;`.
  **L165 CN**: 添加一条独立语句或声明：`__m256bh_u __v;`。
- **L166 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L166 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L167 EN**: Returns from the current function with `((const struct __loadu_pbh *)__p)->__v`.
  **L167 CN**: 以 `((const struct __loadu_pbh *)__p)->__v` 从当前函数返回。
- **L168 EN**: Closes the current lexical scope or compound statement.
  **L168 CN**: 结束当前词法作用域或复合语句块。

### Lines 169-192

````c

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_loadu_pbh(void const *__p) {
  struct __loadu_pbh {
    __m128bh_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_pbh *)__p)->__v;
}

static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_sbh(void *__dp,
                                                           __m128bh __a) {
  struct __mm_store_sbh_struct {
    __bf16 __u;
  } __attribute__((__packed__, __may_alias__));
  ((struct __mm_store_sbh_struct *)__dp)->__u = __a[0];
}

static __inline__ void __DEFAULT_FN_ATTRS128 _mm_mask_store_sbh(void *__W,
                                                                __mmask8 __U,
                                                                __m128bh __A) {
  __builtin_ia32_storesbf16128_mask((__v8bf *)__W, __A, __U & 1);
}

static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_store_pbh(void *__P,
````
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L170 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L170 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L171 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_loadu_pbh(void const *__p) {`.
  **L171 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_loadu_pbh(void const *__p) {`。
- **L172 EN**: Declares struct `__loadu_pbh`.
  **L172 CN**: 声明 struct `__loadu_pbh`。
- **L173 EN**: Adds a standalone statement or declaration: `__m128bh_u __v;`.
  **L173 CN**: 添加一条独立语句或声明：`__m128bh_u __v;`。
- **L174 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L174 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L175 EN**: Returns from the current function with `((const struct __loadu_pbh *)__p)->__v`.
  **L175 CN**: 以 `((const struct __loadu_pbh *)__p)->__v` 从当前函数返回。
- **L176 EN**: Closes the current lexical scope or compound statement.
  **L176 CN**: 结束当前词法作用域或复合语句块。
- **L177 EN**: Blank line separating nearby declarations or logic blocks.
  **L177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_sbh(void *__dp,`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_sbh(void *__dp,`。
- **L179 EN**: Continues the surrounding expression or declaration: `__m128bh __a) {`.
  **L179 CN**: 继续构造周围的表达式或声明：`__m128bh __a) {`。
- **L180 EN**: Declares struct `__mm_store_sbh_struct`.
  **L180 CN**: 声明 struct `__mm_store_sbh_struct`。
- **L181 EN**: Adds a standalone statement or declaration: `__bf16 __u;`.
  **L181 CN**: 添加一条独立语句或声明：`__bf16 __u;`。
- **L182 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L182 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L183 EN**: Executes a call or declaration centered on `statement`.
  **L183 CN**: 执行以 `statement` 为核心的调用或声明。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS128 _mm_mask_store_sbh(void *__W,`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS128 _mm_mask_store_sbh(void *__W,`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__mmask8 __U,`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`__mmask8 __U,`。
- **L188 EN**: Continues the surrounding expression or declaration: `__m128bh __A) {`.
  **L188 CN**: 继续构造周围的表达式或声明：`__m128bh __A) {`。
- **L189 EN**: Executes a call or declaration centered on `__builtin_ia32_storesbf16128_mask`.
  **L189 CN**: 执行以 `__builtin_ia32_storesbf16128_mask` 为核心的调用或声明。
- **L190 EN**: Closes the current lexical scope or compound statement.
  **L190 CN**: 结束当前词法作用域或复合语句块。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_store_pbh(void *__P,`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_store_pbh(void *__P,`。

### Lines 193-216

````c
                                                              __m256bh __A) {
  *(__m256bh *)__P = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_pbh(void *__P,
                                                           __m128bh __A) {
  *(__m128bh *)__P = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_storeu_pbh(void *__P,
                                                               __m256bh __A) {
  struct __storeu_pbh {
    __m256bh_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_pbh *)__P)->__v = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS128 _mm_storeu_pbh(void *__P,
                                                            __m128bh __A) {
  struct __storeu_pbh {
    __m128bh_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_pbh *)__P)->__v = __A;
}
````
- **L193 EN**: Continues the surrounding expression or declaration: `__m256bh __A) {`.
  **L193 CN**: 继续构造周围的表达式或声明：`__m256bh __A) {`。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `(__m256bh *)__P __A;`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__m256bh *)__P __A;`。
- **L195 EN**: Closes the current lexical scope or compound statement.
  **L195 CN**: 结束当前词法作用域或复合语句块。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_pbh(void *__P,`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS128 _mm_store_pbh(void *__P,`。
- **L198 EN**: Continues the surrounding expression or declaration: `__m128bh __A) {`.
  **L198 CN**: 继续构造周围的表达式或声明：`__m128bh __A) {`。
- **L199 EN**: Comment explains nearby logic, constraints, or intent: `(__m128bh *)__P __A;`.
  **L199 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(__m128bh *)__P __A;`。
- **L200 EN**: Closes the current lexical scope or compound statement.
  **L200 CN**: 结束当前词法作用域或复合语句块。
- **L201 EN**: Blank line separating nearby declarations or logic blocks.
  **L201 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_storeu_pbh(void *__P,`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS256 _mm256_storeu_pbh(void *__P,`。
- **L203 EN**: Continues the surrounding expression or declaration: `__m256bh __A) {`.
  **L203 CN**: 继续构造周围的表达式或声明：`__m256bh __A) {`。
- **L204 EN**: Declares struct `__storeu_pbh`.
  **L204 CN**: 声明 struct `__storeu_pbh`。
- **L205 EN**: Adds a standalone statement or declaration: `__m256bh_u __v;`.
  **L205 CN**: 添加一条独立语句或声明：`__m256bh_u __v;`。
- **L206 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L206 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L207 EN**: Executes a call or declaration centered on `statement`.
  **L207 CN**: 执行以 `statement` 为核心的调用或声明。
- **L208 EN**: Closes the current lexical scope or compound statement.
  **L208 CN**: 结束当前词法作用域或复合语句块。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS128 _mm_storeu_pbh(void *__P,`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS128 _mm_storeu_pbh(void *__P,`。
- **L211 EN**: Continues the surrounding expression or declaration: `__m128bh __A) {`.
  **L211 CN**: 继续构造周围的表达式或声明：`__m128bh __A) {`。
- **L212 EN**: Declares struct `__storeu_pbh`.
  **L212 CN**: 声明 struct `__storeu_pbh`。
- **L213 EN**: Adds a standalone statement or declaration: `__m128bh_u __v;`.
  **L213 CN**: 添加一条独立语句或声明：`__m128bh_u __v;`。
- **L214 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L214 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L215 EN**: Executes a call or declaration centered on `statement`.
  **L215 CN**: 执行以 `statement` 为核心的调用或声明。
- **L216 EN**: Closes the current lexical scope or compound statement.
  **L216 CN**: 结束当前词法作用域或复合语句块。

### Lines 217-240

````c

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_move_sbh(__m128bh __a,
                                                              __m128bh __b) {
  __a[0] = __b[0];
  return __a;
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_move_sbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {
  return __builtin_ia32_selectsbf_128(__U, _mm_move_sbh(__A, __B), __W);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_move_sbh(__mmask8 __U, __m128bh __A, __m128bh __B) {
  return __builtin_ia32_selectsbf_128(__U, _mm_move_sbh(__A, __B),
                                      _mm_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_undefined_pbh(void) {
  return (__m128bh)__builtin_ia32_undef128();
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_set_sbh(__bf16 bf) {
  return (__v8bf)__builtin_shufflevector(
````
- **L217 EN**: Blank line separating nearby declarations or logic blocks.
  **L217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_move_sbh(__m128bh __a,`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_move_sbh(__m128bh __a,`。
- **L219 EN**: Continues the surrounding expression or declaration: `__m128bh __b) {`.
  **L219 CN**: 继续构造周围的表达式或声明：`__m128bh __b) {`。
- **L220 EN**: Adds a standalone statement or declaration: `__a[0] = __b[0];`.
  **L220 CN**: 添加一条独立语句或声明：`__a[0] = __b[0];`。
- **L221 EN**: Returns from the current function with `__a`.
  **L221 CN**: 以 `__a` 从当前函数返回。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L224 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L224 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L225 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_move_sbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L225 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_move_sbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L226 EN**: Returns from the current function with `__builtin_ia32_selectsbf_128(__U, _mm_move_sbh(__A, __B), __W)`.
  **L226 CN**: 以 `__builtin_ia32_selectsbf_128(__U, _mm_move_sbh(__A, __B), __W)` 从当前函数返回。
- **L227 EN**: Closes the current lexical scope or compound statement.
  **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L229 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L229 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L230 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_move_sbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L230 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_move_sbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L231 EN**: Returns from the current function with `__builtin_ia32_selectsbf_128(__U, _mm_move_sbh(__A, __B),`.
  **L231 CN**: 以 `__builtin_ia32_selectsbf_128(__U, _mm_move_sbh(__A, __B),` 从当前函数返回。
- **L232 EN**: Executes a call or declaration centered on `_mm_setzero_pbh`.
  **L232 CN**: 执行以 `_mm_setzero_pbh` 为核心的调用或声明。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L235 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_undefined_pbh(void) {`.
  **L235 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_undefined_pbh(void) {`。
- **L236 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_undef128()`.
  **L236 CN**: 以 `(__m128bh)__builtin_ia32_undef128()` 从当前函数返回。
- **L237 EN**: Closes the current lexical scope or compound statement.
  **L237 CN**: 结束当前词法作用域或复合语句块。
- **L238 EN**: Blank line separating nearby declarations or logic blocks.
  **L238 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L239 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_set_sbh(__bf16 bf) {`.
  **L239 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_set_sbh(__bf16 bf) {`。
- **L240 EN**: Returns from the current function with `(__v8bf)__builtin_shufflevector(`.
  **L240 CN**: 以 `(__v8bf)__builtin_shufflevector(` 从当前函数返回。

### Lines 241-264

````c
      (__v8bf){bf, bf, bf, bf, bf, bf, bf, bf}, (__v8bf)_mm_setzero_pbh(), 0, 8,
      8, 8, 8, 8, 8, 8);
}

static __inline __m128bh __DEFAULT_FN_ATTRS128 _mm_set1_pbh(__bf16 bf) {
  return (__m128bh)(__v8bf){bf, bf, bf, bf, bf, bf, bf, bf};
}

static __inline __m256bh __DEFAULT_FN_ATTRS256 _mm256_set1_pbh(__bf16 bf) {
  return (__m256bh)(__v16bf){bf, bf, bf, bf, bf, bf, bf, bf,
                             bf, bf, bf, bf, bf, bf, bf, bf};
}

static __inline __m128bh __DEFAULT_FN_ATTRS128
_mm_set_pbh(__bf16 bf1, __bf16 bf2, __bf16 bf3, __bf16 bf4, __bf16 bf5,
            __bf16 bf6, __bf16 bf7, __bf16 bf8) {
  return (__m128bh)(__v8bf){bf1, bf2, bf3, bf4, bf5, bf6, bf7, bf8};
}

static __inline __m256bh __DEFAULT_FN_ATTRS256 _mm256_set_pbh(
    __bf16 bf1, __bf16 bf2, __bf16 bf3, __bf16 bf4, __bf16 bf5, __bf16 bf6,
    __bf16 bf7, __bf16 bf8, __bf16 bf9, __bf16 bf10, __bf16 bf11, __bf16 bf12,
    __bf16 bf13, __bf16 bf14, __bf16 bf15, __bf16 bf16) {
  return (__m256bh)(__v16bf){bf1, bf2,  bf3,  bf4,  bf5,  bf6,  bf7,  bf8,
````
- **L241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8bf){bf, bf, bf, bf, bf, bf, bf, bf}, (__v8bf)_mm_setzero_pbh(), 0, 8,`.
  **L241 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8bf){bf, bf, bf, bf, bf, bf, bf, bf}, (__v8bf)_mm_setzero_pbh(), 0, 8,`。
- **L242 EN**: Adds a standalone statement or declaration: `8, 8, 8, 8, 8, 8);`.
  **L242 CN**: 添加一条独立语句或声明：`8, 8, 8, 8, 8, 8);`。
- **L243 EN**: Closes the current lexical scope or compound statement.
  **L243 CN**: 结束当前词法作用域或复合语句块。
- **L244 EN**: Blank line separating nearby declarations or logic blocks.
  **L244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L245 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline __m128bh __DEFAULT_FN_ATTRS128 _mm_set1_pbh(__bf16 bf) {`.
  **L245 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline __m128bh __DEFAULT_FN_ATTRS128 _mm_set1_pbh(__bf16 bf) {`。
- **L246 EN**: Returns from the current function with `(__m128bh)(__v8bf){bf, bf, bf, bf, bf, bf, bf, bf}`.
  **L246 CN**: 以 `(__m128bh)(__v8bf){bf, bf, bf, bf, bf, bf, bf, bf}` 从当前函数返回。
- **L247 EN**: Closes the current lexical scope or compound statement.
  **L247 CN**: 结束当前词法作用域或复合语句块。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L249 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline __m256bh __DEFAULT_FN_ATTRS256 _mm256_set1_pbh(__bf16 bf) {`.
  **L249 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline __m256bh __DEFAULT_FN_ATTRS256 _mm256_set1_pbh(__bf16 bf) {`。
- **L250 EN**: Returns from the current function with `(__m256bh)(__v16bf){bf, bf, bf, bf, bf, bf, bf, bf,`.
  **L250 CN**: 以 `(__m256bh)(__v16bf){bf, bf, bf, bf, bf, bf, bf, bf,` 从当前函数返回。
- **L251 EN**: Adds a standalone statement or declaration: `bf, bf, bf, bf, bf, bf, bf, bf};`.
  **L251 CN**: 添加一条独立语句或声明：`bf, bf, bf, bf, bf, bf, bf, bf};`。
- **L252 EN**: Closes the current lexical scope or compound statement.
  **L252 CN**: 结束当前词法作用域或复合语句块。
- **L253 EN**: Blank line separating nearby declarations or logic blocks.
  **L253 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L254 EN**: Continues the surrounding expression or declaration: `static __inline __m128bh __DEFAULT_FN_ATTRS128`.
  **L254 CN**: 继续构造周围的表达式或声明：`static __inline __m128bh __DEFAULT_FN_ATTRS128`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_set_pbh(__bf16 bf1, __bf16 bf2, __bf16 bf3, __bf16 bf4, __bf16 bf5,`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_set_pbh(__bf16 bf1, __bf16 bf2, __bf16 bf3, __bf16 bf4, __bf16 bf5,`。
- **L256 EN**: Continues the surrounding expression or declaration: `__bf16 bf6, __bf16 bf7, __bf16 bf8) {`.
  **L256 CN**: 继续构造周围的表达式或声明：`__bf16 bf6, __bf16 bf7, __bf16 bf8) {`。
- **L257 EN**: Returns from the current function with `(__m128bh)(__v8bf){bf1, bf2, bf3, bf4, bf5, bf6, bf7, bf8}`.
  **L257 CN**: 以 `(__m128bh)(__v8bf){bf1, bf2, bf3, bf4, bf5, bf6, bf7, bf8}` 从当前函数返回。
- **L258 EN**: Closes the current lexical scope or compound statement.
  **L258 CN**: 结束当前词法作用域或复合语句块。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L260 EN**: Continues logic associated with callable symbol `_mm256_set_pbh`.
  **L260 CN**: 继续与可调用符号 `_mm256_set_pbh` 相关的逻辑。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__bf16 bf1, __bf16 bf2, __bf16 bf3, __bf16 bf4, __bf16 bf5, __bf16 bf6,`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`__bf16 bf1, __bf16 bf2, __bf16 bf3, __bf16 bf4, __bf16 bf5, __bf16 bf6,`。
- **L262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__bf16 bf7, __bf16 bf8, __bf16 bf9, __bf16 bf10, __bf16 bf11, __bf16 bf12,`.
  **L262 CN**: 继续一个多行参数列表、初始化器或聚合项：`__bf16 bf7, __bf16 bf8, __bf16 bf9, __bf16 bf10, __bf16 bf11, __bf16 bf12,`。
- **L263 EN**: Continues the surrounding expression or declaration: `__bf16 bf13, __bf16 bf14, __bf16 bf15, __bf16 bf16) {`.
  **L263 CN**: 继续构造周围的表达式或声明：`__bf16 bf13, __bf16 bf14, __bf16 bf15, __bf16 bf16) {`。
- **L264 EN**: Returns from the current function with `(__m256bh)(__v16bf){bf1, bf2,  bf3,  bf4,  bf5,  bf6,  bf7,  bf8,`.
  **L264 CN**: 以 `(__m256bh)(__v16bf){bf1, bf2,  bf3,  bf4,  bf5,  bf6,  bf7,  bf8,` 从当前函数返回。

### Lines 265-288

````c
                             bf9, bf10, bf11, bf12, bf13, bf14, bf15, bf16};
}

#define _mm_setr_pbh(bf1, bf2, bf3, bf4, bf5, bf6, bf7, bf8)                   \
  _mm_set_pbh((bf8), (bf7), (bf6), (bf5), (bf4), (bf3), (bf2), (bf1))

#define _mm256_setr_pbh(bf1, bf2, bf3, bf4, bf5, bf6, bf7, bf8, bf9, bf10,     \
                        bf11, bf12, bf13, bf14, bf15, bf16)                    \
  _mm256_set_pbh((bf16), (bf15), (bf14), (bf13), (bf12), (bf11), (bf10),       \
                 (bf9), (bf8), (bf7), (bf6), (bf5), (bf4), (bf3), (bf2),       \
                 (bf1))

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_abs_pbh(__m256bh __A) {
  return (__m256bh)_mm256_and_epi32(_mm256_set1_epi32(0x7FFF7FFF),
                                    (__m256i)__A);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_abs_pbh(__m128bh __A) {
  return (__m128bh)_mm_and_epi32(_mm_set1_epi32(0x7FFF7FFF), (__m128i)__A);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_blend_pbh(__mmask8 __U, __m128bh __A, __m128bh __W) {
  return (__m128bh)__builtin_ia32_selectpbf_128((__mmask8)__U, (__v8bf)__W,
````
- **L265 EN**: Adds a standalone statement or declaration: `bf9, bf10, bf11, bf12, bf13, bf14, bf15, bf16};`.
  **L265 CN**: 添加一条独立语句或声明：`bf9, bf10, bf11, bf12, bf13, bf14, bf15, bf16};`。
- **L266 EN**: Closes the current lexical scope or compound statement.
  **L266 CN**: 结束当前词法作用域或复合语句块。
- **L267 EN**: Blank line separating nearby declarations or logic blocks.
  **L267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L268 EN**: Defines macro `_mm_setr_pbh(bf1, bf2, bf3, bf4, bf5, bf6, bf7, bf8)` for conditional compilation, shorthand, or API generation.
  **L268 CN**: 定义宏 `_mm_setr_pbh(bf1, bf2, bf3, bf4, bf5, bf6, bf7, bf8)`，用于条件编译、简写或 API 生成。
- **L269 EN**: Continues logic associated with callable symbol `_mm_set_pbh`.
  **L269 CN**: 继续与可调用符号 `_mm_set_pbh` 相关的逻辑。
- **L270 EN**: Blank line separating nearby declarations or logic blocks.
  **L270 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L271 EN**: Defines macro `_mm256_setr_pbh` for conditional compilation, shorthand, or API generation.
  **L271 CN**: 定义宏 `_mm256_setr_pbh`，用于条件编译、简写或 API 生成。
- **L272 EN**: Continues the surrounding expression or declaration: `bf11, bf12, bf13, bf14, bf15, bf16)                    \`.
  **L272 CN**: 继续构造周围的表达式或声明：`bf11, bf12, bf13, bf14, bf15, bf16)                    \`。
- **L273 EN**: Continues logic associated with callable symbol `_mm256_set_pbh`.
  **L273 CN**: 继续与可调用符号 `_mm256_set_pbh` 相关的逻辑。
- **L274 EN**: Continues the surrounding expression or declaration: `(bf9), (bf8), (bf7), (bf6), (bf5), (bf4), (bf3), (bf2),       \`.
  **L274 CN**: 继续构造周围的表达式或声明：`(bf9), (bf8), (bf7), (bf6), (bf5), (bf4), (bf3), (bf2),       \`。
- **L275 EN**: Continues the surrounding expression or declaration: `(bf1))`.
  **L275 CN**: 继续构造周围的表达式或声明：`(bf1))`。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L277 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_abs_pbh(__m256bh __A) {`.
  **L277 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_abs_pbh(__m256bh __A) {`。
- **L278 EN**: Returns from the current function with `(__m256bh)_mm256_and_epi32(_mm256_set1_epi32(0x7FFF7FFF),`.
  **L278 CN**: 以 `(__m256bh)_mm256_and_epi32(_mm256_set1_epi32(0x7FFF7FFF),` 从当前函数返回。
- **L279 EN**: Executes a call or declaration centered on `statement`.
  **L279 CN**: 执行以 `statement` 为核心的调用或声明。
- **L280 EN**: Closes the current lexical scope or compound statement.
  **L280 CN**: 结束当前词法作用域或复合语句块。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L282 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_abs_pbh(__m128bh __A) {`.
  **L282 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_abs_pbh(__m128bh __A) {`。
- **L283 EN**: Returns from the current function with `(__m128bh)_mm_and_epi32(_mm_set1_epi32(0x7FFF7FFF), (__m128i)__A)`.
  **L283 CN**: 以 `(__m128bh)_mm_and_epi32(_mm_set1_epi32(0x7FFF7FFF), (__m128i)__A)` 从当前函数返回。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L286 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L286 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L287 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_blend_pbh(__mmask8 __U, __m128bh __A, __m128bh __W) {`.
  **L287 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_blend_pbh(__mmask8 __U, __m128bh __A, __m128bh __W) {`。
- **L288 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128((__mmask8)__U, (__v8bf)__W,`.
  **L288 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128((__mmask8)__U, (__v8bf)__W,` 从当前函数返回。

### Lines 289-312

````c
                                                (__v8bf)__A);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_blend_pbh(__mmask16 __U, __m256bh __A, __m256bh __W) {
  return (__m256bh)__builtin_ia32_selectpbf_256((__mmask16)__U, (__v16bf)__W,
                                                (__v16bf)__A);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_permutex2var_pbh(__m128bh __A, __m128i __I, __m128bh __B) {
  return (__m128bh)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,
                                                  (__v8hi)__B);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_permutex2var_pbh(__m256bh __A, __m256i __I, __m256bh __B) {
  return (__m256bh)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,
                                                  (__v16hi)__B);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_permutexvar_pbh(__m128i __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_permvarhi128((__v8hi)__B, (__v8hi)__A);
````
- **L289 EN**: Executes a call or declaration centered on `statement`.
  **L289 CN**: 执行以 `statement` 为核心的调用或声明。
- **L290 EN**: Closes the current lexical scope or compound statement.
  **L290 CN**: 结束当前词法作用域或复合语句块。
- **L291 EN**: Blank line separating nearby declarations or logic blocks.
  **L291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L292 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L292 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L293 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_blend_pbh(__mmask16 __U, __m256bh __A, __m256bh __W) {`.
  **L293 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_blend_pbh(__mmask16 __U, __m256bh __A, __m256bh __W) {`。
- **L294 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256((__mmask16)__U, (__v16bf)__W,`.
  **L294 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256((__mmask16)__U, (__v16bf)__W,` 从当前函数返回。
- **L295 EN**: Executes a call or declaration centered on `statement`.
  **L295 CN**: 执行以 `statement` 为核心的调用或声明。
- **L296 EN**: Closes the current lexical scope or compound statement.
  **L296 CN**: 结束当前词法作用域或复合语句块。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L298 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L298 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L299 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_permutex2var_pbh(__m128bh __A, __m128i __I, __m128bh __B) {`.
  **L299 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_permutex2var_pbh(__m128bh __A, __m128i __I, __m128bh __B) {`。
- **L300 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,`.
  **L300 CN**: 以 `(__m128bh)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,` 从当前函数返回。
- **L301 EN**: Executes a call or declaration centered on `statement`.
  **L301 CN**: 执行以 `statement` 为核心的调用或声明。
- **L302 EN**: Closes the current lexical scope or compound statement.
  **L302 CN**: 结束当前词法作用域或复合语句块。
- **L303 EN**: Blank line separating nearby declarations or logic blocks.
  **L303 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L304 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L304 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L305 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_permutex2var_pbh(__m256bh __A, __m256i __I, __m256bh __B) {`.
  **L305 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_permutex2var_pbh(__m256bh __A, __m256i __I, __m256bh __B) {`。
- **L306 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,`.
  **L306 CN**: 以 `(__m256bh)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,` 从当前函数返回。
- **L307 EN**: Executes a call or declaration centered on `statement`.
  **L307 CN**: 执行以 `statement` 为核心的调用或声明。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L310 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L311 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_permutexvar_pbh(__m128i __A, __m128bh __B) {`.
  **L311 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_permutexvar_pbh(__m128i __A, __m128bh __B) {`。
- **L312 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_permvarhi128((__v8hi)__B, (__v8hi)__A)`.
  **L312 CN**: 以 `(__m128bh)__builtin_ia32_permvarhi128((__v8hi)__B, (__v8hi)__A)` 从当前函数返回。

### Lines 313-336

````c
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_permutexvar_pbh(__m256i __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_permvarhi256((__v16hi)__B, (__v16hi)__A);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_add_pbh(__m256bh __A,
                                                                __m256bh __B) {
  return (__m256bh)((__v16bf)__A + (__v16bf)__B);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_add_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_add_pbh(__A, __B), (__v16bf)__W);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_add_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_add_pbh(__A, __B),
      (__v16bf)_mm256_setzero_pbh());
}
````
- **L313 EN**: Closes the current lexical scope or compound statement.
  **L313 CN**: 结束当前词法作用域或复合语句块。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L315 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L316 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_permutexvar_pbh(__m256i __A, __m256bh __B) {`.
  **L316 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_permutexvar_pbh(__m256i __A, __m256bh __B) {`。
- **L317 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_permvarhi256((__v16hi)__B, (__v16hi)__A)`.
  **L317 CN**: 以 `(__m256bh)__builtin_ia32_permvarhi256((__v16hi)__B, (__v16hi)__A)` 从当前函数返回。
- **L318 EN**: Closes the current lexical scope or compound statement.
  **L318 CN**: 结束当前词法作用域或复合语句块。
- **L319 EN**: Blank line separating nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_add_pbh(__m256bh __A,`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_add_pbh(__m256bh __A,`。
- **L321 EN**: Continues the surrounding expression or declaration: `__m256bh __B) {`.
  **L321 CN**: 继续构造周围的表达式或声明：`__m256bh __B) {`。
- **L322 EN**: Returns from the current function with `(__m256bh)((__v16bf)__A + (__v16bf)__B)`.
  **L322 CN**: 以 `(__m256bh)((__v16bf)__A + (__v16bf)__B)` 从当前函数返回。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L325 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L325 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L326 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_add_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L326 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_add_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L327 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L327 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L328 EN**: Executes a call or declaration centered on `statement`.
  **L328 CN**: 执行以 `statement` 为核心的调用或声明。
- **L329 EN**: Closes the current lexical scope or compound statement.
  **L329 CN**: 结束当前词法作用域或复合语句块。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L331 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L331 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L332 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_add_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L332 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_add_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L333 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L333 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16bf)_mm256_add_pbh(__A, __B),`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16bf)_mm256_add_pbh(__A, __B),`。
- **L335 EN**: Executes a call or declaration centered on `statement`.
  **L335 CN**: 执行以 `statement` 为核心的调用或声明。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。

### Lines 337-360

````c

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_add_pbh(__m128bh __A,
                                                             __m128bh __B) {
  return (__m128bh)((__v8bf)__A + (__v8bf)__B);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_add_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_add_pbh(__A, __B), (__v8bf)__W);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_add_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_add_pbh(__A, __B), (__v8bf)_mm_setzero_pbh());
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_sub_pbh(__m256bh __A,
                                                                __m256bh __B) {
  return (__m256bh)((__v16bf)__A - (__v16bf)__B);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
````
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_add_pbh(__m128bh __A,`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_add_pbh(__m128bh __A,`。
- **L339 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L339 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L340 EN**: Returns from the current function with `(__m128bh)((__v8bf)__A + (__v8bf)__B)`.
  **L340 CN**: 以 `(__m128bh)((__v8bf)__A + (__v8bf)__B)` 从当前函数返回。
- **L341 EN**: Closes the current lexical scope or compound statement.
  **L341 CN**: 结束当前词法作用域或复合语句块。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L343 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L343 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L344 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_add_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L344 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_add_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L345 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L345 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L346 EN**: Executes a call or declaration centered on `statement`.
  **L346 CN**: 执行以 `statement` 为核心的调用或声明。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L349 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L350 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_add_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L350 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_add_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L351 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L351 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L352 EN**: Executes a call or declaration centered on `statement`.
  **L352 CN**: 执行以 `statement` 为核心的调用或声明。
- **L353 EN**: Closes the current lexical scope or compound statement.
  **L353 CN**: 结束当前词法作用域或复合语句块。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_sub_pbh(__m256bh __A,`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_sub_pbh(__m256bh __A,`。
- **L356 EN**: Continues the surrounding expression or declaration: `__m256bh __B) {`.
  **L356 CN**: 继续构造周围的表达式或声明：`__m256bh __B) {`。
- **L357 EN**: Returns from the current function with `(__m256bh)((__v16bf)__A - (__v16bf)__B)`.
  **L357 CN**: 以 `(__m256bh)((__v16bf)__A - (__v16bf)__B)` 从当前函数返回。
- **L358 EN**: Closes the current lexical scope or compound statement.
  **L358 CN**: 结束当前词法作用域或复合语句块。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L360 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L360 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。

### Lines 361-384

````c
_mm256_mask_sub_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_sub_pbh(__A, __B), (__v16bf)__W);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_sub_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_sub_pbh(__A, __B),
      (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_sub_pbh(__m128bh __A,
                                                             __m128bh __B) {
  return (__m128bh)((__v8bf)__A - (__v8bf)__B);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_sub_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_sub_pbh(__A, __B), (__v8bf)__W);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
````
- **L361 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_sub_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L361 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_sub_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L362 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L362 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L363 EN**: Executes a call or declaration centered on `statement`.
  **L363 CN**: 执行以 `statement` 为核心的调用或声明。
- **L364 EN**: Closes the current lexical scope or compound statement.
  **L364 CN**: 结束当前词法作用域或复合语句块。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L366 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L366 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L367 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_sub_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L367 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_sub_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L368 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L368 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16bf)_mm256_sub_pbh(__A, __B),`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16bf)_mm256_sub_pbh(__A, __B),`。
- **L370 EN**: Executes a call or declaration centered on `statement`.
  **L370 CN**: 执行以 `statement` 为核心的调用或声明。
- **L371 EN**: Closes the current lexical scope or compound statement.
  **L371 CN**: 结束当前词法作用域或复合语句块。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_sub_pbh(__m128bh __A,`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_sub_pbh(__m128bh __A,`。
- **L374 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L374 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L375 EN**: Returns from the current function with `(__m128bh)((__v8bf)__A - (__v8bf)__B)`.
  **L375 CN**: 以 `(__m128bh)((__v8bf)__A - (__v8bf)__B)` 从当前函数返回。
- **L376 EN**: Closes the current lexical scope or compound statement.
  **L376 CN**: 结束当前词法作用域或复合语句块。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L378 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L378 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_sub_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_sub_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L380 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L380 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L381 EN**: Executes a call or declaration centered on `statement`.
  **L381 CN**: 执行以 `statement` 为核心的调用或声明。
- **L382 EN**: Closes the current lexical scope or compound statement.
  **L382 CN**: 结束当前词法作用域或复合语句块。
- **L383 EN**: Blank line separating nearby declarations or logic blocks.
  **L383 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L384 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L384 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。

### Lines 385-408

````c
_mm_maskz_sub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_sub_pbh(__A, __B), (__v8bf)_mm_setzero_pbh());
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mul_pbh(__m256bh __A,
                                                                __m256bh __B) {
  return (__m256bh)((__v16bf)__A * (__v16bf)__B);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_mul_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_mul_pbh(__A, __B), (__v16bf)__W);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_mul_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_mul_pbh(__A, __B),
      (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_mul_pbh(__m128bh __A,
````
- **L385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_sub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_sub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L386 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L386 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L387 EN**: Executes a call or declaration centered on `statement`.
  **L387 CN**: 执行以 `statement` 为核心的调用或声明。
- **L388 EN**: Closes the current lexical scope or compound statement.
  **L388 CN**: 结束当前词法作用域或复合语句块。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mul_pbh(__m256bh __A,`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mul_pbh(__m256bh __A,`。
- **L391 EN**: Continues the surrounding expression or declaration: `__m256bh __B) {`.
  **L391 CN**: 继续构造周围的表达式或声明：`__m256bh __B) {`。
- **L392 EN**: Returns from the current function with `(__m256bh)((__v16bf)__A * (__v16bf)__B)`.
  **L392 CN**: 以 `(__m256bh)((__v16bf)__A * (__v16bf)__B)` 从当前函数返回。
- **L393 EN**: Closes the current lexical scope or compound statement.
  **L393 CN**: 结束当前词法作用域或复合语句块。
- **L394 EN**: Blank line separating nearby declarations or logic blocks.
  **L394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L395 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L395 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L396 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mul_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L396 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mul_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L397 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L397 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L398 EN**: Executes a call or declaration centered on `statement`.
  **L398 CN**: 执行以 `statement` 为核心的调用或声明。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L401 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L401 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L402 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mul_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L402 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mul_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L403 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L403 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16bf)_mm256_mul_pbh(__A, __B),`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16bf)_mm256_mul_pbh(__A, __B),`。
- **L405 EN**: Executes a call or declaration centered on `statement`.
  **L405 CN**: 执行以 `statement` 为核心的调用或声明。
- **L406 EN**: Closes the current lexical scope or compound statement.
  **L406 CN**: 结束当前词法作用域或复合语句块。
- **L407 EN**: Blank line separating nearby declarations or logic blocks.
  **L407 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_mul_pbh(__m128bh __A,`.
  **L408 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_mul_pbh(__m128bh __A,`。

### Lines 409-432

````c
                                                             __m128bh __B) {
  return (__m128bh)((__v8bf)__A * (__v8bf)__B);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_mul_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_mul_pbh(__A, __B), (__v8bf)__W);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_mul_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_mul_pbh(__A, __B), (__v8bf)_mm_setzero_pbh());
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_div_pbh(__m256bh __A,
                                                                __m256bh __B) {
  return (__m256bh)((__v16bf)__A / (__v16bf)__B);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_div_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
````
- **L409 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L409 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L410 EN**: Returns from the current function with `(__m128bh)((__v8bf)__A * (__v8bf)__B)`.
  **L410 CN**: 以 `(__m128bh)((__v8bf)__A * (__v8bf)__B)` 从当前函数返回。
- **L411 EN**: Closes the current lexical scope or compound statement.
  **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L413 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L413 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L414 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mul_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L414 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mul_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L415 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L415 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L416 EN**: Executes a call or declaration centered on `statement`.
  **L416 CN**: 执行以 `statement` 为核心的调用或声明。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L419 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L419 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L420 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mul_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L420 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mul_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L421 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L421 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L422 EN**: Executes a call or declaration centered on `statement`.
  **L422 CN**: 执行以 `statement` 为核心的调用或声明。
- **L423 EN**: Closes the current lexical scope or compound statement.
  **L423 CN**: 结束当前词法作用域或复合语句块。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_div_pbh(__m256bh __A,`.
  **L425 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_div_pbh(__m256bh __A,`。
- **L426 EN**: Continues the surrounding expression or declaration: `__m256bh __B) {`.
  **L426 CN**: 继续构造周围的表达式或声明：`__m256bh __B) {`。
- **L427 EN**: Returns from the current function with `(__m256bh)((__v16bf)__A / (__v16bf)__B)`.
  **L427 CN**: 以 `(__m256bh)((__v16bf)__A / (__v16bf)__B)` 从当前函数返回。
- **L428 EN**: Closes the current lexical scope or compound statement.
  **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L430 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L430 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L431 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_div_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L431 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_div_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L432 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L432 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。

### Lines 433-456

````c
      (__mmask16)__U, (__v16bf)_mm256_div_pbh(__A, __B), (__v16bf)__W);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_div_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_div_pbh(__A, __B),
      (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_div_pbh(__m128bh __A,
                                                             __m128bh __B) {
  return (__m128bh)((__v8bf)__A / (__v8bf)__B);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_div_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_div_pbh(__A, __B), (__v8bf)__W);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_div_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
````
- **L433 EN**: Executes a call or declaration centered on `statement`.
  **L433 CN**: 执行以 `statement` 为核心的调用或声明。
- **L434 EN**: Closes the current lexical scope or compound statement.
  **L434 CN**: 结束当前词法作用域或复合语句块。
- **L435 EN**: Blank line separating nearby declarations or logic blocks.
  **L435 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L436 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L436 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L437 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_div_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L437 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_div_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L438 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L438 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16bf)_mm256_div_pbh(__A, __B),`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16bf)_mm256_div_pbh(__A, __B),`。
- **L440 EN**: Executes a call or declaration centered on `statement`.
  **L440 CN**: 执行以 `statement` 为核心的调用或声明。
- **L441 EN**: Closes the current lexical scope or compound statement.
  **L441 CN**: 结束当前词法作用域或复合语句块。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_div_pbh(__m128bh __A,`.
  **L443 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_div_pbh(__m128bh __A,`。
- **L444 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L444 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L445 EN**: Returns from the current function with `(__m128bh)((__v8bf)__A / (__v8bf)__B)`.
  **L445 CN**: 以 `(__m128bh)((__v8bf)__A / (__v8bf)__B)` 从当前函数返回。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L448 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L449 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_div_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L449 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_div_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L450 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L450 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L451 EN**: Executes a call or declaration centered on `statement`.
  **L451 CN**: 执行以 `statement` 为核心的调用或声明。
- **L452 EN**: Closes the current lexical scope or compound statement.
  **L452 CN**: 结束当前词法作用域或复合语句块。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L454 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L454 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L455 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_div_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L455 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_div_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L456 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L456 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。

### Lines 457-480

````c
      (__mmask8)__U, (__v8bf)_mm_div_pbh(__A, __B), (__v8bf)_mm_setzero_pbh());
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_max_pbh(__m256bh __A,
                                                                __m256bh __B) {
  return (__m256bh)__builtin_ia32_vmaxbf16256((__v16bf)__A, (__v16bf)__B);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_max_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_max_pbh(__A, __B), (__v16bf)__W);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_max_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_max_pbh(__A, __B),
      (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_max_pbh(__m128bh __A,
                                                             __m128bh __B) {
  return (__m128bh)__builtin_ia32_vmaxbf16128((__v8bf)__A, (__v8bf)__B);
````
- **L457 EN**: Executes a call or declaration centered on `statement`.
  **L457 CN**: 执行以 `statement` 为核心的调用或声明。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Blank line separating nearby declarations or logic blocks.
  **L459 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_max_pbh(__m256bh __A,`.
  **L460 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_max_pbh(__m256bh __A,`。
- **L461 EN**: Continues the surrounding expression or declaration: `__m256bh __B) {`.
  **L461 CN**: 继续构造周围的表达式或声明：`__m256bh __B) {`。
- **L462 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vmaxbf16256((__v16bf)__A, (__v16bf)__B)`.
  **L462 CN**: 以 `(__m256bh)__builtin_ia32_vmaxbf16256((__v16bf)__A, (__v16bf)__B)` 从当前函数返回。
- **L463 EN**: Closes the current lexical scope or compound statement.
  **L463 CN**: 结束当前词法作用域或复合语句块。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L465 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L465 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L466 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_max_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L466 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_max_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L467 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L467 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L468 EN**: Executes a call or declaration centered on `statement`.
  **L468 CN**: 执行以 `statement` 为核心的调用或声明。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L471 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L472 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_max_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L472 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_max_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L473 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L473 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16bf)_mm256_max_pbh(__A, __B),`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16bf)_mm256_max_pbh(__A, __B),`。
- **L475 EN**: Executes a call or declaration centered on `statement`.
  **L475 CN**: 执行以 `statement` 为核心的调用或声明。
- **L476 EN**: Closes the current lexical scope or compound statement.
  **L476 CN**: 结束当前词法作用域或复合语句块。
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_max_pbh(__m128bh __A,`.
  **L478 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_max_pbh(__m128bh __A,`。
- **L479 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L479 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L480 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vmaxbf16128((__v8bf)__A, (__v8bf)__B)`.
  **L480 CN**: 以 `(__m128bh)__builtin_ia32_vmaxbf16128((__v8bf)__A, (__v8bf)__B)` 从当前函数返回。

### Lines 481-504

````c
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_max_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_max_pbh(__A, __B), (__v8bf)__W);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_max_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_max_pbh(__A, __B), (__v8bf)_mm_setzero_pbh());
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_min_pbh(__m256bh __A,
                                                                __m256bh __B) {
  return (__m256bh)__builtin_ia32_vminbf16256((__v16bf)__A, (__v16bf)__B);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_min_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_min_pbh(__A, __B), (__v16bf)__W);
}
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L483 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L483 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L484 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_max_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L484 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_max_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L485 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L485 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L486 EN**: Executes a call or declaration centered on `statement`.
  **L486 CN**: 执行以 `statement` 为核心的调用或声明。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L489 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L489 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L490 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_max_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L490 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_max_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L491 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L491 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L492 EN**: Executes a call or declaration centered on `statement`.
  **L492 CN**: 执行以 `statement` 为核心的调用或声明。
- **L493 EN**: Closes the current lexical scope or compound statement.
  **L493 CN**: 结束当前词法作用域或复合语句块。
- **L494 EN**: Blank line separating nearby declarations or logic blocks.
  **L494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_min_pbh(__m256bh __A,`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_min_pbh(__m256bh __A,`。
- **L496 EN**: Continues the surrounding expression or declaration: `__m256bh __B) {`.
  **L496 CN**: 继续构造周围的表达式或声明：`__m256bh __B) {`。
- **L497 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vminbf16256((__v16bf)__A, (__v16bf)__B)`.
  **L497 CN**: 以 `(__m256bh)__builtin_ia32_vminbf16256((__v16bf)__A, (__v16bf)__B)` 从当前函数返回。
- **L498 EN**: Closes the current lexical scope or compound statement.
  **L498 CN**: 结束当前词法作用域或复合语句块。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L500 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L500 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L501 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_min_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L501 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_min_pbh(__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L502 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L502 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L503 EN**: Executes a call or declaration centered on `statement`.
  **L503 CN**: 执行以 `statement` 为核心的调用或声明。
- **L504 EN**: Closes the current lexical scope or compound statement.
  **L504 CN**: 结束当前词法作用域或复合语句块。

### Lines 505-528

````c

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_min_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_min_pbh(__A, __B),
      (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_min_pbh(__m128bh __A,
                                                             __m128bh __B) {
  return (__m128bh)__builtin_ia32_vminbf16128((__v8bf)__A, (__v8bf)__B);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_min_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_min_pbh(__A, __B), (__v8bf)__W);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_min_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_min_pbh(__A, __B), (__v8bf)_mm_setzero_pbh());
}
````
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L506 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L506 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L507 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_min_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L507 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_min_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L508 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L508 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16bf)_mm256_min_pbh(__A, __B),`.
  **L509 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16bf)_mm256_min_pbh(__A, __B),`。
- **L510 EN**: Executes a call or declaration centered on `statement`.
  **L510 CN**: 执行以 `statement` 为核心的调用或声明。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_min_pbh(__m128bh __A,`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_min_pbh(__m128bh __A,`。
- **L514 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L514 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L515 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vminbf16128((__v8bf)__A, (__v8bf)__B)`.
  **L515 CN**: 以 `(__m128bh)__builtin_ia32_vminbf16128((__v8bf)__A, (__v8bf)__B)` 从当前函数返回。
- **L516 EN**: Closes the current lexical scope or compound statement.
  **L516 CN**: 结束当前词法作用域或复合语句块。
- **L517 EN**: Blank line separating nearby declarations or logic blocks.
  **L517 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L518 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L518 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L519 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_min_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L519 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_min_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L520 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L520 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L521 EN**: Executes a call or declaration centered on `statement`.
  **L521 CN**: 执行以 `statement` 为核心的调用或声明。
- **L522 EN**: Closes the current lexical scope or compound statement.
  **L522 CN**: 结束当前词法作用域或复合语句块。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L524 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L524 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L525 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_min_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L525 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_min_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L526 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L526 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L527 EN**: Executes a call or declaration centered on `statement`.
  **L527 CN**: 执行以 `statement` 为核心的调用或声明。
- **L528 EN**: Closes the current lexical scope or compound statement.
  **L528 CN**: 结束当前词法作用域或复合语句块。

### Lines 529-552

````c

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comieq_sbh(__m128bh __A,
                                                           __m128bh __B) {
  return __builtin_ia32_vcomisbf16eq((__v8bf)__A, (__v8bf)__B);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comilt_sbh(__m128bh __A,
                                                           __m128bh __B) {
  return __builtin_ia32_vcomisbf16lt((__v8bf)__A, (__v8bf)__B);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comile_sbh(__m128bh __A,
                                                           __m128bh __B) {
  return __builtin_ia32_vcomisbf16le((__v8bf)__A, (__v8bf)__B);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comigt_sbh(__m128bh __A,
                                                           __m128bh __B) {
  return __builtin_ia32_vcomisbf16gt((__v8bf)__A, (__v8bf)__B);
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comige_sbh(__m128bh __A,
                                                           __m128bh __B) {
  return __builtin_ia32_vcomisbf16ge((__v8bf)__A, (__v8bf)__B);
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comieq_sbh(__m128bh __A,`.
  **L530 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comieq_sbh(__m128bh __A,`。
- **L531 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L531 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L532 EN**: Returns from the current function with `__builtin_ia32_vcomisbf16eq((__v8bf)__A, (__v8bf)__B)`.
  **L532 CN**: 以 `__builtin_ia32_vcomisbf16eq((__v8bf)__A, (__v8bf)__B)` 从当前函数返回。
- **L533 EN**: Closes the current lexical scope or compound statement.
  **L533 CN**: 结束当前词法作用域或复合语句块。
- **L534 EN**: Blank line separating nearby declarations or logic blocks.
  **L534 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comilt_sbh(__m128bh __A,`.
  **L535 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comilt_sbh(__m128bh __A,`。
- **L536 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L536 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L537 EN**: Returns from the current function with `__builtin_ia32_vcomisbf16lt((__v8bf)__A, (__v8bf)__B)`.
  **L537 CN**: 以 `__builtin_ia32_vcomisbf16lt((__v8bf)__A, (__v8bf)__B)` 从当前函数返回。
- **L538 EN**: Closes the current lexical scope or compound statement.
  **L538 CN**: 结束当前词法作用域或复合语句块。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comile_sbh(__m128bh __A,`.
  **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comile_sbh(__m128bh __A,`。
- **L541 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L541 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L542 EN**: Returns from the current function with `__builtin_ia32_vcomisbf16le((__v8bf)__A, (__v8bf)__B)`.
  **L542 CN**: 以 `__builtin_ia32_vcomisbf16le((__v8bf)__A, (__v8bf)__B)` 从当前函数返回。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comigt_sbh(__m128bh __A,`.
  **L545 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comigt_sbh(__m128bh __A,`。
- **L546 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L546 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L547 EN**: Returns from the current function with `__builtin_ia32_vcomisbf16gt((__v8bf)__A, (__v8bf)__B)`.
  **L547 CN**: 以 `__builtin_ia32_vcomisbf16gt((__v8bf)__A, (__v8bf)__B)` 从当前函数返回。
- **L548 EN**: Closes the current lexical scope or compound statement.
  **L548 CN**: 结束当前词法作用域或复合语句块。
- **L549 EN**: Blank line separating nearby declarations or logic blocks.
  **L549 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comige_sbh(__m128bh __A,`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comige_sbh(__m128bh __A,`。
- **L551 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L551 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L552 EN**: Returns from the current function with `__builtin_ia32_vcomisbf16ge((__v8bf)__A, (__v8bf)__B)`.
  **L552 CN**: 以 `__builtin_ia32_vcomisbf16ge((__v8bf)__A, (__v8bf)__B)` 从当前函数返回。

### Lines 553-576

````c
}

static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comineq_sbh(__m128bh __A,
                                                            __m128bh __B) {
  return __builtin_ia32_vcomisbf16neq((__v8bf)__A, (__v8bf)__B);
}

#define _mm256_cmp_pbh_mask(__A, __B, __P)                                     \
  ((__mmask16)__builtin_ia32_vcmpbf16256_mask((__v16bf)(__m256bh)(__A),        \
                                              (__v16bf)(__m256bh)(__B),        \
                                              (int)(__P), (__mmask16) - 1))

#define _mm256_mask_cmp_pbh_mask(__U, __A, __B, __P)                           \
  ((__mmask16)__builtin_ia32_vcmpbf16256_mask((__v16bf)(__m256bh)(__A),        \
                                              (__v16bf)(__m256bh)(__B),        \
                                              (int)(__P), (__mmask16)(__U)))

#define _mm_cmp_pbh_mask(__A, __B, __P)                                        \
  ((__mmask8)__builtin_ia32_vcmpbf16128_mask((__v8bf)(__m128bh)(__A),          \
                                             (__v8bf)(__m128bh)(__B),          \
                                             (int)(__P), (__mmask8) - 1))

#define _mm_mask_cmp_pbh_mask(__U, __A, __B, __P)                              \
  ((__mmask8)__builtin_ia32_vcmpbf16128_mask((__v8bf)(__m128bh)(__A),          \
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comineq_sbh(__m128bh __A,`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int __DEFAULT_FN_ATTRS128 _mm_comineq_sbh(__m128bh __A,`。
- **L556 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L556 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L557 EN**: Returns from the current function with `__builtin_ia32_vcomisbf16neq((__v8bf)__A, (__v8bf)__B)`.
  **L557 CN**: 以 `__builtin_ia32_vcomisbf16neq((__v8bf)__A, (__v8bf)__B)` 从当前函数返回。
- **L558 EN**: Closes the current lexical scope or compound statement.
  **L558 CN**: 结束当前词法作用域或复合语句块。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L560 EN**: Defines macro `_mm256_cmp_pbh_mask(__A, __B, __P)` for conditional compilation, shorthand, or API generation.
  **L560 CN**: 定义宏 `_mm256_cmp_pbh_mask(__A, __B, __P)`，用于条件编译、简写或 API 生成。
- **L561 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcmpbf16256_mask`.
  **L561 CN**: 继续与可调用符号 `__builtin_ia32_vcmpbf16256_mask` 相关的逻辑。
- **L562 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__B),        \`.
  **L562 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__B),        \`。
- **L563 EN**: Continues the surrounding expression or declaration: `(int)(__P), (__mmask16) - 1))`.
  **L563 CN**: 继续构造周围的表达式或声明：`(int)(__P), (__mmask16) - 1))`。
- **L564 EN**: Blank line separating nearby declarations or logic blocks.
  **L564 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L565 EN**: Defines macro `_mm256_mask_cmp_pbh_mask(__U, __A, __B, __P)` for conditional compilation, shorthand, or API generation.
  **L565 CN**: 定义宏 `_mm256_mask_cmp_pbh_mask(__U, __A, __B, __P)`，用于条件编译、简写或 API 生成。
- **L566 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcmpbf16256_mask`.
  **L566 CN**: 继续与可调用符号 `__builtin_ia32_vcmpbf16256_mask` 相关的逻辑。
- **L567 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__B),        \`.
  **L567 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__B),        \`。
- **L568 EN**: Continues the surrounding expression or declaration: `(int)(__P), (__mmask16)(__U)))`.
  **L568 CN**: 继续构造周围的表达式或声明：`(int)(__P), (__mmask16)(__U)))`。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L570 EN**: Defines macro `_mm_cmp_pbh_mask(__A, __B, __P)` for conditional compilation, shorthand, or API generation.
  **L570 CN**: 定义宏 `_mm_cmp_pbh_mask(__A, __B, __P)`，用于条件编译、简写或 API 生成。
- **L571 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcmpbf16128_mask`.
  **L571 CN**: 继续与可调用符号 `__builtin_ia32_vcmpbf16128_mask` 相关的逻辑。
- **L572 EN**: Continues the surrounding expression or declaration: `(__v8bf)(__m128bh)(__B),          \`.
  **L572 CN**: 继续构造周围的表达式或声明：`(__v8bf)(__m128bh)(__B),          \`。
- **L573 EN**: Continues the surrounding expression or declaration: `(int)(__P), (__mmask8) - 1))`.
  **L573 CN**: 继续构造周围的表达式或声明：`(int)(__P), (__mmask8) - 1))`。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Defines macro `_mm_mask_cmp_pbh_mask(__U, __A, __B, __P)` for conditional compilation, shorthand, or API generation.
  **L575 CN**: 定义宏 `_mm_mask_cmp_pbh_mask(__U, __A, __B, __P)`，用于条件编译、简写或 API 生成。
- **L576 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcmpbf16128_mask`.
  **L576 CN**: 继续与可调用符号 `__builtin_ia32_vcmpbf16128_mask` 相关的逻辑。

### Lines 577-600

````c
                                             (__v8bf)(__m128bh)(__B),          \
                                             (int)(__P), (__mmask8)(__U)))

#define _mm256_mask_fpclass_pbh_mask(__U, __A, imm)                            \
  ((__mmask16)__builtin_ia32_vfpclassbf16256_mask(                             \
      (__v16bf)(__m256bh)(__A), (int)(imm), (__mmask16)(__U)))

#define _mm256_fpclass_pbh_mask(__A, imm)                                      \
  ((__mmask16)__builtin_ia32_vfpclassbf16256_mask(                             \
      (__v16bf)(__m256bh)(__A), (int)(imm), (__mmask16) - 1))

#define _mm_mask_fpclass_pbh_mask(__U, __A, imm)                               \
  ((__mmask8)__builtin_ia32_vfpclassbf16128_mask((__v8bf)(__m128bh)(__A),      \
                                                 (int)(imm), (__mmask8)(__U)))

#define _mm_fpclass_pbh_mask(__A, imm)                                         \
  ((__mmask8)__builtin_ia32_vfpclassbf16128_mask((__v8bf)(__m128bh)(__A),      \
                                                 (int)(imm), (__mmask8) - 1))

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_scalef_pbh(__m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_vscalefbf16256_mask(
      (__v16bf)__A, (__v16bf)__B, (__v16bf)_mm256_undefined_pbh(),
      (__mmask16)-1);
````
- **L577 EN**: Continues the surrounding expression or declaration: `(__v8bf)(__m128bh)(__B),          \`.
  **L577 CN**: 继续构造周围的表达式或声明：`(__v8bf)(__m128bh)(__B),          \`。
- **L578 EN**: Continues the surrounding expression or declaration: `(int)(__P), (__mmask8)(__U)))`.
  **L578 CN**: 继续构造周围的表达式或声明：`(int)(__P), (__mmask8)(__U)))`。
- **L579 EN**: Blank line separating nearby declarations or logic blocks.
  **L579 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L580 EN**: Defines macro `_mm256_mask_fpclass_pbh_mask(__U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L580 CN**: 定义宏 `_mm256_mask_fpclass_pbh_mask(__U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L581 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfpclassbf16256_mask`.
  **L581 CN**: 继续与可调用符号 `__builtin_ia32_vfpclassbf16256_mask` 相关的逻辑。
- **L582 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__A), (int)(imm), (__mmask16)(__U)))`.
  **L582 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__A), (int)(imm), (__mmask16)(__U)))`。
- **L583 EN**: Blank line separating nearby declarations or logic blocks.
  **L583 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L584 EN**: Defines macro `_mm256_fpclass_pbh_mask(__A, imm)` for conditional compilation, shorthand, or API generation.
  **L584 CN**: 定义宏 `_mm256_fpclass_pbh_mask(__A, imm)`，用于条件编译、简写或 API 生成。
- **L585 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfpclassbf16256_mask`.
  **L585 CN**: 继续与可调用符号 `__builtin_ia32_vfpclassbf16256_mask` 相关的逻辑。
- **L586 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__A), (int)(imm), (__mmask16) - 1))`.
  **L586 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__A), (int)(imm), (__mmask16) - 1))`。
- **L587 EN**: Blank line separating nearby declarations or logic blocks.
  **L587 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L588 EN**: Defines macro `_mm_mask_fpclass_pbh_mask(__U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L588 CN**: 定义宏 `_mm_mask_fpclass_pbh_mask(__U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L589 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfpclassbf16128_mask`.
  **L589 CN**: 继续与可调用符号 `__builtin_ia32_vfpclassbf16128_mask` 相关的逻辑。
- **L590 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask8)(__U)))`.
  **L590 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask8)(__U)))`。
- **L591 EN**: Blank line separating nearby declarations or logic blocks.
  **L591 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L592 EN**: Defines macro `_mm_fpclass_pbh_mask(__A, imm)` for conditional compilation, shorthand, or API generation.
  **L592 CN**: 定义宏 `_mm_fpclass_pbh_mask(__A, imm)`，用于条件编译、简写或 API 生成。
- **L593 EN**: Continues logic associated with callable symbol `__builtin_ia32_vfpclassbf16128_mask`.
  **L593 CN**: 继续与可调用符号 `__builtin_ia32_vfpclassbf16128_mask` 相关的逻辑。
- **L594 EN**: Continues the surrounding expression or declaration: `(int)(imm), (__mmask8) - 1))`.
  **L594 CN**: 继续构造周围的表达式或声明：`(int)(imm), (__mmask8) - 1))`。
- **L595 EN**: Blank line separating nearby declarations or logic blocks.
  **L595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L596 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L596 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L597 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_scalef_pbh(__m256bh __A, __m256bh __B) {`.
  **L597 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_scalef_pbh(__m256bh __A, __m256bh __B) {`。
- **L598 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vscalefbf16256_mask(`.
  **L598 CN**: 以 `(__m256bh)__builtin_ia32_vscalefbf16256_mask(` 从当前函数返回。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16bf)__A, (__v16bf)__B, (__v16bf)_mm256_undefined_pbh(),`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16bf)__A, (__v16bf)__B, (__v16bf)_mm256_undefined_pbh(),`。
- **L600 EN**: Executes a call or declaration centered on `statement`.
  **L600 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 601-624

````c
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mask_scalef_pbh(
    __m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_vscalefbf16256_mask(
      (__v16bf)__A, (__v16bf)__B, (__v16bf)__W, (__mmask16)__U);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_scalef_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {
  return (__m256bh)__builtin_ia32_vscalefbf16256_mask(
      (__v16bf)__A, (__v16bf)__B, (__v16bf)_mm256_setzero_pbh(),
      (__mmask16)__U);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_scalef_pbh(__m128bh __A,
                                                                __m128bh __B) {
  return (__m128bh)__builtin_ia32_vscalefbf16128_mask(
      (__v8bf)__A, (__v8bf)__B, (__v8bf)_mm_undefined_pbh(), (__mmask8)-1);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_scalef_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_vscalefbf16128_mask(
````
- **L601 EN**: Closes the current lexical scope or compound statement.
  **L601 CN**: 结束当前词法作用域或复合语句块。
- **L602 EN**: Blank line separating nearby declarations or logic blocks.
  **L602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L603 EN**: Continues logic associated with callable symbol `_mm256_mask_scalef_pbh`.
  **L603 CN**: 继续与可调用符号 `_mm256_mask_scalef_pbh` 相关的逻辑。
- **L604 EN**: Continues the surrounding expression or declaration: `__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L604 CN**: 继续构造周围的表达式或声明：`__m256bh __W, __mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L605 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vscalefbf16256_mask(`.
  **L605 CN**: 以 `(__m256bh)__builtin_ia32_vscalefbf16256_mask(` 从当前函数返回。
- **L606 EN**: Executes a call or declaration centered on `statement`.
  **L606 CN**: 执行以 `statement` 为核心的调用或声明。
- **L607 EN**: Closes the current lexical scope or compound statement.
  **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L609 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L610 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_scalef_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`.
  **L610 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_scalef_pbh(__mmask16 __U, __m256bh __A, __m256bh __B) {`。
- **L611 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vscalefbf16256_mask(`.
  **L611 CN**: 以 `(__m256bh)__builtin_ia32_vscalefbf16256_mask(` 从当前函数返回。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16bf)__A, (__v16bf)__B, (__v16bf)_mm256_setzero_pbh(),`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16bf)__A, (__v16bf)__B, (__v16bf)_mm256_setzero_pbh(),`。
- **L613 EN**: Executes a call or declaration centered on `statement`.
  **L613 CN**: 执行以 `statement` 为核心的调用或声明。
- **L614 EN**: Closes the current lexical scope or compound statement.
  **L614 CN**: 结束当前词法作用域或复合语句块。
- **L615 EN**: Blank line separating nearby declarations or logic blocks.
  **L615 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_scalef_pbh(__m128bh __A,`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_scalef_pbh(__m128bh __A,`。
- **L617 EN**: Continues the surrounding expression or declaration: `__m128bh __B) {`.
  **L617 CN**: 继续构造周围的表达式或声明：`__m128bh __B) {`。
- **L618 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vscalefbf16128_mask(`.
  **L618 CN**: 以 `(__m128bh)__builtin_ia32_vscalefbf16128_mask(` 从当前函数返回。
- **L619 EN**: Executes a call or declaration centered on `statement`.
  **L619 CN**: 执行以 `statement` 为核心的调用或声明。
- **L620 EN**: Closes the current lexical scope or compound statement.
  **L620 CN**: 结束当前词法作用域或复合语句块。
- **L621 EN**: Blank line separating nearby declarations or logic blocks.
  **L621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L622 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L622 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L623 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_scalef_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L623 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_scalef_pbh(__m128bh __W, __mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L624 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vscalefbf16128_mask(`.
  **L624 CN**: 以 `(__m128bh)__builtin_ia32_vscalefbf16128_mask(` 从当前函数返回。

### Lines 625-648

````c
      (__v8bf)__A, (__v8bf)__B, (__v8bf)__W, (__mmask8)__U);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_scalef_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {
  return (__m128bh)__builtin_ia32_vscalefbf16128_mask(
      (__v8bf)__A, (__v8bf)__B, (__v8bf)_mm_setzero_pbh(), (__mmask8)__U);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_rcp_pbh(__m256bh __A) {
  return (__m256bh)__builtin_ia32_vrcpbf16256_mask(
      (__v16bf)__A, (__v16bf)_mm256_undefined_pbh(), (__mmask16)-1);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_rcp_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {
  return (__m256bh)__builtin_ia32_vrcpbf16256_mask((__v16bf)__A, (__v16bf)__W,
                                                   (__mmask16)__U);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_rcp_pbh(__mmask16 __U, __m256bh __A) {
  return (__m256bh)__builtin_ia32_vrcpbf16256_mask(
      (__v16bf)__A, (__v16bf)_mm256_setzero_pbh(), (__mmask16)__U);
````
- **L625 EN**: Executes a call or declaration centered on `statement`.
  **L625 CN**: 执行以 `statement` 为核心的调用或声明。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Blank line separating nearby declarations or logic blocks.
  **L627 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L628 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L628 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L629 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_scalef_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`.
  **L629 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_scalef_pbh(__mmask8 __U, __m128bh __A, __m128bh __B) {`。
- **L630 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vscalefbf16128_mask(`.
  **L630 CN**: 以 `(__m128bh)__builtin_ia32_vscalefbf16128_mask(` 从当前函数返回。
- **L631 EN**: Executes a call or declaration centered on `statement`.
  **L631 CN**: 执行以 `statement` 为核心的调用或声明。
- **L632 EN**: Closes the current lexical scope or compound statement.
  **L632 CN**: 结束当前词法作用域或复合语句块。
- **L633 EN**: Blank line separating nearby declarations or logic blocks.
  **L633 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L634 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_rcp_pbh(__m256bh __A) {`.
  **L634 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_rcp_pbh(__m256bh __A) {`。
- **L635 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vrcpbf16256_mask(`.
  **L635 CN**: 以 `(__m256bh)__builtin_ia32_vrcpbf16256_mask(` 从当前函数返回。
- **L636 EN**: Executes a call or declaration centered on `statement`.
  **L636 CN**: 执行以 `statement` 为核心的调用或声明。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L639 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L639 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L640 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_rcp_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {`.
  **L640 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_rcp_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {`。
- **L641 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vrcpbf16256_mask((__v16bf)__A, (__v16bf)__W,`.
  **L641 CN**: 以 `(__m256bh)__builtin_ia32_vrcpbf16256_mask((__v16bf)__A, (__v16bf)__W,` 从当前函数返回。
- **L642 EN**: Executes a call or declaration centered on `statement`.
  **L642 CN**: 执行以 `statement` 为核心的调用或声明。
- **L643 EN**: Closes the current lexical scope or compound statement.
  **L643 CN**: 结束当前词法作用域或复合语句块。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L645 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L645 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L646 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_rcp_pbh(__mmask16 __U, __m256bh __A) {`.
  **L646 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_rcp_pbh(__mmask16 __U, __m256bh __A) {`。
- **L647 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vrcpbf16256_mask(`.
  **L647 CN**: 以 `(__m256bh)__builtin_ia32_vrcpbf16256_mask(` 从当前函数返回。
- **L648 EN**: Executes a call or declaration centered on `statement`.
  **L648 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 649-672

````c
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_rcp_pbh(__m128bh __A) {
  return (__m128bh)__builtin_ia32_vrcpbf16128_mask(
      (__v8bf)__A, (__v8bf)_mm_undefined_pbh(), (__mmask8)-1);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_rcp_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {
  return (__m128bh)__builtin_ia32_vrcpbf16128_mask((__v8bf)__A, (__v8bf)__W,
                                                   (__mmask8)__U);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_rcp_pbh(__mmask8 __U, __m128bh __A) {
  return (__m128bh)__builtin_ia32_vrcpbf16128_mask(
      (__v8bf)__A, (__v8bf)_mm_setzero_pbh(), (__mmask8)__U);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_getexp_pbh(__m256bh __A) {
  return (__m256bh)__builtin_ia32_vgetexpbf16256_mask(
      (__v16bf)__A, (__v16bf)_mm256_undefined_pbh(), (__mmask16)-1);
}
````
- **L649 EN**: Closes the current lexical scope or compound statement.
  **L649 CN**: 结束当前词法作用域或复合语句块。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L651 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_rcp_pbh(__m128bh __A) {`.
  **L651 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_rcp_pbh(__m128bh __A) {`。
- **L652 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vrcpbf16128_mask(`.
  **L652 CN**: 以 `(__m128bh)__builtin_ia32_vrcpbf16128_mask(` 从当前函数返回。
- **L653 EN**: Executes a call or declaration centered on `statement`.
  **L653 CN**: 执行以 `statement` 为核心的调用或声明。
- **L654 EN**: Closes the current lexical scope or compound statement.
  **L654 CN**: 结束当前词法作用域或复合语句块。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L656 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L656 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L657 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_rcp_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {`.
  **L657 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_rcp_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {`。
- **L658 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vrcpbf16128_mask((__v8bf)__A, (__v8bf)__W,`.
  **L658 CN**: 以 `(__m128bh)__builtin_ia32_vrcpbf16128_mask((__v8bf)__A, (__v8bf)__W,` 从当前函数返回。
- **L659 EN**: Executes a call or declaration centered on `statement`.
  **L659 CN**: 执行以 `statement` 为核心的调用或声明。
- **L660 EN**: Closes the current lexical scope or compound statement.
  **L660 CN**: 结束当前词法作用域或复合语句块。
- **L661 EN**: Blank line separating nearby declarations or logic blocks.
  **L661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L662 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L662 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L663 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_rcp_pbh(__mmask8 __U, __m128bh __A) {`.
  **L663 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_rcp_pbh(__mmask8 __U, __m128bh __A) {`。
- **L664 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vrcpbf16128_mask(`.
  **L664 CN**: 以 `(__m128bh)__builtin_ia32_vrcpbf16128_mask(` 从当前函数返回。
- **L665 EN**: Executes a call or declaration centered on `statement`.
  **L665 CN**: 执行以 `statement` 为核心的调用或声明。
- **L666 EN**: Closes the current lexical scope or compound statement.
  **L666 CN**: 结束当前词法作用域或复合语句块。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L668 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L668 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L669 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_getexp_pbh(__m256bh __A) {`.
  **L669 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_getexp_pbh(__m256bh __A) {`。
- **L670 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vgetexpbf16256_mask(`.
  **L670 CN**: 以 `(__m256bh)__builtin_ia32_vgetexpbf16256_mask(` 从当前函数返回。
- **L671 EN**: Executes a call or declaration centered on `statement`.
  **L671 CN**: 执行以 `statement` 为核心的调用或声明。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````c

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_getexp_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {
  return (__m256bh)__builtin_ia32_vgetexpbf16256_mask(
      (__v16bf)__A, (__v16bf)__W, (__mmask16)__U);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_getexp_pbh(__mmask16 __U, __m256bh __A) {
  return (__m256bh)__builtin_ia32_vgetexpbf16256_mask(
      (__v16bf)__A, (__v16bf)_mm256_setzero_pbh(), (__mmask16)__U);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_getexp_pbh(__m128bh __A) {
  return (__m128bh)__builtin_ia32_vgetexpbf16128_mask(
      (__v8bf)__A, (__v8bf)_mm_undefined_pbh(), (__mmask8)-1);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_getexp_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {
  return (__m128bh)__builtin_ia32_vgetexpbf16128_mask((__v8bf)__A, (__v8bf)__W,
                                                      (__mmask8)__U);
}

````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L674 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L675 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_getexp_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {`.
  **L675 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_getexp_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {`。
- **L676 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vgetexpbf16256_mask(`.
  **L676 CN**: 以 `(__m256bh)__builtin_ia32_vgetexpbf16256_mask(` 从当前函数返回。
- **L677 EN**: Executes a call or declaration centered on `statement`.
  **L677 CN**: 执行以 `statement` 为核心的调用或声明。
- **L678 EN**: Closes the current lexical scope or compound statement.
  **L678 CN**: 结束当前词法作用域或复合语句块。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L680 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L680 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L681 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_getexp_pbh(__mmask16 __U, __m256bh __A) {`.
  **L681 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_getexp_pbh(__mmask16 __U, __m256bh __A) {`。
- **L682 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vgetexpbf16256_mask(`.
  **L682 CN**: 以 `(__m256bh)__builtin_ia32_vgetexpbf16256_mask(` 从当前函数返回。
- **L683 EN**: Executes a call or declaration centered on `statement`.
  **L683 CN**: 执行以 `statement` 为核心的调用或声明。
- **L684 EN**: Closes the current lexical scope or compound statement.
  **L684 CN**: 结束当前词法作用域或复合语句块。
- **L685 EN**: Blank line separating nearby declarations or logic blocks.
  **L685 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L686 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_getexp_pbh(__m128bh __A) {`.
  **L686 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_getexp_pbh(__m128bh __A) {`。
- **L687 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vgetexpbf16128_mask(`.
  **L687 CN**: 以 `(__m128bh)__builtin_ia32_vgetexpbf16128_mask(` 从当前函数返回。
- **L688 EN**: Executes a call or declaration centered on `statement`.
  **L688 CN**: 执行以 `statement` 为核心的调用或声明。
- **L689 EN**: Closes the current lexical scope or compound statement.
  **L689 CN**: 结束当前词法作用域或复合语句块。
- **L690 EN**: Blank line separating nearby declarations or logic blocks.
  **L690 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L691 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L691 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L692 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_getexp_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {`.
  **L692 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_getexp_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {`。
- **L693 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vgetexpbf16128_mask((__v8bf)__A, (__v8bf)__W,`.
  **L693 CN**: 以 `(__m128bh)__builtin_ia32_vgetexpbf16128_mask((__v8bf)__A, (__v8bf)__W,` 从当前函数返回。
- **L694 EN**: Executes a call or declaration centered on `statement`.
  **L694 CN**: 执行以 `statement` 为核心的调用或声明。
- **L695 EN**: Closes the current lexical scope or compound statement.
  **L695 CN**: 结束当前词法作用域或复合语句块。
- **L696 EN**: Blank line separating nearby declarations or logic blocks.
  **L696 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 697-720

````c
static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_getexp_pbh(__mmask8 __U, __m128bh __A) {
  return (__m128bh)__builtin_ia32_vgetexpbf16128_mask(
      (__v8bf)__A, (__v8bf)_mm_setzero_pbh(), (__mmask8)__U);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_rsqrt_pbh(__m256bh __A) {
  return (__m256bh)__builtin_ia32_vrsqrtbf16256_mask(
      (__v16bf)__A, (__v16bf)_mm256_undefined_pbh(), (__mmask16)-1);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_rsqrt_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {
  return (__m256bh)__builtin_ia32_vrsqrtbf16256_mask((__v16bf)__A, (__v16bf)__W,
                                                     (__mmask16)__U);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_rsqrt_pbh(__mmask16 __U, __m256bh __A) {
  return (__m256bh)__builtin_ia32_vrsqrtbf16256_mask(
      (__v16bf)__A, (__v16bf)_mm256_setzero_pbh(), (__mmask16)__U);
}

````
- **L697 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L697 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L698 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_getexp_pbh(__mmask8 __U, __m128bh __A) {`.
  **L698 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_getexp_pbh(__mmask8 __U, __m128bh __A) {`。
- **L699 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vgetexpbf16128_mask(`.
  **L699 CN**: 以 `(__m128bh)__builtin_ia32_vgetexpbf16128_mask(` 从当前函数返回。
- **L700 EN**: Executes a call or declaration centered on `statement`.
  **L700 CN**: 执行以 `statement` 为核心的调用或声明。
- **L701 EN**: Closes the current lexical scope or compound statement.
  **L701 CN**: 结束当前词法作用域或复合语句块。
- **L702 EN**: Blank line separating nearby declarations or logic blocks.
  **L702 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L703 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L703 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L704 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_rsqrt_pbh(__m256bh __A) {`.
  **L704 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_rsqrt_pbh(__m256bh __A) {`。
- **L705 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vrsqrtbf16256_mask(`.
  **L705 CN**: 以 `(__m256bh)__builtin_ia32_vrsqrtbf16256_mask(` 从当前函数返回。
- **L706 EN**: Executes a call or declaration centered on `statement`.
  **L706 CN**: 执行以 `statement` 为核心的调用或声明。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L709 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L709 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L710 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_rsqrt_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {`.
  **L710 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_rsqrt_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {`。
- **L711 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vrsqrtbf16256_mask((__v16bf)__A, (__v16bf)__W,`.
  **L711 CN**: 以 `(__m256bh)__builtin_ia32_vrsqrtbf16256_mask((__v16bf)__A, (__v16bf)__W,` 从当前函数返回。
- **L712 EN**: Executes a call or declaration centered on `statement`.
  **L712 CN**: 执行以 `statement` 为核心的调用或声明。
- **L713 EN**: Closes the current lexical scope or compound statement.
  **L713 CN**: 结束当前词法作用域或复合语句块。
- **L714 EN**: Blank line separating nearby declarations or logic blocks.
  **L714 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L715 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L715 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L716 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_rsqrt_pbh(__mmask16 __U, __m256bh __A) {`.
  **L716 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_rsqrt_pbh(__mmask16 __U, __m256bh __A) {`。
- **L717 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_vrsqrtbf16256_mask(`.
  **L717 CN**: 以 `(__m256bh)__builtin_ia32_vrsqrtbf16256_mask(` 从当前函数返回。
- **L718 EN**: Executes a call or declaration centered on `statement`.
  **L718 CN**: 执行以 `statement` 为核心的调用或声明。
- **L719 EN**: Closes the current lexical scope or compound statement.
  **L719 CN**: 结束当前词法作用域或复合语句块。
- **L720 EN**: Blank line separating nearby declarations or logic blocks.
  **L720 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 721-744

````c
static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_rsqrt_pbh(__m128bh __A) {
  return (__m128bh)__builtin_ia32_vrsqrtbf16128_mask(
      (__v8bf)__A, (__v8bf)_mm_undefined_pbh(), (__mmask8)-1);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_rsqrt_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {
  return (__m128bh)__builtin_ia32_vrsqrtbf16128_mask((__v8bf)__A, (__v8bf)__W,
                                                     (__mmask8)__U);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_rsqrt_pbh(__mmask8 __U, __m128bh __A) {
  return (__m128bh)__builtin_ia32_vrsqrtbf16128_mask(
      (__v8bf)__A, (__v8bf)_mm_setzero_pbh(), (__mmask8)__U);
}

#define _mm256_reduce_pbh(__A, imm)                                            \
  ((__m256bh)__builtin_ia32_vreducebf16256_mask(                               \
      (__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)_mm256_undefined_pbh(),   \
      (__mmask16) - 1))

#define _mm256_mask_reduce_pbh(__W, __U, __A, imm)                             \
  ((__m256bh)__builtin_ia32_vreducebf16256_mask(                               \
````
- **L721 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_rsqrt_pbh(__m128bh __A) {`.
  **L721 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_rsqrt_pbh(__m128bh __A) {`。
- **L722 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vrsqrtbf16128_mask(`.
  **L722 CN**: 以 `(__m128bh)__builtin_ia32_vrsqrtbf16128_mask(` 从当前函数返回。
- **L723 EN**: Executes a call or declaration centered on `statement`.
  **L723 CN**: 执行以 `statement` 为核心的调用或声明。
- **L724 EN**: Closes the current lexical scope or compound statement.
  **L724 CN**: 结束当前词法作用域或复合语句块。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L726 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L726 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L727 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_rsqrt_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {`.
  **L727 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_rsqrt_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {`。
- **L728 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vrsqrtbf16128_mask((__v8bf)__A, (__v8bf)__W,`.
  **L728 CN**: 以 `(__m128bh)__builtin_ia32_vrsqrtbf16128_mask((__v8bf)__A, (__v8bf)__W,` 从当前函数返回。
- **L729 EN**: Executes a call or declaration centered on `statement`.
  **L729 CN**: 执行以 `statement` 为核心的调用或声明。
- **L730 EN**: Closes the current lexical scope or compound statement.
  **L730 CN**: 结束当前词法作用域或复合语句块。
- **L731 EN**: Blank line separating nearby declarations or logic blocks.
  **L731 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L732 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L732 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L733 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_rsqrt_pbh(__mmask8 __U, __m128bh __A) {`.
  **L733 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_rsqrt_pbh(__mmask8 __U, __m128bh __A) {`。
- **L734 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_vrsqrtbf16128_mask(`.
  **L734 CN**: 以 `(__m128bh)__builtin_ia32_vrsqrtbf16128_mask(` 从当前函数返回。
- **L735 EN**: Executes a call or declaration centered on `statement`.
  **L735 CN**: 执行以 `statement` 为核心的调用或声明。
- **L736 EN**: Closes the current lexical scope or compound statement.
  **L736 CN**: 结束当前词法作用域或复合语句块。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L738 EN**: Defines macro `_mm256_reduce_pbh(__A, imm)` for conditional compilation, shorthand, or API generation.
  **L738 CN**: 定义宏 `_mm256_reduce_pbh(__A, imm)`，用于条件编译、简写或 API 生成。
- **L739 EN**: Continues logic associated with callable symbol `__builtin_ia32_vreducebf16256_mask`.
  **L739 CN**: 继续与可调用符号 `__builtin_ia32_vreducebf16256_mask` 相关的逻辑。
- **L740 EN**: Continues logic associated with callable symbol `_mm256_undefined_pbh`.
  **L740 CN**: 继续与可调用符号 `_mm256_undefined_pbh` 相关的逻辑。
- **L741 EN**: Continues the surrounding expression or declaration: `(__mmask16) - 1))`.
  **L741 CN**: 继续构造周围的表达式或声明：`(__mmask16) - 1))`。
- **L742 EN**: Blank line separating nearby declarations or logic blocks.
  **L742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L743 EN**: Defines macro `_mm256_mask_reduce_pbh(__W, __U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L743 CN**: 定义宏 `_mm256_mask_reduce_pbh(__W, __U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L744 EN**: Continues logic associated with callable symbol `__builtin_ia32_vreducebf16256_mask`.
  **L744 CN**: 继续与可调用符号 `__builtin_ia32_vreducebf16256_mask` 相关的逻辑。

### Lines 745-768

````c
      (__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)(__m256bh)(__W),          \
      (__mmask16)(__U)))

#define _mm256_maskz_reduce_pbh(__U, __A, imm)                                 \
  ((__m256bh)__builtin_ia32_vreducebf16256_mask(                               \
      (__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)_mm256_setzero_pbh(),     \
      (__mmask16)(__U)))

#define _mm_reduce_pbh(__A, imm)                                               \
  ((__m128bh)__builtin_ia32_vreducebf16128_mask(                               \
      (__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)_mm_undefined_pbh(),        \
      (__mmask8) - 1))

#define _mm_mask_reduce_pbh(__W, __U, __A, imm)                                \
  ((__m128bh)__builtin_ia32_vreducebf16128_mask(                               \
      (__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)(__m128bh)(__W),            \
      (__mmask8)(__U)))

#define _mm_maskz_reduce_pbh(__U, __A, imm)                                    \
  ((__m128bh)__builtin_ia32_vreducebf16128_mask(                               \
      (__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)_mm_setzero_pbh(),          \
      (__mmask8)(__U)))

#define _mm256_roundscale_pbh(__A, imm)                                        \
````
- **L745 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)(__m256bh)(__W),          \`.
  **L745 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)(__m256bh)(__W),          \`。
- **L746 EN**: Continues the surrounding expression or declaration: `(__mmask16)(__U)))`.
  **L746 CN**: 继续构造周围的表达式或声明：`(__mmask16)(__U)))`。
- **L747 EN**: Blank line separating nearby declarations or logic blocks.
  **L747 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L748 EN**: Defines macro `_mm256_maskz_reduce_pbh(__U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L748 CN**: 定义宏 `_mm256_maskz_reduce_pbh(__U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L749 EN**: Continues logic associated with callable symbol `__builtin_ia32_vreducebf16256_mask`.
  **L749 CN**: 继续与可调用符号 `__builtin_ia32_vreducebf16256_mask` 相关的逻辑。
- **L750 EN**: Continues logic associated with callable symbol `_mm256_setzero_pbh`.
  **L750 CN**: 继续与可调用符号 `_mm256_setzero_pbh` 相关的逻辑。
- **L751 EN**: Continues the surrounding expression or declaration: `(__mmask16)(__U)))`.
  **L751 CN**: 继续构造周围的表达式或声明：`(__mmask16)(__U)))`。
- **L752 EN**: Blank line separating nearby declarations or logic blocks.
  **L752 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L753 EN**: Defines macro `_mm_reduce_pbh(__A, imm)` for conditional compilation, shorthand, or API generation.
  **L753 CN**: 定义宏 `_mm_reduce_pbh(__A, imm)`，用于条件编译、简写或 API 生成。
- **L754 EN**: Continues logic associated with callable symbol `__builtin_ia32_vreducebf16128_mask`.
  **L754 CN**: 继续与可调用符号 `__builtin_ia32_vreducebf16128_mask` 相关的逻辑。
- **L755 EN**: Continues logic associated with callable symbol `_mm_undefined_pbh`.
  **L755 CN**: 继续与可调用符号 `_mm_undefined_pbh` 相关的逻辑。
- **L756 EN**: Continues the surrounding expression or declaration: `(__mmask8) - 1))`.
  **L756 CN**: 继续构造周围的表达式或声明：`(__mmask8) - 1))`。
- **L757 EN**: Blank line separating nearby declarations or logic blocks.
  **L757 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L758 EN**: Defines macro `_mm_mask_reduce_pbh(__W, __U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L758 CN**: 定义宏 `_mm_mask_reduce_pbh(__W, __U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L759 EN**: Continues logic associated with callable symbol `__builtin_ia32_vreducebf16128_mask`.
  **L759 CN**: 继续与可调用符号 `__builtin_ia32_vreducebf16128_mask` 相关的逻辑。
- **L760 EN**: Continues the surrounding expression or declaration: `(__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)(__m128bh)(__W),            \`.
  **L760 CN**: 继续构造周围的表达式或声明：`(__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)(__m128bh)(__W),            \`。
- **L761 EN**: Continues the surrounding expression or declaration: `(__mmask8)(__U)))`.
  **L761 CN**: 继续构造周围的表达式或声明：`(__mmask8)(__U)))`。
- **L762 EN**: Blank line separating nearby declarations or logic blocks.
  **L762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L763 EN**: Defines macro `_mm_maskz_reduce_pbh(__U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L763 CN**: 定义宏 `_mm_maskz_reduce_pbh(__U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L764 EN**: Continues logic associated with callable symbol `__builtin_ia32_vreducebf16128_mask`.
  **L764 CN**: 继续与可调用符号 `__builtin_ia32_vreducebf16128_mask` 相关的逻辑。
- **L765 EN**: Continues logic associated with callable symbol `_mm_setzero_pbh`.
  **L765 CN**: 继续与可调用符号 `_mm_setzero_pbh` 相关的逻辑。
- **L766 EN**: Continues the surrounding expression or declaration: `(__mmask8)(__U)))`.
  **L766 CN**: 继续构造周围的表达式或声明：`(__mmask8)(__U)))`。
- **L767 EN**: Blank line separating nearby declarations or logic blocks.
  **L767 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L768 EN**: Defines macro `_mm256_roundscale_pbh(__A, imm)` for conditional compilation, shorthand, or API generation.
  **L768 CN**: 定义宏 `_mm256_roundscale_pbh(__A, imm)`，用于条件编译、简写或 API 生成。

### Lines 769-792

````c
  ((__m256bh)__builtin_ia32_vrndscalebf16_256_mask(                            \
      (__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)_mm256_setzero_pbh(),     \
      (__mmask16) - 1))

#define _mm256_mask_roundscale_pbh(__W, __U, __A, imm)                         \
  ((__m256bh)__builtin_ia32_vrndscalebf16_256_mask(                            \
      (__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)(__m256bh)(__W),          \
      (__mmask16)(__U)))

#define _mm256_maskz_roundscale_pbh(__U, __A, imm)                             \
  ((__m256bh)__builtin_ia32_vrndscalebf16_256_mask(                            \
      (__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)_mm256_setzero_pbh(),     \
      (__mmask16)(__U)))

#define _mm_roundscale_pbh(__A, imm)                                           \
  ((__m128bh)__builtin_ia32_vrndscalebf16_128_mask(                            \
      (__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)_mm_setzero_pbh(),          \
      (__mmask8) - 1))

#define _mm_mask_roundscale_pbh(__W, __U, __A, imm)                            \
  ((__m128bh)__builtin_ia32_vrndscalebf16_128_mask(                            \
      (__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)(__m128bh)(__W),            \
      (__mmask8)(__U)))

````
- **L769 EN**: Continues logic associated with callable symbol `__builtin_ia32_vrndscalebf16_256_mask`.
  **L769 CN**: 继续与可调用符号 `__builtin_ia32_vrndscalebf16_256_mask` 相关的逻辑。
- **L770 EN**: Continues logic associated with callable symbol `_mm256_setzero_pbh`.
  **L770 CN**: 继续与可调用符号 `_mm256_setzero_pbh` 相关的逻辑。
- **L771 EN**: Continues the surrounding expression or declaration: `(__mmask16) - 1))`.
  **L771 CN**: 继续构造周围的表达式或声明：`(__mmask16) - 1))`。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L773 EN**: Defines macro `_mm256_mask_roundscale_pbh(__W, __U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L773 CN**: 定义宏 `_mm256_mask_roundscale_pbh(__W, __U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L774 EN**: Continues logic associated with callable symbol `__builtin_ia32_vrndscalebf16_256_mask`.
  **L774 CN**: 继续与可调用符号 `__builtin_ia32_vrndscalebf16_256_mask` 相关的逻辑。
- **L775 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)(__m256bh)(__W),          \`.
  **L775 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__A), (int)(imm), (__v16bf)(__m256bh)(__W),          \`。
- **L776 EN**: Continues the surrounding expression or declaration: `(__mmask16)(__U)))`.
  **L776 CN**: 继续构造周围的表达式或声明：`(__mmask16)(__U)))`。
- **L777 EN**: Blank line separating nearby declarations or logic blocks.
  **L777 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L778 EN**: Defines macro `_mm256_maskz_roundscale_pbh(__U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L778 CN**: 定义宏 `_mm256_maskz_roundscale_pbh(__U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L779 EN**: Continues logic associated with callable symbol `__builtin_ia32_vrndscalebf16_256_mask`.
  **L779 CN**: 继续与可调用符号 `__builtin_ia32_vrndscalebf16_256_mask` 相关的逻辑。
- **L780 EN**: Continues logic associated with callable symbol `_mm256_setzero_pbh`.
  **L780 CN**: 继续与可调用符号 `_mm256_setzero_pbh` 相关的逻辑。
- **L781 EN**: Continues the surrounding expression or declaration: `(__mmask16)(__U)))`.
  **L781 CN**: 继续构造周围的表达式或声明：`(__mmask16)(__U)))`。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L783 EN**: Defines macro `_mm_roundscale_pbh(__A, imm)` for conditional compilation, shorthand, or API generation.
  **L783 CN**: 定义宏 `_mm_roundscale_pbh(__A, imm)`，用于条件编译、简写或 API 生成。
- **L784 EN**: Continues logic associated with callable symbol `__builtin_ia32_vrndscalebf16_128_mask`.
  **L784 CN**: 继续与可调用符号 `__builtin_ia32_vrndscalebf16_128_mask` 相关的逻辑。
- **L785 EN**: Continues logic associated with callable symbol `_mm_setzero_pbh`.
  **L785 CN**: 继续与可调用符号 `_mm_setzero_pbh` 相关的逻辑。
- **L786 EN**: Continues the surrounding expression or declaration: `(__mmask8) - 1))`.
  **L786 CN**: 继续构造周围的表达式或声明：`(__mmask8) - 1))`。
- **L787 EN**: Blank line separating nearby declarations or logic blocks.
  **L787 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L788 EN**: Defines macro `_mm_mask_roundscale_pbh(__W, __U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L788 CN**: 定义宏 `_mm_mask_roundscale_pbh(__W, __U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L789 EN**: Continues logic associated with callable symbol `__builtin_ia32_vrndscalebf16_128_mask`.
  **L789 CN**: 继续与可调用符号 `__builtin_ia32_vrndscalebf16_128_mask` 相关的逻辑。
- **L790 EN**: Continues the surrounding expression or declaration: `(__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)(__m128bh)(__W),            \`.
  **L790 CN**: 继续构造周围的表达式或声明：`(__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)(__m128bh)(__W),            \`。
- **L791 EN**: Continues the surrounding expression or declaration: `(__mmask8)(__U)))`.
  **L791 CN**: 继续构造周围的表达式或声明：`(__mmask8)(__U)))`。
- **L792 EN**: Blank line separating nearby declarations or logic blocks.
  **L792 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 793-816

````c
#define _mm_maskz_roundscale_pbh(__U, __A, imm)                                \
  ((__m128bh)__builtin_ia32_vrndscalebf16_128_mask(                            \
      (__v8bf)(__m128bh)(__A), (int)(imm), (__v8bf)_mm_setzero_pbh(),          \
      (__mmask8)(__U)))

#define _mm256_getmant_pbh(__A, __B, __C)                                      \
  ((__m256bh)__builtin_ia32_vgetmantbf16256_mask(                              \
      (__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \
      (__v16bf)_mm256_undefined_pbh(), (__mmask16) - 1))

#define _mm256_mask_getmant_pbh(__W, __U, __A, __B, __C)                       \
  ((__m256bh)__builtin_ia32_vgetmantbf16256_mask(                              \
      (__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \
      (__v16bf)(__m256bh)(__W), (__mmask16)(__U)))

#define _mm256_maskz_getmant_pbh(__U, __A, __B, __C)                           \
  ((__m256bh)__builtin_ia32_vgetmantbf16256_mask(                              \
      (__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \
      (__v16bf)_mm256_setzero_pbh(), (__mmask16)(__U)))

#define _mm_getmant_pbh(__A, __B, __C)                                         \
  ((__m128bh)__builtin_ia32_vgetmantbf16128_mask(                              \
      (__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \
      (__v8bf)_mm_undefined_pbh(), (__mmask8) - 1))
````
- **L793 EN**: Defines macro `_mm_maskz_roundscale_pbh(__U, __A, imm)` for conditional compilation, shorthand, or API generation.
  **L793 CN**: 定义宏 `_mm_maskz_roundscale_pbh(__U, __A, imm)`，用于条件编译、简写或 API 生成。
- **L794 EN**: Continues logic associated with callable symbol `__builtin_ia32_vrndscalebf16_128_mask`.
  **L794 CN**: 继续与可调用符号 `__builtin_ia32_vrndscalebf16_128_mask` 相关的逻辑。
- **L795 EN**: Continues logic associated with callable symbol `_mm_setzero_pbh`.
  **L795 CN**: 继续与可调用符号 `_mm_setzero_pbh` 相关的逻辑。
- **L796 EN**: Continues the surrounding expression or declaration: `(__mmask8)(__U)))`.
  **L796 CN**: 继续构造周围的表达式或声明：`(__mmask8)(__U)))`。
- **L797 EN**: Blank line separating nearby declarations or logic blocks.
  **L797 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L798 EN**: Defines macro `_mm256_getmant_pbh(__A, __B, __C)` for conditional compilation, shorthand, or API generation.
  **L798 CN**: 定义宏 `_mm256_getmant_pbh(__A, __B, __C)`，用于条件编译、简写或 API 生成。
- **L799 EN**: Continues logic associated with callable symbol `__builtin_ia32_vgetmantbf16256_mask`.
  **L799 CN**: 继续与可调用符号 `__builtin_ia32_vgetmantbf16256_mask` 相关的逻辑。
- **L800 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \`.
  **L800 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \`。
- **L801 EN**: Continues logic associated with callable symbol `_mm256_undefined_pbh`.
  **L801 CN**: 继续与可调用符号 `_mm256_undefined_pbh` 相关的逻辑。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L803 EN**: Defines macro `_mm256_mask_getmant_pbh(__W, __U, __A, __B, __C)` for conditional compilation, shorthand, or API generation.
  **L803 CN**: 定义宏 `_mm256_mask_getmant_pbh(__W, __U, __A, __B, __C)`，用于条件编译、简写或 API 生成。
- **L804 EN**: Continues logic associated with callable symbol `__builtin_ia32_vgetmantbf16256_mask`.
  **L804 CN**: 继续与可调用符号 `__builtin_ia32_vgetmantbf16256_mask` 相关的逻辑。
- **L805 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \`.
  **L805 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \`。
- **L806 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__W), (__mmask16)(__U)))`.
  **L806 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__W), (__mmask16)(__U)))`。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L808 EN**: Defines macro `_mm256_maskz_getmant_pbh(__U, __A, __B, __C)` for conditional compilation, shorthand, or API generation.
  **L808 CN**: 定义宏 `_mm256_maskz_getmant_pbh(__U, __A, __B, __C)`，用于条件编译、简写或 API 生成。
- **L809 EN**: Continues logic associated with callable symbol `__builtin_ia32_vgetmantbf16256_mask`.
  **L809 CN**: 继续与可调用符号 `__builtin_ia32_vgetmantbf16256_mask` 相关的逻辑。
- **L810 EN**: Continues the surrounding expression or declaration: `(__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \`.
  **L810 CN**: 继续构造周围的表达式或声明：`(__v16bf)(__m256bh)(__A), (int)(((__C) << 2) | (__B)),                   \`。
- **L811 EN**: Continues logic associated with callable symbol `_mm256_setzero_pbh`.
  **L811 CN**: 继续与可调用符号 `_mm256_setzero_pbh` 相关的逻辑。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Defines macro `_mm_getmant_pbh(__A, __B, __C)` for conditional compilation, shorthand, or API generation.
  **L813 CN**: 定义宏 `_mm_getmant_pbh(__A, __B, __C)`，用于条件编译、简写或 API 生成。
- **L814 EN**: Continues logic associated with callable symbol `__builtin_ia32_vgetmantbf16128_mask`.
  **L814 CN**: 继续与可调用符号 `__builtin_ia32_vgetmantbf16128_mask` 相关的逻辑。
- **L815 EN**: Continues the surrounding expression or declaration: `(__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \`.
  **L815 CN**: 继续构造周围的表达式或声明：`(__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \`。
- **L816 EN**: Continues logic associated with callable symbol `_mm_undefined_pbh`.
  **L816 CN**: 继续与可调用符号 `_mm_undefined_pbh` 相关的逻辑。

### Lines 817-840

````c

#define _mm_mask_getmant_pbh(__W, __U, __A, __B, __C)                          \
  ((__m128bh)__builtin_ia32_vgetmantbf16128_mask(                              \
      (__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \
      (__v8bf)(__m128bh)(__W), (__mmask8)(__U)))

#define _mm_maskz_getmant_pbh(__U, __A, __B, __C)                              \
  ((__m128bh)__builtin_ia32_vgetmantbf16128_mask(                              \
      (__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \
      (__v8bf)_mm_setzero_pbh(), (__mmask8)(__U)))

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_sqrt_pbh(__m256bh __A) {
  return __builtin_elementwise_sqrt(__A);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_sqrt_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U, (__v16bf)_mm256_sqrt_pbh(__A), (__v16bf)__W);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_maskz_sqrt_pbh(__mmask16 __U, __m256bh __A) {
  return (__m256bh)__builtin_ia32_selectpbf_256((__mmask16)__U,
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L818 EN**: Defines macro `_mm_mask_getmant_pbh(__W, __U, __A, __B, __C)` for conditional compilation, shorthand, or API generation.
  **L818 CN**: 定义宏 `_mm_mask_getmant_pbh(__W, __U, __A, __B, __C)`，用于条件编译、简写或 API 生成。
- **L819 EN**: Continues logic associated with callable symbol `__builtin_ia32_vgetmantbf16128_mask`.
  **L819 CN**: 继续与可调用符号 `__builtin_ia32_vgetmantbf16128_mask` 相关的逻辑。
- **L820 EN**: Continues the surrounding expression or declaration: `(__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \`.
  **L820 CN**: 继续构造周围的表达式或声明：`(__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \`。
- **L821 EN**: Continues the surrounding expression or declaration: `(__v8bf)(__m128bh)(__W), (__mmask8)(__U)))`.
  **L821 CN**: 继续构造周围的表达式或声明：`(__v8bf)(__m128bh)(__W), (__mmask8)(__U)))`。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Defines macro `_mm_maskz_getmant_pbh(__U, __A, __B, __C)` for conditional compilation, shorthand, or API generation.
  **L823 CN**: 定义宏 `_mm_maskz_getmant_pbh(__U, __A, __B, __C)`，用于条件编译、简写或 API 生成。
- **L824 EN**: Continues logic associated with callable symbol `__builtin_ia32_vgetmantbf16128_mask`.
  **L824 CN**: 继续与可调用符号 `__builtin_ia32_vgetmantbf16128_mask` 相关的逻辑。
- **L825 EN**: Continues the surrounding expression or declaration: `(__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \`.
  **L825 CN**: 继续构造周围的表达式或声明：`(__v8bf)(__m128bh)(__A), (int)(((__C) << 2) | (__B)),                    \`。
- **L826 EN**: Continues logic associated with callable symbol `_mm_setzero_pbh`.
  **L826 CN**: 继续与可调用符号 `_mm_setzero_pbh` 相关的逻辑。
- **L827 EN**: Blank line separating nearby declarations or logic blocks.
  **L827 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L828 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_sqrt_pbh(__m256bh __A) {`.
  **L828 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_sqrt_pbh(__m256bh __A) {`。
- **L829 EN**: Returns from the current function with `__builtin_elementwise_sqrt(__A)`.
  **L829 CN**: 以 `__builtin_elementwise_sqrt(__A)` 从当前函数返回。
- **L830 EN**: Closes the current lexical scope or compound statement.
  **L830 CN**: 结束当前词法作用域或复合语句块。
- **L831 EN**: Blank line separating nearby declarations or logic blocks.
  **L831 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L832 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L832 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L833 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_sqrt_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {`.
  **L833 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_sqrt_pbh(__m256bh __W, __mmask16 __U, __m256bh __A) {`。
- **L834 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L834 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L835 EN**: Executes a call or declaration centered on `statement`.
  **L835 CN**: 执行以 `statement` 为核心的调用或声明。
- **L836 EN**: Closes the current lexical scope or compound statement.
  **L836 CN**: 结束当前词法作用域或复合语句块。
- **L837 EN**: Blank line separating nearby declarations or logic blocks.
  **L837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L838 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L838 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L839 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_sqrt_pbh(__mmask16 __U, __m256bh __A) {`.
  **L839 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_sqrt_pbh(__mmask16 __U, __m256bh __A) {`。
- **L840 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256((__mmask16)__U,`.
  **L840 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256((__mmask16)__U,` 从当前函数返回。

### Lines 841-864

````c
                                                (__v16bf)_mm256_sqrt_pbh(__A),
                                                (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_sqrt_pbh(__m128bh __A) {
  return __builtin_elementwise_sqrt(__A);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_sqrt_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_sqrt_pbh(__A), (__v8bf)__W);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_sqrt_pbh(__mmask8 __U, __m128bh __A) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, (__v8bf)_mm_sqrt_pbh(__A), (__v8bf)_mm_setzero_pbh());
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_fmadd_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_elementwise_fma((__v16bf)__A, (__v16bf)__B,
                                             (__v16bf)__C);
````
- **L841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16bf)_mm256_sqrt_pbh(__A),`.
  **L841 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16bf)_mm256_sqrt_pbh(__A),`。
- **L842 EN**: Executes a call or declaration centered on `statement`.
  **L842 CN**: 执行以 `statement` 为核心的调用或声明。
- **L843 EN**: Closes the current lexical scope or compound statement.
  **L843 CN**: 结束当前词法作用域或复合语句块。
- **L844 EN**: Blank line separating nearby declarations or logic blocks.
  **L844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L845 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_sqrt_pbh(__m128bh __A) {`.
  **L845 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_sqrt_pbh(__m128bh __A) {`。
- **L846 EN**: Returns from the current function with `__builtin_elementwise_sqrt(__A)`.
  **L846 CN**: 以 `__builtin_elementwise_sqrt(__A)` 从当前函数返回。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L849 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L850 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_sqrt_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {`.
  **L850 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_sqrt_pbh(__m128bh __W, __mmask8 __U, __m128bh __A) {`。
- **L851 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L851 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L852 EN**: Executes a call or declaration centered on `statement`.
  **L852 CN**: 执行以 `statement` 为核心的调用或声明。
- **L853 EN**: Closes the current lexical scope or compound statement.
  **L853 CN**: 结束当前词法作用域或复合语句块。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L855 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L855 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L856 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_sqrt_pbh(__mmask8 __U, __m128bh __A) {`.
  **L856 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_sqrt_pbh(__mmask8 __U, __m128bh __A) {`。
- **L857 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L857 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L858 EN**: Executes a call or declaration centered on `statement`.
  **L858 CN**: 执行以 `statement` 为核心的调用或声明。
- **L859 EN**: Closes the current lexical scope or compound statement.
  **L859 CN**: 结束当前词法作用域或复合语句块。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L861 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L861 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L862 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_fmadd_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {`.
  **L862 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_fmadd_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {`。
- **L863 EN**: Returns from the current function with `(__m256bh)__builtin_elementwise_fma((__v16bf)__A, (__v16bf)__B,`.
  **L863 CN**: 以 `(__m256bh)__builtin_elementwise_fma((__v16bf)__A, (__v16bf)__B,` 从当前函数返回。
- **L864 EN**: Executes a call or declaration centered on `statement`.
  **L864 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 865-888

````c
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_fmadd_pbh(__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C), (__v16bf)__A);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mask3_fmadd_pbh(
    __m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C), (__v16bf)__C);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_maskz_fmadd_pbh(
    __mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),
      (__v16bf)_mm256_setzero_pbh());
}

````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L867 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L868 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fmadd_pbh(__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {`.
  **L868 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fmadd_pbh(__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {`。
- **L869 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L869 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L871 EN**: Executes a call or declaration centered on `_mm256_fmadd_pbh`.
  **L871 CN**: 执行以 `_mm256_fmadd_pbh` 为核心的调用或声明。
- **L872 EN**: Closes the current lexical scope or compound statement.
  **L872 CN**: 结束当前词法作用域或复合语句块。
- **L873 EN**: Blank line separating nearby declarations or logic blocks.
  **L873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L874 EN**: Continues logic associated with callable symbol `_mm256_mask3_fmadd_pbh`.
  **L874 CN**: 继续与可调用符号 `_mm256_mask3_fmadd_pbh` 相关的逻辑。
- **L875 EN**: Continues the surrounding expression or declaration: `__m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {`.
  **L875 CN**: 继续构造周围的表达式或声明：`__m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {`。
- **L876 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L876 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L878 EN**: Executes a call or declaration centered on `_mm256_fmadd_pbh`.
  **L878 CN**: 执行以 `_mm256_fmadd_pbh` 为核心的调用或声明。
- **L879 EN**: Closes the current lexical scope or compound statement.
  **L879 CN**: 结束当前词法作用域或复合语句块。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L881 EN**: Continues logic associated with callable symbol `_mm256_maskz_fmadd_pbh`.
  **L881 CN**: 继续与可调用符号 `_mm256_maskz_fmadd_pbh` 相关的逻辑。
- **L882 EN**: Continues the surrounding expression or declaration: `__mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {`.
  **L882 CN**: 继续构造周围的表达式或声明：`__mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {`。
- **L883 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L883 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_fmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`.
  **L885 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_fmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`。
- **L886 EN**: Executes a call or declaration centered on `statement`.
  **L886 CN**: 执行以 `statement` 为核心的调用或声明。
- **L887 EN**: Closes the current lexical scope or compound statement.
  **L887 CN**: 结束当前词法作用域或复合语句块。
- **L888 EN**: Blank line separating nearby declarations or logic blocks.
  **L888 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 889-912

````c
static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_fmsub_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_elementwise_fma((__v16bf)__A, (__v16bf)__B,
                                             -(__v16bf)__C);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_mask_fmsub_pbh(__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C), (__v16bf)__A);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mask3_fmsub_pbh(
    __m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C), (__v16bf)__C);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_maskz_fmsub_pbh(
    __mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
````
- **L889 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L889 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L890 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_fmsub_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {`.
  **L890 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_fmsub_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {`。
- **L891 EN**: Returns from the current function with `(__m256bh)__builtin_elementwise_fma((__v16bf)__A, (__v16bf)__B,`.
  **L891 CN**: 以 `(__m256bh)__builtin_elementwise_fma((__v16bf)__A, (__v16bf)__B,` 从当前函数返回。
- **L892 EN**: Executes a call or declaration centered on `-`.
  **L892 CN**: 执行以 `-` 为核心的调用或声明。
- **L893 EN**: Closes the current lexical scope or compound statement.
  **L893 CN**: 结束当前词法作用域或复合语句块。
- **L894 EN**: Blank line separating nearby declarations or logic blocks.
  **L894 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L895 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L895 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L896 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_fmsub_pbh(__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {`.
  **L896 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_fmsub_pbh(__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {`。
- **L897 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L897 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L899 EN**: Executes a call or declaration centered on `_mm256_fmsub_pbh`.
  **L899 CN**: 执行以 `_mm256_fmsub_pbh` 为核心的调用或声明。
- **L900 EN**: Closes the current lexical scope or compound statement.
  **L900 CN**: 结束当前词法作用域或复合语句块。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L902 EN**: Continues logic associated with callable symbol `_mm256_mask3_fmsub_pbh`.
  **L902 CN**: 继续与可调用符号 `_mm256_mask3_fmsub_pbh` 相关的逻辑。
- **L903 EN**: Continues the surrounding expression or declaration: `__m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {`.
  **L903 CN**: 继续构造周围的表达式或声明：`__m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {`。
- **L904 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L904 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L905 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L906 EN**: Executes a call or declaration centered on `_mm256_fmsub_pbh`.
  **L906 CN**: 执行以 `_mm256_fmsub_pbh` 为核心的调用或声明。
- **L907 EN**: Closes the current lexical scope or compound statement.
  **L907 CN**: 结束当前词法作用域或复合语句块。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L909 EN**: Continues logic associated with callable symbol `_mm256_maskz_fmsub_pbh`.
  **L909 CN**: 继续与可调用符号 `_mm256_maskz_fmsub_pbh` 相关的逻辑。
- **L910 EN**: Continues the surrounding expression or declaration: `__mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {`.
  **L910 CN**: 继续构造周围的表达式或声明：`__mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {`。
- **L911 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L911 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。

### Lines 913-936

````c
      _mm256_fmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),
      (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_fnmadd_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_elementwise_fma((__v16bf)__A, -(__v16bf)__B,
                                             (__v16bf)__C);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mask_fnmadd_pbh(
    __m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),
      (__v16bf)__A);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mask3_fnmadd_pbh(
    __m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),
      (__v16bf)__C);
````
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_fmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`.
  **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_fmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`。
- **L914 EN**: Executes a call or declaration centered on `statement`.
  **L914 CN**: 执行以 `statement` 为核心的调用或声明。
- **L915 EN**: Closes the current lexical scope or compound statement.
  **L915 CN**: 结束当前词法作用域或复合语句块。
- **L916 EN**: Blank line separating nearby declarations or logic blocks.
  **L916 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L917 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L917 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L918 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_fnmadd_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {`.
  **L918 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_fnmadd_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {`。
- **L919 EN**: Returns from the current function with `(__m256bh)__builtin_elementwise_fma((__v16bf)__A, -(__v16bf)__B,`.
  **L919 CN**: 以 `(__m256bh)__builtin_elementwise_fma((__v16bf)__A, -(__v16bf)__B,` 从当前函数返回。
- **L920 EN**: Executes a call or declaration centered on `statement`.
  **L920 CN**: 执行以 `statement` 为核心的调用或声明。
- **L921 EN**: Closes the current lexical scope or compound statement.
  **L921 CN**: 结束当前词法作用域或复合语句块。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L923 EN**: Continues logic associated with callable symbol `_mm256_mask_fnmadd_pbh`.
  **L923 CN**: 继续与可调用符号 `_mm256_mask_fnmadd_pbh` 相关的逻辑。
- **L924 EN**: Continues the surrounding expression or declaration: `__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {`.
  **L924 CN**: 继续构造周围的表达式或声明：`__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {`。
- **L925 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L925 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`。
- **L928 EN**: Executes a call or declaration centered on `statement`.
  **L928 CN**: 执行以 `statement` 为核心的调用或声明。
- **L929 EN**: Closes the current lexical scope or compound statement.
  **L929 CN**: 结束当前词法作用域或复合语句块。
- **L930 EN**: Blank line separating nearby declarations or logic blocks.
  **L930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L931 EN**: Continues logic associated with callable symbol `_mm256_mask3_fnmadd_pbh`.
  **L931 CN**: 继续与可调用符号 `_mm256_mask3_fnmadd_pbh` 相关的逻辑。
- **L932 EN**: Continues the surrounding expression or declaration: `__m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {`.
  **L932 CN**: 继续构造周围的表达式或声明：`__m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {`。
- **L933 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L933 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`。
- **L936 EN**: Executes a call or declaration centered on `statement`.
  **L936 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 937-960

````c
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_maskz_fnmadd_pbh(
    __mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),
      (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256
_mm256_fnmsub_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_elementwise_fma((__v16bf)__A, -(__v16bf)__B,
                                             -(__v16bf)__C);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mask_fnmsub_pbh(
    __m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),
      (__v16bf)__A);
}

````
- **L937 EN**: Closes the current lexical scope or compound statement.
  **L937 CN**: 结束当前词法作用域或复合语句块。
- **L938 EN**: Blank line separating nearby declarations or logic blocks.
  **L938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L939 EN**: Continues logic associated with callable symbol `_mm256_maskz_fnmadd_pbh`.
  **L939 CN**: 继续与可调用符号 `_mm256_maskz_fnmadd_pbh` 相关的逻辑。
- **L940 EN**: Continues the surrounding expression or declaration: `__mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {`.
  **L940 CN**: 继续构造周围的表达式或声明：`__mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {`。
- **L941 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L941 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L942 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L942 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L943 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`.
  **L943 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_fnmadd_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`。
- **L944 EN**: Executes a call or declaration centered on `statement`.
  **L944 CN**: 执行以 `statement` 为核心的调用或声明。
- **L945 EN**: Closes the current lexical scope or compound statement.
  **L945 CN**: 结束当前词法作用域或复合语句块。
- **L946 EN**: Blank line separating nearby declarations or logic blocks.
  **L946 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L947 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256bh __DEFAULT_FN_ATTRS256`.
  **L947 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256bh __DEFAULT_FN_ATTRS256`。
- **L948 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_fnmsub_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {`.
  **L948 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_fnmsub_pbh(__m256bh __A, __m256bh __B, __m256bh __C) {`。
- **L949 EN**: Returns from the current function with `(__m256bh)__builtin_elementwise_fma((__v16bf)__A, -(__v16bf)__B,`.
  **L949 CN**: 以 `(__m256bh)__builtin_elementwise_fma((__v16bf)__A, -(__v16bf)__B,` 从当前函数返回。
- **L950 EN**: Executes a call or declaration centered on `-`.
  **L950 CN**: 执行以 `-` 为核心的调用或声明。
- **L951 EN**: Closes the current lexical scope or compound statement.
  **L951 CN**: 结束当前词法作用域或复合语句块。
- **L952 EN**: Blank line separating nearby declarations or logic blocks.
  **L952 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L953 EN**: Continues logic associated with callable symbol `_mm256_mask_fnmsub_pbh`.
  **L953 CN**: 继续与可调用符号 `_mm256_mask_fnmsub_pbh` 相关的逻辑。
- **L954 EN**: Continues the surrounding expression or declaration: `__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {`.
  **L954 CN**: 继续构造周围的表达式或声明：`__m256bh __A, __mmask16 __U, __m256bh __B, __m256bh __C) {`。
- **L955 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L955 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L956 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L957 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`.
  **L957 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`。
- **L958 EN**: Executes a call or declaration centered on `statement`.
  **L958 CN**: 执行以 `statement` 为核心的调用或声明。
- **L959 EN**: Closes the current lexical scope or compound statement.
  **L959 CN**: 结束当前词法作用域或复合语句块。
- **L960 EN**: Blank line separating nearby declarations or logic blocks.
  **L960 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 961-984

````c
static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_mask3_fnmsub_pbh(
    __m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),
      (__v16bf)__C);
}

static __inline__ __m256bh __DEFAULT_FN_ATTRS256 _mm256_maskz_fnmsub_pbh(
    __mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {
  return (__m256bh)__builtin_ia32_selectpbf_256(
      (__mmask16)__U,
      _mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),
      (__v16bf)_mm256_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fmadd_pbh(__m128bh __A,
                                                               __m128bh __B,
                                                               __m128bh __C) {
  return (__m128bh)__builtin_elementwise_fma((__v8bf)__A, (__v8bf)__B,
                                             (__v8bf)__C);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
````
- **L961 EN**: Continues logic associated with callable symbol `_mm256_mask3_fnmsub_pbh`.
  **L961 CN**: 继续与可调用符号 `_mm256_mask3_fnmsub_pbh` 相关的逻辑。
- **L962 EN**: Continues the surrounding expression or declaration: `__m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {`.
  **L962 CN**: 继续构造周围的表达式或声明：`__m256bh __A, __m256bh __B, __m256bh __C, __mmask16 __U) {`。
- **L963 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L963 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L964 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`.
  **L965 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`。
- **L966 EN**: Executes a call or declaration centered on `statement`.
  **L966 CN**: 执行以 `statement` 为核心的调用或声明。
- **L967 EN**: Closes the current lexical scope or compound statement.
  **L967 CN**: 结束当前词法作用域或复合语句块。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Continues logic associated with callable symbol `_mm256_maskz_fnmsub_pbh`.
  **L969 CN**: 继续与可调用符号 `_mm256_maskz_fnmsub_pbh` 相关的逻辑。
- **L970 EN**: Continues the surrounding expression or declaration: `__mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {`.
  **L970 CN**: 继续构造周围的表达式或声明：`__mmask16 __U, __m256bh __A, __m256bh __B, __m256bh __C) {`。
- **L971 EN**: Returns from the current function with `(__m256bh)__builtin_ia32_selectpbf_256(`.
  **L971 CN**: 以 `(__m256bh)__builtin_ia32_selectpbf_256(` 从当前函数返回。
- **L972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U,`.
  **L972 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U,`。
- **L973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`.
  **L973 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_fnmsub_pbh((__v16bf)__A, (__v16bf)__B, (__v16bf)__C),`。
- **L974 EN**: Executes a call or declaration centered on `statement`.
  **L974 CN**: 执行以 `statement` 为核心的调用或声明。
- **L975 EN**: Closes the current lexical scope or compound statement.
  **L975 CN**: 结束当前词法作用域或复合语句块。
- **L976 EN**: Blank line separating nearby declarations or logic blocks.
  **L976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fmadd_pbh(__m128bh __A,`.
  **L977 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fmadd_pbh(__m128bh __A,`。
- **L978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128bh __B,`.
  **L978 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128bh __B,`。
- **L979 EN**: Continues the surrounding expression or declaration: `__m128bh __C) {`.
  **L979 CN**: 继续构造周围的表达式或声明：`__m128bh __C) {`。
- **L980 EN**: Returns from the current function with `(__m128bh)__builtin_elementwise_fma((__v8bf)__A, (__v8bf)__B,`.
  **L980 CN**: 以 `(__m128bh)__builtin_elementwise_fma((__v8bf)__A, (__v8bf)__B,` 从当前函数返回。
- **L981 EN**: Executes a call or declaration centered on `statement`.
  **L981 CN**: 执行以 `statement` 为核心的调用或声明。
- **L982 EN**: Closes the current lexical scope or compound statement.
  **L982 CN**: 结束当前词法作用域或复合语句块。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L984 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L984 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。

### Lines 985-1008

````c
_mm_mask_fmadd_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)__A);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask3_fmadd_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)__C);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_fmadd_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)_mm_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fmsub_pbh(__m128bh __A,
                                                               __m128bh __B,
                                                               __m128bh __C) {
  return (__m128bh)__builtin_elementwise_fma((__v8bf)__A, (__v8bf)__B,
````
- **L985 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fmadd_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {`.
  **L985 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fmadd_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {`。
- **L986 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L986 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L987 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L988 EN**: Executes a call or declaration centered on `statement`.
  **L988 CN**: 执行以 `statement` 为核心的调用或声明。
- **L989 EN**: Closes the current lexical scope or compound statement.
  **L989 CN**: 结束当前词法作用域或复合语句块。
- **L990 EN**: Blank line separating nearby declarations or logic blocks.
  **L990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L991 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L991 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L992 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmadd_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {`.
  **L992 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmadd_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {`。
- **L993 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L993 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L994 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L995 EN**: Executes a call or declaration centered on `statement`.
  **L995 CN**: 执行以 `statement` 为核心的调用或声明。
- **L996 EN**: Closes the current lexical scope or compound statement.
  **L996 CN**: 结束当前词法作用域或复合语句块。
- **L997 EN**: Blank line separating nearby declarations or logic blocks.
  **L997 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L998 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L998 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L999 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmadd_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {`.
  **L999 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmadd_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {`。
- **L1000 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1000 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1002 EN**: Executes a call or declaration centered on `statement`.
  **L1002 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1003 EN**: Closes the current lexical scope or compound statement.
  **L1003 CN**: 结束当前词法作用域或复合语句块。
- **L1004 EN**: Blank line separating nearby declarations or logic blocks.
  **L1004 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fmsub_pbh(__m128bh __A,`.
  **L1005 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fmsub_pbh(__m128bh __A,`。
- **L1006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128bh __B,`.
  **L1006 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128bh __B,`。
- **L1007 EN**: Continues the surrounding expression or declaration: `__m128bh __C) {`.
  **L1007 CN**: 继续构造周围的表达式或声明：`__m128bh __C) {`。
- **L1008 EN**: Returns from the current function with `(__m128bh)__builtin_elementwise_fma((__v8bf)__A, (__v8bf)__B,`.
  **L1008 CN**: 以 `(__m128bh)__builtin_elementwise_fma((__v8bf)__A, (__v8bf)__B,` 从当前函数返回。

### Lines 1009-1032

````c
                                             -(__v8bf)__C);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_fmsub_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)__A);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask3_fmsub_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)__C);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_fmsub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)_mm_setzero_pbh());
}

````
- **L1009 EN**: Executes a call or declaration centered on `-`.
  **L1009 CN**: 执行以 `-` 为核心的调用或声明。
- **L1010 EN**: Closes the current lexical scope or compound statement.
  **L1010 CN**: 结束当前词法作用域或复合语句块。
- **L1011 EN**: Blank line separating nearby declarations or logic blocks.
  **L1011 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1012 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1012 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1013 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fmsub_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {`.
  **L1013 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fmsub_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {`。
- **L1014 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1014 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1015 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1016 EN**: Executes a call or declaration centered on `statement`.
  **L1016 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1017 EN**: Closes the current lexical scope or compound statement.
  **L1017 CN**: 结束当前词法作用域或复合语句块。
- **L1018 EN**: Blank line separating nearby declarations or logic blocks.
  **L1018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1019 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1019 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1020 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fmsub_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {`.
  **L1020 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fmsub_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {`。
- **L1021 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1021 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1022 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1023 EN**: Executes a call or declaration centered on `statement`.
  **L1023 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1024 EN**: Closes the current lexical scope or compound statement.
  **L1024 CN**: 结束当前词法作用域或复合语句块。
- **L1025 EN**: Blank line separating nearby declarations or logic blocks.
  **L1025 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1026 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1026 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1027 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fmsub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {`.
  **L1027 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fmsub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {`。
- **L1028 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1028 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1029 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1030 EN**: Executes a call or declaration centered on `statement`.
  **L1030 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1031 EN**: Closes the current lexical scope or compound statement.
  **L1031 CN**: 结束当前词法作用域或复合语句块。
- **L1032 EN**: Blank line separating nearby declarations or logic blocks.
  **L1032 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1033-1056

````c
static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fnmadd_pbh(__m128bh __A,
                                                                __m128bh __B,
                                                                __m128bh __C) {
  return (__m128bh)__builtin_elementwise_fma((__v8bf)__A, -(__v8bf)__B,
                                             (__v8bf)__C);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_fnmadd_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)__A);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask3_fnmadd_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)__C);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_fnmadd_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
````
- **L1033 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fnmadd_pbh(__m128bh __A,`.
  **L1033 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fnmadd_pbh(__m128bh __A,`。
- **L1034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128bh __B,`.
  **L1034 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128bh __B,`。
- **L1035 EN**: Continues the surrounding expression or declaration: `__m128bh __C) {`.
  **L1035 CN**: 继续构造周围的表达式或声明：`__m128bh __C) {`。
- **L1036 EN**: Returns from the current function with `(__m128bh)__builtin_elementwise_fma((__v8bf)__A, -(__v8bf)__B,`.
  **L1036 CN**: 以 `(__m128bh)__builtin_elementwise_fma((__v8bf)__A, -(__v8bf)__B,` 从当前函数返回。
- **L1037 EN**: Executes a call or declaration centered on `statement`.
  **L1037 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1038 EN**: Closes the current lexical scope or compound statement.
  **L1038 CN**: 结束当前词法作用域或复合语句块。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1040 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1040 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1041 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fnmadd_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {`.
  **L1041 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fnmadd_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {`。
- **L1042 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1042 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1044 EN**: Executes a call or declaration centered on `statement`.
  **L1044 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1045 EN**: Closes the current lexical scope or compound statement.
  **L1045 CN**: 结束当前词法作用域或复合语句块。
- **L1046 EN**: Blank line separating nearby declarations or logic blocks.
  **L1046 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1047 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1047 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1048 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fnmadd_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {`.
  **L1048 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fnmadd_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {`。
- **L1049 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1049 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1050 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1051 EN**: Executes a call or declaration centered on `statement`.
  **L1051 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1052 EN**: Closes the current lexical scope or compound statement.
  **L1052 CN**: 结束当前词法作用域或复合语句块。
- **L1053 EN**: Blank line separating nearby declarations or logic blocks.
  **L1053 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1054 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1054 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1055 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fnmadd_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {`.
  **L1055 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fnmadd_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {`。
- **L1056 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1056 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。

### Lines 1057-1080

````c
      (__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)_mm_setzero_pbh());
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fnmsub_pbh(__m128bh __A,
                                                                __m128bh __B,
                                                                __m128bh __C) {
  return (__m128bh)__builtin_elementwise_fma((__v8bf)__A, -(__v8bf)__B,
                                             -(__v8bf)__C);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask_fnmsub_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)__A);
}

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_mask3_fnmsub_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)__C);
}
````
- **L1057 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1057 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fnmadd_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1058 EN**: Executes a call or declaration centered on `statement`.
  **L1058 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1059 EN**: Closes the current lexical scope or compound statement.
  **L1059 CN**: 结束当前词法作用域或复合语句块。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fnmsub_pbh(__m128bh __A,`.
  **L1061 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128 _mm_fnmsub_pbh(__m128bh __A,`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__m128bh __B,`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`__m128bh __B,`。
- **L1063 EN**: Continues the surrounding expression or declaration: `__m128bh __C) {`.
  **L1063 CN**: 继续构造周围的表达式或声明：`__m128bh __C) {`。
- **L1064 EN**: Returns from the current function with `(__m128bh)__builtin_elementwise_fma((__v8bf)__A, -(__v8bf)__B,`.
  **L1064 CN**: 以 `(__m128bh)__builtin_elementwise_fma((__v8bf)__A, -(__v8bf)__B,` 从当前函数返回。
- **L1065 EN**: Executes a call or declaration centered on `-`.
  **L1065 CN**: 执行以 `-` 为核心的调用或声明。
- **L1066 EN**: Closes the current lexical scope or compound statement.
  **L1066 CN**: 结束当前词法作用域或复合语句块。
- **L1067 EN**: Blank line separating nearby declarations or logic blocks.
  **L1067 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1068 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1068 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1069 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_fnmsub_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {`.
  **L1069 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_fnmsub_pbh(__m128bh __A, __mmask8 __U, __m128bh __B, __m128bh __C) {`。
- **L1070 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1070 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1072 EN**: Executes a call or declaration centered on `statement`.
  **L1072 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1073 EN**: Closes the current lexical scope or compound statement.
  **L1073 CN**: 结束当前词法作用域或复合语句块。
- **L1074 EN**: Blank line separating nearby declarations or logic blocks.
  **L1074 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1075 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1075 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1076 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask3_fnmsub_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {`.
  **L1076 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask3_fnmsub_pbh(__m128bh __A, __m128bh __B, __m128bh __C, __mmask8 __U) {`。
- **L1077 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1077 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1078 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1078 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1079 EN**: Executes a call or declaration centered on `statement`.
  **L1079 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1080 EN**: Closes the current lexical scope or compound statement.
  **L1080 CN**: 结束当前词法作用域或复合语句块。

### Lines 1081-1094

````c

static __inline__ __m128bh __DEFAULT_FN_ATTRS128
_mm_maskz_fnmsub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {
  return (__m128bh)__builtin_ia32_selectpbf_128(
      (__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),
      (__v8bf)_mm_setzero_pbh());
}

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256
#undef __DEFAULT_FN_ATTRS128_CONSTEXPR
#undef __DEFAULT_FN_ATTRS256_CONSTEXPR
#endif
#endif
````
- **L1081 EN**: Blank line separating nearby declarations or logic blocks.
  **L1081 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1082 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128bh __DEFAULT_FN_ATTRS128`.
  **L1082 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128bh __DEFAULT_FN_ATTRS128`。
- **L1083 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_fnmsub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {`.
  **L1083 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_fnmsub_pbh(__mmask8 __U, __m128bh __A, __m128bh __B, __m128bh __C) {`。
- **L1084 EN**: Returns from the current function with `(__m128bh)__builtin_ia32_selectpbf_128(`.
  **L1084 CN**: 以 `(__m128bh)__builtin_ia32_selectpbf_128(` 从当前函数返回。
- **L1085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`.
  **L1085 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask8)__U, _mm_fnmsub_pbh((__v8bf)__A, (__v8bf)__B, (__v8bf)__C),`。
- **L1086 EN**: Executes a call or declaration centered on `statement`.
  **L1086 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1087 EN**: Closes the current lexical scope or compound statement.
  **L1087 CN**: 结束当前词法作用域或复合语句块。
- **L1088 EN**: Blank line separating nearby declarations or logic blocks.
  **L1088 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1089 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L1089 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L1090 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L1090 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。
- **L1091 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1091 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1092 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1092 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1093 EN**: Closes the current preprocessor conditional block.
  **L1093 CN**: 结束当前预处理条件块。
- **L1094 EN**: Closes the current preprocessor conditional block.
  **L1094 CN**: 结束当前预处理条件块。

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
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__SSE2__`, `__AVX10_2BF16INTRIN_H`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_bit_cast`, `__builtin_shufflevector`, `__builtin_ia32_undef256`, `__builtin_ia32_loadsbf16128_mask`, `__builtin_ia32_storesbf16128_mask`, `__builtin_ia32_selectsbf_128`, `__builtin_ia32_undef128`, `__builtin_ia32_selectpbf_128`, `__builtin_ia32_selectpbf_256`, `__builtin_ia32_vpermi2varhi128`, `__builtin_ia32_vpermi2varhi256`, `__builtin_ia32_permvarhi128`
