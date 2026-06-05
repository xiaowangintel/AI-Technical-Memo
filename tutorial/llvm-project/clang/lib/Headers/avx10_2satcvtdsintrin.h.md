# avx10_2satcvtdsintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx10_2satcvtdsintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: AVX512SATCVTDS intrinsics.
- **Purpose (CN)**: 提供 AVX512SATCVTDS intrinsic 接口。
- **Line Count / 行数**: 376

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*===----------- avx10_2satcvtdsintrin.h - AVX512SATCVTDS intrinsics --------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __IMMINTRIN_H
#error                                                                         \
    "Never use <avx10_2satcvtdsintrin.h> directly; include <immintrin.h> instead."
#endif // __IMMINTRIN_H

#ifndef __AVX10_2SATCVTDSINTRIN_H
#define __AVX10_2SATCVTDSINTRIN_H

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
- **L9 EN**: Blank line separating nearby declarations or logic blocks.
  **L9 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __IMMINTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __IMMINTRIN_H`。
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error                                                                         \`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error                                                                         \`。
- **L12 EN**: Continues the surrounding expression or declaration: `"Never use <avx10_2satcvtdsintrin.h> directly; include <immintrin.h> instead."`.
  **L12 CN**: 继续构造周围的表达式或声明：`"Never use <avx10_2satcvtdsintrin.h> directly; include <immintrin.h> instead."`。
- **L13 EN**: Closes the current preprocessor conditional block.
  **L13 CN**: 结束当前预处理条件块。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Starts a preprocessor conditional block: `#ifndef __AVX10_2SATCVTDSINTRIN_H`.
  **L15 CN**: 开始一个预处理条件块：`#ifndef __AVX10_2SATCVTDSINTRIN_H`。
- **L16 EN**: Defines macro `__AVX10_2SATCVTDSINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L16 CN**: 定义宏 `__AVX10_2SATCVTDSINTRIN_H`，用于条件编译、简写或 API 生成。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `Define the default attributes for the functions in this file.`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define the default attributes for the functions in this file.`。
- **L19 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L19 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L20 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`.
  **L20 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`。

### Lines 21-40

````c
                 __min_vector_width__(256)))

#define __DEFAULT_FN_ATTRS128                                                  \
  __attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \
                 __min_vector_width__(128)))

#define _mm_cvtts_roundsd_i32(__A, __R)                                        \
  ((int)__builtin_ia32_vcvttsd2sis32((__v2df)(__m128)(__A), (const int)(__R)))

#define _mm_cvtts_roundsd_si32(__A, __R)                                       \
  ((int)__builtin_ia32_vcvttsd2sis32((__v2df)(__m128d)(__A), (const int)(__R)))

#define _mm_cvtts_roundsd_u32(__A, __R)                                        \
  ((unsigned int)__builtin_ia32_vcvttsd2usis32((__v2df)(__m128d)(__A),         \
                                               (const int)(__R)))

#define _mm_cvtts_roundss_i32(__A, __R)                                        \
  ((int)__builtin_ia32_vcvttss2sis32((__v4sf)(__m128)(__A), (const int)(__R)))

#define _mm_cvtts_roundss_si32(__A, __R)                                       \
````
- **L21 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L21 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Defines macro `__DEFAULT_FN_ATTRS128` for conditional compilation, shorthand, or API generation.
  **L23 CN**: 定义宏 `__DEFAULT_FN_ATTRS128`，用于条件编译、简写或 API 生成。
