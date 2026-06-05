# avx512vldqintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx512vldqintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: AVX512VL and AVX512DQ intrinsics.
- **Purpose (CN)**: 提供 AVX512VL and AVX512DQ intrinsic 接口。
- **Line Count / 行数**: 1156

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- avx512vldqintrin.h - AVX512VL and AVX512DQ intrinsics ------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __IMMINTRIN_H
#error "Never use <avx512vldqintrin.h> directly; include <immintrin.h> instead."
#endif

#ifndef __AVX512VLDQINTRIN_H
#define __AVX512VLDQINTRIN_H

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS128                                                  \
  __attribute__((__always_inline__, __nodebug__,                               \
                 __target__("avx512vl,avx512dq"), __min_vector_width__(128)))
#define __DEFAULT_FN_ATTRS256                                                  \
  __attribute__((__always_inline__, __nodebug__,                               \
                 __target__("avx512vl,avx512dq"), __min_vector_width__(256)))

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
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __IMMINTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __IMMINTRIN_H`。
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <avx512vldqintrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error "Never use <avx512vldqintrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifndef __AVX512VLDQINTRIN_H`.
  **L14 CN**: 开始一个预处理条件块：`#ifndef __AVX512VLDQINTRIN_H`。
- **L15 EN**: Defines macro `__AVX512VLDQINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `__AVX512VLDQINTRIN_H`，用于条件编译、简写或 API 生成。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L18 EN**: Defines macro `__DEFAULT_FN_ATTRS128` for conditional compilation, shorthand, or API generation.
  **L18 CN**: 定义宏 `__DEFAULT_FN_ATTRS128`，用于条件编译、简写或 API 生成。
- **L19 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__,                               \`.
  **L19 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__,                               \`。
- **L20 EN**: Continues logic associated with callable symbol `__target__`.
  **L20 CN**: 继续与可调用符号 `__target__` 相关的逻辑。
- **L21 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L21 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L22 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__,                               \`.
  **L22 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__,                               \`。
- **L23 EN**: Continues logic associated with callable symbol `__target__`.
  **L23 CN**: 继续与可调用符号 `__target__` 相关的逻辑。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 25-48

````c
#if defined(__cplusplus) && (__cplusplus >= 201103L)
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256 constexpr
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128 constexpr
#else
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128
#endif

static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm256_mullo_epi64(__m256i __A, __m256i __B) {
  return (__m256i) ((__v4du) __A * (__v4du) __B);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm256_mask_mullo_epi64(__m256i __W, __mmask8 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectq_256((__mmask8)__U,
                                             (__v4di)_mm256_mullo_epi64(__A, __B),
                                             (__v4di)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm256_maskz_mullo_epi64(__mmask8 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectq_256((__mmask8)__U,
                                             (__v4di)_mm256_mullo_epi64(__A, __B),
````
- **L25 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L25 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L26 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L26 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L27 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L27 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L28 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L28 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L29 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L29 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L30 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L30 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L31 EN**: Closes the current preprocessor conditional block.
  **L31 CN**: 结束当前预处理条件块。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L33 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L33 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L34 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mullo_epi64(__m256i __A, __m256i __B) {`.
  **L34 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mullo_epi64(__m256i __A, __m256i __B) {`。
- **L35 EN**: Returns from the current function with `(__m256i) ((__v4du) __A * (__v4du) __B)`.
  **L35 CN**: 以 `(__m256i) ((__v4du) __A * (__v4du) __B)` 从当前函数返回。
- **L36 EN**: Closes the current lexical scope or compound statement.
  **L36 CN**: 结束当前词法作用域或复合语句块。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L38 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L39 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mullo_epi64(__m256i __W, __mmask8 __U, __m256i __A, __m256i __B) {`.
  **L39 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mullo_epi64(__m256i __W, __mmask8 __U, __m256i __A, __m256i __B) {`。
- **L40 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectq_256((__mmask8)__U,`.
  **L40 CN**: 以 `(__m256i)__builtin_ia32_selectq_256((__mmask8)__U,` 从当前函数返回。
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di)_mm256_mullo_epi64(__A, __B),`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di)_mm256_mullo_epi64(__A, __B),`。
- **L42 EN**: Executes a call or declaration centered on `statement`.
  **L42 CN**: 执行以 `statement` 为核心的调用或声明。
- **L43 EN**: Closes the current lexical scope or compound statement.
  **L43 CN**: 结束当前词法作用域或复合语句块。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L45 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L45 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L46 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mullo_epi64(__mmask8 __U, __m256i __A, __m256i __B) {`.
  **L46 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mullo_epi64(__mmask8 __U, __m256i __A, __m256i __B) {`。
- **L47 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectq_256((__mmask8)__U,`.
  **L47 CN**: 以 `(__m256i)__builtin_ia32_selectq_256((__mmask8)__U,` 从当前函数返回。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di)_mm256_mullo_epi64(__A, __B),`.
  **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di)_mm256_mullo_epi64(__A, __B),`。

### Lines 49-72

````c
                                             (__v4di)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mullo_epi64(__m128i __A, __m128i __B) {
  return (__m128i) ((__v2du) __A * (__v2du) __B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_mullo_epi64(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectq_128((__mmask8)__U,
                                             (__v2di)_mm_mullo_epi64(__A, __B),
                                             (__v2di)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_mullo_epi64(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectq_128((__mmask8)__U,
                                             (__v2di)_mm_mullo_epi64(__A, __B),
                                             (__v2di)_mm_setzero_si128());
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_mask_andnot_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {
````
- **L49 EN**: Executes a call or declaration centered on `statement`.
  **L49 CN**: 执行以 `statement` 为核心的调用或声明。
- **L50 EN**: Closes the current lexical scope or compound statement.
  **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L52 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L53 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mullo_epi64(__m128i __A, __m128i __B) {`.
  **L53 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mullo_epi64(__m128i __A, __m128i __B) {`。
- **L54 EN**: Returns from the current function with `(__m128i) ((__v2du) __A * (__v2du) __B)`.
  **L54 CN**: 以 `(__m128i) ((__v2du) __A * (__v2du) __B)` 从当前函数返回。
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L57 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L57 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L58 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mullo_epi64(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L58 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mullo_epi64(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L59 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectq_128((__mmask8)__U,`.
  **L59 CN**: 以 `(__m128i)__builtin_ia32_selectq_128((__mmask8)__U,` 从当前函数返回。
- **L60 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di)_mm_mullo_epi64(__A, __B),`.
  **L60 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di)_mm_mullo_epi64(__A, __B),`。
- **L61 EN**: Executes a call or declaration centered on `statement`.
  **L61 CN**: 执行以 `statement` 为核心的调用或声明。
- **L62 EN**: Closes the current lexical scope or compound statement.
  **L62 CN**: 结束当前词法作用域或复合语句块。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L64 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L65 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mullo_epi64(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L65 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mullo_epi64(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L66 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectq_128((__mmask8)__U,`.
  **L66 CN**: 以 `(__m128i)__builtin_ia32_selectq_128((__mmask8)__U,` 从当前函数返回。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di)_mm_mullo_epi64(__A, __B),`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di)_mm_mullo_epi64(__A, __B),`。
- **L68 EN**: Executes a call or declaration centered on `statement`.
  **L68 CN**: 执行以 `statement` 为核心的调用或声明。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L71 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L71 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L72 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_andnot_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {`.
  **L72 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_andnot_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {`。

### Lines 73-96

````c
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_andnot_pd(__A, __B),
                                              (__v4df)__W);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_maskz_andnot_pd(__mmask8 __U, __m256d __A, __m256d __B) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_andnot_pd(__A, __B),
                                              (__v4df)_mm256_setzero_pd());
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_mask_andnot_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_andnot_pd(__A, __B),
                                              (__v2df)__W);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_maskz_andnot_pd(__mmask8 __U, __m128d __A, __m128d __B) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_andnot_pd(__A, __B),
                                              (__v2df)_mm_setzero_pd());
````
- **L73 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L73 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_andnot_pd(__A, __B),`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_andnot_pd(__A, __B),`。
- **L75 EN**: Executes a call or declaration centered on `statement`.
  **L75 CN**: 执行以 `statement` 为核心的调用或声明。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L78 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L78 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L79 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_andnot_pd(__mmask8 __U, __m256d __A, __m256d __B) {`.
  **L79 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_andnot_pd(__mmask8 __U, __m256d __A, __m256d __B) {`。
- **L80 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L80 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_andnot_pd(__A, __B),`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_andnot_pd(__A, __B),`。
- **L82 EN**: Executes a call or declaration centered on `statement`.
  **L82 CN**: 执行以 `statement` 为核心的调用或声明。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L85 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L86 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_andnot_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {`.
  **L86 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_andnot_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {`。
- **L87 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L87 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_andnot_pd(__A, __B),`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_andnot_pd(__A, __B),`。
- **L89 EN**: Executes a call or declaration centered on `statement`.
  **L89 CN**: 执行以 `statement` 为核心的调用或声明。
- **L90 EN**: Closes the current lexical scope or compound statement.
  **L90 CN**: 结束当前词法作用域或复合语句块。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L92 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L92 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L93 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_andnot_pd(__mmask8 __U, __m128d __A, __m128d __B) {`.
  **L93 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_andnot_pd(__mmask8 __U, __m128d __A, __m128d __B) {`。
- **L94 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L94 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_andnot_pd(__A, __B),`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_andnot_pd(__A, __B),`。
- **L96 EN**: Executes a call or declaration centered on `statement`.
  **L96 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 97-120

````c
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_mask_andnot_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__U,
                                             (__v8sf)_mm256_andnot_ps(__A, __B),
                                             (__v8sf)__W);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_maskz_andnot_ps(__mmask8 __U, __m256 __A, __m256 __B) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__U,
                                             (__v8sf)_mm256_andnot_ps(__A, __B),
                                             (__v8sf)_mm256_setzero_ps());
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_mask_andnot_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm_andnot_ps(__A, __B),
                                             (__v4sf)__W);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L99 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L99 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L100 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_andnot_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {`.
  **L100 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_andnot_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {`。
- **L101 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,`.
  **L101 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,` 从当前函数返回。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_andnot_ps(__A, __B),`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_andnot_ps(__A, __B),`。
- **L103 EN**: Executes a call or declaration centered on `statement`.
  **L103 CN**: 执行以 `statement` 为核心的调用或声明。
- **L104 EN**: Closes the current lexical scope or compound statement.
  **L104 CN**: 结束当前词法作用域或复合语句块。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L106 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L106 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L107 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_andnot_ps(__mmask8 __U, __m256 __A, __m256 __B) {`.
  **L107 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_andnot_ps(__mmask8 __U, __m256 __A, __m256 __B) {`。
- **L108 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,`.
  **L108 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,` 从当前函数返回。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_andnot_ps(__A, __B),`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_andnot_ps(__A, __B),`。
- **L110 EN**: Executes a call or declaration centered on `statement`.
  **L110 CN**: 执行以 `statement` 为核心的调用或声明。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L113 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L113 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L114 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_andnot_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {`.
  **L114 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_andnot_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {`。
- **L115 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L115 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm_andnot_ps(__A, __B),`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm_andnot_ps(__A, __B),`。
- **L117 EN**: Executes a call or declaration centered on `statement`.
  **L117 CN**: 执行以 `statement` 为核心的调用或声明。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L120 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L120 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。

### Lines 121-144

````c
_mm_maskz_andnot_ps(__mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm_andnot_ps(__A, __B),
                                             (__v4sf)_mm_setzero_ps());
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_mask_and_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_and_pd(__A, __B),
                                              (__v4df)__W);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_maskz_and_pd(__mmask8 __U, __m256d __A, __m256d __B) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_and_pd(__A, __B),
                                              (__v4df)_mm256_setzero_pd());
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_mask_and_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_and_pd(__A, __B),
````
- **L121 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_andnot_ps(__mmask8 __U, __m128 __A, __m128 __B) {`.
  **L121 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_andnot_ps(__mmask8 __U, __m128 __A, __m128 __B) {`。
- **L122 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L122 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm_andnot_ps(__A, __B),`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm_andnot_ps(__A, __B),`。
- **L124 EN**: Executes a call or declaration centered on `statement`.
  **L124 CN**: 执行以 `statement` 为核心的调用或声明。
- **L125 EN**: Closes the current lexical scope or compound statement.
  **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L127 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L127 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L128 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_and_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {`.
  **L128 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_and_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {`。
- **L129 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L129 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_and_pd(__A, __B),`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_and_pd(__A, __B),`。
- **L131 EN**: Executes a call or declaration centered on `statement`.
  **L131 CN**: 执行以 `statement` 为核心的调用或声明。
- **L132 EN**: Closes the current lexical scope or compound statement.
  **L132 CN**: 结束当前词法作用域或复合语句块。
- **L133 EN**: Blank line separating nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L134 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L134 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L135 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_and_pd(__mmask8 __U, __m256d __A, __m256d __B) {`.
  **L135 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_and_pd(__mmask8 __U, __m256d __A, __m256d __B) {`。
- **L136 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L136 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_and_pd(__A, __B),`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_and_pd(__A, __B),`。
- **L138 EN**: Executes a call or declaration centered on `statement`.
  **L138 CN**: 执行以 `statement` 为核心的调用或声明。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L141 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L141 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_and_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {`.
  **L142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_and_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {`。
- **L143 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L143 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_and_pd(__A, __B),`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_and_pd(__A, __B),`。

### Lines 145-168

````c
                                              (__v2df)__W);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_maskz_and_pd(__mmask8 __U, __m128d __A, __m128d __B) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_and_pd(__A, __B),
                                              (__v2df)_mm_setzero_pd());
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_mask_and_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__U,
                                             (__v8sf)_mm256_and_ps(__A, __B),
                                             (__v8sf)__W);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_maskz_and_ps(__mmask8 __U, __m256 __A, __m256 __B) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__U,
                                             (__v8sf)_mm256_and_ps(__A, __B),
                                             (__v8sf)_mm256_setzero_ps());
}

````
- **L145 EN**: Executes a call or declaration centered on `statement`.
  **L145 CN**: 执行以 `statement` 为核心的调用或声明。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L148 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_and_pd(__mmask8 __U, __m128d __A, __m128d __B) {`.
  **L149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_and_pd(__mmask8 __U, __m128d __A, __m128d __B) {`。
- **L150 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L150 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_and_pd(__A, __B),`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_and_pd(__A, __B),`。
- **L152 EN**: Executes a call or declaration centered on `statement`.
  **L152 CN**: 执行以 `statement` 为核心的调用或声明。
- **L153 EN**: Closes the current lexical scope or compound statement.
  **L153 CN**: 结束当前词法作用域或复合语句块。
- **L154 EN**: Blank line separating nearby declarations or logic blocks.
  **L154 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L155 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L155 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_and_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {`.
  **L156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_and_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {`。
- **L157 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,`.
  **L157 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,` 从当前函数返回。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_and_ps(__A, __B),`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_and_ps(__A, __B),`。
- **L159 EN**: Executes a call or declaration centered on `statement`.
  **L159 CN**: 执行以 `statement` 为核心的调用或声明。
- **L160 EN**: Closes the current lexical scope or compound statement.
  **L160 CN**: 结束当前词法作用域或复合语句块。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L162 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_and_ps(__mmask8 __U, __m256 __A, __m256 __B) {`.
  **L163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_and_ps(__mmask8 __U, __m256 __A, __m256 __B) {`。
- **L164 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,`.
  **L164 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,` 从当前函数返回。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_and_ps(__A, __B),`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_and_ps(__A, __B),`。
- **L166 EN**: Executes a call or declaration centered on `statement`.
  **L166 CN**: 执行以 `statement` 为核心的调用或声明。
- **L167 EN**: Closes the current lexical scope or compound statement.
  **L167 CN**: 结束当前词法作用域或复合语句块。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 169-192

````c
static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_mask_and_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm_and_ps(__A, __B),
                                             (__v4sf)__W);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_maskz_and_ps(__mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm_and_ps(__A, __B),
                                             (__v4sf)_mm_setzero_ps());
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_mask_xor_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_xor_pd(__A, __B),
                                              (__v4df)__W);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_maskz_xor_pd(__mmask8 __U, __m256d __A, __m256d __B) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
````
- **L169 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L169 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L170 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_and_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {`.
  **L170 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_and_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {`。
- **L171 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L171 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm_and_ps(__A, __B),`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm_and_ps(__A, __B),`。
- **L173 EN**: Executes a call or declaration centered on `statement`.
  **L173 CN**: 执行以 `statement` 为核心的调用或声明。
- **L174 EN**: Closes the current lexical scope or compound statement.
  **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Blank line separating nearby declarations or logic blocks.
  **L175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L176 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L176 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L177 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_and_ps(__mmask8 __U, __m128 __A, __m128 __B) {`.
  **L177 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_and_ps(__mmask8 __U, __m128 __A, __m128 __B) {`。
- **L178 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L178 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm_and_ps(__A, __B),`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm_and_ps(__A, __B),`。
- **L180 EN**: Executes a call or declaration centered on `statement`.
  **L180 CN**: 执行以 `statement` 为核心的调用或声明。
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L183 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L183 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L184 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_xor_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {`.
  **L184 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_xor_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {`。
- **L185 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L185 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_xor_pd(__A, __B),`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_xor_pd(__A, __B),`。
- **L187 EN**: Executes a call or declaration centered on `statement`.
  **L187 CN**: 执行以 `statement` 为核心的调用或声明。
- **L188 EN**: Closes the current lexical scope or compound statement.
  **L188 CN**: 结束当前词法作用域或复合语句块。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L190 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L190 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L191 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_xor_pd(__mmask8 __U, __m256d __A, __m256d __B) {`.
  **L191 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_xor_pd(__mmask8 __U, __m256d __A, __m256d __B) {`。
- **L192 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L192 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。

### Lines 193-216

````c
                                              (__v4df)_mm256_xor_pd(__A, __B),
                                              (__v4df)_mm256_setzero_pd());
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_mask_xor_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_xor_pd(__A, __B),
                                              (__v2df)__W);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_maskz_xor_pd (__mmask8 __U, __m128d __A, __m128d __B) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_xor_pd(__A, __B),
                                              (__v2df)_mm_setzero_pd());
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_mask_xor_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__U,
                                             (__v8sf)_mm256_xor_ps(__A, __B),
                                             (__v8sf)__W);
}
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_xor_pd(__A, __B),`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_xor_pd(__A, __B),`。
- **L194 EN**: Executes a call or declaration centered on `statement`.
  **L194 CN**: 执行以 `statement` 为核心的调用或声明。
- **L195 EN**: Closes the current lexical scope or compound statement.
  **L195 CN**: 结束当前词法作用域或复合语句块。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L197 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L197 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L198 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_xor_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {`.
  **L198 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_xor_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {`。
- **L199 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L199 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_xor_pd(__A, __B),`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_xor_pd(__A, __B),`。
- **L201 EN**: Executes a call or declaration centered on `statement`.
  **L201 CN**: 执行以 `statement` 为核心的调用或声明。
- **L202 EN**: Closes the current lexical scope or compound statement.
  **L202 CN**: 结束当前词法作用域或复合语句块。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L204 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L204 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L205 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_xor_pd (__mmask8 __U, __m128d __A, __m128d __B) {`.
  **L205 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_xor_pd (__mmask8 __U, __m128d __A, __m128d __B) {`。
- **L206 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L206 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_xor_pd(__A, __B),`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_xor_pd(__A, __B),`。
- **L208 EN**: Executes a call or declaration centered on `statement`.
  **L208 CN**: 执行以 `statement` 为核心的调用或声明。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L211 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L211 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L212 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_xor_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {`.
  **L212 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_xor_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {`。
- **L213 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,`.
  **L213 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,` 从当前函数返回。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_xor_ps(__A, __B),`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_xor_ps(__A, __B),`。
- **L215 EN**: Executes a call or declaration centered on `statement`.
  **L215 CN**: 执行以 `statement` 为核心的调用或声明。
- **L216 EN**: Closes the current lexical scope or compound statement.
  **L216 CN**: 结束当前词法作用域或复合语句块。

### Lines 217-240

````c

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_maskz_xor_ps(__mmask8 __U, __m256 __A, __m256 __B) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__U,
                                             (__v8sf)_mm256_xor_ps(__A, __B),
                                             (__v8sf)_mm256_setzero_ps());
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_mask_xor_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm_xor_ps(__A, __B),
                                             (__v4sf)__W);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_maskz_xor_ps(__mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm_xor_ps(__A, __B),
                                             (__v4sf)_mm_setzero_ps());
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_mask_or_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {
````
- **L217 EN**: Blank line separating nearby declarations or logic blocks.
  **L217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L218 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L218 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L219 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_xor_ps(__mmask8 __U, __m256 __A, __m256 __B) {`.
  **L219 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_xor_ps(__mmask8 __U, __m256 __A, __m256 __B) {`。
- **L220 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,`.
  **L220 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,` 从当前函数返回。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_xor_ps(__A, __B),`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_xor_ps(__A, __B),`。
- **L222 EN**: Executes a call or declaration centered on `statement`.
  **L222 CN**: 执行以 `statement` 为核心的调用或声明。
- **L223 EN**: Closes the current lexical scope or compound statement.
  **L223 CN**: 结束当前词法作用域或复合语句块。
- **L224 EN**: Blank line separating nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L225 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L225 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L226 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_xor_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {`.
  **L226 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_xor_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {`。
- **L227 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L227 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm_xor_ps(__A, __B),`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm_xor_ps(__A, __B),`。
- **L229 EN**: Executes a call or declaration centered on `statement`.
  **L229 CN**: 执行以 `statement` 为核心的调用或声明。
- **L230 EN**: Closes the current lexical scope or compound statement.
  **L230 CN**: 结束当前词法作用域或复合语句块。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L232 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L232 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L233 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_xor_ps(__mmask8 __U, __m128 __A, __m128 __B) {`.
  **L233 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_xor_ps(__mmask8 __U, __m128 __A, __m128 __B) {`。
- **L234 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L234 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm_xor_ps(__A, __B),`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm_xor_ps(__A, __B),`。
- **L236 EN**: Executes a call or declaration centered on `statement`.
  **L236 CN**: 执行以 `statement` 为核心的调用或声明。
- **L237 EN**: Closes the current lexical scope or compound statement.
  **L237 CN**: 结束当前词法作用域或复合语句块。
- **L238 EN**: Blank line separating nearby declarations or logic blocks.
  **L238 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L239 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L239 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L240 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_or_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {`.
  **L240 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_or_pd(__m256d __W, __mmask8 __U, __m256d __A, __m256d __B) {`。

### Lines 241-264

````c
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_or_pd(__A, __B),
                                              (__v4df)__W);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256
_mm256_maskz_or_pd(__mmask8 __U, __m256d __A, __m256d __B) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_or_pd(__A, __B),
                                              (__v4df)_mm256_setzero_pd());
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_mask_or_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_or_pd(__A, __B),
                                              (__v2df)__W);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128
_mm_maskz_or_pd(__mmask8 __U, __m128d __A, __m128d __B) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_or_pd(__A, __B),
                                              (__v2df)_mm_setzero_pd());
````
- **L241 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L241 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_or_pd(__A, __B),`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_or_pd(__A, __B),`。
- **L243 EN**: Executes a call or declaration centered on `statement`.
  **L243 CN**: 执行以 `statement` 为核心的调用或声明。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L246 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256`.
  **L246 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256`。
- **L247 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_or_pd(__mmask8 __U, __m256d __A, __m256d __B) {`.
  **L247 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_or_pd(__mmask8 __U, __m256d __A, __m256d __B) {`。
- **L248 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L248 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_or_pd(__A, __B),`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_or_pd(__A, __B),`。
- **L250 EN**: Executes a call or declaration centered on `statement`.
  **L250 CN**: 执行以 `statement` 为核心的调用或声明。
- **L251 EN**: Closes the current lexical scope or compound statement.
  **L251 CN**: 结束当前词法作用域或复合语句块。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L253 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L253 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L254 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_or_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {`.
  **L254 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_or_pd(__m128d __W, __mmask8 __U, __m128d __A, __m128d __B) {`。
- **L255 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L255 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_or_pd(__A, __B),`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_or_pd(__A, __B),`。
- **L257 EN**: Executes a call or declaration centered on `statement`.
  **L257 CN**: 执行以 `statement` 为核心的调用或声明。
- **L258 EN**: Closes the current lexical scope or compound statement.
  **L258 CN**: 结束当前词法作用域或复合语句块。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L260 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128`.
  **L260 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128`。
- **L261 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_or_pd(__mmask8 __U, __m128d __A, __m128d __B) {`.
  **L261 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_or_pd(__mmask8 __U, __m128d __A, __m128d __B) {`。
- **L262 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L262 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_or_pd(__A, __B),`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_or_pd(__A, __B),`。
- **L264 EN**: Executes a call or declaration centered on `statement`.
  **L264 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 265-288

````c
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_mask_or_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__U,
                                             (__v8sf)_mm256_or_ps(__A, __B),
                                             (__v8sf)__W);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256
_mm256_maskz_or_ps(__mmask8 __U, __m256 __A, __m256 __B) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__U,
                                             (__v8sf)_mm256_or_ps(__A, __B),
                                             (__v8sf)_mm256_setzero_ps());
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_mask_or_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm_or_ps(__A, __B),
                                             (__v4sf)__W);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
````
- **L265 EN**: Closes the current lexical scope or compound statement.
  **L265 CN**: 结束当前词法作用域或复合语句块。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L267 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L267 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L268 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_or_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {`.
  **L268 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_or_ps(__m256 __W, __mmask8 __U, __m256 __A, __m256 __B) {`。
- **L269 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,`.
  **L269 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,` 从当前函数返回。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_or_ps(__A, __B),`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_or_ps(__A, __B),`。
- **L271 EN**: Executes a call or declaration centered on `statement`.
  **L271 CN**: 执行以 `statement` 为核心的调用或声明。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Blank line separating nearby declarations or logic blocks.
  **L273 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L274 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256`.
  **L274 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256`。
- **L275 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_or_ps(__mmask8 __U, __m256 __A, __m256 __B) {`.
  **L275 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_or_ps(__mmask8 __U, __m256 __A, __m256 __B) {`。
- **L276 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,`.
  **L276 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__U,` 从当前函数返回。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_or_ps(__A, __B),`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_or_ps(__A, __B),`。
- **L278 EN**: Executes a call or declaration centered on `statement`.
  **L278 CN**: 执行以 `statement` 为核心的调用或声明。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L281 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L281 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L282 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_or_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {`.
  **L282 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_or_ps(__m128 __W, __mmask8 __U, __m128 __A, __m128 __B) {`。
- **L283 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L283 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm_or_ps(__A, __B),`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm_or_ps(__A, __B),`。
- **L285 EN**: Executes a call or declaration centered on `statement`.
  **L285 CN**: 执行以 `statement` 为核心的调用或声明。
- **L286 EN**: Closes the current lexical scope or compound statement.
  **L286 CN**: 结束当前词法作用域或复合语句块。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L288 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L288 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。

### Lines 289-312

````c
_mm_maskz_or_ps(__mmask8 __U, __m128 __A, __m128 __B) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm_or_ps(__A, __B),
                                             (__v4sf)_mm_setzero_ps());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtpd_epi64 (__m128d __A) {
  return (__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtpd_epi64 (__m128i __W, __mmask8 __U, __m128d __A) {
  return (__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,
                (__v2di) __W,
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtpd_epi64 (__mmask8 __U, __m128d __A) {
  return (__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,
                (__v2di) _mm_setzero_si128(),
````
- **L289 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_or_ps(__mmask8 __U, __m128 __A, __m128 __B) {`.
  **L289 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_or_ps(__mmask8 __U, __m128 __A, __m128 __B) {`。
- **L290 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L290 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm_or_ps(__A, __B),`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm_or_ps(__A, __B),`。
- **L292 EN**: Executes a call or declaration centered on `statement`.
  **L292 CN**: 执行以 `statement` 为核心的调用或声明。
- **L293 EN**: Closes the current lexical scope or compound statement.
  **L293 CN**: 结束当前词法作用域或复合语句块。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L295 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L295 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L296 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpd_epi64 (__m128d __A) {`.
  **L296 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpd_epi64 (__m128d __A) {`。
- **L297 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,`.
  **L297 CN**: 以 `(__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,` 从当前函数返回。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L299 EN**: Executes a call or declaration centered on `statement`.
  **L299 CN**: 执行以 `statement` 为核心的调用或声明。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L302 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L302 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L303 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtpd_epi64 (__m128i __W, __mmask8 __U, __m128d __A) {`.
  **L303 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtpd_epi64 (__m128i __W, __mmask8 __U, __m128d __A) {`。
- **L304 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,`.
  **L304 CN**: 以 `(__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,` 从当前函数返回。
- **L305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) __W,`.
  **L305 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) __W,`。
- **L306 EN**: Executes a call or declaration centered on `statement`.
  **L306 CN**: 执行以 `statement` 为核心的调用或声明。
- **L307 EN**: Closes the current lexical scope or compound statement.
  **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L309 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L309 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L310 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtpd_epi64 (__mmask8 __U, __m128d __A) {`.
  **L310 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtpd_epi64 (__mmask8 __U, __m128d __A) {`。
- **L311 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,`.
  **L311 CN**: 以 `(__m128i) __builtin_ia32_cvtpd2qq128_mask ((__v2df) __A,` 从当前函数返回。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。

### Lines 313-336

````c
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtpd_epi64 (__m256d __A) {
  return (__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) -1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtpd_epi64 (__m256i __W, __mmask8 __U, __m256d __A) {
  return (__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,
                (__v4di) __W,
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtpd_epi64 (__mmask8 __U, __m256d __A) {
  return (__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) __U);
}

````
- **L313 EN**: Executes a call or declaration centered on `statement`.
  **L313 CN**: 执行以 `statement` 为核心的调用或声明。
- **L314 EN**: Closes the current lexical scope or compound statement.
  **L314 CN**: 结束当前词法作用域或复合语句块。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L316 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L316 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L317 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtpd_epi64 (__m256d __A) {`.
  **L317 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtpd_epi64 (__m256d __A) {`。
- **L318 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,`.
  **L318 CN**: 以 `(__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,` 从当前函数返回。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L320 EN**: Executes a call or declaration centered on `statement`.
  **L320 CN**: 执行以 `statement` 为核心的调用或声明。
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L323 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L324 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtpd_epi64 (__m256i __W, __mmask8 __U, __m256d __A) {`.
  **L324 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtpd_epi64 (__m256i __W, __mmask8 __U, __m256d __A) {`。
- **L325 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,`.
  **L325 CN**: 以 `(__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,` 从当前函数返回。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) __W,`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) __W,`。
- **L327 EN**: Executes a call or declaration centered on `statement`.
  **L327 CN**: 执行以 `statement` 为核心的调用或声明。
- **L328 EN**: Closes the current lexical scope or compound statement.
  **L328 CN**: 结束当前词法作用域或复合语句块。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L330 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L330 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L331 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtpd_epi64 (__mmask8 __U, __m256d __A) {`.
  **L331 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtpd_epi64 (__mmask8 __U, __m256d __A) {`。
- **L332 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,`.
  **L332 CN**: 以 `(__m256i) __builtin_ia32_cvtpd2qq256_mask ((__v4df) __A,` 从当前函数返回。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L334 EN**: Executes a call or declaration centered on `statement`.
  **L334 CN**: 执行以 `statement` 为核心的调用或声明。
- **L335 EN**: Closes the current lexical scope or compound statement.
  **L335 CN**: 结束当前词法作用域或复合语句块。
- **L336 EN**: Blank line separating nearby declarations or logic blocks.
  **L336 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 337-360

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtpd_epu64 (__m128d __A) {
  return (__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtpd_epu64 (__m128i __W, __mmask8 __U, __m128d __A) {
  return (__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,
                (__v2di) __W,
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtpd_epu64 (__mmask8 __U, __m128d __A) {
  return (__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtpd_epu64 (__m256d __A) {
  return (__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,
````
- **L337 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L337 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L338 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtpd_epu64 (__m128d __A) {`.
  **L338 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtpd_epu64 (__m128d __A) {`。
- **L339 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,`.
  **L339 CN**: 以 `(__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,` 从当前函数返回。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L341 EN**: Executes a call or declaration centered on `statement`.
  **L341 CN**: 执行以 `statement` 为核心的调用或声明。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L344 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L344 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L345 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtpd_epu64 (__m128i __W, __mmask8 __U, __m128d __A) {`.
  **L345 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtpd_epu64 (__m128i __W, __mmask8 __U, __m128d __A) {`。
- **L346 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,`.
  **L346 CN**: 以 `(__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,` 从当前函数返回。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) __W,`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) __W,`。
- **L348 EN**: Executes a call or declaration centered on `statement`.
  **L348 CN**: 执行以 `statement` 为核心的调用或声明。
- **L349 EN**: Closes the current lexical scope or compound statement.
  **L349 CN**: 结束当前词法作用域或复合语句块。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L351 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L351 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L352 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtpd_epu64 (__mmask8 __U, __m128d __A) {`.
  **L352 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtpd_epu64 (__mmask8 __U, __m128d __A) {`。
- **L353 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,`.
  **L353 CN**: 以 `(__m128i) __builtin_ia32_cvtpd2uqq128_mask ((__v2df) __A,` 从当前函数返回。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L355 EN**: Executes a call or declaration centered on `statement`.
  **L355 CN**: 执行以 `statement` 为核心的调用或声明。
- **L356 EN**: Closes the current lexical scope or compound statement.
  **L356 CN**: 结束当前词法作用域或复合语句块。
- **L357 EN**: Blank line separating nearby declarations or logic blocks.
  **L357 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L358 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L358 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L359 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtpd_epu64 (__m256d __A) {`.
  **L359 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtpd_epu64 (__m256d __A) {`。
- **L360 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,`.
  **L360 CN**: 以 `(__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,` 从当前函数返回。

### Lines 361-384

````c
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) -1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtpd_epu64 (__m256i __W, __mmask8 __U, __m256d __A) {
  return (__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,
                (__v4di) __W,
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtpd_epu64 (__mmask8 __U, __m256d __A) {
  return (__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtps_epi64 (__m128 __A) {
  return (__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) -1);
}
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L362 EN**: Executes a call or declaration centered on `statement`.
  **L362 CN**: 执行以 `statement` 为核心的调用或声明。
- **L363 EN**: Closes the current lexical scope or compound statement.
  **L363 CN**: 结束当前词法作用域或复合语句块。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L365 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L365 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L366 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtpd_epu64 (__m256i __W, __mmask8 __U, __m256d __A) {`.
  **L366 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtpd_epu64 (__m256i __W, __mmask8 __U, __m256d __A) {`。
- **L367 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,`.
  **L367 CN**: 以 `(__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,` 从当前函数返回。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) __W,`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) __W,`。
- **L369 EN**: Executes a call or declaration centered on `statement`.
  **L369 CN**: 执行以 `statement` 为核心的调用或声明。
- **L370 EN**: Closes the current lexical scope or compound statement.
  **L370 CN**: 结束当前词法作用域或复合语句块。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L372 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L372 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L373 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtpd_epu64 (__mmask8 __U, __m256d __A) {`.
  **L373 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtpd_epu64 (__mmask8 __U, __m256d __A) {`。
- **L374 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,`.
  **L374 CN**: 以 `(__m256i) __builtin_ia32_cvtpd2uqq256_mask ((__v4df) __A,` 从当前函数返回。
- **L375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L375 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L376 EN**: Executes a call or declaration centered on `statement`.
  **L376 CN**: 执行以 `statement` 为核心的调用或声明。
- **L377 EN**: Closes the current lexical scope or compound statement.
  **L377 CN**: 结束当前词法作用域或复合语句块。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L379 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L379 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L380 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtps_epi64 (__m128 __A) {`.
  **L380 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtps_epi64 (__m128 __A) {`。
- **L381 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,`.
  **L381 CN**: 以 `(__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L383 EN**: Executes a call or declaration centered on `statement`.
  **L383 CN**: 执行以 `statement` 为核心的调用或声明。
- **L384 EN**: Closes the current lexical scope or compound statement.
  **L384 CN**: 结束当前词法作用域或复合语句块。

### Lines 385-408

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtps_epi64 (__m128i __W, __mmask8 __U, __m128 __A) {
  return (__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,
                (__v2di) __W,
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtps_epi64 (__mmask8 __U, __m128 __A) {
  return (__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtps_epi64 (__m128 __A) {
  return (__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) -1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtps_epi64 (__m256i __W, __mmask8 __U, __m128 __A) {
````
- **L385 EN**: Blank line separating nearby declarations or logic blocks.
  **L385 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L386 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L386 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L387 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtps_epi64 (__m128i __W, __mmask8 __U, __m128 __A) {`.
  **L387 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtps_epi64 (__m128i __W, __mmask8 __U, __m128 __A) {`。
- **L388 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,`.
  **L388 CN**: 以 `(__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) __W,`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) __W,`。
- **L390 EN**: Executes a call or declaration centered on `statement`.
  **L390 CN**: 执行以 `statement` 为核心的调用或声明。
- **L391 EN**: Closes the current lexical scope or compound statement.
  **L391 CN**: 结束当前词法作用域或复合语句块。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L393 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L393 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L394 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtps_epi64 (__mmask8 __U, __m128 __A) {`.
  **L394 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtps_epi64 (__mmask8 __U, __m128 __A) {`。
- **L395 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,`.
  **L395 CN**: 以 `(__m128i) __builtin_ia32_cvtps2qq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L397 EN**: Executes a call or declaration centered on `statement`.
  **L397 CN**: 执行以 `statement` 为核心的调用或声明。
- **L398 EN**: Closes the current lexical scope or compound statement.
  **L398 CN**: 结束当前词法作用域或复合语句块。
- **L399 EN**: Blank line separating nearby declarations or logic blocks.
  **L399 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L400 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L400 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L401 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtps_epi64 (__m128 __A) {`.
  **L401 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtps_epi64 (__m128 __A) {`。
- **L402 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,`.
  **L402 CN**: 以 `(__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L404 EN**: Executes a call or declaration centered on `statement`.
  **L404 CN**: 执行以 `statement` 为核心的调用或声明。
- **L405 EN**: Closes the current lexical scope or compound statement.
  **L405 CN**: 结束当前词法作用域或复合语句块。
- **L406 EN**: Blank line separating nearby declarations or logic blocks.
  **L406 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L407 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L407 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L408 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtps_epi64 (__m256i __W, __mmask8 __U, __m128 __A) {`.
  **L408 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtps_epi64 (__m256i __W, __mmask8 __U, __m128 __A) {`。

### Lines 409-432

````c
  return (__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,
                (__v4di) __W,
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtps_epi64 (__mmask8 __U, __m128 __A) {
  return (__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtps_epu64 (__m128 __A) {
  return (__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtps_epu64 (__m128i __W, __mmask8 __U, __m128 __A) {
  return (__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,
                (__v2di) __W,
                (__mmask8) __U);
````
- **L409 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,`.
  **L409 CN**: 以 `(__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) __W,`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) __W,`。
- **L411 EN**: Executes a call or declaration centered on `statement`.
  **L411 CN**: 执行以 `statement` 为核心的调用或声明。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L414 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L415 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtps_epi64 (__mmask8 __U, __m128 __A) {`.
  **L415 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtps_epi64 (__mmask8 __U, __m128 __A) {`。
- **L416 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,`.
  **L416 CN**: 以 `(__m256i) __builtin_ia32_cvtps2qq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L418 EN**: Executes a call or declaration centered on `statement`.
  **L418 CN**: 执行以 `statement` 为核心的调用或声明。
- **L419 EN**: Closes the current lexical scope or compound statement.
  **L419 CN**: 结束当前词法作用域或复合语句块。
- **L420 EN**: Blank line separating nearby declarations or logic blocks.
  **L420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L421 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L421 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L422 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtps_epu64 (__m128 __A) {`.
  **L422 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtps_epu64 (__m128 __A) {`。
- **L423 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,`.
  **L423 CN**: 以 `(__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L424 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L425 EN**: Executes a call or declaration centered on `statement`.
  **L425 CN**: 执行以 `statement` 为核心的调用或声明。
- **L426 EN**: Closes the current lexical scope or compound statement.
  **L426 CN**: 结束当前词法作用域或复合语句块。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L428 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L428 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L429 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtps_epu64 (__m128i __W, __mmask8 __U, __m128 __A) {`.
  **L429 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtps_epu64 (__m128i __W, __mmask8 __U, __m128 __A) {`。
- **L430 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,`.
  **L430 CN**: 以 `(__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) __W,`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) __W,`。
- **L432 EN**: Executes a call or declaration centered on `statement`.
  **L432 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 433-456

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtps_epu64 (__mmask8 __U, __m128 __A) {
  return (__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtps_epu64 (__m128 __A) {
  return (__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) -1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtps_epu64 (__m256i __W, __mmask8 __U, __m128 __A) {
  return (__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,
                (__v4di) __W,
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
````
- **L433 EN**: Closes the current lexical scope or compound statement.
  **L433 CN**: 结束当前词法作用域或复合语句块。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L435 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L435 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L436 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtps_epu64 (__mmask8 __U, __m128 __A) {`.
  **L436 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtps_epu64 (__mmask8 __U, __m128 __A) {`。
- **L437 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,`.
  **L437 CN**: 以 `(__m128i) __builtin_ia32_cvtps2uqq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L439 EN**: Executes a call or declaration centered on `statement`.
  **L439 CN**: 执行以 `statement` 为核心的调用或声明。
- **L440 EN**: Closes the current lexical scope or compound statement.
  **L440 CN**: 结束当前词法作用域或复合语句块。
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L442 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L442 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L443 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtps_epu64 (__m128 __A) {`.
  **L443 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtps_epu64 (__m128 __A) {`。
- **L444 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,`.
  **L444 CN**: 以 `(__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L446 EN**: Executes a call or declaration centered on `statement`.
  **L446 CN**: 执行以 `statement` 为核心的调用或声明。
- **L447 EN**: Closes the current lexical scope or compound statement.
  **L447 CN**: 结束当前词法作用域或复合语句块。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L449 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L449 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L450 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtps_epu64 (__m256i __W, __mmask8 __U, __m128 __A) {`.
  **L450 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtps_epu64 (__m256i __W, __mmask8 __U, __m128 __A) {`。
- **L451 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,`.
  **L451 CN**: 以 `(__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) __W,`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) __W,`。
- **L453 EN**: Executes a call or declaration centered on `statement`.
  **L453 CN**: 执行以 `statement` 为核心的调用或声明。
- **L454 EN**: Closes the current lexical scope or compound statement.
  **L454 CN**: 结束当前词法作用域或复合语句块。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L456 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L456 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。

### Lines 457-480

````c
_mm256_maskz_cvtps_epu64 (__mmask8 __U, __m128 __A) {
  return (__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) __U);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_cvtepi64_pd(__m128i __A) {
  return (__m128d)__builtin_convertvector((__v2di)__A, __v2df);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_cvtepi64_pd(__m128d __W, __mmask8 __U, __m128i __A) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_cvtepi64_pd(__A),
                                              (__v2df)__W);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_cvtepi64_pd(__mmask8 __U, __m128i __A) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_cvtepi64_pd(__A),
                                              (__v2df)_mm_setzero_pd());
}
````
- **L457 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtps_epu64 (__mmask8 __U, __m128 __A) {`.
  **L457 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtps_epu64 (__mmask8 __U, __m128 __A) {`。
- **L458 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,`.
  **L458 CN**: 以 `(__m256i) __builtin_ia32_cvtps2uqq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L460 EN**: Executes a call or declaration centered on `statement`.
  **L460 CN**: 执行以 `statement` 为核心的调用或声明。
- **L461 EN**: Closes the current lexical scope or compound statement.
  **L461 CN**: 结束当前词法作用域或复合语句块。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L463 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L463 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L464 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi64_pd(__m128i __A) {`.
  **L464 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi64_pd(__m128i __A) {`。
- **L465 EN**: Returns from the current function with `(__m128d)__builtin_convertvector((__v2di)__A, __v2df)`.
  **L465 CN**: 以 `(__m128d)__builtin_convertvector((__v2di)__A, __v2df)` 从当前函数返回。
- **L466 EN**: Closes the current lexical scope or compound statement.
  **L466 CN**: 结束当前词法作用域或复合语句块。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L468 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L469 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepi64_pd(__m128d __W, __mmask8 __U, __m128i __A) {`.
  **L469 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepi64_pd(__m128d __W, __mmask8 __U, __m128i __A) {`。
- **L470 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L470 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_cvtepi64_pd(__A),`.
  **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_cvtepi64_pd(__A),`。
- **L472 EN**: Executes a call or declaration centered on `statement`.
  **L472 CN**: 执行以 `statement` 为核心的调用或声明。
- **L473 EN**: Closes the current lexical scope or compound statement.
  **L473 CN**: 结束当前词法作用域或复合语句块。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L475 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L476 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepi64_pd(__mmask8 __U, __m128i __A) {`.
  **L476 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepi64_pd(__mmask8 __U, __m128i __A) {`。
- **L477 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L477 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_cvtepi64_pd(__A),`.
  **L478 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_cvtepi64_pd(__A),`。
- **L479 EN**: Executes a call or declaration centered on `statement`.
  **L479 CN**: 执行以 `statement` 为核心的调用或声明。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-504

````c

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtepi64_pd(__m256i __A) {
  return (__m256d)__builtin_convertvector((__v4di)__A, __v4df);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepi64_pd(__m256d __W, __mmask8 __U, __m256i __A) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_cvtepi64_pd(__A),
                                              (__v4df)__W);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepi64_pd(__mmask8 __U, __m256i __A) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_cvtepi64_pd(__A),
                                              (__v4df)_mm256_setzero_pd());
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_cvtepi64_ps (__m128i __A) {
  return (__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,
                (__v4sf) _mm_setzero_ps(),
````
- **L481 EN**: Blank line separating nearby declarations or logic blocks.
  **L481 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L482 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L482 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L483 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepi64_pd(__m256i __A) {`.
  **L483 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepi64_pd(__m256i __A) {`。
- **L484 EN**: Returns from the current function with `(__m256d)__builtin_convertvector((__v4di)__A, __v4df)`.
  **L484 CN**: 以 `(__m256d)__builtin_convertvector((__v4di)__A, __v4df)` 从当前函数返回。
- **L485 EN**: Closes the current lexical scope or compound statement.
  **L485 CN**: 结束当前词法作用域或复合语句块。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L487 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L487 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L488 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepi64_pd(__m256d __W, __mmask8 __U, __m256i __A) {`.
  **L488 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepi64_pd(__m256d __W, __mmask8 __U, __m256i __A) {`。
- **L489 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L489 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_cvtepi64_pd(__A),`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_cvtepi64_pd(__A),`。
- **L491 EN**: Executes a call or declaration centered on `statement`.
  **L491 CN**: 执行以 `statement` 为核心的调用或声明。
- **L492 EN**: Closes the current lexical scope or compound statement.
  **L492 CN**: 结束当前词法作用域或复合语句块。
- **L493 EN**: Blank line separating nearby declarations or logic blocks.
  **L493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L494 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L494 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L495 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepi64_pd(__mmask8 __U, __m256i __A) {`.
  **L495 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepi64_pd(__mmask8 __U, __m256i __A) {`。
- **L496 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L496 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_cvtepi64_pd(__A),`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_cvtepi64_pd(__A),`。
- **L498 EN**: Executes a call or declaration centered on `statement`.
  **L498 CN**: 执行以 `statement` 为核心的调用或声明。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Blank line separating nearby declarations or logic blocks.
  **L500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L501 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L501 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L502 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi64_ps (__m128i __A) {`.
  **L502 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi64_ps (__m128i __A) {`。
- **L503 EN**: Returns from the current function with `(__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,`.
  **L503 CN**: 以 `(__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,` 从当前函数返回。
- **L504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf) _mm_setzero_ps(),`.
  **L504 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf) _mm_setzero_ps(),`。

### Lines 505-528

````c
                (__mmask8) -1);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_mask_cvtepi64_ps (__m128 __W, __mmask8 __U, __m128i __A) {
  return (__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,
                (__v4sf) __W,
                (__mmask8) __U);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_maskz_cvtepi64_ps (__mmask8 __U, __m128i __A) {
  return (__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,
                (__v4sf) _mm_setzero_ps(),
                (__mmask8) __U);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtepi64_ps(__m256i __A) {
  return (__m128)__builtin_convertvector((__v4di)__A, __v4sf);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepi64_ps(__m128 __W, __mmask8 __U, __m256i __A) {
````
- **L505 EN**: Executes a call or declaration centered on `statement`.
  **L505 CN**: 执行以 `statement` 为核心的调用或声明。
- **L506 EN**: Closes the current lexical scope or compound statement.
  **L506 CN**: 结束当前词法作用域或复合语句块。
- **L507 EN**: Blank line separating nearby declarations or logic blocks.
  **L507 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L508 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L508 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L509 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepi64_ps (__m128 __W, __mmask8 __U, __m128i __A) {`.
  **L509 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepi64_ps (__m128 __W, __mmask8 __U, __m128i __A) {`。
- **L510 EN**: Returns from the current function with `(__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,`.
  **L510 CN**: 以 `(__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,` 从当前函数返回。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf) __W,`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf) __W,`。
- **L512 EN**: Executes a call or declaration centered on `statement`.
  **L512 CN**: 执行以 `statement` 为核心的调用或声明。
- **L513 EN**: Closes the current lexical scope or compound statement.
  **L513 CN**: 结束当前词法作用域或复合语句块。
- **L514 EN**: Blank line separating nearby declarations or logic blocks.
  **L514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L515 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L515 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L516 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepi64_ps (__mmask8 __U, __m128i __A) {`.
  **L516 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepi64_ps (__mmask8 __U, __m128i __A) {`。
- **L517 EN**: Returns from the current function with `(__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,`.
  **L517 CN**: 以 `(__m128) __builtin_ia32_cvtqq2ps128_mask ((__v2di) __A,` 从当前函数返回。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf) _mm_setzero_ps(),`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf) _mm_setzero_ps(),`。
- **L519 EN**: Executes a call or declaration centered on `statement`.
  **L519 CN**: 执行以 `statement` 为核心的调用或声明。
- **L520 EN**: Closes the current lexical scope or compound statement.
  **L520 CN**: 结束当前词法作用域或复合语句块。
- **L521 EN**: Blank line separating nearby declarations or logic blocks.
  **L521 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L522 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L522 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L523 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepi64_ps(__m256i __A) {`.
  **L523 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepi64_ps(__m256i __A) {`。
- **L524 EN**: Returns from the current function with `(__m128)__builtin_convertvector((__v4di)__A, __v4sf)`.
  **L524 CN**: 以 `(__m128)__builtin_convertvector((__v4di)__A, __v4sf)` 从当前函数返回。
- **L525 EN**: Closes the current lexical scope or compound statement.
  **L525 CN**: 结束当前词法作用域或复合语句块。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L527 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L527 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L528 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepi64_ps(__m128 __W, __mmask8 __U, __m256i __A) {`.
  **L528 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepi64_ps(__m128 __W, __mmask8 __U, __m256i __A) {`。

### Lines 529-552

````c
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm256_cvtepi64_ps(__A),
                                             (__v4sf)__W);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepi64_ps(__mmask8 __U, __m256i __A) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm256_cvtepi64_ps(__A),
                                             (__v4sf)_mm_setzero_ps());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvttpd_epi64 (__m128d __A) {
  return (__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttpd_epi64 (__m128i __W, __mmask8 __U, __m128d __A) {
  return (__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,
                (__v2di) __W,
                (__mmask8) __U);
````
- **L529 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L529 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm256_cvtepi64_ps(__A),`.
  **L530 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm256_cvtepi64_ps(__A),`。
- **L531 EN**: Executes a call or declaration centered on `statement`.
  **L531 CN**: 执行以 `statement` 为核心的调用或声明。
- **L532 EN**: Closes the current lexical scope or compound statement.
  **L532 CN**: 结束当前词法作用域或复合语句块。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L534 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L534 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L535 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepi64_ps(__mmask8 __U, __m256i __A) {`.
  **L535 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepi64_ps(__mmask8 __U, __m256i __A) {`。
- **L536 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L536 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm256_cvtepi64_ps(__A),`.
  **L537 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm256_cvtepi64_ps(__A),`。
- **L538 EN**: Executes a call or declaration centered on `statement`.
  **L538 CN**: 执行以 `statement` 为核心的调用或声明。
- **L539 EN**: Closes the current lexical scope or compound statement.
  **L539 CN**: 结束当前词法作用域或复合语句块。
- **L540 EN**: Blank line separating nearby declarations or logic blocks.
  **L540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L541 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L541 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L542 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttpd_epi64 (__m128d __A) {`.
  **L542 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttpd_epi64 (__m128d __A) {`。
- **L543 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,`.
  **L543 CN**: 以 `(__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,` 从当前函数返回。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L545 EN**: Executes a call or declaration centered on `statement`.
  **L545 CN**: 执行以 `statement` 为核心的调用或声明。
- **L546 EN**: Closes the current lexical scope or compound statement.
  **L546 CN**: 结束当前词法作用域或复合语句块。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L548 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L548 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L549 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttpd_epi64 (__m128i __W, __mmask8 __U, __m128d __A) {`.
  **L549 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttpd_epi64 (__m128i __W, __mmask8 __U, __m128d __A) {`。
- **L550 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,`.
  **L550 CN**: 以 `(__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,` 从当前函数返回。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) __W,`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) __W,`。
- **L552 EN**: Executes a call or declaration centered on `statement`.
  **L552 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 553-576

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttpd_epi64 (__mmask8 __U, __m128d __A) {
  return (__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttpd_epi64 (__m256d __A) {
  return (__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) -1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttpd_epi64 (__m256i __W, __mmask8 __U, __m256d __A) {
  return (__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,
                (__v4di) __W,
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L555 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L556 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttpd_epi64 (__mmask8 __U, __m128d __A) {`.
  **L556 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttpd_epi64 (__mmask8 __U, __m128d __A) {`。
- **L557 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,`.
  **L557 CN**: 以 `(__m128i) __builtin_ia32_cvttpd2qq128_mask ((__v2df) __A,` 从当前函数返回。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L559 EN**: Executes a call or declaration centered on `statement`.
  **L559 CN**: 执行以 `statement` 为核心的调用或声明。
- **L560 EN**: Closes the current lexical scope or compound statement.
  **L560 CN**: 结束当前词法作用域或复合语句块。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L562 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L562 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L563 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttpd_epi64 (__m256d __A) {`.
  **L563 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttpd_epi64 (__m256d __A) {`。
- **L564 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,`.
  **L564 CN**: 以 `(__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,` 从当前函数返回。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L566 EN**: Executes a call or declaration centered on `statement`.
  **L566 CN**: 执行以 `statement` 为核心的调用或声明。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L569 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L569 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L570 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttpd_epi64 (__m256i __W, __mmask8 __U, __m256d __A) {`.
  **L570 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttpd_epi64 (__m256i __W, __mmask8 __U, __m256d __A) {`。
- **L571 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,`.
  **L571 CN**: 以 `(__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,` 从当前函数返回。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) __W,`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) __W,`。
- **L573 EN**: Executes a call or declaration centered on `statement`.
  **L573 CN**: 执行以 `statement` 为核心的调用或声明。
- **L574 EN**: Closes the current lexical scope or compound statement.
  **L574 CN**: 结束当前词法作用域或复合语句块。
- **L575 EN**: Blank line separating nearby declarations or logic blocks.
  **L575 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L576 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L576 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。

### Lines 577-600

````c
_mm256_maskz_cvttpd_epi64 (__mmask8 __U, __m256d __A) {
  return (__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvttpd_epu64 (__m128d __A) {
  return (__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttpd_epu64 (__m128i __W, __mmask8 __U, __m128d __A) {
  return (__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,
                (__v2di) __W,
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttpd_epu64 (__mmask8 __U, __m128d __A) {
  return (__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,
                (__v2di) _mm_setzero_si128(),
````
- **L577 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttpd_epi64 (__mmask8 __U, __m256d __A) {`.
  **L577 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttpd_epi64 (__mmask8 __U, __m256d __A) {`。
- **L578 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,`.
  **L578 CN**: 以 `(__m256i) __builtin_ia32_cvttpd2qq256_mask ((__v4df) __A,` 从当前函数返回。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L580 EN**: Executes a call or declaration centered on `statement`.
  **L580 CN**: 执行以 `statement` 为核心的调用或声明。
- **L581 EN**: Closes the current lexical scope or compound statement.
  **L581 CN**: 结束当前词法作用域或复合语句块。
- **L582 EN**: Blank line separating nearby declarations or logic blocks.
  **L582 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L583 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L583 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L584 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttpd_epu64 (__m128d __A) {`.
  **L584 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttpd_epu64 (__m128d __A) {`。
- **L585 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,`.
  **L585 CN**: 以 `(__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,` 从当前函数返回。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L587 EN**: Executes a call or declaration centered on `statement`.
  **L587 CN**: 执行以 `statement` 为核心的调用或声明。
- **L588 EN**: Closes the current lexical scope or compound statement.
  **L588 CN**: 结束当前词法作用域或复合语句块。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L590 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L590 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L591 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttpd_epu64 (__m128i __W, __mmask8 __U, __m128d __A) {`.
  **L591 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttpd_epu64 (__m128i __W, __mmask8 __U, __m128d __A) {`。
- **L592 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,`.
  **L592 CN**: 以 `(__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,` 从当前函数返回。
- **L593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) __W,`.
  **L593 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) __W,`。
- **L594 EN**: Executes a call or declaration centered on `statement`.
  **L594 CN**: 执行以 `statement` 为核心的调用或声明。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L597 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L597 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L598 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttpd_epu64 (__mmask8 __U, __m128d __A) {`.
  **L598 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttpd_epu64 (__mmask8 __U, __m128d __A) {`。
- **L599 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,`.
  **L599 CN**: 以 `(__m128i) __builtin_ia32_cvttpd2uqq128_mask ((__v2df) __A,` 从当前函数返回。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。

### Lines 601-624

````c
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttpd_epu64 (__m256d __A) {
  return (__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) -1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttpd_epu64 (__m256i __W, __mmask8 __U, __m256d __A) {
  return (__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,
                (__v4di) __W,
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttpd_epu64 (__mmask8 __U, __m256d __A) {
  return (__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) __U);
}

````
- **L601 EN**: Executes a call or declaration centered on `statement`.
  **L601 CN**: 执行以 `statement` 为核心的调用或声明。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L604 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L604 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L605 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttpd_epu64 (__m256d __A) {`.
  **L605 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttpd_epu64 (__m256d __A) {`。
- **L606 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,`.
  **L606 CN**: 以 `(__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,` 从当前函数返回。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L608 EN**: Executes a call or declaration centered on `statement`.
  **L608 CN**: 执行以 `statement` 为核心的调用或声明。
- **L609 EN**: Closes the current lexical scope or compound statement.
  **L609 CN**: 结束当前词法作用域或复合语句块。
- **L610 EN**: Blank line separating nearby declarations or logic blocks.
  **L610 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L611 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L611 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L612 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttpd_epu64 (__m256i __W, __mmask8 __U, __m256d __A) {`.
  **L612 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttpd_epu64 (__m256i __W, __mmask8 __U, __m256d __A) {`。
- **L613 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,`.
  **L613 CN**: 以 `(__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,` 从当前函数返回。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) __W,`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) __W,`。
- **L615 EN**: Executes a call or declaration centered on `statement`.
  **L615 CN**: 执行以 `statement` 为核心的调用或声明。
- **L616 EN**: Closes the current lexical scope or compound statement.
  **L616 CN**: 结束当前词法作用域或复合语句块。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L618 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L618 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L619 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttpd_epu64 (__mmask8 __U, __m256d __A) {`.
  **L619 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttpd_epu64 (__mmask8 __U, __m256d __A) {`。
- **L620 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,`.
  **L620 CN**: 以 `(__m256i) __builtin_ia32_cvttpd2uqq256_mask ((__v4df) __A,` 从当前函数返回。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L622 EN**: Executes a call or declaration centered on `statement`.
  **L622 CN**: 执行以 `statement` 为核心的调用或声明。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 625-648

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvttps_epi64 (__m128 __A) {
  return (__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttps_epi64 (__m128i __W, __mmask8 __U, __m128 __A) {
  return (__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,
                (__v2di) __W,
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttps_epi64 (__mmask8 __U, __m128 __A) {
  return (__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttps_epi64 (__m128 __A) {
  return (__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,
````
- **L625 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L625 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L626 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttps_epi64 (__m128 __A) {`.
  **L626 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttps_epi64 (__m128 __A) {`。
- **L627 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,`.
  **L627 CN**: 以 `(__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L629 EN**: Executes a call or declaration centered on `statement`.
  **L629 CN**: 执行以 `statement` 为核心的调用或声明。
- **L630 EN**: Closes the current lexical scope or compound statement.
  **L630 CN**: 结束当前词法作用域或复合语句块。
- **L631 EN**: Blank line separating nearby declarations or logic blocks.
  **L631 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L632 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L632 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L633 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttps_epi64 (__m128i __W, __mmask8 __U, __m128 __A) {`.
  **L633 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttps_epi64 (__m128i __W, __mmask8 __U, __m128 __A) {`。
- **L634 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,`.
  **L634 CN**: 以 `(__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) __W,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) __W,`。
- **L636 EN**: Executes a call or declaration centered on `statement`.
  **L636 CN**: 执行以 `statement` 为核心的调用或声明。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L639 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L639 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L640 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttps_epi64 (__mmask8 __U, __m128 __A) {`.
  **L640 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttps_epi64 (__mmask8 __U, __m128 __A) {`。
- **L641 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,`.
  **L641 CN**: 以 `(__m128i) __builtin_ia32_cvttps2qq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L643 EN**: Executes a call or declaration centered on `statement`.
  **L643 CN**: 执行以 `statement` 为核心的调用或声明。
- **L644 EN**: Closes the current lexical scope or compound statement.
  **L644 CN**: 结束当前词法作用域或复合语句块。
- **L645 EN**: Blank line separating nearby declarations or logic blocks.
  **L645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L646 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L646 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L647 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttps_epi64 (__m128 __A) {`.
  **L647 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttps_epi64 (__m128 __A) {`。
- **L648 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,`.
  **L648 CN**: 以 `(__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,` 从当前函数返回。

### Lines 649-672

````c
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) -1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttps_epi64 (__m256i __W, __mmask8 __U, __m128 __A) {
  return (__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,
                (__v4di) __W,
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttps_epi64 (__mmask8 __U, __m128 __A) {
  return (__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvttps_epu64 (__m128 __A) {
  return (__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) -1);
}
````
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L650 EN**: Executes a call or declaration centered on `statement`.
  **L650 CN**: 执行以 `statement` 为核心的调用或声明。
- **L651 EN**: Closes the current lexical scope or compound statement.
  **L651 CN**: 结束当前词法作用域或复合语句块。
- **L652 EN**: Blank line separating nearby declarations or logic blocks.
  **L652 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L653 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L653 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L654 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttps_epi64 (__m256i __W, __mmask8 __U, __m128 __A) {`.
  **L654 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttps_epi64 (__m256i __W, __mmask8 __U, __m128 __A) {`。
- **L655 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,`.
  **L655 CN**: 以 `(__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) __W,`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) __W,`。
- **L657 EN**: Executes a call or declaration centered on `statement`.
  **L657 CN**: 执行以 `statement` 为核心的调用或声明。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L660 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L660 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L661 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttps_epi64 (__mmask8 __U, __m128 __A) {`.
  **L661 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttps_epi64 (__mmask8 __U, __m128 __A) {`。
- **L662 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,`.
  **L662 CN**: 以 `(__m256i) __builtin_ia32_cvttps2qq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L663 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L664 EN**: Executes a call or declaration centered on `statement`.
  **L664 CN**: 执行以 `statement` 为核心的调用或声明。
- **L665 EN**: Closes the current lexical scope or compound statement.
  **L665 CN**: 结束当前词法作用域或复合语句块。
- **L666 EN**: Blank line separating nearby declarations or logic blocks.
  **L666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L667 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L667 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L668 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvttps_epu64 (__m128 __A) {`.
  **L668 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvttps_epu64 (__m128 __A) {`。
- **L669 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,`.
  **L669 CN**: 以 `(__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L671 EN**: Executes a call or declaration centered on `statement`.
  **L671 CN**: 执行以 `statement` 为核心的调用或声明。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvttps_epu64 (__m128i __W, __mmask8 __U, __m128 __A) {
  return (__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,
                (__v2di) __W,
                (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvttps_epu64 (__mmask8 __U, __m128 __A) {
  return (__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,
                (__v2di) _mm_setzero_si128(),
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvttps_epu64 (__m128 __A) {
  return (__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) -1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvttps_epu64 (__m256i __W, __mmask8 __U, __m128 __A) {
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L674 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L675 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvttps_epu64 (__m128i __W, __mmask8 __U, __m128 __A) {`.
  **L675 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvttps_epu64 (__m128i __W, __mmask8 __U, __m128 __A) {`。
- **L676 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,`.
  **L676 CN**: 以 `(__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) __W,`.
  **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) __W,`。
- **L678 EN**: Executes a call or declaration centered on `statement`.
  **L678 CN**: 执行以 `statement` 为核心的调用或声明。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L681 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L681 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L682 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvttps_epu64 (__mmask8 __U, __m128 __A) {`.
  **L682 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvttps_epu64 (__mmask8 __U, __m128 __A) {`。
- **L683 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,`.
  **L683 CN**: 以 `(__m128i) __builtin_ia32_cvttps2uqq128_mask ((__v4sf) __A,` 从当前函数返回。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di) _mm_setzero_si128(),`.
  **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di) _mm_setzero_si128(),`。
- **L685 EN**: Executes a call or declaration centered on `statement`.
  **L685 CN**: 执行以 `statement` 为核心的调用或声明。
- **L686 EN**: Closes the current lexical scope or compound statement.
  **L686 CN**: 结束当前词法作用域或复合语句块。
- **L687 EN**: Blank line separating nearby declarations or logic blocks.
  **L687 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L688 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L688 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L689 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvttps_epu64 (__m128 __A) {`.
  **L689 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvttps_epu64 (__m128 __A) {`。
- **L690 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,`.
  **L690 CN**: 以 `(__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L692 EN**: Executes a call or declaration centered on `statement`.
  **L692 CN**: 执行以 `statement` 为核心的调用或声明。
- **L693 EN**: Closes the current lexical scope or compound statement.
  **L693 CN**: 结束当前词法作用域或复合语句块。
- **L694 EN**: Blank line separating nearby declarations or logic blocks.
  **L694 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L695 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L695 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L696 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvttps_epu64 (__m256i __W, __mmask8 __U, __m128 __A) {`.
  **L696 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvttps_epu64 (__m256i __W, __mmask8 __U, __m128 __A) {`。

### Lines 697-720

````c
  return (__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,
                (__v4di) __W,
                (__mmask8) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvttps_epu64 (__mmask8 __U, __m128 __A) {
  return (__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,
                (__v4di) _mm256_setzero_si256(),
                (__mmask8) __U);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_cvtepu64_pd(__m128i __A) {
  return (__m128d)__builtin_convertvector((__v2du)__A, __v2df);
}

static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_cvtepu64_pd(__m128d __W, __mmask8 __U, __m128i __A) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_cvtepu64_pd(__A),
                                              (__v2df)__W);
}

````
- **L697 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,`.
  **L697 CN**: 以 `(__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) __W,`.
  **L698 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) __W,`。
- **L699 EN**: Executes a call or declaration centered on `statement`.
  **L699 CN**: 执行以 `statement` 为核心的调用或声明。
- **L700 EN**: Closes the current lexical scope or compound statement.
  **L700 CN**: 结束当前词法作用域或复合语句块。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L702 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L703 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvttps_epu64 (__mmask8 __U, __m128 __A) {`.
  **L703 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvttps_epu64 (__mmask8 __U, __m128 __A) {`。
- **L704 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,`.
  **L704 CN**: 以 `(__m256i) __builtin_ia32_cvttps2uqq256_mask ((__v4sf) __A,` 从当前函数返回。
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di) _mm256_setzero_si256(),`.
  **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di) _mm256_setzero_si256(),`。
- **L706 EN**: Executes a call or declaration centered on `statement`.
  **L706 CN**: 执行以 `statement` 为核心的调用或声明。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L709 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L709 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L710 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu64_pd(__m128i __A) {`.
  **L710 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu64_pd(__m128i __A) {`。
- **L711 EN**: Returns from the current function with `(__m128d)__builtin_convertvector((__v2du)__A, __v2df)`.
  **L711 CN**: 以 `(__m128d)__builtin_convertvector((__v2du)__A, __v2df)` 从当前函数返回。
- **L712 EN**: Closes the current lexical scope or compound statement.
  **L712 CN**: 结束当前词法作用域或复合语句块。
- **L713 EN**: Blank line separating nearby declarations or logic blocks.
  **L713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L714 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L714 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L715 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepu64_pd(__m128d __W, __mmask8 __U, __m128i __A) {`.
  **L715 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepu64_pd(__m128d __W, __mmask8 __U, __m128i __A) {`。
- **L716 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L716 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_cvtepu64_pd(__A),`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_cvtepu64_pd(__A),`。
- **L718 EN**: Executes a call or declaration centered on `statement`.
  **L718 CN**: 执行以 `statement` 为核心的调用或声明。
- **L719 EN**: Closes the current lexical scope or compound statement.
  **L719 CN**: 结束当前词法作用域或复合语句块。
- **L720 EN**: Blank line separating nearby declarations or logic blocks.
  **L720 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 721-744

````c
static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_cvtepu64_pd(__mmask8 __U, __m128i __A) {
  return (__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,
                                              (__v2df)_mm_cvtepu64_pd(__A),
                                              (__v2df)_mm_setzero_pd());
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtepu64_pd(__m256i __A) {
  return (__m256d)__builtin_convertvector((__v4du)__A, __v4df);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepu64_pd(__m256d __W, __mmask8 __U, __m256i __A) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_cvtepu64_pd(__A),
                                              (__v4df)__W);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepu64_pd(__mmask8 __U, __m256i __A) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,
                                              (__v4df)_mm256_cvtepu64_pd(__A),
                                              (__v4df)_mm256_setzero_pd());
````
- **L721 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L721 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128d __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L722 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepu64_pd(__mmask8 __U, __m128i __A) {`.
  **L722 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepu64_pd(__mmask8 __U, __m128i __A) {`。
- **L723 EN**: Returns from the current function with `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,`.
  **L723 CN**: 以 `(__m128d)__builtin_ia32_selectpd_128((__mmask8)__U,` 从当前函数返回。
- **L724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2df)_mm_cvtepu64_pd(__A),`.
  **L724 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2df)_mm_cvtepu64_pd(__A),`。
- **L725 EN**: Executes a call or declaration centered on `statement`.
  **L725 CN**: 执行以 `statement` 为核心的调用或声明。
- **L726 EN**: Closes the current lexical scope or compound statement.
  **L726 CN**: 结束当前词法作用域或复合语句块。
- **L727 EN**: Blank line separating nearby declarations or logic blocks.
  **L727 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L728 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L728 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L729 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepu64_pd(__m256i __A) {`.
  **L729 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepu64_pd(__m256i __A) {`。
- **L730 EN**: Returns from the current function with `(__m256d)__builtin_convertvector((__v4du)__A, __v4df)`.
  **L730 CN**: 以 `(__m256d)__builtin_convertvector((__v4du)__A, __v4df)` 从当前函数返回。
- **L731 EN**: Closes the current lexical scope or compound statement.
  **L731 CN**: 结束当前词法作用域或复合语句块。
- **L732 EN**: Blank line separating nearby declarations or logic blocks.
  **L732 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L733 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L733 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L734 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepu64_pd(__m256d __W, __mmask8 __U, __m256i __A) {`.
  **L734 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepu64_pd(__m256d __W, __mmask8 __U, __m256i __A) {`。
- **L735 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L735 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_cvtepu64_pd(__A),`.
  **L736 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_cvtepu64_pd(__A),`。
- **L737 EN**: Executes a call or declaration centered on `statement`.
  **L737 CN**: 执行以 `statement` 为核心的调用或声明。
- **L738 EN**: Closes the current lexical scope or compound statement.
  **L738 CN**: 结束当前词法作用域或复合语句块。
- **L739 EN**: Blank line separating nearby declarations or logic blocks.
  **L739 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L740 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L740 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L741 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepu64_pd(__mmask8 __U, __m256i __A) {`.
  **L741 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepu64_pd(__mmask8 __U, __m256i __A) {`。
- **L742 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,`.
  **L742 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__U,` 从当前函数返回。
- **L743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_cvtepu64_pd(__A),`.
  **L743 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_cvtepu64_pd(__A),`。
- **L744 EN**: Executes a call or declaration centered on `statement`.
  **L744 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 745-768

````c
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_cvtepu64_ps (__m128i __A) {
  return (__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,
                (__v4sf) _mm_setzero_ps(),
                (__mmask8) -1);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_mask_cvtepu64_ps (__m128 __W, __mmask8 __U, __m128i __A) {
  return (__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,
                (__v4sf) __W,
                (__mmask8) __U);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS128
_mm_maskz_cvtepu64_ps (__mmask8 __U, __m128i __A) {
  return (__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,
                (__v4sf) _mm_setzero_ps(),
                (__mmask8) __U);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L745 EN**: Closes the current lexical scope or compound statement.
  **L745 CN**: 结束当前词法作用域或复合语句块。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L747 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L747 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L748 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepu64_ps (__m128i __A) {`.
  **L748 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepu64_ps (__m128i __A) {`。
- **L749 EN**: Returns from the current function with `(__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,`.
  **L749 CN**: 以 `(__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,` 从当前函数返回。
- **L750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf) _mm_setzero_ps(),`.
  **L750 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf) _mm_setzero_ps(),`。
- **L751 EN**: Executes a call or declaration centered on `statement`.
  **L751 CN**: 执行以 `statement` 为核心的调用或声明。
- **L752 EN**: Closes the current lexical scope or compound statement.
  **L752 CN**: 结束当前词法作用域或复合语句块。
- **L753 EN**: Blank line separating nearby declarations or logic blocks.
  **L753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L754 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L754 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L755 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepu64_ps (__m128 __W, __mmask8 __U, __m128i __A) {`.
  **L755 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepu64_ps (__m128 __W, __mmask8 __U, __m128i __A) {`。
- **L756 EN**: Returns from the current function with `(__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,`.
  **L756 CN**: 以 `(__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,` 从当前函数返回。
- **L757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf) __W,`.
  **L757 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf) __W,`。
- **L758 EN**: Executes a call or declaration centered on `statement`.
  **L758 CN**: 执行以 `statement` 为核心的调用或声明。
- **L759 EN**: Closes the current lexical scope or compound statement.
  **L759 CN**: 结束当前词法作用域或复合语句块。
- **L760 EN**: Blank line separating nearby declarations or logic blocks.
  **L760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L761 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS128`.
  **L761 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS128`。
- **L762 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepu64_ps (__mmask8 __U, __m128i __A) {`.
  **L762 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepu64_ps (__mmask8 __U, __m128i __A) {`。
- **L763 EN**: Returns from the current function with `(__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,`.
  **L763 CN**: 以 `(__m128) __builtin_ia32_cvtuqq2ps128_mask ((__v2di) __A,` 从当前函数返回。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf) _mm_setzero_ps(),`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf) _mm_setzero_ps(),`。
- **L765 EN**: Executes a call or declaration centered on `statement`.
  **L765 CN**: 执行以 `statement` 为核心的调用或声明。
- **L766 EN**: Closes the current lexical scope or compound statement.
  **L766 CN**: 结束当前词法作用域或复合语句块。
- **L767 EN**: Blank line separating nearby declarations or logic blocks.
  **L767 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L768 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L768 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 769-792

````c
_mm256_cvtepu64_ps(__m256i __A) {
  return (__m128)__builtin_convertvector((__v4du)__A, __v4sf);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepu64_ps(__m128 __W, __mmask8 __U, __m256i __A) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm256_cvtepu64_ps(__A),
                                             (__v4sf)__W);
}

static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepu64_ps(__mmask8 __U, __m256i __A) {
  return (__m128)__builtin_ia32_selectps_128((__mmask8)__U,
                                             (__v4sf)_mm256_cvtepu64_ps(__A),
                                             (__v4sf)_mm_setzero_ps());
}

#define _mm_range_pd(A, B, C) \
  ((__m128d)__builtin_ia32_rangepd128_mask((__v2df)(__m128d)(A), \
                                           (__v2df)(__m128d)(B), (int)(C), \
                                           (__v2df)_mm_setzero_pd(), \
                                           (__mmask8)-1))

````
- **L769 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepu64_ps(__m256i __A) {`.
  **L769 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepu64_ps(__m256i __A) {`。
- **L770 EN**: Returns from the current function with `(__m128)__builtin_convertvector((__v4du)__A, __v4sf)`.
  **L770 CN**: 以 `(__m128)__builtin_convertvector((__v4du)__A, __v4sf)` 从当前函数返回。
- **L771 EN**: Closes the current lexical scope or compound statement.
  **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Blank line separating nearby declarations or logic blocks.
  **L772 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L773 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L773 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L774 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepu64_ps(__m128 __W, __mmask8 __U, __m256i __A) {`.
  **L774 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepu64_ps(__m128 __W, __mmask8 __U, __m256i __A) {`。
- **L775 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L775 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm256_cvtepu64_ps(__A),`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm256_cvtepu64_ps(__A),`。
- **L777 EN**: Executes a call or declaration centered on `statement`.
  **L777 CN**: 执行以 `statement` 为核心的调用或声明。
- **L778 EN**: Closes the current lexical scope or compound statement.
  **L778 CN**: 结束当前词法作用域或复合语句块。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L780 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L780 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L781 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepu64_ps(__mmask8 __U, __m256i __A) {`.
  **L781 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepu64_ps(__mmask8 __U, __m256i __A) {`。
- **L782 EN**: Returns from the current function with `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,`.
  **L782 CN**: 以 `(__m128)__builtin_ia32_selectps_128((__mmask8)__U,` 从当前函数返回。
- **L783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4sf)_mm256_cvtepu64_ps(__A),`.
  **L783 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4sf)_mm256_cvtepu64_ps(__A),`。
- **L784 EN**: Executes a call or declaration centered on `statement`.
  **L784 CN**: 执行以 `statement` 为核心的调用或声明。
- **L785 EN**: Closes the current lexical scope or compound statement.
  **L785 CN**: 结束当前词法作用域或复合语句块。
- **L786 EN**: Blank line separating nearby declarations or logic blocks.
  **L786 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L787 EN**: Defines macro `_mm_range_pd(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L787 CN**: 定义宏 `_mm_range_pd(A, B, C)`，用于条件编译、简写或 API 生成。
- **L788 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd128_mask`.
  **L788 CN**: 继续与可调用符号 `__builtin_ia32_rangepd128_mask` 相关的逻辑。
- **L789 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), (int)(C), \`.
  **L789 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), (int)(C), \`。
- **L790 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L790 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L791 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L791 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L792 EN**: Blank line separating nearby declarations or logic blocks.
  **L792 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 793-816

````c
#define _mm_mask_range_pd(W, U, A, B, C) \
  ((__m128d)__builtin_ia32_rangepd128_mask((__v2df)(__m128d)(A), \
                                           (__v2df)(__m128d)(B), (int)(C), \
                                           (__v2df)(__m128d)(W), \
                                           (__mmask8)(U)))

#define _mm_maskz_range_pd(U, A, B, C) \
  ((__m128d)__builtin_ia32_rangepd128_mask((__v2df)(__m128d)(A), \
                                           (__v2df)(__m128d)(B), (int)(C), \
                                           (__v2df)_mm_setzero_pd(), \
                                           (__mmask8)(U)))

#define _mm256_range_pd(A, B, C) \
  ((__m256d)__builtin_ia32_rangepd256_mask((__v4df)(__m256d)(A), \
                                           (__v4df)(__m256d)(B), (int)(C), \
                                           (__v4df)_mm256_setzero_pd(), \
                                           (__mmask8)-1))

#define _mm256_mask_range_pd(W, U, A, B, C) \
  ((__m256d)__builtin_ia32_rangepd256_mask((__v4df)(__m256d)(A), \
                                           (__v4df)(__m256d)(B), (int)(C), \
                                           (__v4df)(__m256d)(W), \
                                           (__mmask8)(U)))

````
- **L793 EN**: Defines macro `_mm_mask_range_pd(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L793 CN**: 定义宏 `_mm_mask_range_pd(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L794 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd128_mask`.
  **L794 CN**: 继续与可调用符号 `__builtin_ia32_rangepd128_mask` 相关的逻辑。
- **L795 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), (int)(C), \`.
  **L795 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), (int)(C), \`。
- **L796 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(W), \`.
  **L796 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(W), \`。
- **L797 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L797 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L798 EN**: Blank line separating nearby declarations or logic blocks.
  **L798 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L799 EN**: Defines macro `_mm_maskz_range_pd(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L799 CN**: 定义宏 `_mm_maskz_range_pd(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L800 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd128_mask`.
  **L800 CN**: 继续与可调用符号 `__builtin_ia32_rangepd128_mask` 相关的逻辑。
- **L801 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), (int)(C), \`.
  **L801 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), (int)(C), \`。
- **L802 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L802 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L803 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L803 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L804 EN**: Blank line separating nearby declarations or logic blocks.
  **L804 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L805 EN**: Defines macro `_mm256_range_pd(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L805 CN**: 定义宏 `_mm256_range_pd(A, B, C)`，用于条件编译、简写或 API 生成。
- **L806 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd256_mask`.
  **L806 CN**: 继续与可调用符号 `__builtin_ia32_rangepd256_mask` 相关的逻辑。
- **L807 EN**: Continues the surrounding expression or declaration: `(__v4df)(__m256d)(B), (int)(C), \`.
  **L807 CN**: 继续构造周围的表达式或声明：`(__v4df)(__m256d)(B), (int)(C), \`。
- **L808 EN**: Continues logic associated with callable symbol `_mm256_setzero_pd`.
  **L808 CN**: 继续与可调用符号 `_mm256_setzero_pd` 相关的逻辑。
- **L809 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L809 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L811 EN**: Defines macro `_mm256_mask_range_pd(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L811 CN**: 定义宏 `_mm256_mask_range_pd(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L812 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd256_mask`.
  **L812 CN**: 继续与可调用符号 `__builtin_ia32_rangepd256_mask` 相关的逻辑。
- **L813 EN**: Continues the surrounding expression or declaration: `(__v4df)(__m256d)(B), (int)(C), \`.
  **L813 CN**: 继续构造周围的表达式或声明：`(__v4df)(__m256d)(B), (int)(C), \`。
- **L814 EN**: Continues the surrounding expression or declaration: `(__v4df)(__m256d)(W), \`.
  **L814 CN**: 继续构造周围的表达式或声明：`(__v4df)(__m256d)(W), \`。
- **L815 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L815 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L816 EN**: Blank line separating nearby declarations or logic blocks.
  **L816 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 817-840

````c
#define _mm256_maskz_range_pd(U, A, B, C) \
  ((__m256d)__builtin_ia32_rangepd256_mask((__v4df)(__m256d)(A), \
                                           (__v4df)(__m256d)(B), (int)(C), \
                                           (__v4df)_mm256_setzero_pd(), \
                                           (__mmask8)(U)))

#define _mm_range_ps(A, B, C) \
  ((__m128)__builtin_ia32_rangeps128_mask((__v4sf)(__m128)(A), \
                                          (__v4sf)(__m128)(B), (int)(C), \
                                          (__v4sf)_mm_setzero_ps(), \
                                          (__mmask8)-1))

#define _mm_mask_range_ps(W, U, A, B, C) \
  ((__m128)__builtin_ia32_rangeps128_mask((__v4sf)(__m128)(A), \
                                          (__v4sf)(__m128)(B), (int)(C), \
                                          (__v4sf)(__m128)(W), (__mmask8)(U)))

#define _mm_maskz_range_ps(U, A, B, C) \
  ((__m128)__builtin_ia32_rangeps128_mask((__v4sf)(__m128)(A), \
                                          (__v4sf)(__m128)(B), (int)(C), \
                                          (__v4sf)_mm_setzero_ps(), \
                                          (__mmask8)(U)))

#define _mm256_range_ps(A, B, C) \
````
- **L817 EN**: Defines macro `_mm256_maskz_range_pd(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L817 CN**: 定义宏 `_mm256_maskz_range_pd(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L818 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangepd256_mask`.
  **L818 CN**: 继续与可调用符号 `__builtin_ia32_rangepd256_mask` 相关的逻辑。
- **L819 EN**: Continues the surrounding expression or declaration: `(__v4df)(__m256d)(B), (int)(C), \`.
  **L819 CN**: 继续构造周围的表达式或声明：`(__v4df)(__m256d)(B), (int)(C), \`。
- **L820 EN**: Continues logic associated with callable symbol `_mm256_setzero_pd`.
  **L820 CN**: 继续与可调用符号 `_mm256_setzero_pd` 相关的逻辑。
- **L821 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L821 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Defines macro `_mm_range_ps(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L823 CN**: 定义宏 `_mm_range_ps(A, B, C)`，用于条件编译、简写或 API 生成。
- **L824 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps128_mask`.
  **L824 CN**: 继续与可调用符号 `__builtin_ia32_rangeps128_mask` 相关的逻辑。
- **L825 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), (int)(C), \`.
  **L825 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), (int)(C), \`。
- **L826 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L826 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L827 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L827 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L829 EN**: Defines macro `_mm_mask_range_ps(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L829 CN**: 定义宏 `_mm_mask_range_ps(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L830 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps128_mask`.
  **L830 CN**: 继续与可调用符号 `__builtin_ia32_rangeps128_mask` 相关的逻辑。
- **L831 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), (int)(C), \`.
  **L831 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), (int)(C), \`。
- **L832 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(W), (__mmask8)(U)))`.
  **L832 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(W), (__mmask8)(U)))`。
- **L833 EN**: Blank line separating nearby declarations or logic blocks.
  **L833 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L834 EN**: Defines macro `_mm_maskz_range_ps(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L834 CN**: 定义宏 `_mm_maskz_range_ps(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L835 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps128_mask`.
  **L835 CN**: 继续与可调用符号 `__builtin_ia32_rangeps128_mask` 相关的逻辑。
- **L836 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(B), (int)(C), \`.
  **L836 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(B), (int)(C), \`。
- **L837 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L837 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L838 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L838 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L839 EN**: Blank line separating nearby declarations or logic blocks.
  **L839 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L840 EN**: Defines macro `_mm256_range_ps(A, B, C)` for conditional compilation, shorthand, or API generation.
  **L840 CN**: 定义宏 `_mm256_range_ps(A, B, C)`，用于条件编译、简写或 API 生成。

### Lines 841-864

````c
  ((__m256)__builtin_ia32_rangeps256_mask((__v8sf)(__m256)(A), \
                                          (__v8sf)(__m256)(B), (int)(C), \
                                          (__v8sf)_mm256_setzero_ps(), \
                                          (__mmask8)-1))

#define _mm256_mask_range_ps(W, U, A, B, C) \
  ((__m256)__builtin_ia32_rangeps256_mask((__v8sf)(__m256)(A), \
                                          (__v8sf)(__m256)(B), (int)(C), \
                                          (__v8sf)(__m256)(W), (__mmask8)(U)))

#define _mm256_maskz_range_ps(U, A, B, C) \
  ((__m256)__builtin_ia32_rangeps256_mask((__v8sf)(__m256)(A), \
                                          (__v8sf)(__m256)(B), (int)(C), \
                                          (__v8sf)_mm256_setzero_ps(), \
                                          (__mmask8)(U)))

#define _mm_reduce_pd(A, B) \
  ((__m128d)__builtin_ia32_reducepd128_mask((__v2df)(__m128d)(A), (int)(B), \
                                            (__v2df)_mm_setzero_pd(), \
                                            (__mmask8)-1))

#define _mm_mask_reduce_pd(W, U, A, B) \
  ((__m128d)__builtin_ia32_reducepd128_mask((__v2df)(__m128d)(A), (int)(B), \
                                            (__v2df)(__m128d)(W), \
````
- **L841 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps256_mask`.
  **L841 CN**: 继续与可调用符号 `__builtin_ia32_rangeps256_mask` 相关的逻辑。
- **L842 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(B), (int)(C), \`.
  **L842 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(B), (int)(C), \`。
- **L843 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L843 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L844 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L844 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L845 EN**: Blank line separating nearby declarations or logic blocks.
  **L845 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L846 EN**: Defines macro `_mm256_mask_range_ps(W, U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L846 CN**: 定义宏 `_mm256_mask_range_ps(W, U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L847 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps256_mask`.
  **L847 CN**: 继续与可调用符号 `__builtin_ia32_rangeps256_mask` 相关的逻辑。
- **L848 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(B), (int)(C), \`.
  **L848 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(B), (int)(C), \`。
- **L849 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(W), (__mmask8)(U)))`.
  **L849 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(W), (__mmask8)(U)))`。
- **L850 EN**: Blank line separating nearby declarations or logic blocks.
  **L850 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L851 EN**: Defines macro `_mm256_maskz_range_ps(U, A, B, C)` for conditional compilation, shorthand, or API generation.
  **L851 CN**: 定义宏 `_mm256_maskz_range_ps(U, A, B, C)`，用于条件编译、简写或 API 生成。
- **L852 EN**: Continues logic associated with callable symbol `__builtin_ia32_rangeps256_mask`.
  **L852 CN**: 继续与可调用符号 `__builtin_ia32_rangeps256_mask` 相关的逻辑。
- **L853 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(B), (int)(C), \`.
  **L853 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(B), (int)(C), \`。
- **L854 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L854 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L855 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L855 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L856 EN**: Blank line separating nearby declarations or logic blocks.
  **L856 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L857 EN**: Defines macro `_mm_reduce_pd(A, B)` for conditional compilation, shorthand, or API generation.
  **L857 CN**: 定义宏 `_mm_reduce_pd(A, B)`，用于条件编译、简写或 API 生成。
- **L858 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd128_mask`.
  **L858 CN**: 继续与可调用符号 `__builtin_ia32_reducepd128_mask` 相关的逻辑。
- **L859 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L859 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L860 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L860 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L861 EN**: Blank line separating nearby declarations or logic blocks.
  **L861 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L862 EN**: Defines macro `_mm_mask_reduce_pd(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L862 CN**: 定义宏 `_mm_mask_reduce_pd(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L863 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd128_mask`.
  **L863 CN**: 继续与可调用符号 `__builtin_ia32_reducepd128_mask` 相关的逻辑。
- **L864 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(W), \`.
  **L864 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(W), \`。

### Lines 865-888

````c
                                            (__mmask8)(U)))

#define _mm_maskz_reduce_pd(U, A, B) \
  ((__m128d)__builtin_ia32_reducepd128_mask((__v2df)(__m128d)(A), (int)(B), \
                                            (__v2df)_mm_setzero_pd(), \
                                            (__mmask8)(U)))

#define _mm256_reduce_pd(A, B) \
  ((__m256d)__builtin_ia32_reducepd256_mask((__v4df)(__m256d)(A), (int)(B), \
                                            (__v4df)_mm256_setzero_pd(), \
                                            (__mmask8)-1))

#define _mm256_mask_reduce_pd(W, U, A, B) \
  ((__m256d)__builtin_ia32_reducepd256_mask((__v4df)(__m256d)(A), (int)(B), \
                                            (__v4df)(__m256d)(W), \
                                            (__mmask8)(U)))

#define _mm256_maskz_reduce_pd(U, A, B) \
  ((__m256d)__builtin_ia32_reducepd256_mask((__v4df)(__m256d)(A), (int)(B), \
                                            (__v4df)_mm256_setzero_pd(), \
                                            (__mmask8)(U)))

#define _mm_reduce_ps(A, B) \
  ((__m128)__builtin_ia32_reduceps128_mask((__v4sf)(__m128)(A), (int)(B), \
````
- **L865 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L865 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Defines macro `_mm_maskz_reduce_pd(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L867 CN**: 定义宏 `_mm_maskz_reduce_pd(U, A, B)`，用于条件编译、简写或 API 生成。
- **L868 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd128_mask`.
  **L868 CN**: 继续与可调用符号 `__builtin_ia32_reducepd128_mask` 相关的逻辑。
- **L869 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L869 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L870 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L870 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L871 EN**: Blank line separating nearby declarations or logic blocks.
  **L871 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L872 EN**: Defines macro `_mm256_reduce_pd(A, B)` for conditional compilation, shorthand, or API generation.
  **L872 CN**: 定义宏 `_mm256_reduce_pd(A, B)`，用于条件编译、简写或 API 生成。
- **L873 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd256_mask`.
  **L873 CN**: 继续与可调用符号 `__builtin_ia32_reducepd256_mask` 相关的逻辑。
- **L874 EN**: Continues logic associated with callable symbol `_mm256_setzero_pd`.
  **L874 CN**: 继续与可调用符号 `_mm256_setzero_pd` 相关的逻辑。
- **L875 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L875 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L876 EN**: Blank line separating nearby declarations or logic blocks.
  **L876 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L877 EN**: Defines macro `_mm256_mask_reduce_pd(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L877 CN**: 定义宏 `_mm256_mask_reduce_pd(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L878 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd256_mask`.
  **L878 CN**: 继续与可调用符号 `__builtin_ia32_reducepd256_mask` 相关的逻辑。
- **L879 EN**: Continues the surrounding expression or declaration: `(__v4df)(__m256d)(W), \`.
  **L879 CN**: 继续构造周围的表达式或声明：`(__v4df)(__m256d)(W), \`。
- **L880 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L880 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L881 EN**: Blank line separating nearby declarations or logic blocks.
  **L881 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L882 EN**: Defines macro `_mm256_maskz_reduce_pd(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L882 CN**: 定义宏 `_mm256_maskz_reduce_pd(U, A, B)`，用于条件编译、简写或 API 生成。
- **L883 EN**: Continues logic associated with callable symbol `__builtin_ia32_reducepd256_mask`.
  **L883 CN**: 继续与可调用符号 `__builtin_ia32_reducepd256_mask` 相关的逻辑。
- **L884 EN**: Continues logic associated with callable symbol `_mm256_setzero_pd`.
  **L884 CN**: 继续与可调用符号 `_mm256_setzero_pd` 相关的逻辑。
- **L885 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L885 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L886 EN**: Blank line separating nearby declarations or logic blocks.
  **L886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L887 EN**: Defines macro `_mm_reduce_ps(A, B)` for conditional compilation, shorthand, or API generation.
  **L887 CN**: 定义宏 `_mm_reduce_ps(A, B)`，用于条件编译、简写或 API 生成。
- **L888 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps128_mask`.
  **L888 CN**: 继续与可调用符号 `__builtin_ia32_reduceps128_mask` 相关的逻辑。

### Lines 889-912

````c
                                           (__v4sf)_mm_setzero_ps(), \
                                           (__mmask8)-1))

#define _mm_mask_reduce_ps(W, U, A, B) \
  ((__m128)__builtin_ia32_reduceps128_mask((__v4sf)(__m128)(A), (int)(B), \
                                           (__v4sf)(__m128)(W), \
                                           (__mmask8)(U)))

#define _mm_maskz_reduce_ps(U, A, B) \
  ((__m128)__builtin_ia32_reduceps128_mask((__v4sf)(__m128)(A), (int)(B), \
                                           (__v4sf)_mm_setzero_ps(), \
                                           (__mmask8)(U)))

#define _mm256_reduce_ps(A, B) \
  ((__m256)__builtin_ia32_reduceps256_mask((__v8sf)(__m256)(A), (int)(B), \
                                           (__v8sf)_mm256_setzero_ps(), \
                                           (__mmask8)-1))

#define _mm256_mask_reduce_ps(W, U, A, B) \
  ((__m256)__builtin_ia32_reduceps256_mask((__v8sf)(__m256)(A), (int)(B), \
                                           (__v8sf)(__m256)(W), \
                                           (__mmask8)(U)))

#define _mm256_maskz_reduce_ps(U, A, B) \
````
- **L889 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L889 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L890 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L890 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L891 EN**: Blank line separating nearby declarations or logic blocks.
  **L891 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L892 EN**: Defines macro `_mm_mask_reduce_ps(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L892 CN**: 定义宏 `_mm_mask_reduce_ps(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L893 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps128_mask`.
  **L893 CN**: 继续与可调用符号 `__builtin_ia32_reduceps128_mask` 相关的逻辑。
- **L894 EN**: Continues the surrounding expression or declaration: `(__v4sf)(__m128)(W), \`.
  **L894 CN**: 继续构造周围的表达式或声明：`(__v4sf)(__m128)(W), \`。
- **L895 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L895 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L897 EN**: Defines macro `_mm_maskz_reduce_ps(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L897 CN**: 定义宏 `_mm_maskz_reduce_ps(U, A, B)`，用于条件编译、简写或 API 生成。
- **L898 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps128_mask`.
  **L898 CN**: 继续与可调用符号 `__builtin_ia32_reduceps128_mask` 相关的逻辑。
- **L899 EN**: Continues logic associated with callable symbol `_mm_setzero_ps`.
  **L899 CN**: 继续与可调用符号 `_mm_setzero_ps` 相关的逻辑。
- **L900 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L900 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L902 EN**: Defines macro `_mm256_reduce_ps(A, B)` for conditional compilation, shorthand, or API generation.
  **L902 CN**: 定义宏 `_mm256_reduce_ps(A, B)`，用于条件编译、简写或 API 生成。
- **L903 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps256_mask`.
  **L903 CN**: 继续与可调用符号 `__builtin_ia32_reduceps256_mask` 相关的逻辑。
- **L904 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L904 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L905 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L905 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L906 EN**: Blank line separating nearby declarations or logic blocks.
  **L906 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L907 EN**: Defines macro `_mm256_mask_reduce_ps(W, U, A, B)` for conditional compilation, shorthand, or API generation.
  **L907 CN**: 定义宏 `_mm256_mask_reduce_ps(W, U, A, B)`，用于条件编译、简写或 API 生成。
- **L908 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps256_mask`.
  **L908 CN**: 继续与可调用符号 `__builtin_ia32_reduceps256_mask` 相关的逻辑。
- **L909 EN**: Continues the surrounding expression or declaration: `(__v8sf)(__m256)(W), \`.
  **L909 CN**: 继续构造周围的表达式或声明：`(__v8sf)(__m256)(W), \`。
- **L910 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L910 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L911 EN**: Blank line separating nearby declarations or logic blocks.
  **L911 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L912 EN**: Defines macro `_mm256_maskz_reduce_ps(U, A, B)` for conditional compilation, shorthand, or API generation.
  **L912 CN**: 定义宏 `_mm256_maskz_reduce_ps(U, A, B)`，用于条件编译、简写或 API 生成。

### Lines 913-936

````c
  ((__m256)__builtin_ia32_reduceps256_mask((__v8sf)(__m256)(A), (int)(B), \
                                           (__v8sf)_mm256_setzero_ps(), \
                                           (__mmask8)(U)))

static __inline__ __mmask8
    __DEFAULT_FN_ATTRS128_CONSTEXPR _mm_movepi32_mask(__m128i __A) {
  return (__mmask8) __builtin_ia32_cvtd2mask128 ((__v4si) __A);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_movepi32_mask(__m256i __A) {
  return (__mmask8) __builtin_ia32_cvtd2mask256 ((__v8si) __A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_movm_epi32(__mmask8 __A) {
  return (__m128i) __builtin_ia32_cvtmask2d128 (__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_movm_epi32(__mmask8 __A) {
  return (__m256i) __builtin_ia32_cvtmask2d256 (__A);
}

````
- **L913 EN**: Continues logic associated with callable symbol `__builtin_ia32_reduceps256_mask`.
  **L913 CN**: 继续与可调用符号 `__builtin_ia32_reduceps256_mask` 相关的逻辑。
- **L914 EN**: Continues logic associated with callable symbol `_mm256_setzero_ps`.
  **L914 CN**: 继续与可调用符号 `_mm256_setzero_ps` 相关的逻辑。
- **L915 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L915 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L916 EN**: Blank line separating nearby declarations or logic blocks.
  **L916 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L917 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8`.
  **L917 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8`。
- **L918 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_movepi32_mask(__m128i __A) {`.
  **L918 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEFAULT_FN_ATTRS128_CONSTEXPR _mm_movepi32_mask(__m128i __A) {`。
- **L919 EN**: Returns from the current function with `(__mmask8) __builtin_ia32_cvtd2mask128 ((__v4si) __A)`.
  **L919 CN**: 以 `(__mmask8) __builtin_ia32_cvtd2mask128 ((__v4si) __A)` 从当前函数返回。
- **L920 EN**: Closes the current lexical scope or compound statement.
  **L920 CN**: 结束当前词法作用域或复合语句块。
- **L921 EN**: Blank line separating nearby declarations or logic blocks.
  **L921 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L922 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L922 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L923 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_movepi32_mask(__m256i __A) {`.
  **L923 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_movepi32_mask(__m256i __A) {`。
- **L924 EN**: Returns from the current function with `(__mmask8) __builtin_ia32_cvtd2mask256 ((__v8si) __A)`.
  **L924 CN**: 以 `(__mmask8) __builtin_ia32_cvtd2mask256 ((__v8si) __A)` 从当前函数返回。
- **L925 EN**: Closes the current lexical scope or compound statement.
  **L925 CN**: 结束当前词法作用域或复合语句块。
- **L926 EN**: Blank line separating nearby declarations or logic blocks.
  **L926 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L927 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L927 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L928 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movm_epi32(__mmask8 __A) {`.
  **L928 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movm_epi32(__mmask8 __A) {`。
- **L929 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtmask2d128 (__A)`.
  **L929 CN**: 以 `(__m128i) __builtin_ia32_cvtmask2d128 (__A)` 从当前函数返回。
- **L930 EN**: Closes the current lexical scope or compound statement.
  **L930 CN**: 结束当前词法作用域或复合语句块。
- **L931 EN**: Blank line separating nearby declarations or logic blocks.
  **L931 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L932 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L932 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L933 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_movm_epi32(__mmask8 __A) {`.
  **L933 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_movm_epi32(__mmask8 __A) {`。
- **L934 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtmask2d256 (__A)`.
  **L934 CN**: 以 `(__m256i) __builtin_ia32_cvtmask2d256 (__A)` 从当前函数返回。
- **L935 EN**: Closes the current lexical scope or compound statement.
  **L935 CN**: 结束当前词法作用域或复合语句块。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 937-960

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_movm_epi64(__mmask8 __A) {
  return (__m128i) __builtin_ia32_cvtmask2q128 (__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_movm_epi64(__mmask8 __A) {
  return (__m256i) __builtin_ia32_cvtmask2q256 (__A);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_movepi64_mask(__m128i __A) {
  return (__mmask8) __builtin_ia32_cvtq2mask128 ((__v2di) __A);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_movepi64_mask(__m256i __A) {
  return (__mmask8) __builtin_ia32_cvtq2mask256 ((__v4di) __A);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_broadcast_f32x2(__m128 __A) {
  return (__m256)__builtin_shufflevector((__v4sf)__A, (__v4sf)__A,
                                         0, 1, 0, 1, 0, 1, 0, 1);
````
- **L937 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L937 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L938 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movm_epi64(__mmask8 __A) {`.
  **L938 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movm_epi64(__mmask8 __A) {`。
- **L939 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtmask2q128 (__A)`.
  **L939 CN**: 以 `(__m128i) __builtin_ia32_cvtmask2q128 (__A)` 从当前函数返回。
- **L940 EN**: Closes the current lexical scope or compound statement.
  **L940 CN**: 结束当前词法作用域或复合语句块。
- **L941 EN**: Blank line separating nearby declarations or logic blocks.
  **L941 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L942 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L942 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L943 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_movm_epi64(__mmask8 __A) {`.
  **L943 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_movm_epi64(__mmask8 __A) {`。
- **L944 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtmask2q256 (__A)`.
  **L944 CN**: 以 `(__m256i) __builtin_ia32_cvtmask2q256 (__A)` 从当前函数返回。
- **L945 EN**: Closes the current lexical scope or compound statement.
  **L945 CN**: 结束当前词法作用域或复合语句块。
- **L946 EN**: Blank line separating nearby declarations or logic blocks.
  **L946 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L947 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L947 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L948 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movepi64_mask(__m128i __A) {`.
  **L948 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movepi64_mask(__m128i __A) {`。
- **L949 EN**: Returns from the current function with `(__mmask8) __builtin_ia32_cvtq2mask128 ((__v2di) __A)`.
  **L949 CN**: 以 `(__mmask8) __builtin_ia32_cvtq2mask128 ((__v2di) __A)` 从当前函数返回。
- **L950 EN**: Closes the current lexical scope or compound statement.
  **L950 CN**: 结束当前词法作用域或复合语句块。
- **L951 EN**: Blank line separating nearby declarations or logic blocks.
  **L951 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L952 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L952 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L953 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_movepi64_mask(__m256i __A) {`.
  **L953 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_movepi64_mask(__m256i __A) {`。
- **L954 EN**: Returns from the current function with `(__mmask8) __builtin_ia32_cvtq2mask256 ((__v4di) __A)`.
  **L954 CN**: 以 `(__mmask8) __builtin_ia32_cvtq2mask256 ((__v4di) __A)` 从当前函数返回。
- **L955 EN**: Closes the current lexical scope or compound statement.
  **L955 CN**: 结束当前词法作用域或复合语句块。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L957 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L957 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L958 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_broadcast_f32x2(__m128 __A) {`.
  **L958 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_broadcast_f32x2(__m128 __A) {`。
- **L959 EN**: Returns from the current function with `(__m256)__builtin_shufflevector((__v4sf)__A, (__v4sf)__A,`.
  **L959 CN**: 以 `(__m256)__builtin_shufflevector((__v4sf)__A, (__v4sf)__A,` 从当前函数返回。
- **L960 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1, 0, 1, 0, 1);`.
  **L960 CN**: 添加一条独立语句或声明：`0, 1, 0, 1, 0, 1, 0, 1);`。

### Lines 961-984

````c
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_broadcast_f32x2(__m256 __O, __mmask8 __M, __m128 __A) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__M,
                                             (__v8sf)_mm256_broadcast_f32x2(__A),
                                             (__v8sf)__O);
}

static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_broadcast_f32x2(__mmask8 __M, __m128 __A) {
  return (__m256)__builtin_ia32_selectps_256((__mmask8)__M,
                                             (__v8sf)_mm256_broadcast_f32x2(__A),
                                             (__v8sf)_mm256_setzero_ps());
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_broadcast_f64x2(__m128d __A) {
  return (__m256d)__builtin_shufflevector((__v2df)__A, (__v2df)__A,
                                          0, 1, 0, 1);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_broadcast_f64x2(__m256d __O, __mmask8 __M, __m128d __A) {
````
- **L961 EN**: Closes the current lexical scope or compound statement.
  **L961 CN**: 结束当前词法作用域或复合语句块。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L963 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L963 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L964 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_broadcast_f32x2(__m256 __O, __mmask8 __M, __m128 __A) {`.
  **L964 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_broadcast_f32x2(__m256 __O, __mmask8 __M, __m128 __A) {`。
- **L965 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__M,`.
  **L965 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__M,` 从当前函数返回。
- **L966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_broadcast_f32x2(__A),`.
  **L966 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_broadcast_f32x2(__A),`。
- **L967 EN**: Executes a call or declaration centered on `statement`.
  **L967 CN**: 执行以 `statement` 为核心的调用或声明。
- **L968 EN**: Closes the current lexical scope or compound statement.
  **L968 CN**: 结束当前词法作用域或复合语句块。
- **L969 EN**: Blank line separating nearby declarations or logic blocks.
  **L969 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L970 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L970 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L971 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_broadcast_f32x2(__mmask8 __M, __m128 __A) {`.
  **L971 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_broadcast_f32x2(__mmask8 __M, __m128 __A) {`。
- **L972 EN**: Returns from the current function with `(__m256)__builtin_ia32_selectps_256((__mmask8)__M,`.
  **L972 CN**: 以 `(__m256)__builtin_ia32_selectps_256((__mmask8)__M,` 从当前函数返回。
- **L973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8sf)_mm256_broadcast_f32x2(__A),`.
  **L973 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8sf)_mm256_broadcast_f32x2(__A),`。
- **L974 EN**: Executes a call or declaration centered on `statement`.
  **L974 CN**: 执行以 `statement` 为核心的调用或声明。
- **L975 EN**: Closes the current lexical scope or compound statement.
  **L975 CN**: 结束当前词法作用域或复合语句块。
- **L976 EN**: Blank line separating nearby declarations or logic blocks.
  **L976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L977 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L977 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L978 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_broadcast_f64x2(__m128d __A) {`.
  **L978 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_broadcast_f64x2(__m128d __A) {`。
- **L979 EN**: Returns from the current function with `(__m256d)__builtin_shufflevector((__v2df)__A, (__v2df)__A,`.
  **L979 CN**: 以 `(__m256d)__builtin_shufflevector((__v2df)__A, (__v2df)__A,` 从当前函数返回。
- **L980 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1);`.
  **L980 CN**: 添加一条独立语句或声明：`0, 1, 0, 1);`。
- **L981 EN**: Closes the current lexical scope or compound statement.
  **L981 CN**: 结束当前词法作用域或复合语句块。
- **L982 EN**: Blank line separating nearby declarations or logic blocks.
  **L982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L983 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L983 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L984 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_broadcast_f64x2(__m256d __O, __mmask8 __M, __m128d __A) {`.
  **L984 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_broadcast_f64x2(__m256d __O, __mmask8 __M, __m128d __A) {`。

### Lines 985-1008

````c
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__M,
                                            (__v4df)_mm256_broadcast_f64x2(__A),
                                            (__v4df)__O);
}

static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_broadcast_f64x2(__mmask8 __M, __m128d __A) {
  return (__m256d)__builtin_ia32_selectpd_256((__mmask8)__M,
                                            (__v4df)_mm256_broadcast_f64x2(__A),
                                            (__v4df)_mm256_setzero_pd());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_broadcast_i32x2(__m128i __A) {
  return (__m128i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,
                                          0, 1, 0, 1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_broadcast_i32x2(__m128i __O, __mmask8 __M, __m128i __A) {
  return (__m128i)__builtin_ia32_selectd_128((__mmask8)__M,
                                             (__v4si)_mm_broadcast_i32x2(__A),
                                             (__v4si)__O);
}
````
- **L985 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__M,`.
  **L985 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__M,` 从当前函数返回。
- **L986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_broadcast_f64x2(__A),`.
  **L986 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_broadcast_f64x2(__A),`。
- **L987 EN**: Executes a call or declaration centered on `statement`.
  **L987 CN**: 执行以 `statement` 为核心的调用或声明。
- **L988 EN**: Closes the current lexical scope or compound statement.
  **L988 CN**: 结束当前词法作用域或复合语句块。
- **L989 EN**: Blank line separating nearby declarations or logic blocks.
  **L989 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L990 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L990 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256d __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L991 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_broadcast_f64x2(__mmask8 __M, __m128d __A) {`.
  **L991 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_broadcast_f64x2(__mmask8 __M, __m128d __A) {`。
- **L992 EN**: Returns from the current function with `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__M,`.
  **L992 CN**: 以 `(__m256d)__builtin_ia32_selectpd_256((__mmask8)__M,` 从当前函数返回。
- **L993 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4df)_mm256_broadcast_f64x2(__A),`.
  **L993 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4df)_mm256_broadcast_f64x2(__A),`。
- **L994 EN**: Executes a call or declaration centered on `statement`.
  **L994 CN**: 执行以 `statement` 为核心的调用或声明。
- **L995 EN**: Closes the current lexical scope or compound statement.
  **L995 CN**: 结束当前词法作用域或复合语句块。
- **L996 EN**: Blank line separating nearby declarations or logic blocks.
  **L996 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L997 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L997 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L998 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_broadcast_i32x2(__m128i __A) {`.
  **L998 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_broadcast_i32x2(__m128i __A) {`。
- **L999 EN**: Returns from the current function with `(__m128i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,`.
  **L999 CN**: 以 `(__m128i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,` 从当前函数返回。
- **L1000 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1);`.
  **L1000 CN**: 添加一条独立语句或声明：`0, 1, 0, 1);`。
- **L1001 EN**: Closes the current lexical scope or compound statement.
  **L1001 CN**: 结束当前词法作用域或复合语句块。
- **L1002 EN**: Blank line separating nearby declarations or logic blocks.
  **L1002 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1003 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1003 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1004 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_broadcast_i32x2(__m128i __O, __mmask8 __M, __m128i __A) {`.
  **L1004 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_broadcast_i32x2(__m128i __O, __mmask8 __M, __m128i __A) {`。
- **L1005 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectd_128((__mmask8)__M,`.
  **L1005 CN**: 以 `(__m128i)__builtin_ia32_selectd_128((__mmask8)__M,` 从当前函数返回。
- **L1006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4si)_mm_broadcast_i32x2(__A),`.
  **L1006 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4si)_mm_broadcast_i32x2(__A),`。
- **L1007 EN**: Executes a call or declaration centered on `statement`.
  **L1007 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1008 EN**: Closes the current lexical scope or compound statement.
  **L1008 CN**: 结束当前词法作用域或复合语句块。

### Lines 1009-1032

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_broadcast_i32x2(__mmask8 __M, __m128i __A) {
  return (__m128i)__builtin_ia32_selectd_128((__mmask8)__M,
                                             (__v4si)_mm_broadcast_i32x2(__A),
                                             (__v4si)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_broadcast_i32x2(__m128i __A) {
  return (__m256i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,
                                          0, 1, 0, 1, 0, 1, 0, 1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_broadcast_i32x2(__m256i __O, __mmask8 __M, __m128i __A) {
  return (__m256i)__builtin_ia32_selectd_256((__mmask8)__M,
                                             (__v8si)_mm256_broadcast_i32x2(__A),
                                             (__v8si)__O);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_broadcast_i32x2(__mmask8 __M, __m128i __A) {
  return (__m256i)__builtin_ia32_selectd_256((__mmask8)__M,
````
- **L1009 EN**: Blank line separating nearby declarations or logic blocks.
  **L1009 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1010 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1010 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1011 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_broadcast_i32x2(__mmask8 __M, __m128i __A) {`.
  **L1011 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_broadcast_i32x2(__mmask8 __M, __m128i __A) {`。
- **L1012 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectd_128((__mmask8)__M,`.
  **L1012 CN**: 以 `(__m128i)__builtin_ia32_selectd_128((__mmask8)__M,` 从当前函数返回。
- **L1013 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4si)_mm_broadcast_i32x2(__A),`.
  **L1013 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4si)_mm_broadcast_i32x2(__A),`。
- **L1014 EN**: Executes a call or declaration centered on `statement`.
  **L1014 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1015 EN**: Closes the current lexical scope or compound statement.
  **L1015 CN**: 结束当前词法作用域或复合语句块。
- **L1016 EN**: Blank line separating nearby declarations or logic blocks.
  **L1016 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1017 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1017 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1018 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_broadcast_i32x2(__m128i __A) {`.
  **L1018 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_broadcast_i32x2(__m128i __A) {`。
- **L1019 EN**: Returns from the current function with `(__m256i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,`.
  **L1019 CN**: 以 `(__m256i)__builtin_shufflevector((__v4si)__A, (__v4si)__A,` 从当前函数返回。
- **L1020 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1, 0, 1, 0, 1);`.
  **L1020 CN**: 添加一条独立语句或声明：`0, 1, 0, 1, 0, 1, 0, 1);`。
- **L1021 EN**: Closes the current lexical scope or compound statement.
  **L1021 CN**: 结束当前词法作用域或复合语句块。
- **L1022 EN**: Blank line separating nearby declarations or logic blocks.
  **L1022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1023 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1023 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1024 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_broadcast_i32x2(__m256i __O, __mmask8 __M, __m128i __A) {`.
  **L1024 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_broadcast_i32x2(__m256i __O, __mmask8 __M, __m128i __A) {`。
- **L1025 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectd_256((__mmask8)__M,`.
  **L1025 CN**: 以 `(__m256i)__builtin_ia32_selectd_256((__mmask8)__M,` 从当前函数返回。
- **L1026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8si)_mm256_broadcast_i32x2(__A),`.
  **L1026 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8si)_mm256_broadcast_i32x2(__A),`。
- **L1027 EN**: Executes a call or declaration centered on `statement`.
  **L1027 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1028 EN**: Closes the current lexical scope or compound statement.
  **L1028 CN**: 结束当前词法作用域或复合语句块。
- **L1029 EN**: Blank line separating nearby declarations or logic blocks.
  **L1029 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1030 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1030 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1031 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_broadcast_i32x2(__mmask8 __M, __m128i __A) {`.
  **L1031 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_broadcast_i32x2(__mmask8 __M, __m128i __A) {`。
- **L1032 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectd_256((__mmask8)__M,`.
  **L1032 CN**: 以 `(__m256i)__builtin_ia32_selectd_256((__mmask8)__M,` 从当前函数返回。

### Lines 1033-1056

````c
                                             (__v8si)_mm256_broadcast_i32x2(__A),
                                             (__v8si)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_broadcast_i64x2(__m128i __A) {
  return (__m256i)__builtin_shufflevector((__v2di)__A, (__v2di)__A,
                                          0, 1, 0, 1);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_broadcast_i64x2(__m256i __O, __mmask8 __M, __m128i __A) {
  return (__m256i)__builtin_ia32_selectq_256((__mmask8)__M,
                                            (__v4di)_mm256_broadcast_i64x2(__A),
                                            (__v4di)__O);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_broadcast_i64x2(__mmask8 __M, __m128i __A) {
  return (__m256i)__builtin_ia32_selectq_256((__mmask8)__M,
                                            (__v4di)_mm256_broadcast_i64x2(__A),
                                            (__v4di)_mm256_setzero_si256());
}

````
- **L1033 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8si)_mm256_broadcast_i32x2(__A),`.
  **L1033 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8si)_mm256_broadcast_i32x2(__A),`。
- **L1034 EN**: Executes a call or declaration centered on `statement`.
  **L1034 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1035 EN**: Closes the current lexical scope or compound statement.
  **L1035 CN**: 结束当前词法作用域或复合语句块。
- **L1036 EN**: Blank line separating nearby declarations or logic blocks.
  **L1036 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1037 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1037 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1038 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_broadcast_i64x2(__m128i __A) {`.
  **L1038 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_broadcast_i64x2(__m128i __A) {`。
- **L1039 EN**: Returns from the current function with `(__m256i)__builtin_shufflevector((__v2di)__A, (__v2di)__A,`.
  **L1039 CN**: 以 `(__m256i)__builtin_shufflevector((__v2di)__A, (__v2di)__A,` 从当前函数返回。
- **L1040 EN**: Adds a standalone statement or declaration: `0, 1, 0, 1);`.
  **L1040 CN**: 添加一条独立语句或声明：`0, 1, 0, 1);`。
- **L1041 EN**: Closes the current lexical scope or compound statement.
  **L1041 CN**: 结束当前词法作用域或复合语句块。
- **L1042 EN**: Blank line separating nearby declarations or logic blocks.
  **L1042 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1043 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1043 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1044 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_broadcast_i64x2(__m256i __O, __mmask8 __M, __m128i __A) {`.
  **L1044 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_broadcast_i64x2(__m256i __O, __mmask8 __M, __m128i __A) {`。
- **L1045 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectq_256((__mmask8)__M,`.
  **L1045 CN**: 以 `(__m256i)__builtin_ia32_selectq_256((__mmask8)__M,` 从当前函数返回。
- **L1046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di)_mm256_broadcast_i64x2(__A),`.
  **L1046 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di)_mm256_broadcast_i64x2(__A),`。
- **L1047 EN**: Executes a call or declaration centered on `statement`.
  **L1047 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1048 EN**: Closes the current lexical scope or compound statement.
  **L1048 CN**: 结束当前词法作用域或复合语句块。
- **L1049 EN**: Blank line separating nearby declarations or logic blocks.
  **L1049 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1050 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1050 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1051 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_broadcast_i64x2(__mmask8 __M, __m128i __A) {`.
  **L1051 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_broadcast_i64x2(__mmask8 __M, __m128i __A) {`。
- **L1052 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectq_256((__mmask8)__M,`.
  **L1052 CN**: 以 `(__m256i)__builtin_ia32_selectq_256((__mmask8)__M,` 从当前函数返回。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di)_mm256_broadcast_i64x2(__A),`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di)_mm256_broadcast_i64x2(__A),`。
- **L1054 EN**: Executes a call or declaration centered on `statement`.
  **L1054 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1055 EN**: Closes the current lexical scope or compound statement.
  **L1055 CN**: 结束当前词法作用域或复合语句块。
- **L1056 EN**: Blank line separating nearby declarations or logic blocks.
  **L1056 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1057-1080

````c
#define _mm256_extractf64x2_pd(A, imm)                                         \
  ((__m128d)__builtin_ia32_extractf64x2_256_mask(                              \
      (__v4df)(__m256d)(A), (int)(imm), (__v2df)_mm_setzero_pd(),              \
      (__mmask8) - 1))

#define _mm256_mask_extractf64x2_pd(W, U, A, imm) \
  ((__m128d)__builtin_ia32_extractf64x2_256_mask((__v4df)(__m256d)(A), \
                                                 (int)(imm), \
                                                 (__v2df)(__m128d)(W), \
                                                 (__mmask8)(U)))

#define _mm256_maskz_extractf64x2_pd(U, A, imm) \
  ((__m128d)__builtin_ia32_extractf64x2_256_mask((__v4df)(__m256d)(A), \
                                                 (int)(imm), \
                                                 (__v2df)_mm_setzero_pd(), \
                                                 (__mmask8)(U)))

#define _mm256_extracti64x2_epi64(A, imm)                                      \
  ((__m128i)__builtin_ia32_extracti64x2_256_mask(                              \
      (__v4di)(__m256i)(A), (int)(imm), (__v2di)_mm_setzero_si128(),           \
      (__mmask8) - 1))

#define _mm256_mask_extracti64x2_epi64(W, U, A, imm) \
  ((__m128i)__builtin_ia32_extracti64x2_256_mask((__v4di)(__m256i)(A), \
````
- **L1057 EN**: Defines macro `_mm256_extractf64x2_pd(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1057 CN**: 定义宏 `_mm256_extractf64x2_pd(A, imm)`，用于条件编译、简写或 API 生成。
- **L1058 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf64x2_256_mask`.
  **L1058 CN**: 继续与可调用符号 `__builtin_ia32_extractf64x2_256_mask` 相关的逻辑。
- **L1059 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1059 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1060 EN**: Continues the surrounding expression or declaration: `(__mmask8) - 1))`.
  **L1060 CN**: 继续构造周围的表达式或声明：`(__mmask8) - 1))`。
- **L1061 EN**: Blank line separating nearby declarations or logic blocks.
  **L1061 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1062 EN**: Defines macro `_mm256_mask_extractf64x2_pd(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1062 CN**: 定义宏 `_mm256_mask_extractf64x2_pd(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1063 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf64x2_256_mask`.
  **L1063 CN**: 继续与可调用符号 `__builtin_ia32_extractf64x2_256_mask` 相关的逻辑。
- **L1064 EN**: Continues the surrounding expression or declaration: `(int)(imm), \`.
  **L1064 CN**: 继续构造周围的表达式或声明：`(int)(imm), \`。
- **L1065 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(W), \`.
  **L1065 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(W), \`。
- **L1066 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1066 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1067 EN**: Blank line separating nearby declarations or logic blocks.
  **L1067 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1068 EN**: Defines macro `_mm256_maskz_extractf64x2_pd(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1068 CN**: 定义宏 `_mm256_maskz_extractf64x2_pd(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1069 EN**: Continues logic associated with callable symbol `__builtin_ia32_extractf64x2_256_mask`.
  **L1069 CN**: 继续与可调用符号 `__builtin_ia32_extractf64x2_256_mask` 相关的逻辑。
- **L1070 EN**: Continues the surrounding expression or declaration: `(int)(imm), \`.
  **L1070 CN**: 继续构造周围的表达式或声明：`(int)(imm), \`。
- **L1071 EN**: Continues logic associated with callable symbol `_mm_setzero_pd`.
  **L1071 CN**: 继续与可调用符号 `_mm_setzero_pd` 相关的逻辑。
- **L1072 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1072 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1073 EN**: Blank line separating nearby declarations or logic blocks.
  **L1073 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1074 EN**: Defines macro `_mm256_extracti64x2_epi64(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1074 CN**: 定义宏 `_mm256_extracti64x2_epi64(A, imm)`，用于条件编译、简写或 API 生成。
- **L1075 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti64x2_256_mask`.
  **L1075 CN**: 继续与可调用符号 `__builtin_ia32_extracti64x2_256_mask` 相关的逻辑。
- **L1076 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L1076 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L1077 EN**: Continues the surrounding expression or declaration: `(__mmask8) - 1))`.
  **L1077 CN**: 继续构造周围的表达式或声明：`(__mmask8) - 1))`。
- **L1078 EN**: Blank line separating nearby declarations or logic blocks.
  **L1078 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1079 EN**: Defines macro `_mm256_mask_extracti64x2_epi64(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1079 CN**: 定义宏 `_mm256_mask_extracti64x2_epi64(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1080 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti64x2_256_mask`.
  **L1080 CN**: 继续与可调用符号 `__builtin_ia32_extracti64x2_256_mask` 相关的逻辑。

### Lines 1081-1104

````c
                                                 (int)(imm), \
                                                 (__v2di)(__m128i)(W), \
                                                 (__mmask8)(U)))

#define _mm256_maskz_extracti64x2_epi64(U, A, imm) \
  ((__m128i)__builtin_ia32_extracti64x2_256_mask((__v4di)(__m256i)(A), \
                                                 (int)(imm), \
                                                 (__v2di)_mm_setzero_si128(), \
                                                 (__mmask8)(U)))

#define _mm256_insertf64x2(A, B, imm) \
  ((__m256d)__builtin_ia32_insertf64x2_256((__v4df)(__m256d)(A), \
                                           (__v2df)(__m128d)(B), (int)(imm)))

#define _mm256_mask_insertf64x2(W, U, A, B, imm) \
  ((__m256d)__builtin_ia32_selectpd_256((__mmask8)(U), \
                                  (__v4df)_mm256_insertf64x2((A), (B), (imm)), \
                                  (__v4df)(__m256d)(W)))

#define _mm256_maskz_insertf64x2(U, A, B, imm) \
  ((__m256d)__builtin_ia32_selectpd_256((__mmask8)(U), \
                                  (__v4df)_mm256_insertf64x2((A), (B), (imm)), \
                                  (__v4df)_mm256_setzero_pd()))

````
- **L1081 EN**: Continues the surrounding expression or declaration: `(int)(imm), \`.
  **L1081 CN**: 继续构造周围的表达式或声明：`(int)(imm), \`。
- **L1082 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(W), \`.
  **L1082 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(W), \`。
- **L1083 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1083 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1085 EN**: Defines macro `_mm256_maskz_extracti64x2_epi64(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1085 CN**: 定义宏 `_mm256_maskz_extracti64x2_epi64(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1086 EN**: Continues logic associated with callable symbol `__builtin_ia32_extracti64x2_256_mask`.
  **L1086 CN**: 继续与可调用符号 `__builtin_ia32_extracti64x2_256_mask` 相关的逻辑。
- **L1087 EN**: Continues the surrounding expression or declaration: `(int)(imm), \`.
  **L1087 CN**: 继续构造周围的表达式或声明：`(int)(imm), \`。
- **L1088 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L1088 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L1089 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1089 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1090 EN**: Blank line separating nearby declarations or logic blocks.
  **L1090 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1091 EN**: Defines macro `_mm256_insertf64x2(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1091 CN**: 定义宏 `_mm256_insertf64x2(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1092 EN**: Continues logic associated with callable symbol `__builtin_ia32_insertf64x2_256`.
  **L1092 CN**: 继续与可调用符号 `__builtin_ia32_insertf64x2_256` 相关的逻辑。
- **L1093 EN**: Continues the surrounding expression or declaration: `(__v2df)(__m128d)(B), (int)(imm)))`.
  **L1093 CN**: 继续构造周围的表达式或声明：`(__v2df)(__m128d)(B), (int)(imm)))`。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1095 EN**: Defines macro `_mm256_mask_insertf64x2(W, U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1095 CN**: 定义宏 `_mm256_mask_insertf64x2(W, U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1096 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectpd_256`.
  **L1096 CN**: 继续与可调用符号 `__builtin_ia32_selectpd_256` 相关的逻辑。
- **L1097 EN**: Continues logic associated with callable symbol `_mm256_insertf64x2`.
  **L1097 CN**: 继续与可调用符号 `_mm256_insertf64x2` 相关的逻辑。
- **L1098 EN**: Continues the surrounding expression or declaration: `(__v4df)(__m256d)(W)))`.
  **L1098 CN**: 继续构造周围的表达式或声明：`(__v4df)(__m256d)(W)))`。
- **L1099 EN**: Blank line separating nearby declarations or logic blocks.
  **L1099 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1100 EN**: Defines macro `_mm256_maskz_insertf64x2(U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1100 CN**: 定义宏 `_mm256_maskz_insertf64x2(U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1101 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectpd_256`.
  **L1101 CN**: 继续与可调用符号 `__builtin_ia32_selectpd_256` 相关的逻辑。
- **L1102 EN**: Continues logic associated with callable symbol `_mm256_insertf64x2`.
  **L1102 CN**: 继续与可调用符号 `_mm256_insertf64x2` 相关的逻辑。
- **L1103 EN**: Continues logic associated with callable symbol `_mm256_setzero_pd`.
  **L1103 CN**: 继续与可调用符号 `_mm256_setzero_pd` 相关的逻辑。
- **L1104 EN**: Blank line separating nearby declarations or logic blocks.
  **L1104 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1105-1128

````c
#define _mm256_inserti64x2(A, B, imm) \
  ((__m256i)__builtin_ia32_inserti64x2_256((__v4di)(__m256i)(A), \
                                           (__v2di)(__m128i)(B), (int)(imm)))

#define _mm256_mask_inserti64x2(W, U, A, B, imm) \
  ((__m256i)__builtin_ia32_selectq_256((__mmask8)(U), \
                                   (__v4di)_mm256_inserti64x2((A), (B), (imm)), \
                                   (__v4di)(__m256i)(W)))

#define _mm256_maskz_inserti64x2(U, A, B, imm) \
  ((__m256i)__builtin_ia32_selectq_256((__mmask8)(U), \
                                   (__v4di)_mm256_inserti64x2((A), (B), (imm)), \
                                   (__v4di)_mm256_setzero_si256()))

#define _mm_mask_fpclass_pd_mask(U, A, imm) \
  ((__mmask8)__builtin_ia32_fpclasspd128_mask((__v2df)(__m128d)(A), (int)(imm), \
                                              (__mmask8)(U)))

#define _mm_fpclass_pd_mask(A, imm) \
  ((__mmask8)__builtin_ia32_fpclasspd128_mask((__v2df)(__m128d)(A), (int)(imm), \
                                              (__mmask8)-1))

#define _mm256_mask_fpclass_pd_mask(U, A, imm) \
  ((__mmask8)__builtin_ia32_fpclasspd256_mask((__v4df)(__m256d)(A), (int)(imm), \
````
- **L1105 EN**: Defines macro `_mm256_inserti64x2(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1105 CN**: 定义宏 `_mm256_inserti64x2(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1106 EN**: Continues logic associated with callable symbol `__builtin_ia32_inserti64x2_256`.
  **L1106 CN**: 继续与可调用符号 `__builtin_ia32_inserti64x2_256` 相关的逻辑。
- **L1107 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(B), (int)(imm)))`.
  **L1107 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(B), (int)(imm)))`。
- **L1108 EN**: Blank line separating nearby declarations or logic blocks.
  **L1108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1109 EN**: Defines macro `_mm256_mask_inserti64x2(W, U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1109 CN**: 定义宏 `_mm256_mask_inserti64x2(W, U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1110 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_256`.
  **L1110 CN**: 继续与可调用符号 `__builtin_ia32_selectq_256` 相关的逻辑。
- **L1111 EN**: Continues logic associated with callable symbol `_mm256_inserti64x2`.
  **L1111 CN**: 继续与可调用符号 `_mm256_inserti64x2` 相关的逻辑。
- **L1112 EN**: Continues the surrounding expression or declaration: `(__v4di)(__m256i)(W)))`.
  **L1112 CN**: 继续构造周围的表达式或声明：`(__v4di)(__m256i)(W)))`。
- **L1113 EN**: Blank line separating nearby declarations or logic blocks.
  **L1113 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1114 EN**: Defines macro `_mm256_maskz_inserti64x2(U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L1114 CN**: 定义宏 `_mm256_maskz_inserti64x2(U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L1115 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_256`.
  **L1115 CN**: 继续与可调用符号 `__builtin_ia32_selectq_256` 相关的逻辑。
- **L1116 EN**: Continues logic associated with callable symbol `_mm256_inserti64x2`.
  **L1116 CN**: 继续与可调用符号 `_mm256_inserti64x2` 相关的逻辑。
- **L1117 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L1117 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L1118 EN**: Blank line separating nearby declarations or logic blocks.
  **L1118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1119 EN**: Defines macro `_mm_mask_fpclass_pd_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1119 CN**: 定义宏 `_mm_mask_fpclass_pd_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1120 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasspd128_mask`.
  **L1120 CN**: 继续与可调用符号 `__builtin_ia32_fpclasspd128_mask` 相关的逻辑。
- **L1121 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1121 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1122 EN**: Blank line separating nearby declarations or logic blocks.
  **L1122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1123 EN**: Defines macro `_mm_fpclass_pd_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1123 CN**: 定义宏 `_mm_fpclass_pd_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1124 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasspd128_mask`.
  **L1124 CN**: 继续与可调用符号 `__builtin_ia32_fpclasspd128_mask` 相关的逻辑。
- **L1125 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L1125 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L1126 EN**: Blank line separating nearby declarations or logic blocks.
  **L1126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1127 EN**: Defines macro `_mm256_mask_fpclass_pd_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1127 CN**: 定义宏 `_mm256_mask_fpclass_pd_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1128 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasspd256_mask`.
  **L1128 CN**: 继续与可调用符号 `__builtin_ia32_fpclasspd256_mask` 相关的逻辑。

### Lines 1129-1152

````c
                                              (__mmask8)(U)))

#define _mm256_fpclass_pd_mask(A, imm) \
  ((__mmask8)__builtin_ia32_fpclasspd256_mask((__v4df)(__m256d)(A), (int)(imm), \
                                              (__mmask8)-1))

#define _mm_mask_fpclass_ps_mask(U, A, imm) \
  ((__mmask8)__builtin_ia32_fpclassps128_mask((__v4sf)(__m128)(A), (int)(imm), \
                                              (__mmask8)(U)))

#define _mm_fpclass_ps_mask(A, imm) \
  ((__mmask8)__builtin_ia32_fpclassps128_mask((__v4sf)(__m128)(A), (int)(imm), \
                                              (__mmask8)-1))

#define _mm256_mask_fpclass_ps_mask(U, A, imm) \
  ((__mmask8)__builtin_ia32_fpclassps256_mask((__v8sf)(__m256)(A), (int)(imm), \
                                              (__mmask8)(U)))

#define _mm256_fpclass_ps_mask(A, imm) \
  ((__mmask8)__builtin_ia32_fpclassps256_mask((__v8sf)(__m256)(A), (int)(imm), \
                                              (__mmask8)-1))

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256
````
- **L1129 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1129 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1131 EN**: Defines macro `_mm256_fpclass_pd_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1131 CN**: 定义宏 `_mm256_fpclass_pd_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1132 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclasspd256_mask`.
  **L1132 CN**: 继续与可调用符号 `__builtin_ia32_fpclasspd256_mask` 相关的逻辑。
- **L1133 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L1133 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L1134 EN**: Blank line separating nearby declarations or logic blocks.
  **L1134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1135 EN**: Defines macro `_mm_mask_fpclass_ps_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1135 CN**: 定义宏 `_mm_mask_fpclass_ps_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1136 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassps128_mask`.
  **L1136 CN**: 继续与可调用符号 `__builtin_ia32_fpclassps128_mask` 相关的逻辑。
- **L1137 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1137 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1138 EN**: Blank line separating nearby declarations or logic blocks.
  **L1138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1139 EN**: Defines macro `_mm_fpclass_ps_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1139 CN**: 定义宏 `_mm_fpclass_ps_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1140 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassps128_mask`.
  **L1140 CN**: 继续与可调用符号 `__builtin_ia32_fpclassps128_mask` 相关的逻辑。
- **L1141 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L1141 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L1142 EN**: Blank line separating nearby declarations or logic blocks.
  **L1142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1143 EN**: Defines macro `_mm256_mask_fpclass_ps_mask(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1143 CN**: 定义宏 `_mm256_mask_fpclass_ps_mask(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1144 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassps256_mask`.
  **L1144 CN**: 继续与可调用符号 `__builtin_ia32_fpclassps256_mask` 相关的逻辑。
- **L1145 EN**: Continues the surrounding expression or declaration: `(__mmask8)(U)))`.
  **L1145 CN**: 继续构造周围的表达式或声明：`(__mmask8)(U)))`。
- **L1146 EN**: Blank line separating nearby declarations or logic blocks.
  **L1146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1147 EN**: Defines macro `_mm256_fpclass_ps_mask(A, imm)` for conditional compilation, shorthand, or API generation.
  **L1147 CN**: 定义宏 `_mm256_fpclass_ps_mask(A, imm)`，用于条件编译、简写或 API 生成。
- **L1148 EN**: Continues logic associated with callable symbol `__builtin_ia32_fpclassps256_mask`.
  **L1148 CN**: 继续与可调用符号 `__builtin_ia32_fpclassps256_mask` 相关的逻辑。
- **L1149 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L1149 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1151 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L1151 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L1152 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L1152 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。

### Lines 1153-1156

````c
#undef __DEFAULT_FN_ATTRS256_CONSTEXPR
#undef __DEFAULT_FN_ATTRS128_CONSTEXPR

#endif
````
- **L1153 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1153 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1154 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1154 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1155 EN**: Blank line separating nearby declarations or logic blocks.
  **L1155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1156 EN**: Closes the current preprocessor conditional block.
  **L1156 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **x86 SIMD or system intrinsics / x86 SIMD 或系统 intrinsic 接口**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Inline wrapper functions / 内联包装函数**
- **Compiler-specific attributes / 编译器特定属性**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__AVX512VLDQINTRIN_H`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_ia32_selectq_256`, `__builtin_ia32_selectq_128`, `__builtin_ia32_selectpd_256`, `__builtin_ia32_selectpd_128`, `__builtin_ia32_selectps_256`, `__builtin_ia32_selectps_128`, `__builtin_ia32_cvtpd2qq128_mask`, `__builtin_ia32_cvtpd2qq256_mask`, `__builtin_ia32_cvtpd2uqq128_mask`, `__builtin_ia32_cvtpd2uqq256_mask`, `__builtin_ia32_cvtps2qq128_mask`, `__builtin_ia32_cvtps2qq256_mask`