- **L24 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`.
  **L24 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("avx10.2"),        \`。
- **L25 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L25 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L27 EN**: Defines macro `_mm_cvtts_roundsd_i32(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L27 CN**: 定义宏 `_mm_cvtts_roundsd_i32(__A, __R)`，用于条件编译、简写或 API 生成。
- **L28 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsd2sis32`.
  **L28 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsd2sis32` 相关的逻辑。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L30 EN**: Defines macro `_mm_cvtts_roundsd_si32(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L30 CN**: 定义宏 `_mm_cvtts_roundsd_si32(__A, __R)`，用于条件编译、简写或 API 生成。
- **L31 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsd2sis32`.
  **L31 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsd2sis32` 相关的逻辑。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L33 EN**: Defines macro `_mm_cvtts_roundsd_u32(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L33 CN**: 定义宏 `_mm_cvtts_roundsd_u32(__A, __R)`，用于条件编译、简写或 API 生成。
- **L34 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsd2usis32`.
  **L34 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsd2usis32` 相关的逻辑。
- **L35 EN**: Continues the surrounding expression or declaration: `(const int)(__R)))`.
  **L35 CN**: 继续构造周围的表达式或声明：`(const int)(__R)))`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L37 EN**: Defines macro `_mm_cvtts_roundss_i32(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L37 CN**: 定义宏 `_mm_cvtts_roundss_i32(__A, __R)`，用于条件编译、简写或 API 生成。
- **L38 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttss2sis32`.
  **L38 CN**: 继续与可调用符号 `__builtin_ia32_vcvttss2sis32` 相关的逻辑。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L40 EN**: Defines macro `_mm_cvtts_roundss_si32(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L40 CN**: 定义宏 `_mm_cvtts_roundss_si32(__A, __R)`，用于条件编译、简写或 API 生成。

### Lines 41-60

````c
  ((int)__builtin_ia32_vcvttss2sis32((__v4sf)(__m128)(__A), (const int)(__R)))

#define _mm_cvtts_roundss_u32(__A, __R)                                        \
  ((unsigned int)__builtin_ia32_vcvttss2usis32((__v4sf)(__m128)(__A),          \
                                               (const int)(__R)))

#ifdef __x86_64__
#define _mm_cvtts_roundss_u64(__A, __R)                                        \
  ((unsigned long long)__builtin_ia32_vcvttss2usis64((__v4sf)(__m128)(__A),    \
                                                     (const int)(__R)))

#define _mm_cvtts_roundsd_u64(__A, __R)                                        \
  ((unsigned long long)__builtin_ia32_vcvttsd2usis64((__v2df)(__m128d)(__A),   \
                                                     (const int)(__R)))

#define _mm_cvtts_roundss_i64(__A, __R)                                        \
  ((long long)__builtin_ia32_vcvttss2sis64((__v4sf)(__m128)(__A),              \
                                           (const int)(__R)))

#define _mm_cvtts_roundss_si64(__A, __R)                                       \
````
- **L41 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttss2sis32`.
  **L41 CN**: 继续与可调用符号 `__builtin_ia32_vcvttss2sis32` 相关的逻辑。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L43 EN**: Defines macro `_mm_cvtts_roundss_u32(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L43 CN**: 定义宏 `_mm_cvtts_roundss_u32(__A, __R)`，用于条件编译、简写或 API 生成。
- **L44 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttss2usis32`.
  **L44 CN**: 继续与可调用符号 `__builtin_ia32_vcvttss2usis32` 相关的逻辑。
- **L45 EN**: Continues the surrounding expression or declaration: `(const int)(__R)))`.
  **L45 CN**: 继续构造周围的表达式或声明：`(const int)(__R)))`。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Starts a preprocessor conditional block: `#ifdef __x86_64__`.
  **L47 CN**: 开始一个预处理条件块：`#ifdef __x86_64__`。
- **L48 EN**: Defines macro `_mm_cvtts_roundss_u64(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L48 CN**: 定义宏 `_mm_cvtts_roundss_u64(__A, __R)`，用于条件编译、简写或 API 生成。
- **L49 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttss2usis64`.
  **L49 CN**: 继续与可调用符号 `__builtin_ia32_vcvttss2usis64` 相关的逻辑。
- **L50 EN**: Continues the surrounding expression or declaration: `(const int)(__R)))`.
  **L50 CN**: 继续构造周围的表达式或声明：`(const int)(__R)))`。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Defines macro `_mm_cvtts_roundsd_u64(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L52 CN**: 定义宏 `_mm_cvtts_roundsd_u64(__A, __R)`，用于条件编译、简写或 API 生成。
- **L53 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsd2usis64`.
  **L53 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsd2usis64` 相关的逻辑。
- **L54 EN**: Continues the surrounding expression or declaration: `(const int)(__R)))`.
  **L54 CN**: 继续构造周围的表达式或声明：`(const int)(__R)))`。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L56 EN**: Defines macro `_mm_cvtts_roundss_i64(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L56 CN**: 定义宏 `_mm_cvtts_roundss_i64(__A, __R)`，用于条件编译、简写或 API 生成。
- **L57 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttss2sis64`.
  **L57 CN**: 继续与可调用符号 `__builtin_ia32_vcvttss2sis64` 相关的逻辑。
- **L58 EN**: Continues the surrounding expression or declaration: `(const int)(__R)))`.
  **L58 CN**: 继续构造周围的表达式或声明：`(const int)(__R)))`。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L60 EN**: Defines macro `_mm_cvtts_roundss_si64(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L60 CN**: 定义宏 `_mm_cvtts_roundss_si64(__A, __R)`，用于条件编译、简写或 API 生成。

### Lines 61-80

````c
  ((long long)__builtin_ia32_vcvttss2sis64((__v4sf)(__m128)(__A),              \
                                           (const int)(__R)))

#define _mm_cvtts_roundsd_si64(__A, __R)                                       \
  ((long long)__builtin_ia32_vcvttsd2sis64((__v2df)(__m128d)(__A),             \
                                           (const int)(__R)))

#define _mm_cvtts_roundsd_i64(__A, __R)                                        \
  ((long long)__builtin_ia32_vcvttsd2sis64((__v2df)(__m128d)(__A),             \
                                           (const int)(__R)))
#endif /* __x86_64__ */

// 128 Bit : Double -> int
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtts_pd_epi32(__m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2dqs128_mask(
      (__v2df)__A, (__v4si)(__m128i)_mm_undefined_si128(), (__mmask8)(-1)));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
````
- **L61 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttss2sis64`.
  **L61 CN**: 继续与可调用符号 `__builtin_ia32_vcvttss2sis64` 相关的逻辑。
- **L62 EN**: Continues the surrounding expression or declaration: `(const int)(__R)))`.
  **L62 CN**: 继续构造周围的表达式或声明：`(const int)(__R)))`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Defines macro `_mm_cvtts_roundsd_si64(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L64 CN**: 定义宏 `_mm_cvtts_roundsd_si64(__A, __R)`，用于条件编译、简写或 API 生成。
- **L65 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsd2sis64`.
  **L65 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsd2sis64` 相关的逻辑。
- **L66 EN**: Continues the surrounding expression or declaration: `(const int)(__R)))`.
  **L66 CN**: 继续构造周围的表达式或声明：`(const int)(__R)))`。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L68 EN**: Defines macro `_mm_cvtts_roundsd_i64(__A, __R)` for conditional compilation, shorthand, or API generation.
  **L68 CN**: 定义宏 `_mm_cvtts_roundsd_i64(__A, __R)`，用于条件编译、简写或 API 生成。
- **L69 EN**: Continues logic associated with callable symbol `__builtin_ia32_vcvttsd2sis64`.
  **L69 CN**: 继续与可调用符号 `__builtin_ia32_vcvttsd2sis64` 相关的逻辑。
- **L70 EN**: Continues the surrounding expression or declaration: `(const int)(__R)))`.
  **L70 CN**: 继续构造周围的表达式或声明：`(const int)(__R)))`。
- **L71 EN**: Closes the current preprocessor conditional block.
  **L71 CN**: 结束当前预处理条件块。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L73 EN**: Comment explains nearby logic, constraints, or intent: `128 Bit : Double -> int`.
  **L73 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128 Bit : Double -> int`。
- **L74 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L74 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L75 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtts_pd_epi32(__m128d __A) {`.
  **L75 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtts_pd_epi32(__m128d __A) {`。
- **L76 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2dqs128_mask(`.
  **L76 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2dqs128_mask(` 从当前函数返回。
- **L77 EN**: Executes a call or declaration centered on `statement`.
  **L77 CN**: 执行以 `statement` 为核心的调用或声明。
- **L78 EN**: Closes the current lexical scope or compound statement.
  **L78 CN**: 结束当前词法作用域或复合语句块。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L80 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L80 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。

### Lines 81-100

````c
_mm_mask_cvtts_pd_epi32(__m128i __W, __mmask8 __U, __m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2dqs128_mask((__v2df)__A, (__v4si)__W,
                                                      __U));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtts_pd_epi32(__mmask16 __U, __m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2dqs128_mask(
      (__v2df)__A, (__v4si)(__m128i)_mm_setzero_si128(), __U));
}

// 256 Bit : Double -> int
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvtts_pd_epi32(__m256d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2dqs256_mask(
      (__v4df)__A, (__v4si)_mm_undefined_si128(), (__mmask8)-1));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtts_pd_epi32(__m128i __W, __mmask8 __U, __m256d __A) {
````
- **L81 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtts_pd_epi32(__m128i __W, __mmask8 __U, __m128d __A) {`.
  **L81 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtts_pd_epi32(__m128i __W, __mmask8 __U, __m128d __A) {`。
- **L82 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2dqs128_mask((__v2df)__A, (__v4si)__W,`.
  **L82 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2dqs128_mask((__v2df)__A, (__v4si)__W,` 从当前函数返回。
- **L83 EN**: Adds a standalone statement or declaration: `__U));`.
  **L83 CN**: 添加一条独立语句或声明：`__U));`。
- **L84 EN**: Closes the current lexical scope or compound statement.
  **L84 CN**: 结束当前词法作用域或复合语句块。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L86 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L86 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L87 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtts_pd_epi32(__mmask16 __U, __m128d __A) {`.
  **L87 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtts_pd_epi32(__mmask16 __U, __m128d __A) {`。
- **L88 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2dqs128_mask(`.
  **L88 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2dqs128_mask(` 从当前函数返回。
- **L89 EN**: Executes a call or declaration centered on `statement`.
  **L89 CN**: 执行以 `statement` 为核心的调用或声明。
- **L90 EN**: Closes the current lexical scope or compound statement.
  **L90 CN**: 结束当前词法作用域或复合语句块。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L92 EN**: Comment explains nearby logic, constraints, or intent: `256 Bit : Double -> int`.
  **L92 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256 Bit : Double -> int`。
- **L93 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L93 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L94 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtts_pd_epi32(__m256d __A) {`.
  **L94 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtts_pd_epi32(__m256d __A) {`。
- **L95 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2dqs256_mask(`.
  **L95 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2dqs256_mask(` 从当前函数返回。
- **L96 EN**: Executes a call or declaration centered on `statement`.
  **L96 CN**: 执行以 `statement` 为核心的调用或声明。
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L99 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L99 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L100 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtts_pd_epi32(__m128i __W, __mmask8 __U, __m256d __A) {`.
  **L100 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtts_pd_epi32(__m128i __W, __mmask8 __U, __m256d __A) {`。

### Lines 101-120

````c
  return ((__m128i)__builtin_ia32_vcvttpd2dqs256_mask((__v4df)__A, (__v4si)__W,
                                                      __U));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtts_pd_epi32(__mmask8 __U, __m256d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2dqs256_mask(
      (__v4df)__A, (__v4si)_mm_setzero_si128(), __U));
}

// 128 Bit : Double -> uint
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtts_pd_epu32(__m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(
      (__v2df)__A, (__v4si)(__m128i)_mm_undefined_si128(), (__mmask8)(-1)));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtts_pd_epu32(__m128i __W, __mmask8 __U, __m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(
````
- **L101 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2dqs256_mask((__v4df)__A, (__v4si)__W,`.
  **L101 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2dqs256_mask((__v4df)__A, (__v4si)__W,` 从当前函数返回。
- **L102 EN**: Adds a standalone statement or declaration: `__U));`.
  **L102 CN**: 添加一条独立语句或声明：`__U));`。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L105 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L106 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtts_pd_epi32(__mmask8 __U, __m256d __A) {`.
  **L106 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtts_pd_epi32(__mmask8 __U, __m256d __A) {`。
- **L107 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2dqs256_mask(`.
  **L107 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2dqs256_mask(` 从当前函数返回。
- **L108 EN**: Executes a call or declaration centered on `statement`.
  **L108 CN**: 执行以 `statement` 为核心的调用或声明。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L111 EN**: Comment explains nearby logic, constraints, or intent: `128 Bit : Double -> uint`.
  **L111 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128 Bit : Double -> uint`。
- **L112 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L112 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L113 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtts_pd_epu32(__m128d __A) {`.
  **L113 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtts_pd_epu32(__m128d __A) {`。
- **L114 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(`.
  **L114 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(` 从当前函数返回。
- **L115 EN**: Executes a call or declaration centered on `statement`.
  **L115 CN**: 执行以 `statement` 为核心的调用或声明。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L118 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L118 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L119 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtts_pd_epu32(__m128i __W, __mmask8 __U, __m128d __A) {`.
  **L119 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtts_pd_epu32(__m128i __W, __mmask8 __U, __m128d __A) {`。
- **L120 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(`.
  **L120 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(` 从当前函数返回。

### Lines 121-140

````c
      (__v2df)__A, (__v4si)(__m128i)__W, (__mmask8)__U));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtts_pd_epu32(__mmask8 __U, __m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(
      (__v2df)__A, (__v4si)(__m128i)_mm_setzero_si128(), __U));
}

// 256 Bit : Double -> uint
static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvtts_pd_epu32(__m256d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2udqs256_mask(
      (__v4df)__A, (__v4si)_mm_undefined_si128(), (__mmask8)-1));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtts_pd_epu32(__m128i __W, __mmask8 __U, __m256d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2udqs256_mask((__v4df)__A, (__v4si)__W,
                                                       __U));
````
- **L121 EN**: Executes a call or declaration centered on `statement`.
  **L121 CN**: 执行以 `statement` 为核心的调用或声明。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Blank line separating nearby declarations or logic blocks.
  **L123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L124 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L124 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtts_pd_epu32(__mmask8 __U, __m128d __A) {`.
  **L125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtts_pd_epu32(__mmask8 __U, __m128d __A) {`。
- **L126 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(`.
  **L126 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2udqs128_mask(` 从当前函数返回。
- **L127 EN**: Executes a call or declaration centered on `statement`.
  **L127 CN**: 执行以 `statement` 为核心的调用或声明。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L130 EN**: Comment explains nearby logic, constraints, or intent: `256 Bit : Double -> uint`.
  **L130 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256 Bit : Double -> uint`。
- **L131 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L131 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L132 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtts_pd_epu32(__m256d __A) {`.
  **L132 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtts_pd_epu32(__m256d __A) {`。
- **L133 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2udqs256_mask(`.
  **L133 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2udqs256_mask(` 从当前函数返回。
- **L134 EN**: Executes a call or declaration centered on `statement`.
  **L134 CN**: 执行以 `statement` 为核心的调用或声明。
- **L135 EN**: Closes the current lexical scope or compound statement.
  **L135 CN**: 结束当前词法作用域或复合语句块。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L137 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L137 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L138 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtts_pd_epu32(__m128i __W, __mmask8 __U, __m256d __A) {`.
  **L138 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtts_pd_epu32(__m128i __W, __mmask8 __U, __m256d __A) {`。
- **L139 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2udqs256_mask((__v4df)__A, (__v4si)__W,`.
  **L139 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2udqs256_mask((__v4df)__A, (__v4si)__W,` 从当前函数返回。
- **L140 EN**: Adds a standalone statement or declaration: `__U));`.
  **L140 CN**: 添加一条独立语句或声明：`__U));`。

### Lines 141-160

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtts_pd_epu32(__mmask8 __U, __m256d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2udqs256_mask(
      (__v4df)__A, (__v4si)_mm_setzero_si128(), __U));
}

// 128 Bit : Double -> long
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtts_pd_epi64(__m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2qqs128_mask(
      (__v2df)__A, (__v2di)_mm_undefined_si128(), (__mmask8)-1));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtts_pd_epi64(__m128i __W, __mmask8 __U, __m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2qqs128_mask((__v2df)__A, (__v2di)__W,
                                                      (__mmask8)__U));
}
````
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L143 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L143 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L144 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtts_pd_epu32(__mmask8 __U, __m256d __A) {`.
  **L144 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtts_pd_epu32(__mmask8 __U, __m256d __A) {`。
- **L145 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2udqs256_mask(`.
  **L145 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2udqs256_mask(` 从当前函数返回。
- **L146 EN**: Executes a call or declaration centered on `statement`.
  **L146 CN**: 执行以 `statement` 为核心的调用或声明。
- **L147 EN**: Closes the current lexical scope or compound statement.
  **L147 CN**: 结束当前词法作用域或复合语句块。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L149 EN**: Comment explains nearby logic, constraints, or intent: `128 Bit : Double -> long`.
  **L149 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128 Bit : Double -> long`。
- **L150 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L150 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L151 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtts_pd_epi64(__m128d __A) {`.
  **L151 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtts_pd_epi64(__m128d __A) {`。
- **L152 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2qqs128_mask(`.
  **L152 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2qqs128_mask(` 从当前函数返回。
- **L153 EN**: Executes a call or declaration centered on `statement`.
  **L153 CN**: 执行以 `statement` 为核心的调用或声明。
- **L154 EN**: Closes the current lexical scope or compound statement.
  **L154 CN**: 结束当前词法作用域或复合语句块。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L156 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L156 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L157 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtts_pd_epi64(__m128i __W, __mmask8 __U, __m128d __A) {`.
  **L157 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtts_pd_epi64(__m128i __W, __mmask8 __U, __m128d __A) {`。
- **L158 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2qqs128_mask((__v2df)__A, (__v2di)__W,`.
  **L158 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2qqs128_mask((__v2df)__A, (__v2di)__W,` 从当前函数返回。
- **L159 EN**: Executes a call or declaration centered on `statement`.
  **L159 CN**: 执行以 `statement` 为核心的调用或声明。
- **L160 EN**: Closes the current lexical scope or compound statement.
  **L160 CN**: 结束当前词法作用域或复合语句块。

### Lines 161-180

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtts_pd_epi64(__mmask8 __U, __m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2qqs128_mask(
      (__v2df)__A, (__v2di)_mm_setzero_si128(), (__mmask8)__U));
}

// 256 Bit : Double -> long
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtts_pd_epi64(__m256d __A) {
  return ((__m256i)__builtin_ia32_vcvttpd2qqs256_mask(
      (__v4df)__A, (__v4di)_mm256_undefined_si256(), (__mmask8)-1));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtts_pd_epi64(__m256i __W, __mmask8 __U, __m256d __A) {
  return ((__m256i)__builtin_ia32_vcvttpd2qqs256_mask((__v4df)__A, (__v4di)__W,
                                                      __U));
}

````
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L162 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtts_pd_epi64(__mmask8 __U, __m128d __A) {`.
  **L163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtts_pd_epi64(__mmask8 __U, __m128d __A) {`。
- **L164 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2qqs128_mask(`.
  **L164 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2qqs128_mask(` 从当前函数返回。
- **L165 EN**: Executes a call or declaration centered on `statement`.
  **L165 CN**: 执行以 `statement` 为核心的调用或声明。
- **L166 EN**: Closes the current lexical scope or compound statement.
  **L166 CN**: 结束当前词法作用域或复合语句块。
- **L167 EN**: Blank line separating nearby declarations or logic blocks.
  **L167 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L168 EN**: Comment explains nearby logic, constraints, or intent: `256 Bit : Double -> long`.
  **L168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256 Bit : Double -> long`。
- **L169 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L169 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L170 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtts_pd_epi64(__m256d __A) {`.
  **L170 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtts_pd_epi64(__m256d __A) {`。
- **L171 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttpd2qqs256_mask(`.
  **L171 CN**: 以 `((__m256i)__builtin_ia32_vcvttpd2qqs256_mask(` 从当前函数返回。
- **L172 EN**: Executes a call or declaration centered on `statement`.
  **L172 CN**: 执行以 `statement` 为核心的调用或声明。
- **L173 EN**: Closes the current lexical scope or compound statement.
  **L173 CN**: 结束当前词法作用域或复合语句块。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L175 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L175 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L176 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtts_pd_epi64(__m256i __W, __mmask8 __U, __m256d __A) {`.
  **L176 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtts_pd_epi64(__m256i __W, __mmask8 __U, __m256d __A) {`。
- **L177 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttpd2qqs256_mask((__v4df)__A, (__v4di)__W,`.
  **L177 CN**: 以 `((__m256i)__builtin_ia32_vcvttpd2qqs256_mask((__v4df)__A, (__v4di)__W,` 从当前函数返回。
- **L178 EN**: Adds a standalone statement or declaration: `__U));`.
  **L178 CN**: 添加一条独立语句或声明：`__U));`。
- **L179 EN**: Closes the current lexical scope or compound statement.
  **L179 CN**: 结束当前词法作用域或复合语句块。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 181-200

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtts_pd_epi64(__mmask8 __U, __m256d __A) {
  return ((__m256i)__builtin_ia32_vcvttpd2qqs256_mask(
      (__v4df)__A, (__v4di)_mm256_setzero_si256(), __U));
}

// 128 Bit : Double -> ulong
static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtts_pd_epu64(__m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask(
      (__v2df)__A, (__v2di)_mm_undefined_si128(), (__mmask8)-1));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtts_pd_epu64(__m128i __W, __mmask8 __U, __m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask((__v2df)__A, (__v2di)__W,
                                                       (__mmask8)__U));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
````
- **L181 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L181 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L182 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtts_pd_epi64(__mmask8 __U, __m256d __A) {`.
  **L182 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtts_pd_epi64(__mmask8 __U, __m256d __A) {`。
- **L183 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttpd2qqs256_mask(`.
  **L183 CN**: 以 `((__m256i)__builtin_ia32_vcvttpd2qqs256_mask(` 从当前函数返回。
- **L184 EN**: Executes a call or declaration centered on `statement`.
  **L184 CN**: 执行以 `statement` 为核心的调用或声明。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Blank line separating nearby declarations or logic blocks.
  **L186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L187 EN**: Comment explains nearby logic, constraints, or intent: `128 Bit : Double -> ulong`.
  **L187 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128 Bit : Double -> ulong`。
- **L188 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L188 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L189 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtts_pd_epu64(__m128d __A) {`.
  **L189 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtts_pd_epu64(__m128d __A) {`。
- **L190 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask(`.
  **L190 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask(` 从当前函数返回。
- **L191 EN**: Executes a call or declaration centered on `statement`.
  **L191 CN**: 执行以 `statement` 为核心的调用或声明。
- **L192 EN**: Closes the current lexical scope or compound statement.
  **L192 CN**: 结束当前词法作用域或复合语句块。
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L194 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L194 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L195 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtts_pd_epu64(__m128i __W, __mmask8 __U, __m128d __A) {`.
  **L195 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtts_pd_epu64(__m128i __W, __mmask8 __U, __m128d __A) {`。
- **L196 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask((__v2df)__A, (__v2di)__W,`.
  **L196 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask((__v2df)__A, (__v2di)__W,` 从当前函数返回。
- **L197 EN**: Executes a call or declaration centered on `statement`.
  **L197 CN**: 执行以 `statement` 为核心的调用或声明。
- **L198 EN**: Closes the current lexical scope or compound statement.
  **L198 CN**: 结束当前词法作用域或复合语句块。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L200 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L200 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。

### Lines 201-220

````c
_mm_maskz_cvtts_pd_epu64(__mmask8 __U, __m128d __A) {
  return ((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask(
      (__v2df)__A, (__v2di)_mm_setzero_si128(), (__mmask8)__U));
}

// 256 Bit : Double -> ulong

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtts_pd_epu64(__m256d __A) {
  return ((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask(
      (__v4df)__A, (__v4di)_mm256_undefined_si256(), (__mmask8)-1));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtts_pd_epu64(__m256i __W, __mmask8 __U, __m256d __A) {
  return ((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask((__v4df)__A, (__v4di)__W,
                                                       __U));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
````
- **L201 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtts_pd_epu64(__mmask8 __U, __m128d __A) {`.
  **L201 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtts_pd_epu64(__mmask8 __U, __m128d __A) {`。
- **L202 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask(`.
  **L202 CN**: 以 `((__m128i)__builtin_ia32_vcvttpd2uqqs128_mask(` 从当前函数返回。
- **L203 EN**: Executes a call or declaration centered on `statement`.
  **L203 CN**: 执行以 `statement` 为核心的调用或声明。
- **L204 EN**: Closes the current lexical scope or compound statement.
  **L204 CN**: 结束当前词法作用域或复合语句块。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Comment explains nearby logic, constraints, or intent: `256 Bit : Double -> ulong`.
  **L206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256 Bit : Double -> ulong`。
- **L207 EN**: Blank line separating nearby declarations or logic blocks.
  **L207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L208 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L208 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L209 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtts_pd_epu64(__m256d __A) {`.
  **L209 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtts_pd_epu64(__m256d __A) {`。
- **L210 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask(`.
  **L210 CN**: 以 `((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask(` 从当前函数返回。
- **L211 EN**: Executes a call or declaration centered on `statement`.
  **L211 CN**: 执行以 `statement` 为核心的调用或声明。
- **L212 EN**: Closes the current lexical scope or compound statement.
  **L212 CN**: 结束当前词法作用域或复合语句块。
- **L213 EN**: Blank line separating nearby declarations or logic blocks.
  **L213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L214 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L214 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L215 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtts_pd_epu64(__m256i __W, __mmask8 __U, __m256d __A) {`.
  **L215 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtts_pd_epu64(__m256i __W, __mmask8 __U, __m256d __A) {`。
- **L216 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask((__v4df)__A, (__v4di)__W,`.
  **L216 CN**: 以 `((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask((__v4df)__A, (__v4di)__W,` 从当前函数返回。
- **L217 EN**: Adds a standalone statement or declaration: `__U));`.
  **L217 CN**: 添加一条独立语句或声明：`__U));`。
- **L218 EN**: Closes the current lexical scope or compound statement.
  **L218 CN**: 结束当前词法作用域或复合语句块。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L220 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L220 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。

### Lines 221-240

````c
_mm256_maskz_cvtts_pd_epu64(__mmask8 __U, __m256d __A) {
  return ((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask(
      (__v4df)__A, (__v4di)_mm256_setzero_si256(), __U));
}

// 128 Bit : float -> int
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epi32(__m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2dqs128_mask(
      (__v4sf)__A, (__v4si)(__m128i)_mm_undefined_si128(), (__mmask8)(-1)));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtts_ps_epi32(__m128i __W, __mmask8 __U, __m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2dqs128_mask((__v4sf)__A, (__v4si)__W,
                                                      (__mmask8)__U));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtts_ps_epi32(__mmask8 __U, __m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2dqs128_mask(
````
- **L221 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtts_pd_epu64(__mmask8 __U, __m256d __A) {`.
  **L221 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtts_pd_epu64(__mmask8 __U, __m256d __A) {`。
- **L222 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask(`.
  **L222 CN**: 以 `((__m256i)__builtin_ia32_vcvttpd2uqqs256_mask(` 从当前函数返回。
- **L223 EN**: Executes a call or declaration centered on `statement`.
  **L223 CN**: 执行以 `statement` 为核心的调用或声明。
- **L224 EN**: Closes the current lexical scope or compound statement.
  **L224 CN**: 结束当前词法作用域或复合语句块。
- **L225 EN**: Blank line separating nearby declarations or logic blocks.
  **L225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L226 EN**: Comment explains nearby logic, constraints, or intent: `128 Bit : float -> int`.
  **L226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128 Bit : float -> int`。
- **L227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epi32(__m128 __A) {`.
  **L227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epi32(__m128 __A) {`。
- **L228 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2dqs128_mask(`.
  **L228 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2dqs128_mask(` 从当前函数返回。
- **L229 EN**: Executes a call or declaration centered on `statement`.
  **L229 CN**: 执行以 `statement` 为核心的调用或声明。
- **L230 EN**: Closes the current lexical scope or compound statement.
  **L230 CN**: 结束当前词法作用域或复合语句块。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L232 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L232 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L233 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtts_ps_epi32(__m128i __W, __mmask8 __U, __m128 __A) {`.
  **L233 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtts_ps_epi32(__m128i __W, __mmask8 __U, __m128 __A) {`。
- **L234 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2dqs128_mask((__v4sf)__A, (__v4si)__W,`.
  **L234 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2dqs128_mask((__v4sf)__A, (__v4si)__W,` 从当前函数返回。
- **L235 EN**: Executes a call or declaration centered on `statement`.
  **L235 CN**: 执行以 `statement` 为核心的调用或声明。
- **L236 EN**: Closes the current lexical scope or compound statement.
  **L236 CN**: 结束当前词法作用域或复合语句块。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L238 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L238 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L239 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtts_ps_epi32(__mmask8 __U, __m128 __A) {`.
  **L239 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtts_ps_epi32(__mmask8 __U, __m128 __A) {`。
- **L240 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2dqs128_mask(`.
  **L240 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2dqs128_mask(` 从当前函数返回。

### Lines 241-260

````c
      (__v4sf)__A, (__v4si)(__m128i)_mm_setzero_si128(), (__mmask8)__U));
}

// 256 Bit : float -> int
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtts_ps_epi32(__m256 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2dqs256_mask(
      (__v8sf)__A, (__v8si)_mm256_undefined_si256(), (__mmask8)-1));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtts_ps_epi32(__m256i __W, __mmask8 __U, __m256 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2dqs256_mask((__v8sf)__A, (__v8si)__W,
                                                      __U));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtts_ps_epi32(__mmask8 __U, __m256 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2dqs256_mask(
      (__v8sf)__A, (__v8si)_mm256_setzero_si256(), __U));
````
- **L241 EN**: Executes a call or declaration centered on `statement`.
  **L241 CN**: 执行以 `statement` 为核心的调用或声明。
- **L242 EN**: Closes the current lexical scope or compound statement.
  **L242 CN**: 结束当前词法作用域或复合语句块。
- **L243 EN**: Blank line separating nearby declarations or logic blocks.
  **L243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L244 EN**: Comment explains nearby logic, constraints, or intent: `256 Bit : float -> int`.
  **L244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256 Bit : float -> int`。
- **L245 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L245 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L246 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtts_ps_epi32(__m256 __A) {`.
  **L246 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtts_ps_epi32(__m256 __A) {`。
- **L247 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2dqs256_mask(`.
  **L247 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2dqs256_mask(` 从当前函数返回。
- **L248 EN**: Executes a call or declaration centered on `statement`.
  **L248 CN**: 执行以 `statement` 为核心的调用或声明。
- **L249 EN**: Closes the current lexical scope or compound statement.
  **L249 CN**: 结束当前词法作用域或复合语句块。
- **L250 EN**: Blank line separating nearby declarations or logic blocks.
  **L250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L251 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L251 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L252 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtts_ps_epi32(__m256i __W, __mmask8 __U, __m256 __A) {`.
  **L252 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtts_ps_epi32(__m256i __W, __mmask8 __U, __m256 __A) {`。
- **L253 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2dqs256_mask((__v8sf)__A, (__v8si)__W,`.
  **L253 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2dqs256_mask((__v8sf)__A, (__v8si)__W,` 从当前函数返回。
- **L254 EN**: Adds a standalone statement or declaration: `__U));`.
  **L254 CN**: 添加一条独立语句或声明：`__U));`。
- **L255 EN**: Closes the current lexical scope or compound statement.
  **L255 CN**: 结束当前词法作用域或复合语句块。
- **L256 EN**: Blank line separating nearby declarations or logic blocks.
  **L256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L257 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L257 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L258 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtts_ps_epi32(__mmask8 __U, __m256 __A) {`.
  **L258 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtts_ps_epi32(__mmask8 __U, __m256 __A) {`。
- **L259 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2dqs256_mask(`.
  **L259 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2dqs256_mask(` 从当前函数返回。
- **L260 EN**: Executes a call or declaration centered on `statement`.
  **L260 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 261-280

````c
}

// 128 Bit : float -> uint
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epu32(__m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2udqs128_mask(
      (__v4sf)__A, (__v4si)(__m128i)_mm_undefined_si128(), (__mmask8)(-1)));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtts_ps_epu32(__m128i __W, __mmask8 __U, __m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2udqs128_mask((__v4sf)__A, (__v4si)__W,
                                                       (__mmask8)__U));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtts_ps_epu32(__mmask8 __U, __m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2udqs128_mask(
      (__v4sf)__A, (__v4si)_mm_setzero_si128(), (__mmask8)__U));
}

````
- **L261 EN**: Closes the current lexical scope or compound statement.
  **L261 CN**: 结束当前词法作用域或复合语句块。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L263 EN**: Comment explains nearby logic, constraints, or intent: `128 Bit : float -> uint`.
  **L263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128 Bit : float -> uint`。
- **L264 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epu32(__m128 __A) {`.
  **L264 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epu32(__m128 __A) {`。
- **L265 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2udqs128_mask(`.
  **L265 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2udqs128_mask(` 从当前函数返回。
- **L266 EN**: Executes a call or declaration centered on `statement`.
  **L266 CN**: 执行以 `statement` 为核心的调用或声明。
- **L267 EN**: Closes the current lexical scope or compound statement.
  **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L269 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L269 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L270 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtts_ps_epu32(__m128i __W, __mmask8 __U, __m128 __A) {`.
  **L270 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtts_ps_epu32(__m128i __W, __mmask8 __U, __m128 __A) {`。
- **L271 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2udqs128_mask((__v4sf)__A, (__v4si)__W,`.
  **L271 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2udqs128_mask((__v4sf)__A, (__v4si)__W,` 从当前函数返回。
- **L272 EN**: Executes a call or declaration centered on `statement`.
  **L272 CN**: 执行以 `statement` 为核心的调用或声明。
- **L273 EN**: Closes the current lexical scope or compound statement.
  **L273 CN**: 结束当前词法作用域或复合语句块。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L275 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L275 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L276 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtts_ps_epu32(__mmask8 __U, __m128 __A) {`.
  **L276 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtts_ps_epu32(__mmask8 __U, __m128 __A) {`。
- **L277 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2udqs128_mask(`.
  **L277 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2udqs128_mask(` 从当前函数返回。
- **L278 EN**: Executes a call or declaration centered on `statement`.
  **L278 CN**: 执行以 `statement` 为核心的调用或声明。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 281-300

````c
// 256 Bit : float -> uint

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtts_ps_epu32(__m256 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2udqs256_mask(
      (__v8sf)__A, (__v8si)_mm256_undefined_si256(), (__mmask8)-1));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtts_ps_epu32(__m256i __W, __mmask8 __U, __m256 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2udqs256_mask((__v8sf)__A, (__v8si)__W,
                                                       __U));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtts_ps_epu32(__mmask8 __U, __m256 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2udqs256_mask(
      (__v8sf)__A, (__v8si)_mm256_setzero_si256(), __U));
}

````
- **L281 EN**: Comment explains nearby logic, constraints, or intent: `256 Bit : float -> uint`.
  **L281 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256 Bit : float -> uint`。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L283 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L283 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L284 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtts_ps_epu32(__m256 __A) {`.
  **L284 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtts_ps_epu32(__m256 __A) {`。
- **L285 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2udqs256_mask(`.
  **L285 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2udqs256_mask(` 从当前函数返回。
- **L286 EN**: Executes a call or declaration centered on `statement`.
  **L286 CN**: 执行以 `statement` 为核心的调用或声明。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L289 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L289 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L290 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtts_ps_epu32(__m256i __W, __mmask8 __U, __m256 __A) {`.
  **L290 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtts_ps_epu32(__m256i __W, __mmask8 __U, __m256 __A) {`。
- **L291 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2udqs256_mask((__v8sf)__A, (__v8si)__W,`.
  **L291 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2udqs256_mask((__v8sf)__A, (__v8si)__W,` 从当前函数返回。
- **L292 EN**: Adds a standalone statement or declaration: `__U));`.
  **L292 CN**: 添加一条独立语句或声明：`__U));`。
- **L293 EN**: Closes the current lexical scope or compound statement.
  **L293 CN**: 结束当前词法作用域或复合语句块。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L295 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L295 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L296 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtts_ps_epu32(__mmask8 __U, __m256 __A) {`.
  **L296 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtts_ps_epu32(__mmask8 __U, __m256 __A) {`。
- **L297 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2udqs256_mask(`.
  **L297 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2udqs256_mask(` 从当前函数返回。
- **L298 EN**: Executes a call or declaration centered on `statement`.
  **L298 CN**: 执行以 `statement` 为核心的调用或声明。
- **L299 EN**: Closes the current lexical scope or compound statement.
  **L299 CN**: 结束当前词法作用域或复合语句块。
- **L300 EN**: Blank line separating nearby declarations or logic blocks.
  **L300 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 301-320

````c
// 128 bit : float -> long
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epi64(__m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2qqs128_mask(
      (__v4sf)__A, (__v2di)_mm_undefined_si128(), (__mmask8)-1));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtts_ps_epi64(__m128i __W, __mmask8 __U, __m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2qqs128_mask(
      (__v4sf)__A, (__v2di)(__m128i)__W, (__mmask8)__U));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtts_ps_epi64(__mmask8 __U, __m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2qqs128_mask(
      (__v4sf)__A, (__v2di)_mm_setzero_si128(), (__mmask8)__U));
}
// 256 bit : float -> long

static __inline__ __m256i __DEFAULT_FN_ATTRS256
````
- **L301 EN**: Comment explains nearby logic, constraints, or intent: `128 bit : float -> long`.
  **L301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128 bit : float -> long`。
- **L302 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epi64(__m128 __A) {`.
  **L302 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epi64(__m128 __A) {`。
- **L303 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2qqs128_mask(`.
  **L303 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2qqs128_mask(` 从当前函数返回。
- **L304 EN**: Executes a call or declaration centered on `statement`.
  **L304 CN**: 执行以 `statement` 为核心的调用或声明。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Blank line separating nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L307 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L307 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L308 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtts_ps_epi64(__m128i __W, __mmask8 __U, __m128 __A) {`.
  **L308 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtts_ps_epi64(__m128i __W, __mmask8 __U, __m128 __A) {`。
- **L309 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2qqs128_mask(`.
  **L309 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2qqs128_mask(` 从当前函数返回。
- **L310 EN**: Executes a call or declaration centered on `statement`.
  **L310 CN**: 执行以 `statement` 为核心的调用或声明。
- **L311 EN**: Closes the current lexical scope or compound statement.
  **L311 CN**: 结束当前词法作用域或复合语句块。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L313 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L313 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L314 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtts_ps_epi64(__mmask8 __U, __m128 __A) {`.
  **L314 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtts_ps_epi64(__mmask8 __U, __m128 __A) {`。
- **L315 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2qqs128_mask(`.
  **L315 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2qqs128_mask(` 从当前函数返回。
- **L316 EN**: Executes a call or declaration centered on `statement`.
  **L316 CN**: 执行以 `statement` 为核心的调用或声明。
- **L317 EN**: Closes the current lexical scope or compound statement.
  **L317 CN**: 结束当前词法作用域或复合语句块。
- **L318 EN**: Comment explains nearby logic, constraints, or intent: `256 bit : float -> long`.
  **L318 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256 bit : float -> long`。
- **L319 EN**: Blank line separating nearby declarations or logic blocks.
  **L319 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L320 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L320 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。

### Lines 321-340

````c
_mm256_cvtts_ps_epi64(__m128 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2qqs256_mask(
      (__v4sf)__A, (__v4di)_mm256_undefined_si256(), (__mmask8)-1));
}
static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtts_ps_epi64(__m256i __W, __mmask8 __U, __m128 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2qqs256_mask((__v4sf)__A, (__v4di)__W,
                                                      __U));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtts_ps_epi64(__mmask8 __U, __m128 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2qqs256_mask(
      (__v4sf)__A, (__v4di)_mm256_setzero_si256(), __U));
}

// 128 bit : float -> ulong
static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epu64(__m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(
      (__v4sf)__A, (__v2di)_mm_undefined_si128(), (__mmask8)-1));
````
- **L321 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtts_ps_epi64(__m128 __A) {`.
  **L321 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtts_ps_epi64(__m128 __A) {`。
- **L322 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2qqs256_mask(`.
  **L322 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2qqs256_mask(` 从当前函数返回。
- **L323 EN**: Executes a call or declaration centered on `statement`.
  **L323 CN**: 执行以 `statement` 为核心的调用或声明。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L325 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L326 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtts_ps_epi64(__m256i __W, __mmask8 __U, __m128 __A) {`.
  **L326 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtts_ps_epi64(__m256i __W, __mmask8 __U, __m128 __A) {`。
- **L327 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2qqs256_mask((__v4sf)__A, (__v4di)__W,`.
  **L327 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2qqs256_mask((__v4sf)__A, (__v4di)__W,` 从当前函数返回。
- **L328 EN**: Adds a standalone statement or declaration: `__U));`.
  **L328 CN**: 添加一条独立语句或声明：`__U));`。
- **L329 EN**: Closes the current lexical scope or compound statement.
  **L329 CN**: 结束当前词法作用域或复合语句块。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L331 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L331 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L332 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtts_ps_epi64(__mmask8 __U, __m128 __A) {`.
  **L332 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtts_ps_epi64(__mmask8 __U, __m128 __A) {`。
- **L333 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2qqs256_mask(`.
  **L333 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2qqs256_mask(` 从当前函数返回。
- **L334 EN**: Executes a call or declaration centered on `statement`.
  **L334 CN**: 执行以 `statement` 为核心的调用或声明。
- **L335 EN**: Closes the current lexical scope or compound statement.
  **L335 CN**: 结束当前词法作用域或复合语句块。
- **L336 EN**: Blank line separating nearby declarations or logic blocks.
  **L336 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L337 EN**: Comment explains nearby logic, constraints, or intent: `128 bit : float -> ulong`.
  **L337 CN**: 注释解释附近代码的逻辑、约束或设计意图：`128 bit : float -> ulong`。
- **L338 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epu64(__m128 __A) {`.
  **L338 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ __m128i __DEFAULT_FN_ATTRS128 _mm_cvtts_ps_epu64(__m128 __A) {`。
- **L339 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(`.
  **L339 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(` 从当前函数返回。
- **L340 EN**: Executes a call or declaration centered on `statement`.
  **L340 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 341-360

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtts_ps_epu64(__m128i __W, __mmask8 __U, __m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(
      (__v4sf)__A, (__v2di)(__m128i)__W, (__mmask8)__U));
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtts_ps_epu64(__mmask8 __U, __m128 __A) {
  return ((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(
      (__v4sf)__A, (__v2di)_mm_setzero_si128(), (__mmask8)__U));
}
// 256 bit : float -> ulong

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_cvtts_ps_epu64(__m128 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2uqqs256_mask(
      (__v4sf)__A, (__v4di)_mm256_undefined_si256(), (__mmask8)-1));
}
````
- **L341 EN**: Closes the current lexical scope or compound statement.
  **L341 CN**: 结束当前词法作用域或复合语句块。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L343 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L343 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L344 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtts_ps_epu64(__m128i __W, __mmask8 __U, __m128 __A) {`.
  **L344 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtts_ps_epu64(__m128i __W, __mmask8 __U, __m128 __A) {`。
- **L345 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(`.
  **L345 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(` 从当前函数返回。
- **L346 EN**: Executes a call or declaration centered on `statement`.
  **L346 CN**: 执行以 `statement` 为核心的调用或声明。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L349 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L350 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtts_ps_epu64(__mmask8 __U, __m128 __A) {`.
  **L350 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtts_ps_epu64(__mmask8 __U, __m128 __A) {`。
- **L351 EN**: Returns from the current function with `((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(`.
  **L351 CN**: 以 `((__m128i)__builtin_ia32_vcvttps2uqqs128_mask(` 从当前函数返回。
- **L352 EN**: Executes a call or declaration centered on `statement`.
  **L352 CN**: 执行以 `statement` 为核心的调用或声明。
- **L353 EN**: Closes the current lexical scope or compound statement.
  **L353 CN**: 结束当前词法作用域或复合语句块。
- **L354 EN**: Comment explains nearby logic, constraints, or intent: `256 bit : float -> ulong`.
  **L354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`256 bit : float -> ulong`。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L356 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L356 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L357 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtts_ps_epu64(__m128 __A) {`.
  **L357 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtts_ps_epu64(__m128 __A) {`。
- **L358 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2uqqs256_mask(`.
  **L358 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2uqqs256_mask(` 从当前函数返回。
- **L359 EN**: Executes a call or declaration centered on `statement`.
  **L359 CN**: 执行以 `statement` 为核心的调用或声明。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-376

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtts_ps_epu64(__m256i __W, __mmask8 __U, __m128 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2uqqs256_mask((__v4sf)__A, (__v4di)__W,
                                                       __U));
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtts_ps_epu64(__mmask8 __U, __m128 __A) {
  return ((__m256i)__builtin_ia32_vcvttps2uqqs256_mask(
      (__v4sf)__A, (__v4di)_mm256_setzero_si256(), __U));
}

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256
#endif // __AVX10_2SATCVTDSINTRIN_H
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L362 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L362 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L363 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtts_ps_epu64(__m256i __W, __mmask8 __U, __m128 __A) {`.
  **L363 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtts_ps_epu64(__m256i __W, __mmask8 __U, __m128 __A) {`。
- **L364 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2uqqs256_mask((__v4sf)__A, (__v4di)__W,`.
  **L364 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2uqqs256_mask((__v4sf)__A, (__v4di)__W,` 从当前函数返回。
- **L365 EN**: Adds a standalone statement or declaration: `__U));`.
  **L365 CN**: 添加一条独立语句或声明：`__U));`。
- **L366 EN**: Closes the current lexical scope or compound statement.
  **L366 CN**: 结束当前词法作用域或复合语句块。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L368 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L368 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L369 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtts_ps_epu64(__mmask8 __U, __m128 __A) {`.
  **L369 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtts_ps_epu64(__mmask8 __U, __m128 __A) {`。
- **L370 EN**: Returns from the current function with `((__m256i)__builtin_ia32_vcvttps2uqqs256_mask(`.
  **L370 CN**: 以 `((__m256i)__builtin_ia32_vcvttps2uqqs256_mask(` 从当前函数返回。
- **L371 EN**: Executes a call or declaration centered on `statement`.
  **L371 CN**: 执行以 `statement` 为核心的调用或声明。
- **L372 EN**: Closes the current lexical scope or compound statement.
  **L372 CN**: 结束当前词法作用域或复合语句块。
- **L373 EN**: Blank line separating nearby declarations or logic blocks.
  **L373 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L374 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L374 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L375 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L375 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。
- **L376 EN**: Closes the current preprocessor conditional block.
  **L376 CN**: 结束当前预处理条件块。

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
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__AVX10_2SATCVTDSINTRIN_H`, `__x86_64__`
- **External builtins / 外部 builtin**: `__builtin_ia32_vcvttsd2sis32`, `__builtin_ia32_vcvttsd2usis32`, `__builtin_ia32_vcvttss2sis32`, `__builtin_ia32_vcvttss2usis32`, `__builtin_ia32_vcvttss2usis64`, `__builtin_ia32_vcvttsd2usis64`, `__builtin_ia32_vcvttss2sis64`, `__builtin_ia32_vcvttsd2sis64`, `__builtin_ia32_vcvttpd2dqs128_mask`, `__builtin_ia32_vcvttpd2dqs256_mask`, `__builtin_ia32_vcvttpd2udqs128_mask`, `__builtin_ia32_vcvttpd2udqs256_mask`
