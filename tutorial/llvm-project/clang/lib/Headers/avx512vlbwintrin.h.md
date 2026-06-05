# avx512vlbwintrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx512vlbwintrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: AVX512VL and AVX512BW intrinsics.
- **Purpose (CN)**: 提供 AVX512VL and AVX512BW intrinsic 接口。
- **Line Count / 行数**: 3052

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- avx512vlbwintrin.h - AVX512VL and AVX512BW intrinsics ------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __IMMINTRIN_H
#error "Never use <avx512vlbwintrin.h> directly; include <immintrin.h> instead."
#endif

#ifndef __AVX512VLBWINTRIN_H
#define __AVX512VLBWINTRIN_H

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS128                                                  \
  __attribute__((__always_inline__, __nodebug__,                               \
                 __target__("avx512vl,avx512bw"), __min_vector_width__(128)))
#define __DEFAULT_FN_ATTRS256                                                  \
  __attribute__((__always_inline__, __nodebug__,                               \
                 __target__("avx512vl,avx512bw"), __min_vector_width__(256)))

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
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <avx512vlbwintrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error "Never use <avx512vlbwintrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifndef __AVX512VLBWINTRIN_H`.
  **L14 CN**: 开始一个预处理条件块：`#ifndef __AVX512VLBWINTRIN_H`。
- **L15 EN**: Defines macro `__AVX512VLBWINTRIN_H` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `__AVX512VLBWINTRIN_H`，用于条件编译、简写或 API 生成。
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
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128 constexpr
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256 constexpr
#else
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256
#endif

/* Integer compare */

#define _mm_cmp_epi8_mask(a, b, p) \
  ((__mmask16)__builtin_ia32_cmpb128_mask((__v16qi)(__m128i)(a), \
                                          (__v16qi)(__m128i)(b), (int)(p), \
                                          (__mmask16)-1))

#define _mm_mask_cmp_epi8_mask(m, a, b, p) \
  ((__mmask16)__builtin_ia32_cmpb128_mask((__v16qi)(__m128i)(a), \
                                          (__v16qi)(__m128i)(b), (int)(p), \
                                          (__mmask16)(m)))

#define _mm_cmp_epu8_mask(a, b, p) \
  ((__mmask16)__builtin_ia32_ucmpb128_mask((__v16qi)(__m128i)(a), \
                                           (__v16qi)(__m128i)(b), (int)(p), \
                                           (__mmask16)-1))
````
- **L25 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L25 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L26 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L26 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L27 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L27 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L28 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L28 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L29 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L29 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L30 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L30 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L31 EN**: Closes the current preprocessor conditional block.
  **L31 CN**: 结束当前预处理条件块。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L33 EN**: Comment explains nearby logic, constraints, or intent: `Integer compare`.
  **L33 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Integer compare`。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Defines macro `_mm_cmp_epi8_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L35 CN**: 定义宏 `_mm_cmp_epi8_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L36 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpb128_mask`.
  **L36 CN**: 继续与可调用符号 `__builtin_ia32_cmpb128_mask` 相关的逻辑。
- **L37 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(b), (int)(p), \`.
  **L37 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(b), (int)(p), \`。
- **L38 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1))`.
  **L38 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1))`。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L40 EN**: Defines macro `_mm_mask_cmp_epi8_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L40 CN**: 定义宏 `_mm_mask_cmp_epi8_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L41 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpb128_mask`.
  **L41 CN**: 继续与可调用符号 `__builtin_ia32_cmpb128_mask` 相关的逻辑。
- **L42 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(b), (int)(p), \`.
  **L42 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(b), (int)(p), \`。
- **L43 EN**: Continues the surrounding expression or declaration: `(__mmask16)(m)))`.
  **L43 CN**: 继续构造周围的表达式或声明：`(__mmask16)(m)))`。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L45 EN**: Defines macro `_mm_cmp_epu8_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L45 CN**: 定义宏 `_mm_cmp_epu8_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L46 EN**: Continues logic associated with callable symbol `__builtin_ia32_ucmpb128_mask`.
  **L46 CN**: 继续与可调用符号 `__builtin_ia32_ucmpb128_mask` 相关的逻辑。
- **L47 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(b), (int)(p), \`.
  **L47 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(b), (int)(p), \`。
- **L48 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1))`.
  **L48 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1))`。

### Lines 49-72

````c

#define _mm_mask_cmp_epu8_mask(m, a, b, p) \
  ((__mmask16)__builtin_ia32_ucmpb128_mask((__v16qi)(__m128i)(a), \
                                           (__v16qi)(__m128i)(b), (int)(p), \
                                           (__mmask16)(m)))

#define _mm256_cmp_epi8_mask(a, b, p) \
  ((__mmask32)__builtin_ia32_cmpb256_mask((__v32qi)(__m256i)(a), \
                                          (__v32qi)(__m256i)(b), (int)(p), \
                                          (__mmask32)-1))

#define _mm256_mask_cmp_epi8_mask(m, a, b, p) \
  ((__mmask32)__builtin_ia32_cmpb256_mask((__v32qi)(__m256i)(a), \
                                          (__v32qi)(__m256i)(b), (int)(p), \
                                          (__mmask32)(m)))

#define _mm256_cmp_epu8_mask(a, b, p) \
  ((__mmask32)__builtin_ia32_ucmpb256_mask((__v32qi)(__m256i)(a), \
                                           (__v32qi)(__m256i)(b), (int)(p), \
                                           (__mmask32)-1))

#define _mm256_mask_cmp_epu8_mask(m, a, b, p) \
  ((__mmask32)__builtin_ia32_ucmpb256_mask((__v32qi)(__m256i)(a), \
                                           (__v32qi)(__m256i)(b), (int)(p), \
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L50 EN**: Defines macro `_mm_mask_cmp_epu8_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L50 CN**: 定义宏 `_mm_mask_cmp_epu8_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L51 EN**: Continues logic associated with callable symbol `__builtin_ia32_ucmpb128_mask`.
  **L51 CN**: 继续与可调用符号 `__builtin_ia32_ucmpb128_mask` 相关的逻辑。
- **L52 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(b), (int)(p), \`.
  **L52 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(b), (int)(p), \`。
- **L53 EN**: Continues the surrounding expression or declaration: `(__mmask16)(m)))`.
  **L53 CN**: 继续构造周围的表达式或声明：`(__mmask16)(m)))`。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L55 EN**: Defines macro `_mm256_cmp_epi8_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L55 CN**: 定义宏 `_mm256_cmp_epi8_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L56 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpb256_mask`.
  **L56 CN**: 继续与可调用符号 `__builtin_ia32_cmpb256_mask` 相关的逻辑。
- **L57 EN**: Continues the surrounding expression or declaration: `(__v32qi)(__m256i)(b), (int)(p), \`.
  **L57 CN**: 继续构造周围的表达式或声明：`(__v32qi)(__m256i)(b), (int)(p), \`。
- **L58 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1))`.
  **L58 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1))`。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L60 EN**: Defines macro `_mm256_mask_cmp_epi8_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L60 CN**: 定义宏 `_mm256_mask_cmp_epi8_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L61 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpb256_mask`.
  **L61 CN**: 继续与可调用符号 `__builtin_ia32_cmpb256_mask` 相关的逻辑。
- **L62 EN**: Continues the surrounding expression or declaration: `(__v32qi)(__m256i)(b), (int)(p), \`.
  **L62 CN**: 继续构造周围的表达式或声明：`(__v32qi)(__m256i)(b), (int)(p), \`。
- **L63 EN**: Continues the surrounding expression or declaration: `(__mmask32)(m)))`.
  **L63 CN**: 继续构造周围的表达式或声明：`(__mmask32)(m)))`。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L65 EN**: Defines macro `_mm256_cmp_epu8_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L65 CN**: 定义宏 `_mm256_cmp_epu8_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L66 EN**: Continues logic associated with callable symbol `__builtin_ia32_ucmpb256_mask`.
  **L66 CN**: 继续与可调用符号 `__builtin_ia32_ucmpb256_mask` 相关的逻辑。
- **L67 EN**: Continues the surrounding expression or declaration: `(__v32qi)(__m256i)(b), (int)(p), \`.
  **L67 CN**: 继续构造周围的表达式或声明：`(__v32qi)(__m256i)(b), (int)(p), \`。
- **L68 EN**: Continues the surrounding expression or declaration: `(__mmask32)-1))`.
  **L68 CN**: 继续构造周围的表达式或声明：`(__mmask32)-1))`。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Defines macro `_mm256_mask_cmp_epu8_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L70 CN**: 定义宏 `_mm256_mask_cmp_epu8_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L71 EN**: Continues logic associated with callable symbol `__builtin_ia32_ucmpb256_mask`.
  **L71 CN**: 继续与可调用符号 `__builtin_ia32_ucmpb256_mask` 相关的逻辑。
- **L72 EN**: Continues the surrounding expression or declaration: `(__v32qi)(__m256i)(b), (int)(p), \`.
  **L72 CN**: 继续构造周围的表达式或声明：`(__v32qi)(__m256i)(b), (int)(p), \`。

### Lines 73-96

````c
                                           (__mmask32)(m)))

#define _mm_cmp_epi16_mask(a, b, p) \
  ((__mmask8)__builtin_ia32_cmpw128_mask((__v8hi)(__m128i)(a), \
                                         (__v8hi)(__m128i)(b), (int)(p), \
                                         (__mmask8)-1))

#define _mm_mask_cmp_epi16_mask(m, a, b, p) \
  ((__mmask8)__builtin_ia32_cmpw128_mask((__v8hi)(__m128i)(a), \
                                         (__v8hi)(__m128i)(b), (int)(p), \
                                         (__mmask8)(m)))

#define _mm_cmp_epu16_mask(a, b, p) \
  ((__mmask8)__builtin_ia32_ucmpw128_mask((__v8hi)(__m128i)(a), \
                                          (__v8hi)(__m128i)(b), (int)(p), \
                                          (__mmask8)-1))

#define _mm_mask_cmp_epu16_mask(m, a, b, p) \
  ((__mmask8)__builtin_ia32_ucmpw128_mask((__v8hi)(__m128i)(a), \
                                          (__v8hi)(__m128i)(b), (int)(p), \
                                          (__mmask8)(m)))

#define _mm256_cmp_epi16_mask(a, b, p) \
  ((__mmask16)__builtin_ia32_cmpw256_mask((__v16hi)(__m256i)(a), \
````
- **L73 EN**: Continues the surrounding expression or declaration: `(__mmask32)(m)))`.
  **L73 CN**: 继续构造周围的表达式或声明：`(__mmask32)(m)))`。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L75 EN**: Defines macro `_mm_cmp_epi16_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L75 CN**: 定义宏 `_mm_cmp_epi16_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L76 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpw128_mask`.
  **L76 CN**: 继续与可调用符号 `__builtin_ia32_cmpw128_mask` 相关的逻辑。
- **L77 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(b), (int)(p), \`.
  **L77 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(b), (int)(p), \`。
- **L78 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L78 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L80 EN**: Defines macro `_mm_mask_cmp_epi16_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L80 CN**: 定义宏 `_mm_mask_cmp_epi16_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L81 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpw128_mask`.
  **L81 CN**: 继续与可调用符号 `__builtin_ia32_cmpw128_mask` 相关的逻辑。
- **L82 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(b), (int)(p), \`.
  **L82 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(b), (int)(p), \`。
- **L83 EN**: Continues the surrounding expression or declaration: `(__mmask8)(m)))`.
  **L83 CN**: 继续构造周围的表达式或声明：`(__mmask8)(m)))`。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Defines macro `_mm_cmp_epu16_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L85 CN**: 定义宏 `_mm_cmp_epu16_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L86 EN**: Continues logic associated with callable symbol `__builtin_ia32_ucmpw128_mask`.
  **L86 CN**: 继续与可调用符号 `__builtin_ia32_ucmpw128_mask` 相关的逻辑。
- **L87 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(b), (int)(p), \`.
  **L87 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(b), (int)(p), \`。
- **L88 EN**: Continues the surrounding expression or declaration: `(__mmask8)-1))`.
  **L88 CN**: 继续构造周围的表达式或声明：`(__mmask8)-1))`。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L90 EN**: Defines macro `_mm_mask_cmp_epu16_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L90 CN**: 定义宏 `_mm_mask_cmp_epu16_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L91 EN**: Continues logic associated with callable symbol `__builtin_ia32_ucmpw128_mask`.
  **L91 CN**: 继续与可调用符号 `__builtin_ia32_ucmpw128_mask` 相关的逻辑。
- **L92 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(b), (int)(p), \`.
  **L92 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(b), (int)(p), \`。
- **L93 EN**: Continues the surrounding expression or declaration: `(__mmask8)(m)))`.
  **L93 CN**: 继续构造周围的表达式或声明：`(__mmask8)(m)))`。
- **L94 EN**: Blank line separating nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L95 EN**: Defines macro `_mm256_cmp_epi16_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L95 CN**: 定义宏 `_mm256_cmp_epi16_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L96 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpw256_mask`.
  **L96 CN**: 继续与可调用符号 `__builtin_ia32_cmpw256_mask` 相关的逻辑。

### Lines 97-120

````c
                                          (__v16hi)(__m256i)(b), (int)(p), \
                                          (__mmask16)-1))

#define _mm256_mask_cmp_epi16_mask(m, a, b, p) \
  ((__mmask16)__builtin_ia32_cmpw256_mask((__v16hi)(__m256i)(a), \
                                          (__v16hi)(__m256i)(b), (int)(p), \
                                          (__mmask16)(m)))

#define _mm256_cmp_epu16_mask(a, b, p) \
  ((__mmask16)__builtin_ia32_ucmpw256_mask((__v16hi)(__m256i)(a), \
                                           (__v16hi)(__m256i)(b), (int)(p), \
                                           (__mmask16)-1))

#define _mm256_mask_cmp_epu16_mask(m, a, b, p) \
  ((__mmask16)__builtin_ia32_ucmpw256_mask((__v16hi)(__m256i)(a), \
                                           (__v16hi)(__m256i)(b), (int)(p), \
                                           (__mmask16)(m)))

#define _mm_cmpeq_epi8_mask(A, B) \
    _mm_cmp_epi8_mask((A), (B), _MM_CMPINT_EQ)
#define _mm_mask_cmpeq_epi8_mask(k, A, B) \
    _mm_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_EQ)
#define _mm_cmpge_epi8_mask(A, B) \
    _mm_cmp_epi8_mask((A), (B), _MM_CMPINT_GE)
````
- **L97 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(b), (int)(p), \`.
  **L97 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(b), (int)(p), \`。
- **L98 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1))`.
  **L98 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1))`。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L100 EN**: Defines macro `_mm256_mask_cmp_epi16_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L100 CN**: 定义宏 `_mm256_mask_cmp_epi16_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L101 EN**: Continues logic associated with callable symbol `__builtin_ia32_cmpw256_mask`.
  **L101 CN**: 继续与可调用符号 `__builtin_ia32_cmpw256_mask` 相关的逻辑。
- **L102 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(b), (int)(p), \`.
  **L102 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(b), (int)(p), \`。
- **L103 EN**: Continues the surrounding expression or declaration: `(__mmask16)(m)))`.
  **L103 CN**: 继续构造周围的表达式或声明：`(__mmask16)(m)))`。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Defines macro `_mm256_cmp_epu16_mask(a, b, p)` for conditional compilation, shorthand, or API generation.
  **L105 CN**: 定义宏 `_mm256_cmp_epu16_mask(a, b, p)`，用于条件编译、简写或 API 生成。
- **L106 EN**: Continues logic associated with callable symbol `__builtin_ia32_ucmpw256_mask`.
  **L106 CN**: 继续与可调用符号 `__builtin_ia32_ucmpw256_mask` 相关的逻辑。
- **L107 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(b), (int)(p), \`.
  **L107 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(b), (int)(p), \`。
- **L108 EN**: Continues the surrounding expression or declaration: `(__mmask16)-1))`.
  **L108 CN**: 继续构造周围的表达式或声明：`(__mmask16)-1))`。
- **L109 EN**: Blank line separating nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L110 EN**: Defines macro `_mm256_mask_cmp_epu16_mask(m, a, b, p)` for conditional compilation, shorthand, or API generation.
  **L110 CN**: 定义宏 `_mm256_mask_cmp_epu16_mask(m, a, b, p)`，用于条件编译、简写或 API 生成。
- **L111 EN**: Continues logic associated with callable symbol `__builtin_ia32_ucmpw256_mask`.
  **L111 CN**: 继续与可调用符号 `__builtin_ia32_ucmpw256_mask` 相关的逻辑。
- **L112 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(b), (int)(p), \`.
  **L112 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(b), (int)(p), \`。
- **L113 EN**: Continues the surrounding expression or declaration: `(__mmask16)(m)))`.
  **L113 CN**: 继续构造周围的表达式或声明：`(__mmask16)(m)))`。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Defines macro `_mm_cmpeq_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L115 CN**: 定义宏 `_mm_cmpeq_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L116 EN**: Continues logic associated with callable symbol `_mm_cmp_epi8_mask`.
  **L116 CN**: 继续与可调用符号 `_mm_cmp_epi8_mask` 相关的逻辑。
- **L117 EN**: Defines macro `_mm_mask_cmpeq_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L117 CN**: 定义宏 `_mm_mask_cmpeq_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L118 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi8_mask`.
  **L118 CN**: 继续与可调用符号 `_mm_mask_cmp_epi8_mask` 相关的逻辑。
- **L119 EN**: Defines macro `_mm_cmpge_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L119 CN**: 定义宏 `_mm_cmpge_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L120 EN**: Continues logic associated with callable symbol `_mm_cmp_epi8_mask`.
  **L120 CN**: 继续与可调用符号 `_mm_cmp_epi8_mask` 相关的逻辑。

### Lines 121-144

````c
#define _mm_mask_cmpge_epi8_mask(k, A, B) \
    _mm_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_GE)
#define _mm_cmpgt_epi8_mask(A, B) \
    _mm_cmp_epi8_mask((A), (B), _MM_CMPINT_GT)
#define _mm_mask_cmpgt_epi8_mask(k, A, B) \
    _mm_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_GT)
#define _mm_cmple_epi8_mask(A, B) \
    _mm_cmp_epi8_mask((A), (B), _MM_CMPINT_LE)
#define _mm_mask_cmple_epi8_mask(k, A, B) \
    _mm_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_LE)
#define _mm_cmplt_epi8_mask(A, B) \
    _mm_cmp_epi8_mask((A), (B), _MM_CMPINT_LT)
#define _mm_mask_cmplt_epi8_mask(k, A, B) \
    _mm_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_LT)
#define _mm_cmpneq_epi8_mask(A, B) \
    _mm_cmp_epi8_mask((A), (B), _MM_CMPINT_NE)
#define _mm_mask_cmpneq_epi8_mask(k, A, B) \
    _mm_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_NE)

#define _mm256_cmpeq_epi8_mask(A, B) \
    _mm256_cmp_epi8_mask((A), (B), _MM_CMPINT_EQ)
#define _mm256_mask_cmpeq_epi8_mask(k, A, B) \
    _mm256_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_EQ)
#define _mm256_cmpge_epi8_mask(A, B) \
````
- **L121 EN**: Defines macro `_mm_mask_cmpge_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L121 CN**: 定义宏 `_mm_mask_cmpge_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L122 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi8_mask`.
  **L122 CN**: 继续与可调用符号 `_mm_mask_cmp_epi8_mask` 相关的逻辑。
- **L123 EN**: Defines macro `_mm_cmpgt_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L123 CN**: 定义宏 `_mm_cmpgt_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L124 EN**: Continues logic associated with callable symbol `_mm_cmp_epi8_mask`.
  **L124 CN**: 继续与可调用符号 `_mm_cmp_epi8_mask` 相关的逻辑。
- **L125 EN**: Defines macro `_mm_mask_cmpgt_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L125 CN**: 定义宏 `_mm_mask_cmpgt_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L126 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi8_mask`.
  **L126 CN**: 继续与可调用符号 `_mm_mask_cmp_epi8_mask` 相关的逻辑。
- **L127 EN**: Defines macro `_mm_cmple_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L127 CN**: 定义宏 `_mm_cmple_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L128 EN**: Continues logic associated with callable symbol `_mm_cmp_epi8_mask`.
  **L128 CN**: 继续与可调用符号 `_mm_cmp_epi8_mask` 相关的逻辑。
- **L129 EN**: Defines macro `_mm_mask_cmple_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L129 CN**: 定义宏 `_mm_mask_cmple_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L130 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi8_mask`.
  **L130 CN**: 继续与可调用符号 `_mm_mask_cmp_epi8_mask` 相关的逻辑。
- **L131 EN**: Defines macro `_mm_cmplt_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L131 CN**: 定义宏 `_mm_cmplt_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L132 EN**: Continues logic associated with callable symbol `_mm_cmp_epi8_mask`.
  **L132 CN**: 继续与可调用符号 `_mm_cmp_epi8_mask` 相关的逻辑。
- **L133 EN**: Defines macro `_mm_mask_cmplt_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L133 CN**: 定义宏 `_mm_mask_cmplt_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L134 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi8_mask`.
  **L134 CN**: 继续与可调用符号 `_mm_mask_cmp_epi8_mask` 相关的逻辑。
- **L135 EN**: Defines macro `_mm_cmpneq_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L135 CN**: 定义宏 `_mm_cmpneq_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L136 EN**: Continues logic associated with callable symbol `_mm_cmp_epi8_mask`.
  **L136 CN**: 继续与可调用符号 `_mm_cmp_epi8_mask` 相关的逻辑。
- **L137 EN**: Defines macro `_mm_mask_cmpneq_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L137 CN**: 定义宏 `_mm_mask_cmpneq_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L138 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi8_mask`.
  **L138 CN**: 继续与可调用符号 `_mm_mask_cmp_epi8_mask` 相关的逻辑。
- **L139 EN**: Blank line separating nearby declarations or logic blocks.
  **L139 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L140 EN**: Defines macro `_mm256_cmpeq_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L140 CN**: 定义宏 `_mm256_cmpeq_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L141 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi8_mask`.
  **L141 CN**: 继续与可调用符号 `_mm256_cmp_epi8_mask` 相关的逻辑。
- **L142 EN**: Defines macro `_mm256_mask_cmpeq_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L142 CN**: 定义宏 `_mm256_mask_cmpeq_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L143 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi8_mask`.
  **L143 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi8_mask` 相关的逻辑。
- **L144 EN**: Defines macro `_mm256_cmpge_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L144 CN**: 定义宏 `_mm256_cmpge_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。

### Lines 145-168

````c
    _mm256_cmp_epi8_mask((A), (B), _MM_CMPINT_GE)
#define _mm256_mask_cmpge_epi8_mask(k, A, B) \
    _mm256_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_GE)
#define _mm256_cmpgt_epi8_mask(A, B) \
    _mm256_cmp_epi8_mask((A), (B), _MM_CMPINT_GT)
#define _mm256_mask_cmpgt_epi8_mask(k, A, B) \
    _mm256_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_GT)
#define _mm256_cmple_epi8_mask(A, B) \
    _mm256_cmp_epi8_mask((A), (B), _MM_CMPINT_LE)
#define _mm256_mask_cmple_epi8_mask(k, A, B) \
    _mm256_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_LE)
#define _mm256_cmplt_epi8_mask(A, B) \
    _mm256_cmp_epi8_mask((A), (B), _MM_CMPINT_LT)
#define _mm256_mask_cmplt_epi8_mask(k, A, B) \
    _mm256_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_LT)
#define _mm256_cmpneq_epi8_mask(A, B) \
    _mm256_cmp_epi8_mask((A), (B), _MM_CMPINT_NE)
#define _mm256_mask_cmpneq_epi8_mask(k, A, B) \
    _mm256_mask_cmp_epi8_mask((k), (A), (B), _MM_CMPINT_NE)

#define _mm_cmpeq_epu8_mask(A, B) \
    _mm_cmp_epu8_mask((A), (B), _MM_CMPINT_EQ)
#define _mm_mask_cmpeq_epu8_mask(k, A, B) \
    _mm_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_EQ)
````
- **L145 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi8_mask`.
  **L145 CN**: 继续与可调用符号 `_mm256_cmp_epi8_mask` 相关的逻辑。
- **L146 EN**: Defines macro `_mm256_mask_cmpge_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L146 CN**: 定义宏 `_mm256_mask_cmpge_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L147 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi8_mask`.
  **L147 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi8_mask` 相关的逻辑。
- **L148 EN**: Defines macro `_mm256_cmpgt_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L148 CN**: 定义宏 `_mm256_cmpgt_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L149 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi8_mask`.
  **L149 CN**: 继续与可调用符号 `_mm256_cmp_epi8_mask` 相关的逻辑。
- **L150 EN**: Defines macro `_mm256_mask_cmpgt_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L150 CN**: 定义宏 `_mm256_mask_cmpgt_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L151 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi8_mask`.
  **L151 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi8_mask` 相关的逻辑。
- **L152 EN**: Defines macro `_mm256_cmple_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L152 CN**: 定义宏 `_mm256_cmple_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L153 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi8_mask`.
  **L153 CN**: 继续与可调用符号 `_mm256_cmp_epi8_mask` 相关的逻辑。
- **L154 EN**: Defines macro `_mm256_mask_cmple_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L154 CN**: 定义宏 `_mm256_mask_cmple_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L155 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi8_mask`.
  **L155 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi8_mask` 相关的逻辑。
- **L156 EN**: Defines macro `_mm256_cmplt_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L156 CN**: 定义宏 `_mm256_cmplt_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L157 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi8_mask`.
  **L157 CN**: 继续与可调用符号 `_mm256_cmp_epi8_mask` 相关的逻辑。
- **L158 EN**: Defines macro `_mm256_mask_cmplt_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L158 CN**: 定义宏 `_mm256_mask_cmplt_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L159 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi8_mask`.
  **L159 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi8_mask` 相关的逻辑。
- **L160 EN**: Defines macro `_mm256_cmpneq_epi8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L160 CN**: 定义宏 `_mm256_cmpneq_epi8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L161 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi8_mask`.
  **L161 CN**: 继续与可调用符号 `_mm256_cmp_epi8_mask` 相关的逻辑。
- **L162 EN**: Defines macro `_mm256_mask_cmpneq_epi8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L162 CN**: 定义宏 `_mm256_mask_cmpneq_epi8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L163 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi8_mask`.
  **L163 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi8_mask` 相关的逻辑。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Defines macro `_mm_cmpeq_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L165 CN**: 定义宏 `_mm_cmpeq_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L166 EN**: Continues logic associated with callable symbol `_mm_cmp_epu8_mask`.
  **L166 CN**: 继续与可调用符号 `_mm_cmp_epu8_mask` 相关的逻辑。
- **L167 EN**: Defines macro `_mm_mask_cmpeq_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L167 CN**: 定义宏 `_mm_mask_cmpeq_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L168 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu8_mask`.
  **L168 CN**: 继续与可调用符号 `_mm_mask_cmp_epu8_mask` 相关的逻辑。

### Lines 169-192

````c
#define _mm_cmpge_epu8_mask(A, B) \
    _mm_cmp_epu8_mask((A), (B), _MM_CMPINT_GE)
#define _mm_mask_cmpge_epu8_mask(k, A, B) \
    _mm_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_GE)
#define _mm_cmpgt_epu8_mask(A, B) \
    _mm_cmp_epu8_mask((A), (B), _MM_CMPINT_GT)
#define _mm_mask_cmpgt_epu8_mask(k, A, B) \
    _mm_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_GT)
#define _mm_cmple_epu8_mask(A, B) \
    _mm_cmp_epu8_mask((A), (B), _MM_CMPINT_LE)
#define _mm_mask_cmple_epu8_mask(k, A, B) \
    _mm_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_LE)
#define _mm_cmplt_epu8_mask(A, B) \
    _mm_cmp_epu8_mask((A), (B), _MM_CMPINT_LT)
#define _mm_mask_cmplt_epu8_mask(k, A, B) \
    _mm_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_LT)
#define _mm_cmpneq_epu8_mask(A, B) \
    _mm_cmp_epu8_mask((A), (B), _MM_CMPINT_NE)
#define _mm_mask_cmpneq_epu8_mask(k, A, B) \
    _mm_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_NE)

#define _mm256_cmpeq_epu8_mask(A, B) \
    _mm256_cmp_epu8_mask((A), (B), _MM_CMPINT_EQ)
#define _mm256_mask_cmpeq_epu8_mask(k, A, B) \
````
- **L169 EN**: Defines macro `_mm_cmpge_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L169 CN**: 定义宏 `_mm_cmpge_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L170 EN**: Continues logic associated with callable symbol `_mm_cmp_epu8_mask`.
  **L170 CN**: 继续与可调用符号 `_mm_cmp_epu8_mask` 相关的逻辑。
- **L171 EN**: Defines macro `_mm_mask_cmpge_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L171 CN**: 定义宏 `_mm_mask_cmpge_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L172 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu8_mask`.
  **L172 CN**: 继续与可调用符号 `_mm_mask_cmp_epu8_mask` 相关的逻辑。
- **L173 EN**: Defines macro `_mm_cmpgt_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L173 CN**: 定义宏 `_mm_cmpgt_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L174 EN**: Continues logic associated with callable symbol `_mm_cmp_epu8_mask`.
  **L174 CN**: 继续与可调用符号 `_mm_cmp_epu8_mask` 相关的逻辑。
- **L175 EN**: Defines macro `_mm_mask_cmpgt_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L175 CN**: 定义宏 `_mm_mask_cmpgt_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L176 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu8_mask`.
  **L176 CN**: 继续与可调用符号 `_mm_mask_cmp_epu8_mask` 相关的逻辑。
- **L177 EN**: Defines macro `_mm_cmple_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L177 CN**: 定义宏 `_mm_cmple_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L178 EN**: Continues logic associated with callable symbol `_mm_cmp_epu8_mask`.
  **L178 CN**: 继续与可调用符号 `_mm_cmp_epu8_mask` 相关的逻辑。
- **L179 EN**: Defines macro `_mm_mask_cmple_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L179 CN**: 定义宏 `_mm_mask_cmple_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L180 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu8_mask`.
  **L180 CN**: 继续与可调用符号 `_mm_mask_cmp_epu8_mask` 相关的逻辑。
- **L181 EN**: Defines macro `_mm_cmplt_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L181 CN**: 定义宏 `_mm_cmplt_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L182 EN**: Continues logic associated with callable symbol `_mm_cmp_epu8_mask`.
  **L182 CN**: 继续与可调用符号 `_mm_cmp_epu8_mask` 相关的逻辑。
- **L183 EN**: Defines macro `_mm_mask_cmplt_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L183 CN**: 定义宏 `_mm_mask_cmplt_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L184 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu8_mask`.
  **L184 CN**: 继续与可调用符号 `_mm_mask_cmp_epu8_mask` 相关的逻辑。
- **L185 EN**: Defines macro `_mm_cmpneq_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L185 CN**: 定义宏 `_mm_cmpneq_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L186 EN**: Continues logic associated with callable symbol `_mm_cmp_epu8_mask`.
  **L186 CN**: 继续与可调用符号 `_mm_cmp_epu8_mask` 相关的逻辑。
- **L187 EN**: Defines macro `_mm_mask_cmpneq_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L187 CN**: 定义宏 `_mm_mask_cmpneq_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L188 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu8_mask`.
  **L188 CN**: 继续与可调用符号 `_mm_mask_cmp_epu8_mask` 相关的逻辑。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L190 EN**: Defines macro `_mm256_cmpeq_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L190 CN**: 定义宏 `_mm256_cmpeq_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L191 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu8_mask`.
  **L191 CN**: 继续与可调用符号 `_mm256_cmp_epu8_mask` 相关的逻辑。
- **L192 EN**: Defines macro `_mm256_mask_cmpeq_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L192 CN**: 定义宏 `_mm256_mask_cmpeq_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。

### Lines 193-216

````c
    _mm256_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_EQ)
#define _mm256_cmpge_epu8_mask(A, B) \
    _mm256_cmp_epu8_mask((A), (B), _MM_CMPINT_GE)
#define _mm256_mask_cmpge_epu8_mask(k, A, B) \
    _mm256_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_GE)
#define _mm256_cmpgt_epu8_mask(A, B) \
    _mm256_cmp_epu8_mask((A), (B), _MM_CMPINT_GT)
#define _mm256_mask_cmpgt_epu8_mask(k, A, B) \
    _mm256_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_GT)
#define _mm256_cmple_epu8_mask(A, B) \
    _mm256_cmp_epu8_mask((A), (B), _MM_CMPINT_LE)
#define _mm256_mask_cmple_epu8_mask(k, A, B) \
    _mm256_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_LE)
#define _mm256_cmplt_epu8_mask(A, B) \
    _mm256_cmp_epu8_mask((A), (B), _MM_CMPINT_LT)
#define _mm256_mask_cmplt_epu8_mask(k, A, B) \
    _mm256_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_LT)
#define _mm256_cmpneq_epu8_mask(A, B) \
    _mm256_cmp_epu8_mask((A), (B), _MM_CMPINT_NE)
#define _mm256_mask_cmpneq_epu8_mask(k, A, B) \
    _mm256_mask_cmp_epu8_mask((k), (A), (B), _MM_CMPINT_NE)

#define _mm_cmpeq_epi16_mask(A, B) \
    _mm_cmp_epi16_mask((A), (B), _MM_CMPINT_EQ)
````
- **L193 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu8_mask`.
  **L193 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu8_mask` 相关的逻辑。
- **L194 EN**: Defines macro `_mm256_cmpge_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L194 CN**: 定义宏 `_mm256_cmpge_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L195 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu8_mask`.
  **L195 CN**: 继续与可调用符号 `_mm256_cmp_epu8_mask` 相关的逻辑。
- **L196 EN**: Defines macro `_mm256_mask_cmpge_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L196 CN**: 定义宏 `_mm256_mask_cmpge_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L197 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu8_mask`.
  **L197 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu8_mask` 相关的逻辑。
- **L198 EN**: Defines macro `_mm256_cmpgt_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L198 CN**: 定义宏 `_mm256_cmpgt_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L199 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu8_mask`.
  **L199 CN**: 继续与可调用符号 `_mm256_cmp_epu8_mask` 相关的逻辑。
- **L200 EN**: Defines macro `_mm256_mask_cmpgt_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L200 CN**: 定义宏 `_mm256_mask_cmpgt_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L201 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu8_mask`.
  **L201 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu8_mask` 相关的逻辑。
- **L202 EN**: Defines macro `_mm256_cmple_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L202 CN**: 定义宏 `_mm256_cmple_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L203 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu8_mask`.
  **L203 CN**: 继续与可调用符号 `_mm256_cmp_epu8_mask` 相关的逻辑。
- **L204 EN**: Defines macro `_mm256_mask_cmple_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L204 CN**: 定义宏 `_mm256_mask_cmple_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L205 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu8_mask`.
  **L205 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu8_mask` 相关的逻辑。
- **L206 EN**: Defines macro `_mm256_cmplt_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L206 CN**: 定义宏 `_mm256_cmplt_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L207 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu8_mask`.
  **L207 CN**: 继续与可调用符号 `_mm256_cmp_epu8_mask` 相关的逻辑。
- **L208 EN**: Defines macro `_mm256_mask_cmplt_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L208 CN**: 定义宏 `_mm256_mask_cmplt_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L209 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu8_mask`.
  **L209 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu8_mask` 相关的逻辑。
- **L210 EN**: Defines macro `_mm256_cmpneq_epu8_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L210 CN**: 定义宏 `_mm256_cmpneq_epu8_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L211 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu8_mask`.
  **L211 CN**: 继续与可调用符号 `_mm256_cmp_epu8_mask` 相关的逻辑。
- **L212 EN**: Defines macro `_mm256_mask_cmpneq_epu8_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L212 CN**: 定义宏 `_mm256_mask_cmpneq_epu8_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L213 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu8_mask`.
  **L213 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu8_mask` 相关的逻辑。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L215 EN**: Defines macro `_mm_cmpeq_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L215 CN**: 定义宏 `_mm_cmpeq_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L216 EN**: Continues logic associated with callable symbol `_mm_cmp_epi16_mask`.
  **L216 CN**: 继续与可调用符号 `_mm_cmp_epi16_mask` 相关的逻辑。

### Lines 217-240

````c
#define _mm_mask_cmpeq_epi16_mask(k, A, B) \
    _mm_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_EQ)
#define _mm_cmpge_epi16_mask(A, B) \
    _mm_cmp_epi16_mask((A), (B), _MM_CMPINT_GE)
#define _mm_mask_cmpge_epi16_mask(k, A, B) \
    _mm_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_GE)
#define _mm_cmpgt_epi16_mask(A, B) \
    _mm_cmp_epi16_mask((A), (B), _MM_CMPINT_GT)
#define _mm_mask_cmpgt_epi16_mask(k, A, B) \
    _mm_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_GT)
#define _mm_cmple_epi16_mask(A, B) \
    _mm_cmp_epi16_mask((A), (B), _MM_CMPINT_LE)
#define _mm_mask_cmple_epi16_mask(k, A, B) \
    _mm_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_LE)
#define _mm_cmplt_epi16_mask(A, B) \
    _mm_cmp_epi16_mask((A), (B), _MM_CMPINT_LT)
#define _mm_mask_cmplt_epi16_mask(k, A, B) \
    _mm_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_LT)
#define _mm_cmpneq_epi16_mask(A, B) \
    _mm_cmp_epi16_mask((A), (B), _MM_CMPINT_NE)
#define _mm_mask_cmpneq_epi16_mask(k, A, B) \
    _mm_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_NE)

#define _mm256_cmpeq_epi16_mask(A, B) \
````
- **L217 EN**: Defines macro `_mm_mask_cmpeq_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L217 CN**: 定义宏 `_mm_mask_cmpeq_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L218 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi16_mask`.
  **L218 CN**: 继续与可调用符号 `_mm_mask_cmp_epi16_mask` 相关的逻辑。
- **L219 EN**: Defines macro `_mm_cmpge_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L219 CN**: 定义宏 `_mm_cmpge_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L220 EN**: Continues logic associated with callable symbol `_mm_cmp_epi16_mask`.
  **L220 CN**: 继续与可调用符号 `_mm_cmp_epi16_mask` 相关的逻辑。
- **L221 EN**: Defines macro `_mm_mask_cmpge_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L221 CN**: 定义宏 `_mm_mask_cmpge_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L222 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi16_mask`.
  **L222 CN**: 继续与可调用符号 `_mm_mask_cmp_epi16_mask` 相关的逻辑。
- **L223 EN**: Defines macro `_mm_cmpgt_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L223 CN**: 定义宏 `_mm_cmpgt_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L224 EN**: Continues logic associated with callable symbol `_mm_cmp_epi16_mask`.
  **L224 CN**: 继续与可调用符号 `_mm_cmp_epi16_mask` 相关的逻辑。
- **L225 EN**: Defines macro `_mm_mask_cmpgt_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L225 CN**: 定义宏 `_mm_mask_cmpgt_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L226 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi16_mask`.
  **L226 CN**: 继续与可调用符号 `_mm_mask_cmp_epi16_mask` 相关的逻辑。
- **L227 EN**: Defines macro `_mm_cmple_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L227 CN**: 定义宏 `_mm_cmple_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L228 EN**: Continues logic associated with callable symbol `_mm_cmp_epi16_mask`.
  **L228 CN**: 继续与可调用符号 `_mm_cmp_epi16_mask` 相关的逻辑。
- **L229 EN**: Defines macro `_mm_mask_cmple_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L229 CN**: 定义宏 `_mm_mask_cmple_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L230 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi16_mask`.
  **L230 CN**: 继续与可调用符号 `_mm_mask_cmp_epi16_mask` 相关的逻辑。
- **L231 EN**: Defines macro `_mm_cmplt_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L231 CN**: 定义宏 `_mm_cmplt_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L232 EN**: Continues logic associated with callable symbol `_mm_cmp_epi16_mask`.
  **L232 CN**: 继续与可调用符号 `_mm_cmp_epi16_mask` 相关的逻辑。
- **L233 EN**: Defines macro `_mm_mask_cmplt_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L233 CN**: 定义宏 `_mm_mask_cmplt_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L234 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi16_mask`.
  **L234 CN**: 继续与可调用符号 `_mm_mask_cmp_epi16_mask` 相关的逻辑。
- **L235 EN**: Defines macro `_mm_cmpneq_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L235 CN**: 定义宏 `_mm_cmpneq_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L236 EN**: Continues logic associated with callable symbol `_mm_cmp_epi16_mask`.
  **L236 CN**: 继续与可调用符号 `_mm_cmp_epi16_mask` 相关的逻辑。
- **L237 EN**: Defines macro `_mm_mask_cmpneq_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L237 CN**: 定义宏 `_mm_mask_cmpneq_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L238 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epi16_mask`.
  **L238 CN**: 继续与可调用符号 `_mm_mask_cmp_epi16_mask` 相关的逻辑。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L240 EN**: Defines macro `_mm256_cmpeq_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L240 CN**: 定义宏 `_mm256_cmpeq_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。

### Lines 241-264

````c
    _mm256_cmp_epi16_mask((A), (B), _MM_CMPINT_EQ)
#define _mm256_mask_cmpeq_epi16_mask(k, A, B) \
    _mm256_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_EQ)
#define _mm256_cmpge_epi16_mask(A, B) \
    _mm256_cmp_epi16_mask((A), (B), _MM_CMPINT_GE)
#define _mm256_mask_cmpge_epi16_mask(k, A, B) \
    _mm256_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_GE)
#define _mm256_cmpgt_epi16_mask(A, B) \
    _mm256_cmp_epi16_mask((A), (B), _MM_CMPINT_GT)
#define _mm256_mask_cmpgt_epi16_mask(k, A, B) \
    _mm256_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_GT)
#define _mm256_cmple_epi16_mask(A, B) \
    _mm256_cmp_epi16_mask((A), (B), _MM_CMPINT_LE)
#define _mm256_mask_cmple_epi16_mask(k, A, B) \
    _mm256_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_LE)
#define _mm256_cmplt_epi16_mask(A, B) \
    _mm256_cmp_epi16_mask((A), (B), _MM_CMPINT_LT)
#define _mm256_mask_cmplt_epi16_mask(k, A, B) \
    _mm256_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_LT)
#define _mm256_cmpneq_epi16_mask(A, B) \
    _mm256_cmp_epi16_mask((A), (B), _MM_CMPINT_NE)
#define _mm256_mask_cmpneq_epi16_mask(k, A, B) \
    _mm256_mask_cmp_epi16_mask((k), (A), (B), _MM_CMPINT_NE)

````
- **L241 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi16_mask`.
  **L241 CN**: 继续与可调用符号 `_mm256_cmp_epi16_mask` 相关的逻辑。
- **L242 EN**: Defines macro `_mm256_mask_cmpeq_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L242 CN**: 定义宏 `_mm256_mask_cmpeq_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L243 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi16_mask`.
  **L243 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi16_mask` 相关的逻辑。
- **L244 EN**: Defines macro `_mm256_cmpge_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L244 CN**: 定义宏 `_mm256_cmpge_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L245 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi16_mask`.
  **L245 CN**: 继续与可调用符号 `_mm256_cmp_epi16_mask` 相关的逻辑。
- **L246 EN**: Defines macro `_mm256_mask_cmpge_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L246 CN**: 定义宏 `_mm256_mask_cmpge_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L247 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi16_mask`.
  **L247 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi16_mask` 相关的逻辑。
- **L248 EN**: Defines macro `_mm256_cmpgt_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L248 CN**: 定义宏 `_mm256_cmpgt_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L249 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi16_mask`.
  **L249 CN**: 继续与可调用符号 `_mm256_cmp_epi16_mask` 相关的逻辑。
- **L250 EN**: Defines macro `_mm256_mask_cmpgt_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L250 CN**: 定义宏 `_mm256_mask_cmpgt_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L251 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi16_mask`.
  **L251 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi16_mask` 相关的逻辑。
- **L252 EN**: Defines macro `_mm256_cmple_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L252 CN**: 定义宏 `_mm256_cmple_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L253 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi16_mask`.
  **L253 CN**: 继续与可调用符号 `_mm256_cmp_epi16_mask` 相关的逻辑。
- **L254 EN**: Defines macro `_mm256_mask_cmple_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L254 CN**: 定义宏 `_mm256_mask_cmple_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L255 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi16_mask`.
  **L255 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi16_mask` 相关的逻辑。
- **L256 EN**: Defines macro `_mm256_cmplt_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L256 CN**: 定义宏 `_mm256_cmplt_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L257 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi16_mask`.
  **L257 CN**: 继续与可调用符号 `_mm256_cmp_epi16_mask` 相关的逻辑。
- **L258 EN**: Defines macro `_mm256_mask_cmplt_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L258 CN**: 定义宏 `_mm256_mask_cmplt_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L259 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi16_mask`.
  **L259 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi16_mask` 相关的逻辑。
- **L260 EN**: Defines macro `_mm256_cmpneq_epi16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L260 CN**: 定义宏 `_mm256_cmpneq_epi16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L261 EN**: Continues logic associated with callable symbol `_mm256_cmp_epi16_mask`.
  **L261 CN**: 继续与可调用符号 `_mm256_cmp_epi16_mask` 相关的逻辑。
- **L262 EN**: Defines macro `_mm256_mask_cmpneq_epi16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L262 CN**: 定义宏 `_mm256_mask_cmpneq_epi16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L263 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epi16_mask`.
  **L263 CN**: 继续与可调用符号 `_mm256_mask_cmp_epi16_mask` 相关的逻辑。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 265-288

````c
#define _mm_cmpeq_epu16_mask(A, B) \
    _mm_cmp_epu16_mask((A), (B), _MM_CMPINT_EQ)
#define _mm_mask_cmpeq_epu16_mask(k, A, B) \
    _mm_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_EQ)
#define _mm_cmpge_epu16_mask(A, B) \
    _mm_cmp_epu16_mask((A), (B), _MM_CMPINT_GE)
#define _mm_mask_cmpge_epu16_mask(k, A, B) \
    _mm_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_GE)
#define _mm_cmpgt_epu16_mask(A, B) \
    _mm_cmp_epu16_mask((A), (B), _MM_CMPINT_GT)
#define _mm_mask_cmpgt_epu16_mask(k, A, B) \
    _mm_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_GT)
#define _mm_cmple_epu16_mask(A, B) \
    _mm_cmp_epu16_mask((A), (B), _MM_CMPINT_LE)
#define _mm_mask_cmple_epu16_mask(k, A, B) \
    _mm_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_LE)
#define _mm_cmplt_epu16_mask(A, B) \
    _mm_cmp_epu16_mask((A), (B), _MM_CMPINT_LT)
#define _mm_mask_cmplt_epu16_mask(k, A, B) \
    _mm_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_LT)
#define _mm_cmpneq_epu16_mask(A, B) \
    _mm_cmp_epu16_mask((A), (B), _MM_CMPINT_NE)
#define _mm_mask_cmpneq_epu16_mask(k, A, B) \
    _mm_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_NE)
````
- **L265 EN**: Defines macro `_mm_cmpeq_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L265 CN**: 定义宏 `_mm_cmpeq_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L266 EN**: Continues logic associated with callable symbol `_mm_cmp_epu16_mask`.
  **L266 CN**: 继续与可调用符号 `_mm_cmp_epu16_mask` 相关的逻辑。
- **L267 EN**: Defines macro `_mm_mask_cmpeq_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L267 CN**: 定义宏 `_mm_mask_cmpeq_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L268 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu16_mask`.
  **L268 CN**: 继续与可调用符号 `_mm_mask_cmp_epu16_mask` 相关的逻辑。
- **L269 EN**: Defines macro `_mm_cmpge_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L269 CN**: 定义宏 `_mm_cmpge_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L270 EN**: Continues logic associated with callable symbol `_mm_cmp_epu16_mask`.
  **L270 CN**: 继续与可调用符号 `_mm_cmp_epu16_mask` 相关的逻辑。
- **L271 EN**: Defines macro `_mm_mask_cmpge_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L271 CN**: 定义宏 `_mm_mask_cmpge_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L272 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu16_mask`.
  **L272 CN**: 继续与可调用符号 `_mm_mask_cmp_epu16_mask` 相关的逻辑。
- **L273 EN**: Defines macro `_mm_cmpgt_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L273 CN**: 定义宏 `_mm_cmpgt_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L274 EN**: Continues logic associated with callable symbol `_mm_cmp_epu16_mask`.
  **L274 CN**: 继续与可调用符号 `_mm_cmp_epu16_mask` 相关的逻辑。
- **L275 EN**: Defines macro `_mm_mask_cmpgt_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L275 CN**: 定义宏 `_mm_mask_cmpgt_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L276 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu16_mask`.
  **L276 CN**: 继续与可调用符号 `_mm_mask_cmp_epu16_mask` 相关的逻辑。
- **L277 EN**: Defines macro `_mm_cmple_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L277 CN**: 定义宏 `_mm_cmple_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L278 EN**: Continues logic associated with callable symbol `_mm_cmp_epu16_mask`.
  **L278 CN**: 继续与可调用符号 `_mm_cmp_epu16_mask` 相关的逻辑。
- **L279 EN**: Defines macro `_mm_mask_cmple_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L279 CN**: 定义宏 `_mm_mask_cmple_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L280 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu16_mask`.
  **L280 CN**: 继续与可调用符号 `_mm_mask_cmp_epu16_mask` 相关的逻辑。
- **L281 EN**: Defines macro `_mm_cmplt_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L281 CN**: 定义宏 `_mm_cmplt_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L282 EN**: Continues logic associated with callable symbol `_mm_cmp_epu16_mask`.
  **L282 CN**: 继续与可调用符号 `_mm_cmp_epu16_mask` 相关的逻辑。
- **L283 EN**: Defines macro `_mm_mask_cmplt_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L283 CN**: 定义宏 `_mm_mask_cmplt_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L284 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu16_mask`.
  **L284 CN**: 继续与可调用符号 `_mm_mask_cmp_epu16_mask` 相关的逻辑。
- **L285 EN**: Defines macro `_mm_cmpneq_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L285 CN**: 定义宏 `_mm_cmpneq_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L286 EN**: Continues logic associated with callable symbol `_mm_cmp_epu16_mask`.
  **L286 CN**: 继续与可调用符号 `_mm_cmp_epu16_mask` 相关的逻辑。
- **L287 EN**: Defines macro `_mm_mask_cmpneq_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L287 CN**: 定义宏 `_mm_mask_cmpneq_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L288 EN**: Continues logic associated with callable symbol `_mm_mask_cmp_epu16_mask`.
  **L288 CN**: 继续与可调用符号 `_mm_mask_cmp_epu16_mask` 相关的逻辑。

### Lines 289-312

````c

#define _mm256_cmpeq_epu16_mask(A, B) \
    _mm256_cmp_epu16_mask((A), (B), _MM_CMPINT_EQ)
#define _mm256_mask_cmpeq_epu16_mask(k, A, B) \
    _mm256_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_EQ)
#define _mm256_cmpge_epu16_mask(A, B) \
    _mm256_cmp_epu16_mask((A), (B), _MM_CMPINT_GE)
#define _mm256_mask_cmpge_epu16_mask(k, A, B) \
    _mm256_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_GE)
#define _mm256_cmpgt_epu16_mask(A, B) \
    _mm256_cmp_epu16_mask((A), (B), _MM_CMPINT_GT)
#define _mm256_mask_cmpgt_epu16_mask(k, A, B) \
    _mm256_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_GT)
#define _mm256_cmple_epu16_mask(A, B) \
    _mm256_cmp_epu16_mask((A), (B), _MM_CMPINT_LE)
#define _mm256_mask_cmple_epu16_mask(k, A, B) \
    _mm256_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_LE)
#define _mm256_cmplt_epu16_mask(A, B) \
    _mm256_cmp_epu16_mask((A), (B), _MM_CMPINT_LT)
#define _mm256_mask_cmplt_epu16_mask(k, A, B) \
    _mm256_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_LT)
#define _mm256_cmpneq_epu16_mask(A, B) \
    _mm256_cmp_epu16_mask((A), (B), _MM_CMPINT_NE)
#define _mm256_mask_cmpneq_epu16_mask(k, A, B) \
````
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L290 EN**: Defines macro `_mm256_cmpeq_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L290 CN**: 定义宏 `_mm256_cmpeq_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L291 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu16_mask`.
  **L291 CN**: 继续与可调用符号 `_mm256_cmp_epu16_mask` 相关的逻辑。
- **L292 EN**: Defines macro `_mm256_mask_cmpeq_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L292 CN**: 定义宏 `_mm256_mask_cmpeq_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L293 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu16_mask`.
  **L293 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu16_mask` 相关的逻辑。
- **L294 EN**: Defines macro `_mm256_cmpge_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L294 CN**: 定义宏 `_mm256_cmpge_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L295 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu16_mask`.
  **L295 CN**: 继续与可调用符号 `_mm256_cmp_epu16_mask` 相关的逻辑。
- **L296 EN**: Defines macro `_mm256_mask_cmpge_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L296 CN**: 定义宏 `_mm256_mask_cmpge_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L297 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu16_mask`.
  **L297 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu16_mask` 相关的逻辑。
- **L298 EN**: Defines macro `_mm256_cmpgt_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L298 CN**: 定义宏 `_mm256_cmpgt_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L299 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu16_mask`.
  **L299 CN**: 继续与可调用符号 `_mm256_cmp_epu16_mask` 相关的逻辑。
- **L300 EN**: Defines macro `_mm256_mask_cmpgt_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L300 CN**: 定义宏 `_mm256_mask_cmpgt_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L301 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu16_mask`.
  **L301 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu16_mask` 相关的逻辑。
- **L302 EN**: Defines macro `_mm256_cmple_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L302 CN**: 定义宏 `_mm256_cmple_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L303 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu16_mask`.
  **L303 CN**: 继续与可调用符号 `_mm256_cmp_epu16_mask` 相关的逻辑。
- **L304 EN**: Defines macro `_mm256_mask_cmple_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L304 CN**: 定义宏 `_mm256_mask_cmple_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L305 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu16_mask`.
  **L305 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu16_mask` 相关的逻辑。
- **L306 EN**: Defines macro `_mm256_cmplt_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L306 CN**: 定义宏 `_mm256_cmplt_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L307 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu16_mask`.
  **L307 CN**: 继续与可调用符号 `_mm256_cmp_epu16_mask` 相关的逻辑。
- **L308 EN**: Defines macro `_mm256_mask_cmplt_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L308 CN**: 定义宏 `_mm256_mask_cmplt_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。
- **L309 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu16_mask`.
  **L309 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu16_mask` 相关的逻辑。
- **L310 EN**: Defines macro `_mm256_cmpneq_epu16_mask(A, B)` for conditional compilation, shorthand, or API generation.
  **L310 CN**: 定义宏 `_mm256_cmpneq_epu16_mask(A, B)`，用于条件编译、简写或 API 生成。
- **L311 EN**: Continues logic associated with callable symbol `_mm256_cmp_epu16_mask`.
  **L311 CN**: 继续与可调用符号 `_mm256_cmp_epu16_mask` 相关的逻辑。
- **L312 EN**: Defines macro `_mm256_mask_cmpneq_epu16_mask(k, A, B)` for conditional compilation, shorthand, or API generation.
  **L312 CN**: 定义宏 `_mm256_mask_cmpneq_epu16_mask(k, A, B)`，用于条件编译、简写或 API 生成。

### Lines 313-336

````c
    _mm256_mask_cmp_epu16_mask((k), (A), (B), _MM_CMPINT_NE)

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_add_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                             (__v32qi)_mm256_add_epi8(__A, __B),
                                             (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_add_epi8(__mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                             (__v32qi)_mm256_add_epi8(__A, __B),
                                             (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_add_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_add_epi16(__A, __B),
                                             (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L313 EN**: Continues logic associated with callable symbol `_mm256_mask_cmp_epu16_mask`.
  **L313 CN**: 继续与可调用符号 `_mm256_mask_cmp_epu16_mask` 相关的逻辑。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L315 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L316 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_add_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`.
  **L316 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_add_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`。
- **L317 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L317 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_add_epi8(__A, __B),`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_add_epi8(__A, __B),`。
- **L319 EN**: Executes a call or declaration centered on `statement`.
  **L319 CN**: 执行以 `statement` 为核心的调用或声明。
- **L320 EN**: Closes the current lexical scope or compound statement.
  **L320 CN**: 结束当前词法作用域或复合语句块。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L322 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L322 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L323 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_add_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`.
  **L323 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_add_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`。
- **L324 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L324 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_add_epi8(__A, __B),`.
  **L325 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_add_epi8(__A, __B),`。
- **L326 EN**: Executes a call or declaration centered on `statement`.
  **L326 CN**: 执行以 `statement` 为核心的调用或声明。
- **L327 EN**: Closes the current lexical scope or compound statement.
  **L327 CN**: 结束当前词法作用域或复合语句块。
- **L328 EN**: Blank line separating nearby declarations or logic blocks.
  **L328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L329 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L329 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L330 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_add_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`.
  **L330 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_add_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`。
- **L331 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L331 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_add_epi16(__A, __B),`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_add_epi16(__A, __B),`。
- **L333 EN**: Executes a call or declaration centered on `statement`.
  **L333 CN**: 执行以 `statement` 为核心的调用或声明。
- **L334 EN**: Closes the current lexical scope or compound statement.
  **L334 CN**: 结束当前词法作用域或复合语句块。
- **L335 EN**: Blank line separating nearby declarations or logic blocks.
  **L335 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L336 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L336 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 337-360

````c
_mm256_maskz_add_epi16(__mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_add_epi16(__A, __B),
                                             (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_sub_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                             (__v32qi)_mm256_sub_epi8(__A, __B),
                                             (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_sub_epi8(__mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                             (__v32qi)_mm256_sub_epi8(__A, __B),
                                             (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_sub_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_sub_epi16(__A, __B),
````
- **L337 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_add_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L337 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_add_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L338 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L338 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_add_epi16(__A, __B),`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_add_epi16(__A, __B),`。
- **L340 EN**: Executes a call or declaration centered on `statement`.
  **L340 CN**: 执行以 `statement` 为核心的调用或声明。
- **L341 EN**: Closes the current lexical scope or compound statement.
  **L341 CN**: 结束当前词法作用域或复合语句块。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L343 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L343 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L344 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_sub_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`.
  **L344 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_sub_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`。
- **L345 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L345 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_sub_epi8(__A, __B),`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_sub_epi8(__A, __B),`。
- **L347 EN**: Executes a call or declaration centered on `statement`.
  **L347 CN**: 执行以 `statement` 为核心的调用或声明。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L350 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L350 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L351 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_sub_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`.
  **L351 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_sub_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`。
- **L352 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L352 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_sub_epi8(__A, __B),`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_sub_epi8(__A, __B),`。
- **L354 EN**: Executes a call or declaration centered on `statement`.
  **L354 CN**: 执行以 `statement` 为核心的调用或声明。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L357 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L357 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L358 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_sub_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`.
  **L358 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_sub_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`。
- **L359 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L359 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_sub_epi16(__A, __B),`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_sub_epi16(__A, __B),`。

### Lines 361-384

````c
                                             (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_sub_epi16(__mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_sub_epi16(__A, __B),
                                             (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_add_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_add_epi8(__A, __B),
                                             (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_add_epi8(__mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_add_epi8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

````
- **L361 EN**: Executes a call or declaration centered on `statement`.
  **L361 CN**: 执行以 `statement` 为核心的调用或声明。
- **L362 EN**: Closes the current lexical scope or compound statement.
  **L362 CN**: 结束当前词法作用域或复合语句块。
- **L363 EN**: Blank line separating nearby declarations or logic blocks.
  **L363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L364 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L364 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L365 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_sub_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L365 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_sub_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L366 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L366 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_sub_epi16(__A, __B),`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_sub_epi16(__A, __B),`。
- **L368 EN**: Executes a call or declaration centered on `statement`.
  **L368 CN**: 执行以 `statement` 为核心的调用或声明。
- **L369 EN**: Closes the current lexical scope or compound statement.
  **L369 CN**: 结束当前词法作用域或复合语句块。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L371 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L371 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L372 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_add_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`.
  **L372 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_add_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`。
- **L373 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L373 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_add_epi8(__A, __B),`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_add_epi8(__A, __B),`。
- **L375 EN**: Executes a call or declaration centered on `statement`.
  **L375 CN**: 执行以 `statement` 为核心的调用或声明。
- **L376 EN**: Closes the current lexical scope or compound statement.
  **L376 CN**: 结束当前词法作用域或复合语句块。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L378 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L378 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_add_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`.
  **L379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_add_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`。
- **L380 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L380 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_add_epi8(__A, __B),`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_add_epi8(__A, __B),`。
- **L382 EN**: Executes a call or declaration centered on `statement`.
  **L382 CN**: 执行以 `statement` 为核心的调用或声明。
- **L383 EN**: Closes the current lexical scope or compound statement.
  **L383 CN**: 结束当前词法作用域或复合语句块。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_add_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_add_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_add_epi16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_add_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_sub_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_sub_epi8(__A, __B),
                                             (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_sub_epi8(__mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
````
- **L385 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L385 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L386 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_add_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L386 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_add_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L387 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L387 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_add_epi16(__A, __B),`.
  **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_add_epi16(__A, __B),`。
- **L389 EN**: Executes a call or declaration centered on `statement`.
  **L389 CN**: 执行以 `statement` 为核心的调用或声明。
- **L390 EN**: Closes the current lexical scope or compound statement.
  **L390 CN**: 结束当前词法作用域或复合语句块。
- **L391 EN**: Blank line separating nearby declarations or logic blocks.
  **L391 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L392 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L392 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L393 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_add_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L393 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_add_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L394 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L394 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_add_epi16(__A, __B),`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_add_epi16(__A, __B),`。
- **L396 EN**: Executes a call or declaration centered on `statement`.
  **L396 CN**: 执行以 `statement` 为核心的调用或声明。
- **L397 EN**: Closes the current lexical scope or compound statement.
  **L397 CN**: 结束当前词法作用域或复合语句块。
- **L398 EN**: Blank line separating nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L399 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L399 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L400 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_sub_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`.
  **L400 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_sub_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`。
- **L401 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L401 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_sub_epi8(__A, __B),`.
  **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_sub_epi8(__A, __B),`。
- **L403 EN**: Executes a call or declaration centered on `statement`.
  **L403 CN**: 执行以 `statement` 为核心的调用或声明。
- **L404 EN**: Closes the current lexical scope or compound statement.
  **L404 CN**: 结束当前词法作用域或复合语句块。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L406 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L406 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L407 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_sub_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`.
  **L407 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_sub_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`。
- **L408 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L408 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。

### Lines 409-432

````c
                                             (__v16qi)_mm_sub_epi8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_sub_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_sub_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_sub_epi16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_sub_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_mullo_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_mullo_epi16(__A, __B),
                                             (__v16hi)__W);
}
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_sub_epi8(__A, __B),`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_sub_epi8(__A, __B),`。
- **L410 EN**: Executes a call or declaration centered on `statement`.
  **L410 CN**: 执行以 `statement` 为核心的调用或声明。
- **L411 EN**: Closes the current lexical scope or compound statement.
  **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L413 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L413 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L414 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_sub_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L414 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_sub_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L415 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L415 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_sub_epi16(__A, __B),`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_sub_epi16(__A, __B),`。
- **L417 EN**: Executes a call or declaration centered on `statement`.
  **L417 CN**: 执行以 `statement` 为核心的调用或声明。
- **L418 EN**: Closes the current lexical scope or compound statement.
  **L418 CN**: 结束当前词法作用域或复合语句块。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L420 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L420 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L421 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_sub_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L421 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_sub_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L422 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L422 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_sub_epi16(__A, __B),`.
  **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_sub_epi16(__A, __B),`。
- **L424 EN**: Executes a call or declaration centered on `statement`.
  **L424 CN**: 执行以 `statement` 为核心的调用或声明。
- **L425 EN**: Closes the current lexical scope or compound statement.
  **L425 CN**: 结束当前词法作用域或复合语句块。
- **L426 EN**: Blank line separating nearby declarations or logic blocks.
  **L426 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L427 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L427 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L428 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mullo_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`.
  **L428 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mullo_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`。
- **L429 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L429 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_mullo_epi16(__A, __B),`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_mullo_epi16(__A, __B),`。
- **L431 EN**: Executes a call or declaration centered on `statement`.
  **L431 CN**: 执行以 `statement` 为核心的调用或声明。
- **L432 EN**: Closes the current lexical scope or compound statement.
  **L432 CN**: 结束当前词法作用域或复合语句块。

### Lines 433-456

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_mullo_epi16(__mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_mullo_epi16(__A, __B),
                                             (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_mullo_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_mullo_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_mullo_epi16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_mullo_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_blend_epi8(__mmask16 __U, __m128i __A, __m128i __W) {
````
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L434 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L434 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L435 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mullo_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L435 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mullo_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L436 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L436 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_mullo_epi16(__A, __B),`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_mullo_epi16(__A, __B),`。
- **L438 EN**: Executes a call or declaration centered on `statement`.
  **L438 CN**: 执行以 `statement` 为核心的调用或声明。
- **L439 EN**: Closes the current lexical scope or compound statement.
  **L439 CN**: 结束当前词法作用域或复合语句块。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L441 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L441 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L442 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mullo_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L442 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mullo_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L443 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L443 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_mullo_epi16(__A, __B),`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_mullo_epi16(__A, __B),`。
- **L445 EN**: Executes a call or declaration centered on `statement`.
  **L445 CN**: 执行以 `statement` 为核心的调用或声明。
- **L446 EN**: Closes the current lexical scope or compound statement.
  **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L448 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L449 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mullo_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L449 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mullo_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L450 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L450 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_mullo_epi16(__A, __B),`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_mullo_epi16(__A, __B),`。
- **L452 EN**: Executes a call or declaration centered on `statement`.
  **L452 CN**: 执行以 `statement` 为核心的调用或声明。
- **L453 EN**: Closes the current lexical scope or compound statement.
  **L453 CN**: 结束当前词法作用域或复合语句块。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L455 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L455 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L456 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_blend_epi8(__mmask16 __U, __m128i __A, __m128i __W) {`.
  **L456 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_blend_epi8(__mmask16 __U, __m128i __A, __m128i __W) {`。

### Lines 457-480

````c
  return (__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,
              (__v16qi) __W,
              (__v16qi) __A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_blend_epi8(__mmask32 __U, __m256i __A, __m256i __W) {
  return (__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,
               (__v32qi) __W,
               (__v32qi) __A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_blend_epi16(__mmask8 __U, __m128i __A, __m128i __W) {
  return (__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,
               (__v8hi) __W,
               (__v8hi) __A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_blend_epi16(__mmask16 __U, __m256i __A, __m256i __W) {
  return (__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,
               (__v16hi) __W,
               (__v16hi) __A);
````
- **L457 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,`.
  **L457 CN**: 以 `(__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,` 从当前函数返回。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __W,`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __W,`。
- **L459 EN**: Executes a call or declaration centered on `statement`.
  **L459 CN**: 执行以 `statement` 为核心的调用或声明。
- **L460 EN**: Closes the current lexical scope or compound statement.
  **L460 CN**: 结束当前词法作用域或复合语句块。
- **L461 EN**: Blank line separating nearby declarations or logic blocks.
  **L461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L462 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L462 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L463 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_blend_epi8(__mmask32 __U, __m256i __A, __m256i __W) {`.
  **L463 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_blend_epi8(__mmask32 __U, __m256i __A, __m256i __W) {`。
- **L464 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,`.
  **L464 CN**: 以 `(__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,` 从当前函数返回。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) __W,`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) __W,`。
- **L466 EN**: Executes a call or declaration centered on `statement`.
  **L466 CN**: 执行以 `statement` 为核心的调用或声明。
- **L467 EN**: Closes the current lexical scope or compound statement.
  **L467 CN**: 结束当前词法作用域或复合语句块。
- **L468 EN**: Blank line separating nearby declarations or logic blocks.
  **L468 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L469 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L469 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L470 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_blend_epi16(__mmask8 __U, __m128i __A, __m128i __W) {`.
  **L470 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_blend_epi16(__mmask8 __U, __m128i __A, __m128i __W) {`。
- **L471 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,`.
  **L471 CN**: 以 `(__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,` 从当前函数返回。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) __W,`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) __W,`。
- **L473 EN**: Executes a call or declaration centered on `statement`.
  **L473 CN**: 执行以 `statement` 为核心的调用或声明。
- **L474 EN**: Closes the current lexical scope or compound statement.
  **L474 CN**: 结束当前词法作用域或复合语句块。
- **L475 EN**: Blank line separating nearby declarations or logic blocks.
  **L475 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L476 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L476 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L477 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_blend_epi16(__mmask16 __U, __m256i __A, __m256i __W) {`.
  **L477 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_blend_epi16(__mmask16 __U, __m256i __A, __m256i __W) {`。
- **L478 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,`.
  **L478 CN**: 以 `(__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,` 从当前函数返回。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) __W,`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) __W,`。
- **L480 EN**: Executes a call or declaration centered on `statement`.
  **L480 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 481-504

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_abs_epi8(__m128i __W, __mmask16 __U, __m128i __A) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_abs_epi8(__A),
                                             (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_abs_epi8(__mmask16 __U, __m128i __A) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_abs_epi8(__A),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_abs_epi8(__m256i __W, __mmask32 __U, __m256i __A) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                             (__v32qi)_mm256_abs_epi8(__A),
                                             (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L483 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L483 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L484 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_abs_epi8(__m128i __W, __mmask16 __U, __m128i __A) {`.
  **L484 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_abs_epi8(__m128i __W, __mmask16 __U, __m128i __A) {`。
- **L485 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L485 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_abs_epi8(__A),`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_abs_epi8(__A),`。
- **L487 EN**: Executes a call or declaration centered on `statement`.
  **L487 CN**: 执行以 `statement` 为核心的调用或声明。
- **L488 EN**: Closes the current lexical scope or compound statement.
  **L488 CN**: 结束当前词法作用域或复合语句块。
- **L489 EN**: Blank line separating nearby declarations or logic blocks.
  **L489 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L490 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L490 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L491 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_abs_epi8(__mmask16 __U, __m128i __A) {`.
  **L491 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_abs_epi8(__mmask16 __U, __m128i __A) {`。
- **L492 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L492 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_abs_epi8(__A),`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_abs_epi8(__A),`。
- **L494 EN**: Executes a call or declaration centered on `statement`.
  **L494 CN**: 执行以 `statement` 为核心的调用或声明。
- **L495 EN**: Closes the current lexical scope or compound statement.
  **L495 CN**: 结束当前词法作用域或复合语句块。
- **L496 EN**: Blank line separating nearby declarations or logic blocks.
  **L496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L497 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L497 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L498 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_abs_epi8(__m256i __W, __mmask32 __U, __m256i __A) {`.
  **L498 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_abs_epi8(__m256i __W, __mmask32 __U, __m256i __A) {`。
- **L499 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L499 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_abs_epi8(__A),`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_abs_epi8(__A),`。
- **L501 EN**: Executes a call or declaration centered on `statement`.
  **L501 CN**: 执行以 `statement` 为核心的调用或声明。
- **L502 EN**: Closes the current lexical scope or compound statement.
  **L502 CN**: 结束当前词法作用域或复合语句块。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L504 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L504 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 505-528

````c
_mm256_maskz_abs_epi8(__mmask32 __U, __m256i __A) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                             (__v32qi)_mm256_abs_epi8(__A),
                                             (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_abs_epi16(__m128i __W, __mmask8 __U, __m128i __A) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_abs_epi16(__A),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_abs_epi16(__mmask8 __U, __m128i __A) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_abs_epi16(__A),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_abs_epi16(__m256i __W, __mmask16 __U, __m256i __A) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_abs_epi16(__A),
````
- **L505 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_abs_epi8(__mmask32 __U, __m256i __A) {`.
  **L505 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_abs_epi8(__mmask32 __U, __m256i __A) {`。
- **L506 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L506 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_abs_epi8(__A),`.
  **L507 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_abs_epi8(__A),`。
- **L508 EN**: Executes a call or declaration centered on `statement`.
  **L508 CN**: 执行以 `statement` 为核心的调用或声明。
- **L509 EN**: Closes the current lexical scope or compound statement.
  **L509 CN**: 结束当前词法作用域或复合语句块。
- **L510 EN**: Blank line separating nearby declarations or logic blocks.
  **L510 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L511 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L511 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L512 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_abs_epi16(__m128i __W, __mmask8 __U, __m128i __A) {`.
  **L512 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_abs_epi16(__m128i __W, __mmask8 __U, __m128i __A) {`。
- **L513 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L513 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_abs_epi16(__A),`.
  **L514 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_abs_epi16(__A),`。
- **L515 EN**: Executes a call or declaration centered on `statement`.
  **L515 CN**: 执行以 `statement` 为核心的调用或声明。
- **L516 EN**: Closes the current lexical scope or compound statement.
  **L516 CN**: 结束当前词法作用域或复合语句块。
- **L517 EN**: Blank line separating nearby declarations or logic blocks.
  **L517 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L518 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L518 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L519 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_abs_epi16(__mmask8 __U, __m128i __A) {`.
  **L519 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_abs_epi16(__mmask8 __U, __m128i __A) {`。
- **L520 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L520 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_abs_epi16(__A),`.
  **L521 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_abs_epi16(__A),`。
- **L522 EN**: Executes a call or declaration centered on `statement`.
  **L522 CN**: 执行以 `statement` 为核心的调用或声明。
- **L523 EN**: Closes the current lexical scope or compound statement.
  **L523 CN**: 结束当前词法作用域或复合语句块。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L525 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L525 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L526 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_abs_epi16(__m256i __W, __mmask16 __U, __m256i __A) {`.
  **L526 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_abs_epi16(__m256i __W, __mmask16 __U, __m256i __A) {`。
- **L527 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L527 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_abs_epi16(__A),`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_abs_epi16(__A),`。

### Lines 529-552

````c
                                             (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_abs_epi16(__mmask16 __U, __m256i __A) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_abs_epi16(__A),
                                             (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_packs_epi32(__mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_packs_epi32(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_packs_epi32(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_packs_epi32(__A, __B),
                                             (__v8hi)__W);
}
````
- **L529 EN**: Executes a call or declaration centered on `statement`.
  **L529 CN**: 执行以 `statement` 为核心的调用或声明。
- **L530 EN**: Closes the current lexical scope or compound statement.
  **L530 CN**: 结束当前词法作用域或复合语句块。
- **L531 EN**: Blank line separating nearby declarations or logic blocks.
  **L531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L532 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L532 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L533 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_abs_epi16(__mmask16 __U, __m256i __A) {`.
  **L533 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_abs_epi16(__mmask16 __U, __m256i __A) {`。
- **L534 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L534 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_abs_epi16(__A),`.
  **L535 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_abs_epi16(__A),`。
- **L536 EN**: Executes a call or declaration centered on `statement`.
  **L536 CN**: 执行以 `statement` 为核心的调用或声明。
- **L537 EN**: Closes the current lexical scope or compound statement.
  **L537 CN**: 结束当前词法作用域或复合语句块。
- **L538 EN**: Blank line separating nearby declarations or logic blocks.
  **L538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L539 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L539 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L540 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_packs_epi32(__mmask8 __M, __m128i __A, __m128i __B) {`.
  **L540 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_packs_epi32(__mmask8 __M, __m128i __A, __m128i __B) {`。
- **L541 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L541 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_packs_epi32(__A, __B),`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_packs_epi32(__A, __B),`。
- **L543 EN**: Executes a call or declaration centered on `statement`.
  **L543 CN**: 执行以 `statement` 为核心的调用或声明。
- **L544 EN**: Closes the current lexical scope or compound statement.
  **L544 CN**: 结束当前词法作用域或复合语句块。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L546 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L546 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L547 EN**: Continues logic associated with callable symbol `_mm_mask_packs_epi32`.
  **L547 CN**: 继续与可调用符号 `_mm_mask_packs_epi32` 相关的逻辑。
- **L548 EN**: Opens a new lexical scope or compound statement.
  **L548 CN**: 打开一个新的词法作用域或复合语句块。
- **L549 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L549 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_packs_epi32(__A, __B),`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_packs_epi32(__A, __B),`。
- **L551 EN**: Executes a call or declaration centered on `statement`.
  **L551 CN**: 执行以 `statement` 为核心的调用或声明。
- **L552 EN**: Closes the current lexical scope or compound statement.
  **L552 CN**: 结束当前词法作用域或复合语句块。

### Lines 553-576

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_packs_epi32(__mmask16 __M, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                          (__v16hi)_mm256_packs_epi32(__A, __B),
                                          (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_packs_epi32(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                          (__v16hi)_mm256_packs_epi32(__A, __B),
                                          (__v16hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_packs_epi16(__mmask16 __M, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_packs_epi16(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}
````
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L554 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L554 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L555 EN**: Continues logic associated with callable symbol `_mm256_maskz_packs_epi32`.
  **L555 CN**: 继续与可调用符号 `_mm256_maskz_packs_epi32` 相关的逻辑。
- **L556 EN**: Opens a new lexical scope or compound statement.
  **L556 CN**: 打开一个新的词法作用域或复合语句块。
- **L557 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L557 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_packs_epi32(__A, __B),`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_packs_epi32(__A, __B),`。
- **L559 EN**: Executes a call or declaration centered on `statement`.
  **L559 CN**: 执行以 `statement` 为核心的调用或声明。
- **L560 EN**: Closes the current lexical scope or compound statement.
  **L560 CN**: 结束当前词法作用域或复合语句块。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L562 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L562 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L563 EN**: Continues logic associated with callable symbol `_mm256_mask_packs_epi32`.
  **L563 CN**: 继续与可调用符号 `_mm256_mask_packs_epi32` 相关的逻辑。
- **L564 EN**: Opens a new lexical scope or compound statement.
  **L564 CN**: 打开一个新的词法作用域或复合语句块。
- **L565 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L565 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_packs_epi32(__A, __B),`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_packs_epi32(__A, __B),`。
- **L567 EN**: Executes a call or declaration centered on `statement`.
  **L567 CN**: 执行以 `statement` 为核心的调用或声明。
- **L568 EN**: Closes the current lexical scope or compound statement.
  **L568 CN**: 结束当前词法作用域或复合语句块。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L570 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L570 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L571 EN**: Continues logic associated with callable symbol `_mm_maskz_packs_epi16`.
  **L571 CN**: 继续与可调用符号 `_mm_maskz_packs_epi16` 相关的逻辑。
- **L572 EN**: Opens a new lexical scope or compound statement.
  **L572 CN**: 打开一个新的词法作用域或复合语句块。
- **L573 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L573 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_packs_epi16(__A, __B),`.
  **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_packs_epi16(__A, __B),`。
- **L575 EN**: Executes a call or declaration centered on `statement`.
  **L575 CN**: 执行以 `statement` 为核心的调用或声明。
- **L576 EN**: Closes the current lexical scope or compound statement.
  **L576 CN**: 结束当前词法作用域或复合语句块。

### Lines 577-600

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_packs_epi16(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_packs_epi16(__A, __B),
                                             (__v16qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_packs_epi16(__mmask32 __M, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                          (__v32qi)_mm256_packs_epi16(__A, __B),
                                          (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_packs_epi16(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                          (__v32qi)_mm256_packs_epi16(__A, __B),
                                          (__v32qi)__W);
}
````
- **L577 EN**: Blank line separating nearby declarations or logic blocks.
  **L577 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L578 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L578 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L579 EN**: Continues logic associated with callable symbol `_mm_mask_packs_epi16`.
  **L579 CN**: 继续与可调用符号 `_mm_mask_packs_epi16` 相关的逻辑。
- **L580 EN**: Opens a new lexical scope or compound statement.
  **L580 CN**: 打开一个新的词法作用域或复合语句块。
- **L581 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L581 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_packs_epi16(__A, __B),`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_packs_epi16(__A, __B),`。
- **L583 EN**: Executes a call or declaration centered on `statement`.
  **L583 CN**: 执行以 `statement` 为核心的调用或声明。
- **L584 EN**: Closes the current lexical scope or compound statement.
  **L584 CN**: 结束当前词法作用域或复合语句块。
- **L585 EN**: Blank line separating nearby declarations or logic blocks.
  **L585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L586 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L586 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L587 EN**: Continues logic associated with callable symbol `_mm256_maskz_packs_epi16`.
  **L587 CN**: 继续与可调用符号 `_mm256_maskz_packs_epi16` 相关的逻辑。
- **L588 EN**: Opens a new lexical scope or compound statement.
  **L588 CN**: 打开一个新的词法作用域或复合语句块。
- **L589 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L589 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_packs_epi16(__A, __B),`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_packs_epi16(__A, __B),`。
- **L591 EN**: Executes a call or declaration centered on `statement`.
  **L591 CN**: 执行以 `statement` 为核心的调用或声明。
- **L592 EN**: Closes the current lexical scope or compound statement.
  **L592 CN**: 结束当前词法作用域或复合语句块。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L594 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L595 EN**: Continues logic associated with callable symbol `_mm256_mask_packs_epi16`.
  **L595 CN**: 继续与可调用符号 `_mm256_mask_packs_epi16` 相关的逻辑。
- **L596 EN**: Opens a new lexical scope or compound statement.
  **L596 CN**: 打开一个新的词法作用域或复合语句块。
- **L597 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L597 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_packs_epi16(__A, __B),`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_packs_epi16(__A, __B),`。
- **L599 EN**: Executes a call or declaration centered on `statement`.
  **L599 CN**: 执行以 `statement` 为核心的调用或声明。
- **L600 EN**: Closes the current lexical scope or compound statement.
  **L600 CN**: 结束当前词法作用域或复合语句块。

### Lines 601-624

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_packus_epi32(__mmask8 __M, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_packus_epi32(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_packus_epi32(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_packus_epi32(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_packus_epi32(__mmask16 __M, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                         (__v16hi)_mm256_packus_epi32(__A, __B),
                                         (__v16hi)_mm256_setzero_si256());
}
````
- **L601 EN**: Blank line separating nearby declarations or logic blocks.
  **L601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L602 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L602 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L603 EN**: Continues logic associated with callable symbol `_mm_maskz_packus_epi32`.
  **L603 CN**: 继续与可调用符号 `_mm_maskz_packus_epi32` 相关的逻辑。
- **L604 EN**: Opens a new lexical scope or compound statement.
  **L604 CN**: 打开一个新的词法作用域或复合语句块。
- **L605 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L605 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_packus_epi32(__A, __B),`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_packus_epi32(__A, __B),`。
- **L607 EN**: Executes a call or declaration centered on `statement`.
  **L607 CN**: 执行以 `statement` 为核心的调用或声明。
- **L608 EN**: Closes the current lexical scope or compound statement.
  **L608 CN**: 结束当前词法作用域或复合语句块。
- **L609 EN**: Blank line separating nearby declarations or logic blocks.
  **L609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L610 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L610 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L611 EN**: Continues logic associated with callable symbol `_mm_mask_packus_epi32`.
  **L611 CN**: 继续与可调用符号 `_mm_mask_packus_epi32` 相关的逻辑。
- **L612 EN**: Opens a new lexical scope or compound statement.
  **L612 CN**: 打开一个新的词法作用域或复合语句块。
- **L613 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L613 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_packus_epi32(__A, __B),`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_packus_epi32(__A, __B),`。
- **L615 EN**: Executes a call or declaration centered on `statement`.
  **L615 CN**: 执行以 `statement` 为核心的调用或声明。
- **L616 EN**: Closes the current lexical scope or compound statement.
  **L616 CN**: 结束当前词法作用域或复合语句块。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L618 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L618 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L619 EN**: Continues logic associated with callable symbol `_mm256_maskz_packus_epi32`.
  **L619 CN**: 继续与可调用符号 `_mm256_maskz_packus_epi32` 相关的逻辑。
- **L620 EN**: Opens a new lexical scope or compound statement.
  **L620 CN**: 打开一个新的词法作用域或复合语句块。
- **L621 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L621 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_packus_epi32(__A, __B),`.
  **L622 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_packus_epi32(__A, __B),`。
- **L623 EN**: Executes a call or declaration centered on `statement`.
  **L623 CN**: 执行以 `statement` 为核心的调用或声明。
- **L624 EN**: Closes the current lexical scope or compound statement.
  **L624 CN**: 结束当前词法作用域或复合语句块。

### Lines 625-648

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_packus_epi32(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                         (__v16hi)_mm256_packus_epi32(__A, __B),
                                         (__v16hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_packus_epi16(__mmask16 __M, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                            (__v16qi)_mm_packus_epi16(__A, __B),
                                            (__v16qi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_packus_epi16(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                            (__v16qi)_mm_packus_epi16(__A, __B),
                                            (__v16qi)__W);
}
````
- **L625 EN**: Blank line separating nearby declarations or logic blocks.
  **L625 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L626 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L626 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L627 EN**: Continues logic associated with callable symbol `_mm256_mask_packus_epi32`.
  **L627 CN**: 继续与可调用符号 `_mm256_mask_packus_epi32` 相关的逻辑。
- **L628 EN**: Opens a new lexical scope or compound statement.
  **L628 CN**: 打开一个新的词法作用域或复合语句块。
- **L629 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L629 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_packus_epi32(__A, __B),`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_packus_epi32(__A, __B),`。
- **L631 EN**: Executes a call or declaration centered on `statement`.
  **L631 CN**: 执行以 `statement` 为核心的调用或声明。
- **L632 EN**: Closes the current lexical scope or compound statement.
  **L632 CN**: 结束当前词法作用域或复合语句块。
- **L633 EN**: Blank line separating nearby declarations or logic blocks.
  **L633 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L634 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L634 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L635 EN**: Continues logic associated with callable symbol `_mm_maskz_packus_epi16`.
  **L635 CN**: 继续与可调用符号 `_mm_maskz_packus_epi16` 相关的逻辑。
- **L636 EN**: Opens a new lexical scope or compound statement.
  **L636 CN**: 打开一个新的词法作用域或复合语句块。
- **L637 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L637 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_packus_epi16(__A, __B),`.
  **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_packus_epi16(__A, __B),`。
- **L639 EN**: Executes a call or declaration centered on `statement`.
  **L639 CN**: 执行以 `statement` 为核心的调用或声明。
- **L640 EN**: Closes the current lexical scope or compound statement.
  **L640 CN**: 结束当前词法作用域或复合语句块。
- **L641 EN**: Blank line separating nearby declarations or logic blocks.
  **L641 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L642 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L642 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L643 EN**: Continues logic associated with callable symbol `_mm_mask_packus_epi16`.
  **L643 CN**: 继续与可调用符号 `_mm_mask_packus_epi16` 相关的逻辑。
- **L644 EN**: Opens a new lexical scope or compound statement.
  **L644 CN**: 打开一个新的词法作用域或复合语句块。
- **L645 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L645 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_packus_epi16(__A, __B),`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_packus_epi16(__A, __B),`。
- **L647 EN**: Executes a call or declaration centered on `statement`.
  **L647 CN**: 执行以 `statement` 为核心的调用或声明。
- **L648 EN**: Closes the current lexical scope or compound statement.
  **L648 CN**: 结束当前词法作用域或复合语句块。

### Lines 649-672

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_packus_epi16(__mmask32 __M, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                         (__v32qi)_mm256_packus_epi16(__A, __B),
                                         (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_packus_epi16(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                         (__v32qi)_mm256_packus_epi16(__A, __B),
                                         (__v32qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_adds_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_adds_epi8(__A, __B),
                                             (__v16qi)__W);
}
````
- **L649 EN**: Blank line separating nearby declarations or logic blocks.
  **L649 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L650 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L650 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L651 EN**: Continues logic associated with callable symbol `_mm256_maskz_packus_epi16`.
  **L651 CN**: 继续与可调用符号 `_mm256_maskz_packus_epi16` 相关的逻辑。
- **L652 EN**: Opens a new lexical scope or compound statement.
  **L652 CN**: 打开一个新的词法作用域或复合语句块。
- **L653 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L653 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_packus_epi16(__A, __B),`.
  **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_packus_epi16(__A, __B),`。
- **L655 EN**: Executes a call or declaration centered on `statement`.
  **L655 CN**: 执行以 `statement` 为核心的调用或声明。
- **L656 EN**: Closes the current lexical scope or compound statement.
  **L656 CN**: 结束当前词法作用域或复合语句块。
- **L657 EN**: Blank line separating nearby declarations or logic blocks.
  **L657 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L658 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L658 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L659 EN**: Continues logic associated with callable symbol `_mm256_mask_packus_epi16`.
  **L659 CN**: 继续与可调用符号 `_mm256_mask_packus_epi16` 相关的逻辑。
- **L660 EN**: Opens a new lexical scope or compound statement.
  **L660 CN**: 打开一个新的词法作用域或复合语句块。
- **L661 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L661 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_packus_epi16(__A, __B),`.
  **L662 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_packus_epi16(__A, __B),`。
- **L663 EN**: Executes a call or declaration centered on `statement`.
  **L663 CN**: 执行以 `statement` 为核心的调用或声明。
- **L664 EN**: Closes the current lexical scope or compound statement.
  **L664 CN**: 结束当前词法作用域或复合语句块。
- **L665 EN**: Blank line separating nearby declarations or logic blocks.
  **L665 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L666 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L666 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L667 EN**: Continues logic associated with callable symbol `_mm_mask_adds_epi8`.
  **L667 CN**: 继续与可调用符号 `_mm_mask_adds_epi8` 相关的逻辑。
- **L668 EN**: Opens a new lexical scope or compound statement.
  **L668 CN**: 打开一个新的词法作用域或复合语句块。
- **L669 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L669 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_adds_epi8(__A, __B),`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_adds_epi8(__A, __B),`。
- **L671 EN**: Executes a call or declaration centered on `statement`.
  **L671 CN**: 执行以 `statement` 为核心的调用或声明。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_adds_epi8(__mmask16 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_adds_epi8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_adds_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                            (__v32qi)_mm256_adds_epi8(__A, __B),
                                            (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_adds_epi8(__mmask32 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                            (__v32qi)_mm256_adds_epi8(__A, __B),
                                            (__v32qi)_mm256_setzero_si256());
}
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L674 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L675 EN**: Continues logic associated with callable symbol `_mm_maskz_adds_epi8`.
  **L675 CN**: 继续与可调用符号 `_mm_maskz_adds_epi8` 相关的逻辑。
- **L676 EN**: Opens a new lexical scope or compound statement.
  **L676 CN**: 打开一个新的词法作用域或复合语句块。
- **L677 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L677 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_adds_epi8(__A, __B),`.
  **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_adds_epi8(__A, __B),`。
- **L679 EN**: Executes a call or declaration centered on `statement`.
  **L679 CN**: 执行以 `statement` 为核心的调用或声明。
- **L680 EN**: Closes the current lexical scope or compound statement.
  **L680 CN**: 结束当前词法作用域或复合语句块。
- **L681 EN**: Blank line separating nearby declarations or logic blocks.
  **L681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L682 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L682 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L683 EN**: Continues logic associated with callable symbol `_mm256_mask_adds_epi8`.
  **L683 CN**: 继续与可调用符号 `_mm256_mask_adds_epi8` 相关的逻辑。
- **L684 EN**: Opens a new lexical scope or compound statement.
  **L684 CN**: 打开一个新的词法作用域或复合语句块。
- **L685 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L685 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_adds_epi8(__A, __B),`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_adds_epi8(__A, __B),`。
- **L687 EN**: Executes a call or declaration centered on `statement`.
  **L687 CN**: 执行以 `statement` 为核心的调用或声明。
- **L688 EN**: Closes the current lexical scope or compound statement.
  **L688 CN**: 结束当前词法作用域或复合语句块。
- **L689 EN**: Blank line separating nearby declarations or logic blocks.
  **L689 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L690 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L690 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L691 EN**: Continues logic associated with callable symbol `_mm256_maskz_adds_epi8`.
  **L691 CN**: 继续与可调用符号 `_mm256_maskz_adds_epi8` 相关的逻辑。
- **L692 EN**: Opens a new lexical scope or compound statement.
  **L692 CN**: 打开一个新的词法作用域或复合语句块。
- **L693 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L693 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_adds_epi8(__A, __B),`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_adds_epi8(__A, __B),`。
- **L695 EN**: Executes a call or declaration centered on `statement`.
  **L695 CN**: 执行以 `statement` 为核心的调用或声明。
- **L696 EN**: Closes the current lexical scope or compound statement.
  **L696 CN**: 结束当前词法作用域或复合语句块。

### Lines 697-720

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_adds_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_adds_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_adds_epi16(__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_adds_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_adds_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_adds_epi16(__A, __B),
                                           (__v16hi)__W);
}
````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L698 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L698 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L699 EN**: Continues logic associated with callable symbol `_mm_mask_adds_epi16`.
  **L699 CN**: 继续与可调用符号 `_mm_mask_adds_epi16` 相关的逻辑。
- **L700 EN**: Opens a new lexical scope or compound statement.
  **L700 CN**: 打开一个新的词法作用域或复合语句块。
- **L701 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L701 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_adds_epi16(__A, __B),`.
  **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_adds_epi16(__A, __B),`。
- **L703 EN**: Executes a call or declaration centered on `statement`.
  **L703 CN**: 执行以 `statement` 为核心的调用或声明。
- **L704 EN**: Closes the current lexical scope or compound statement.
  **L704 CN**: 结束当前词法作用域或复合语句块。
- **L705 EN**: Blank line separating nearby declarations or logic blocks.
  **L705 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L706 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L706 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L707 EN**: Continues logic associated with callable symbol `_mm_maskz_adds_epi16`.
  **L707 CN**: 继续与可调用符号 `_mm_maskz_adds_epi16` 相关的逻辑。
- **L708 EN**: Opens a new lexical scope or compound statement.
  **L708 CN**: 打开一个新的词法作用域或复合语句块。
- **L709 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L709 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_adds_epi16(__A, __B),`.
  **L710 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_adds_epi16(__A, __B),`。
- **L711 EN**: Executes a call or declaration centered on `statement`.
  **L711 CN**: 执行以 `statement` 为核心的调用或声明。
- **L712 EN**: Closes the current lexical scope or compound statement.
  **L712 CN**: 结束当前词法作用域或复合语句块。
- **L713 EN**: Blank line separating nearby declarations or logic blocks.
  **L713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L714 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L714 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L715 EN**: Continues logic associated with callable symbol `_mm256_mask_adds_epi16`.
  **L715 CN**: 继续与可调用符号 `_mm256_mask_adds_epi16` 相关的逻辑。
- **L716 EN**: Opens a new lexical scope or compound statement.
  **L716 CN**: 打开一个新的词法作用域或复合语句块。
- **L717 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L717 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_adds_epi16(__A, __B),`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_adds_epi16(__A, __B),`。
- **L719 EN**: Executes a call or declaration centered on `statement`.
  **L719 CN**: 执行以 `statement` 为核心的调用或声明。
- **L720 EN**: Closes the current lexical scope or compound statement.
  **L720 CN**: 结束当前词法作用域或复合语句块。

### Lines 721-744

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_adds_epi16(__mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_adds_epi16(__A, __B),
                                           (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_adds_epu8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_adds_epu8(__A, __B),
                                             (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_adds_epu8(__mmask16 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_adds_epu8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}
````
- **L721 EN**: Blank line separating nearby declarations or logic blocks.
  **L721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L722 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L722 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L723 EN**: Continues logic associated with callable symbol `_mm256_maskz_adds_epi16`.
  **L723 CN**: 继续与可调用符号 `_mm256_maskz_adds_epi16` 相关的逻辑。
- **L724 EN**: Opens a new lexical scope or compound statement.
  **L724 CN**: 打开一个新的词法作用域或复合语句块。
- **L725 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L725 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_adds_epi16(__A, __B),`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_adds_epi16(__A, __B),`。
- **L727 EN**: Executes a call or declaration centered on `statement`.
  **L727 CN**: 执行以 `statement` 为核心的调用或声明。
- **L728 EN**: Closes the current lexical scope or compound statement.
  **L728 CN**: 结束当前词法作用域或复合语句块。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L730 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L730 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L731 EN**: Continues logic associated with callable symbol `_mm_mask_adds_epu8`.
  **L731 CN**: 继续与可调用符号 `_mm_mask_adds_epu8` 相关的逻辑。
- **L732 EN**: Opens a new lexical scope or compound statement.
  **L732 CN**: 打开一个新的词法作用域或复合语句块。
- **L733 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L733 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_adds_epu8(__A, __B),`.
  **L734 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_adds_epu8(__A, __B),`。
- **L735 EN**: Executes a call or declaration centered on `statement`.
  **L735 CN**: 执行以 `statement` 为核心的调用或声明。
- **L736 EN**: Closes the current lexical scope or compound statement.
  **L736 CN**: 结束当前词法作用域或复合语句块。
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L738 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L738 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L739 EN**: Continues logic associated with callable symbol `_mm_maskz_adds_epu8`.
  **L739 CN**: 继续与可调用符号 `_mm_maskz_adds_epu8` 相关的逻辑。
- **L740 EN**: Opens a new lexical scope or compound statement.
  **L740 CN**: 打开一个新的词法作用域或复合语句块。
- **L741 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L741 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_adds_epu8(__A, __B),`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_adds_epu8(__A, __B),`。
- **L743 EN**: Executes a call or declaration centered on `statement`.
  **L743 CN**: 执行以 `statement` 为核心的调用或声明。
- **L744 EN**: Closes the current lexical scope or compound statement.
  **L744 CN**: 结束当前词法作用域或复合语句块。

### Lines 745-768

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_adds_epu8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                            (__v32qi)_mm256_adds_epu8(__A, __B),
                                            (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_adds_epu8(__mmask32 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                            (__v32qi)_mm256_adds_epu8(__A, __B),
                                            (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_adds_epu16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_adds_epu16(__A, __B),
                                             (__v8hi)__W);
}
````
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L746 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L746 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L747 EN**: Continues logic associated with callable symbol `_mm256_mask_adds_epu8`.
  **L747 CN**: 继续与可调用符号 `_mm256_mask_adds_epu8` 相关的逻辑。
- **L748 EN**: Opens a new lexical scope or compound statement.
  **L748 CN**: 打开一个新的词法作用域或复合语句块。
- **L749 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L749 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_adds_epu8(__A, __B),`.
  **L750 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_adds_epu8(__A, __B),`。
- **L751 EN**: Executes a call or declaration centered on `statement`.
  **L751 CN**: 执行以 `statement` 为核心的调用或声明。
- **L752 EN**: Closes the current lexical scope or compound statement.
  **L752 CN**: 结束当前词法作用域或复合语句块。
- **L753 EN**: Blank line separating nearby declarations or logic blocks.
  **L753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L754 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L754 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L755 EN**: Continues logic associated with callable symbol `_mm256_maskz_adds_epu8`.
  **L755 CN**: 继续与可调用符号 `_mm256_maskz_adds_epu8` 相关的逻辑。
- **L756 EN**: Opens a new lexical scope or compound statement.
  **L756 CN**: 打开一个新的词法作用域或复合语句块。
- **L757 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L757 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_adds_epu8(__A, __B),`.
  **L758 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_adds_epu8(__A, __B),`。
- **L759 EN**: Executes a call or declaration centered on `statement`.
  **L759 CN**: 执行以 `statement` 为核心的调用或声明。
- **L760 EN**: Closes the current lexical scope or compound statement.
  **L760 CN**: 结束当前词法作用域或复合语句块。
- **L761 EN**: Blank line separating nearby declarations or logic blocks.
  **L761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L762 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L762 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L763 EN**: Continues logic associated with callable symbol `_mm_mask_adds_epu16`.
  **L763 CN**: 继续与可调用符号 `_mm_mask_adds_epu16` 相关的逻辑。
- **L764 EN**: Opens a new lexical scope or compound statement.
  **L764 CN**: 打开一个新的词法作用域或复合语句块。
- **L765 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L765 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_adds_epu16(__A, __B),`.
  **L766 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_adds_epu16(__A, __B),`。
- **L767 EN**: Executes a call or declaration centered on `statement`.
  **L767 CN**: 执行以 `statement` 为核心的调用或声明。
- **L768 EN**: Closes the current lexical scope or compound statement.
  **L768 CN**: 结束当前词法作用域或复合语句块。

### Lines 769-792

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_adds_epu16(__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_adds_epu16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_adds_epu16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_adds_epu16(__A, __B),
                                           (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_adds_epu16(__mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_adds_epu16(__A, __B),
                                           (__v16hi)_mm256_setzero_si256());
}
````
- **L769 EN**: Blank line separating nearby declarations or logic blocks.
  **L769 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L770 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L770 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L771 EN**: Continues logic associated with callable symbol `_mm_maskz_adds_epu16`.
  **L771 CN**: 继续与可调用符号 `_mm_maskz_adds_epu16` 相关的逻辑。
- **L772 EN**: Opens a new lexical scope or compound statement.
  **L772 CN**: 打开一个新的词法作用域或复合语句块。
- **L773 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L773 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_adds_epu16(__A, __B),`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_adds_epu16(__A, __B),`。
- **L775 EN**: Executes a call or declaration centered on `statement`.
  **L775 CN**: 执行以 `statement` 为核心的调用或声明。
- **L776 EN**: Closes the current lexical scope or compound statement.
  **L776 CN**: 结束当前词法作用域或复合语句块。
- **L777 EN**: Blank line separating nearby declarations or logic blocks.
  **L777 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L778 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L778 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L779 EN**: Continues logic associated with callable symbol `_mm256_mask_adds_epu16`.
  **L779 CN**: 继续与可调用符号 `_mm256_mask_adds_epu16` 相关的逻辑。
- **L780 EN**: Opens a new lexical scope or compound statement.
  **L780 CN**: 打开一个新的词法作用域或复合语句块。
- **L781 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L781 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_adds_epu16(__A, __B),`.
  **L782 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_adds_epu16(__A, __B),`。
- **L783 EN**: Executes a call or declaration centered on `statement`.
  **L783 CN**: 执行以 `statement` 为核心的调用或声明。
- **L784 EN**: Closes the current lexical scope or compound statement.
  **L784 CN**: 结束当前词法作用域或复合语句块。
- **L785 EN**: Blank line separating nearby declarations or logic blocks.
  **L785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L786 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L786 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L787 EN**: Continues logic associated with callable symbol `_mm256_maskz_adds_epu16`.
  **L787 CN**: 继续与可调用符号 `_mm256_maskz_adds_epu16` 相关的逻辑。
- **L788 EN**: Opens a new lexical scope or compound statement.
  **L788 CN**: 打开一个新的词法作用域或复合语句块。
- **L789 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L789 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_adds_epu16(__A, __B),`.
  **L790 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_adds_epu16(__A, __B),`。
- **L791 EN**: Executes a call or declaration centered on `statement`.
  **L791 CN**: 执行以 `statement` 为核心的调用或声明。
- **L792 EN**: Closes the current lexical scope or compound statement.
  **L792 CN**: 结束当前词法作用域或复合语句块。

### Lines 793-816

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_avg_epu8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_avg_epu8(__A, __B), (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_avg_epu8(__mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_avg_epu8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_avg_epu8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256(
      (__mmask32)__U, (__v32qi)_mm256_avg_epu8(__A, __B), (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_avg_epu8(__mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                             (__v32qi)_mm256_avg_epu8(__A, __B),
````
- **L793 EN**: Blank line separating nearby declarations or logic blocks.
  **L793 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L794 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L794 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L795 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_avg_epu8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`.
  **L795 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_avg_epu8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`。
- **L796 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L796 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L797 EN**: Executes a call or declaration centered on `statement`.
  **L797 CN**: 执行以 `statement` 为核心的调用或声明。
- **L798 EN**: Closes the current lexical scope or compound statement.
  **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L800 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L800 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L801 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_avg_epu8(__mmask16 __U, __m128i __A, __m128i __B) {`.
  **L801 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_avg_epu8(__mmask16 __U, __m128i __A, __m128i __B) {`。
- **L802 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L802 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_avg_epu8(__A, __B),`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_avg_epu8(__A, __B),`。
- **L804 EN**: Executes a call or declaration centered on `statement`.
  **L804 CN**: 执行以 `statement` 为核心的调用或声明。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L807 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L807 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L808 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_avg_epu8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`.
  **L808 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_avg_epu8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`。
- **L809 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(`.
  **L809 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(` 从当前函数返回。
- **L810 EN**: Executes a call or declaration centered on `statement`.
  **L810 CN**: 执行以 `statement` 为核心的调用或声明。
- **L811 EN**: Closes the current lexical scope or compound statement.
  **L811 CN**: 结束当前词法作用域或复合语句块。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L813 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L814 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_avg_epu8(__mmask32 __U, __m256i __A, __m256i __B) {`.
  **L814 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_avg_epu8(__mmask32 __U, __m256i __A, __m256i __B) {`。
- **L815 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L815 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_avg_epu8(__A, __B),`.
  **L816 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_avg_epu8(__A, __B),`。

### Lines 817-840

````c
                                             (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_avg_epu16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128(
      (__mmask8)__U, (__v8hi)_mm_avg_epu16(__A, __B), (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_avg_epu16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_avg_epu16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_avg_epu16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256(
      (__mmask16)__U, (__v16hi)_mm256_avg_epu16(__A, __B), (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_avg_epu16(__mmask16 __U, __m256i __A, __m256i __B) {
````
- **L817 EN**: Executes a call or declaration centered on `statement`.
  **L817 CN**: 执行以 `statement` 为核心的调用或声明。
- **L818 EN**: Closes the current lexical scope or compound statement.
  **L818 CN**: 结束当前词法作用域或复合语句块。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L820 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L820 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L821 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_avg_epu16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L821 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_avg_epu16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L822 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(`.
  **L822 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(` 从当前函数返回。
- **L823 EN**: Executes a call or declaration centered on `statement`.
  **L823 CN**: 执行以 `statement` 为核心的调用或声明。
- **L824 EN**: Closes the current lexical scope or compound statement.
  **L824 CN**: 结束当前词法作用域或复合语句块。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L826 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L827 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_avg_epu16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L827 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_avg_epu16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L828 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L828 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_avg_epu16(__A, __B),`.
  **L829 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_avg_epu16(__A, __B),`。
- **L830 EN**: Executes a call or declaration centered on `statement`.
  **L830 CN**: 执行以 `statement` 为核心的调用或声明。
- **L831 EN**: Closes the current lexical scope or compound statement.
  **L831 CN**: 结束当前词法作用域或复合语句块。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L833 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L833 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L834 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_avg_epu16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`.
  **L834 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_avg_epu16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`。
- **L835 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(`.
  **L835 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(` 从当前函数返回。
- **L836 EN**: Executes a call or declaration centered on `statement`.
  **L836 CN**: 执行以 `statement` 为核心的调用或声明。
- **L837 EN**: Closes the current lexical scope or compound statement.
  **L837 CN**: 结束当前词法作用域或复合语句块。
- **L838 EN**: Blank line separating nearby declarations or logic blocks.
  **L838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L839 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L839 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L840 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_avg_epu16(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L840 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_avg_epu16(__mmask16 __U, __m256i __A, __m256i __B) {`。

### Lines 841-864

````c
  return (__m256i)__builtin_ia32_selectw_256(
      (__mmask16)__U, (__v16hi)_mm256_avg_epu16(__A, __B),
      (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_max_epi8(__mmask16 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_max_epi8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_max_epi8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_max_epi8(__A, __B),
                                             (__v16qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_max_epi8(__mmask32 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                             (__v32qi)_mm256_max_epi8(__A, __B),
                                             (__v32qi)_mm256_setzero_si256());
````
- **L841 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(`.
  **L841 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(` 从当前函数返回。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16hi)_mm256_avg_epu16(__A, __B),`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16hi)_mm256_avg_epu16(__A, __B),`。
- **L843 EN**: Executes a call or declaration centered on `statement`.
  **L843 CN**: 执行以 `statement` 为核心的调用或声明。
- **L844 EN**: Closes the current lexical scope or compound statement.
  **L844 CN**: 结束当前词法作用域或复合语句块。
- **L845 EN**: Blank line separating nearby declarations or logic blocks.
  **L845 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L846 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L846 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L847 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_max_epi8(__mmask16 __M, __m128i __A, __m128i __B) {`.
  **L847 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_max_epi8(__mmask16 __M, __m128i __A, __m128i __B) {`。
- **L848 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L848 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_max_epi8(__A, __B),`.
  **L849 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_max_epi8(__A, __B),`。
- **L850 EN**: Executes a call or declaration centered on `statement`.
  **L850 CN**: 执行以 `statement` 为核心的调用或声明。
- **L851 EN**: Closes the current lexical scope or compound statement.
  **L851 CN**: 结束当前词法作用域或复合语句块。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L853 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L854 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_max_epi8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {`.
  **L854 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_max_epi8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {`。
- **L855 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L855 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_max_epi8(__A, __B),`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_max_epi8(__A, __B),`。
- **L857 EN**: Executes a call or declaration centered on `statement`.
  **L857 CN**: 执行以 `statement` 为核心的调用或声明。
- **L858 EN**: Closes the current lexical scope or compound statement.
  **L858 CN**: 结束当前词法作用域或复合语句块。
- **L859 EN**: Blank line separating nearby declarations or logic blocks.
  **L859 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L860 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L860 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L861 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_max_epi8(__mmask32 __M, __m256i __A, __m256i __B) {`.
  **L861 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_max_epi8(__mmask32 __M, __m256i __A, __m256i __B) {`。
- **L862 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L862 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_max_epi8(__A, __B),`.
  **L863 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_max_epi8(__A, __B),`。
- **L864 EN**: Executes a call or declaration centered on `statement`.
  **L864 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 865-888

````c
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_max_epi8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                             (__v32qi)_mm256_max_epi8(__A, __B),
                                             (__v32qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_max_epi16(__mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_max_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_max_epi16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_max_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L867 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L868 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_max_epi8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {`.
  **L868 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_max_epi8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {`。
- **L869 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L869 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_max_epi8(__A, __B),`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_max_epi8(__A, __B),`。
- **L871 EN**: Executes a call or declaration centered on `statement`.
  **L871 CN**: 执行以 `statement` 为核心的调用或声明。
- **L872 EN**: Closes the current lexical scope or compound statement.
  **L872 CN**: 结束当前词法作用域或复合语句块。
- **L873 EN**: Blank line separating nearby declarations or logic blocks.
  **L873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L874 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L874 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L875 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_max_epi16(__mmask8 __M, __m128i __A, __m128i __B) {`.
  **L875 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_max_epi16(__mmask8 __M, __m128i __A, __m128i __B) {`。
- **L876 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L876 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_max_epi16(__A, __B),`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_max_epi16(__A, __B),`。
- **L878 EN**: Executes a call or declaration centered on `statement`.
  **L878 CN**: 执行以 `statement` 为核心的调用或声明。
- **L879 EN**: Closes the current lexical scope or compound statement.
  **L879 CN**: 结束当前词法作用域或复合语句块。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L881 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L881 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L882 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_max_epi16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {`.
  **L882 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_max_epi16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {`。
- **L883 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L883 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_max_epi16(__A, __B),`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_max_epi16(__A, __B),`。
- **L885 EN**: Executes a call or declaration centered on `statement`.
  **L885 CN**: 执行以 `statement` 为核心的调用或声明。
- **L886 EN**: Closes the current lexical scope or compound statement.
  **L886 CN**: 结束当前词法作用域或复合语句块。
- **L887 EN**: Blank line separating nearby declarations or logic blocks.
  **L887 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L888 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L888 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 889-912

````c
_mm256_maskz_max_epi16(__mmask16 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                            (__v16hi)_mm256_max_epi16(__A, __B),
                                            (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_max_epi16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                            (__v16hi)_mm256_max_epi16(__A, __B),
                                            (__v16hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_max_epu8(__mmask16 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_max_epu8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_max_epu8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_max_epu8(__A, __B),
````
- **L889 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_max_epi16(__mmask16 __M, __m256i __A, __m256i __B) {`.
  **L889 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_max_epi16(__mmask16 __M, __m256i __A, __m256i __B) {`。
- **L890 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L890 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_max_epi16(__A, __B),`.
  **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_max_epi16(__A, __B),`。
- **L892 EN**: Executes a call or declaration centered on `statement`.
  **L892 CN**: 执行以 `statement` 为核心的调用或声明。
- **L893 EN**: Closes the current lexical scope or compound statement.
  **L893 CN**: 结束当前词法作用域或复合语句块。
- **L894 EN**: Blank line separating nearby declarations or logic blocks.
  **L894 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L895 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L895 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L896 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_max_epi16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {`.
  **L896 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_max_epi16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {`。
- **L897 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L897 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_max_epi16(__A, __B),`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_max_epi16(__A, __B),`。
- **L899 EN**: Executes a call or declaration centered on `statement`.
  **L899 CN**: 执行以 `statement` 为核心的调用或声明。
- **L900 EN**: Closes the current lexical scope or compound statement.
  **L900 CN**: 结束当前词法作用域或复合语句块。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L902 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L902 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L903 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_max_epu8(__mmask16 __M, __m128i __A, __m128i __B) {`.
  **L903 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_max_epu8(__mmask16 __M, __m128i __A, __m128i __B) {`。
- **L904 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L904 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_max_epu8(__A, __B),`.
  **L905 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_max_epu8(__A, __B),`。
- **L906 EN**: Executes a call or declaration centered on `statement`.
  **L906 CN**: 执行以 `statement` 为核心的调用或声明。
- **L907 EN**: Closes the current lexical scope or compound statement.
  **L907 CN**: 结束当前词法作用域或复合语句块。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L909 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L909 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L910 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_max_epu8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {`.
  **L910 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_max_epu8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {`。
- **L911 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L911 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_max_epu8(__A, __B),`.
  **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_max_epu8(__A, __B),`。

### Lines 913-936

````c
                                             (__v16qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_max_epu8(__mmask32 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                             (__v32qi)_mm256_max_epu8(__A, __B),
                                             (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_max_epu8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                             (__v32qi)_mm256_max_epu8(__A, __B),
                                             (__v32qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_max_epu16(__mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_max_epu16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

````
- **L913 EN**: Executes a call or declaration centered on `statement`.
  **L913 CN**: 执行以 `statement` 为核心的调用或声明。
- **L914 EN**: Closes the current lexical scope or compound statement.
  **L914 CN**: 结束当前词法作用域或复合语句块。
- **L915 EN**: Blank line separating nearby declarations or logic blocks.
  **L915 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L916 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L916 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L917 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_max_epu8(__mmask32 __M, __m256i __A, __m256i __B) {`.
  **L917 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_max_epu8(__mmask32 __M, __m256i __A, __m256i __B) {`。
- **L918 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L918 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_max_epu8(__A, __B),`.
  **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_max_epu8(__A, __B),`。
- **L920 EN**: Executes a call or declaration centered on `statement`.
  **L920 CN**: 执行以 `statement` 为核心的调用或声明。
- **L921 EN**: Closes the current lexical scope or compound statement.
  **L921 CN**: 结束当前词法作用域或复合语句块。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L923 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L923 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L924 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_max_epu8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {`.
  **L924 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_max_epu8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {`。
- **L925 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L925 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_max_epu8(__A, __B),`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_max_epu8(__A, __B),`。
- **L927 EN**: Executes a call or declaration centered on `statement`.
  **L927 CN**: 执行以 `statement` 为核心的调用或声明。
- **L928 EN**: Closes the current lexical scope or compound statement.
  **L928 CN**: 结束当前词法作用域或复合语句块。
- **L929 EN**: Blank line separating nearby declarations or logic blocks.
  **L929 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L930 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L930 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L931 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_max_epu16(__mmask8 __M, __m128i __A, __m128i __B) {`.
  **L931 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_max_epu16(__mmask8 __M, __m128i __A, __m128i __B) {`。
- **L932 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L932 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_max_epu16(__A, __B),`.
  **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_max_epu16(__A, __B),`。
- **L934 EN**: Executes a call or declaration centered on `statement`.
  **L934 CN**: 执行以 `statement` 为核心的调用或声明。
- **L935 EN**: Closes the current lexical scope or compound statement.
  **L935 CN**: 结束当前词法作用域或复合语句块。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 937-960

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_max_epu16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_max_epu16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_max_epu16(__mmask16 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                            (__v16hi)_mm256_max_epu16(__A, __B),
                                            (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_max_epu16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                            (__v16hi)_mm256_max_epu16(__A, __B),
                                            (__v16hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_min_epi8(__mmask16 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
````
- **L937 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L937 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L938 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_max_epu16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {`.
  **L938 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_max_epu16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {`。
- **L939 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L939 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_max_epu16(__A, __B),`.
  **L940 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_max_epu16(__A, __B),`。
- **L941 EN**: Executes a call or declaration centered on `statement`.
  **L941 CN**: 执行以 `statement` 为核心的调用或声明。
- **L942 EN**: Closes the current lexical scope or compound statement.
  **L942 CN**: 结束当前词法作用域或复合语句块。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L944 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L945 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_max_epu16(__mmask16 __M, __m256i __A, __m256i __B) {`.
  **L945 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_max_epu16(__mmask16 __M, __m256i __A, __m256i __B) {`。
- **L946 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L946 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_max_epu16(__A, __B),`.
  **L947 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_max_epu16(__A, __B),`。
- **L948 EN**: Executes a call or declaration centered on `statement`.
  **L948 CN**: 执行以 `statement` 为核心的调用或声明。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Blank line separating nearby declarations or logic blocks.
  **L950 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L951 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L951 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_max_epu16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {`.
  **L952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_max_epu16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {`。
- **L953 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L953 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_max_epu16(__A, __B),`.
  **L954 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_max_epu16(__A, __B),`。
- **L955 EN**: Executes a call or declaration centered on `statement`.
  **L955 CN**: 执行以 `statement` 为核心的调用或声明。
- **L956 EN**: Closes the current lexical scope or compound statement.
  **L956 CN**: 结束当前词法作用域或复合语句块。
- **L957 EN**: Blank line separating nearby declarations or logic blocks.
  **L957 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L958 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L958 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L959 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_min_epi8(__mmask16 __M, __m128i __A, __m128i __B) {`.
  **L959 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_min_epi8(__mmask16 __M, __m128i __A, __m128i __B) {`。
- **L960 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L960 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。

### Lines 961-984

````c
                                             (__v16qi)_mm_min_epi8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_min_epi8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_min_epi8(__A, __B),
                                             (__v16qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_min_epi8(__mmask32 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                             (__v32qi)_mm256_min_epi8(__A, __B),
                                             (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_min_epi8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                             (__v32qi)_mm256_min_epi8(__A, __B),
                                             (__v32qi)__W);
}
````
- **L961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_min_epi8(__A, __B),`.
  **L961 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_min_epi8(__A, __B),`。
- **L962 EN**: Executes a call or declaration centered on `statement`.
  **L962 CN**: 执行以 `statement` 为核心的调用或声明。
- **L963 EN**: Closes the current lexical scope or compound statement.
  **L963 CN**: 结束当前词法作用域或复合语句块。
- **L964 EN**: Blank line separating nearby declarations or logic blocks.
  **L964 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L965 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L965 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L966 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_min_epi8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {`.
  **L966 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_min_epi8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {`。
- **L967 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L967 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_min_epi8(__A, __B),`.
  **L968 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_min_epi8(__A, __B),`。
- **L969 EN**: Executes a call or declaration centered on `statement`.
  **L969 CN**: 执行以 `statement` 为核心的调用或声明。
- **L970 EN**: Closes the current lexical scope or compound statement.
  **L970 CN**: 结束当前词法作用域或复合语句块。
- **L971 EN**: Blank line separating nearby declarations or logic blocks.
  **L971 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L972 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L972 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L973 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_min_epi8(__mmask32 __M, __m256i __A, __m256i __B) {`.
  **L973 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_min_epi8(__mmask32 __M, __m256i __A, __m256i __B) {`。
- **L974 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L974 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_min_epi8(__A, __B),`.
  **L975 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_min_epi8(__A, __B),`。
- **L976 EN**: Executes a call or declaration centered on `statement`.
  **L976 CN**: 执行以 `statement` 为核心的调用或声明。
- **L977 EN**: Closes the current lexical scope or compound statement.
  **L977 CN**: 结束当前词法作用域或复合语句块。
- **L978 EN**: Blank line separating nearby declarations or logic blocks.
  **L978 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L979 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L979 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L980 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_min_epi8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {`.
  **L980 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_min_epi8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {`。
- **L981 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L981 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_min_epi8(__A, __B),`.
  **L982 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_min_epi8(__A, __B),`。
- **L983 EN**: Executes a call or declaration centered on `statement`.
  **L983 CN**: 执行以 `statement` 为核心的调用或声明。
- **L984 EN**: Closes the current lexical scope or compound statement.
  **L984 CN**: 结束当前词法作用域或复合语句块。

### Lines 985-1008

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_min_epi16(__mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_min_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_min_epi16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_min_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_min_epi16(__mmask16 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                            (__v16hi)_mm256_min_epi16(__A, __B),
                                            (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_min_epi16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {
````
- **L985 EN**: Blank line separating nearby declarations or logic blocks.
  **L985 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L986 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L986 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L987 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_min_epi16(__mmask8 __M, __m128i __A, __m128i __B) {`.
  **L987 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_min_epi16(__mmask8 __M, __m128i __A, __m128i __B) {`。
- **L988 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L988 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_min_epi16(__A, __B),`.
  **L989 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_min_epi16(__A, __B),`。
- **L990 EN**: Executes a call or declaration centered on `statement`.
  **L990 CN**: 执行以 `statement` 为核心的调用或声明。
- **L991 EN**: Closes the current lexical scope or compound statement.
  **L991 CN**: 结束当前词法作用域或复合语句块。
- **L992 EN**: Blank line separating nearby declarations or logic blocks.
  **L992 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L993 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L993 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L994 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_min_epi16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {`.
  **L994 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_min_epi16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {`。
- **L995 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L995 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_min_epi16(__A, __B),`.
  **L996 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_min_epi16(__A, __B),`。
- **L997 EN**: Executes a call or declaration centered on `statement`.
  **L997 CN**: 执行以 `statement` 为核心的调用或声明。
- **L998 EN**: Closes the current lexical scope or compound statement.
  **L998 CN**: 结束当前词法作用域或复合语句块。
- **L999 EN**: Blank line separating nearby declarations or logic blocks.
  **L999 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1000 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1000 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1001 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_min_epi16(__mmask16 __M, __m256i __A, __m256i __B) {`.
  **L1001 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_min_epi16(__mmask16 __M, __m256i __A, __m256i __B) {`。
- **L1002 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L1002 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L1003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_min_epi16(__A, __B),`.
  **L1003 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_min_epi16(__A, __B),`。
- **L1004 EN**: Executes a call or declaration centered on `statement`.
  **L1004 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1005 EN**: Closes the current lexical scope or compound statement.
  **L1005 CN**: 结束当前词法作用域或复合语句块。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1007 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1007 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1008 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_min_epi16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {`.
  **L1008 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_min_epi16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {`。

### Lines 1009-1032

````c
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                            (__v16hi)_mm256_min_epi16(__A, __B),
                                            (__v16hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_min_epu8(__mmask16 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_min_epu8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_min_epu8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm_min_epu8(__A, __B),
                                             (__v16qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_min_epu8(__mmask32 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                             (__v32qi)_mm256_min_epu8(__A, __B),
                                             (__v32qi)_mm256_setzero_si256());
````
- **L1009 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L1009 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_min_epi16(__A, __B),`.
  **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_min_epi16(__A, __B),`。
- **L1011 EN**: Executes a call or declaration centered on `statement`.
  **L1011 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1012 EN**: Closes the current lexical scope or compound statement.
  **L1012 CN**: 结束当前词法作用域或复合语句块。
- **L1013 EN**: Blank line separating nearby declarations or logic blocks.
  **L1013 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1014 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1014 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1015 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_min_epu8(__mmask16 __M, __m128i __A, __m128i __B) {`.
  **L1015 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_min_epu8(__mmask16 __M, __m128i __A, __m128i __B) {`。
- **L1016 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L1016 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L1017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_min_epu8(__A, __B),`.
  **L1017 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_min_epu8(__A, __B),`。
- **L1018 EN**: Executes a call or declaration centered on `statement`.
  **L1018 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1019 EN**: Closes the current lexical scope or compound statement.
  **L1019 CN**: 结束当前词法作用域或复合语句块。
- **L1020 EN**: Blank line separating nearby declarations or logic blocks.
  **L1020 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1021 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1021 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1022 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_min_epu8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {`.
  **L1022 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_min_epu8(__m128i __W, __mmask16 __M, __m128i __A, __m128i __B) {`。
- **L1023 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L1023 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L1024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_min_epu8(__A, __B),`.
  **L1024 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_min_epu8(__A, __B),`。
- **L1025 EN**: Executes a call or declaration centered on `statement`.
  **L1025 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1026 EN**: Closes the current lexical scope or compound statement.
  **L1026 CN**: 结束当前词法作用域或复合语句块。
- **L1027 EN**: Blank line separating nearby declarations or logic blocks.
  **L1027 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1028 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1028 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1029 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_min_epu8(__mmask32 __M, __m256i __A, __m256i __B) {`.
  **L1029 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_min_epu8(__mmask32 __M, __m256i __A, __m256i __B) {`。
- **L1030 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L1030 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L1031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_min_epu8(__A, __B),`.
  **L1031 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_min_epu8(__A, __B),`。
- **L1032 EN**: Executes a call or declaration centered on `statement`.
  **L1032 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1033-1056

````c
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_min_epu8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__M,
                                             (__v32qi)_mm256_min_epu8(__A, __B),
                                             (__v32qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_min_epu16(__mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_min_epu16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_min_epu16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                             (__v8hi)_mm_min_epu16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L1033 EN**: Closes the current lexical scope or compound statement.
  **L1033 CN**: 结束当前词法作用域或复合语句块。
- **L1034 EN**: Blank line separating nearby declarations or logic blocks.
  **L1034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1035 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1035 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1036 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_min_epu8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {`.
  **L1036 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_min_epu8(__m256i __W, __mmask32 __M, __m256i __A, __m256i __B) {`。
- **L1037 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,`.
  **L1037 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__M,` 从当前函数返回。
- **L1038 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_min_epu8(__A, __B),`.
  **L1038 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_min_epu8(__A, __B),`。
- **L1039 EN**: Executes a call or declaration centered on `statement`.
  **L1039 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1040 EN**: Closes the current lexical scope or compound statement.
  **L1040 CN**: 结束当前词法作用域或复合语句块。
- **L1041 EN**: Blank line separating nearby declarations or logic blocks.
  **L1041 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1042 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1042 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1043 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_min_epu16(__mmask8 __M, __m128i __A, __m128i __B) {`.
  **L1043 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_min_epu16(__mmask8 __M, __m128i __A, __m128i __B) {`。
- **L1044 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L1044 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L1045 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_min_epu16(__A, __B),`.
  **L1045 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_min_epu16(__A, __B),`。
- **L1046 EN**: Executes a call or declaration centered on `statement`.
  **L1046 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1047 EN**: Closes the current lexical scope or compound statement.
  **L1047 CN**: 结束当前词法作用域或复合语句块。
- **L1048 EN**: Blank line separating nearby declarations or logic blocks.
  **L1048 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1049 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1049 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1050 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_min_epu16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {`.
  **L1050 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_min_epu16(__m128i __W, __mmask8 __M, __m128i __A, __m128i __B) {`。
- **L1051 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L1051 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L1052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_min_epu16(__A, __B),`.
  **L1052 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_min_epu16(__A, __B),`。
- **L1053 EN**: Executes a call or declaration centered on `statement`.
  **L1053 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1054 EN**: Closes the current lexical scope or compound statement.
  **L1054 CN**: 结束当前词法作用域或复合语句块。
- **L1055 EN**: Blank line separating nearby declarations or logic blocks.
  **L1055 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1056 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1056 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 1057-1080

````c
_mm256_maskz_min_epu16(__mmask16 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                            (__v16hi)_mm256_min_epu16(__A, __B),
                                            (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_min_epu16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                            (__v16hi)_mm256_min_epu16(__A, __B),
                                            (__v16hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_shuffle_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                            (__v16qi)_mm_shuffle_epi8(__A, __B),
                                            (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_shuffle_epi8(__mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                            (__v16qi)_mm_shuffle_epi8(__A, __B),
````
- **L1057 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_min_epu16(__mmask16 __M, __m256i __A, __m256i __B) {`.
  **L1057 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_min_epu16(__mmask16 __M, __m256i __A, __m256i __B) {`。
- **L1058 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L1058 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_min_epu16(__A, __B),`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_min_epu16(__A, __B),`。
- **L1060 EN**: Executes a call or declaration centered on `statement`.
  **L1060 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1061 EN**: Closes the current lexical scope or compound statement.
  **L1061 CN**: 结束当前词法作用域或复合语句块。
- **L1062 EN**: Blank line separating nearby declarations or logic blocks.
  **L1062 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1063 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1063 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1064 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_min_epu16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {`.
  **L1064 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_min_epu16(__m256i __W, __mmask16 __M, __m256i __A, __m256i __B) {`。
- **L1065 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L1065 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_min_epu16(__A, __B),`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_min_epu16(__A, __B),`。
- **L1067 EN**: Executes a call or declaration centered on `statement`.
  **L1067 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1068 EN**: Closes the current lexical scope or compound statement.
  **L1068 CN**: 结束当前词法作用域或复合语句块。
- **L1069 EN**: Blank line separating nearby declarations or logic blocks.
  **L1069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1070 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1070 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1071 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_shuffle_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`.
  **L1071 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_shuffle_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`。
- **L1072 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1072 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_shuffle_epi8(__A, __B),`.
  **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_shuffle_epi8(__A, __B),`。
- **L1074 EN**: Executes a call or declaration centered on `statement`.
  **L1074 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Blank line separating nearby declarations or logic blocks.
  **L1076 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1077 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1077 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1078 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_shuffle_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`.
  **L1078 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_shuffle_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`。
- **L1079 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1079 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_shuffle_epi8(__A, __B),`.
  **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_shuffle_epi8(__A, __B),`。

### Lines 1081-1104

````c
                                            (__v16qi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_shuffle_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                         (__v32qi)_mm256_shuffle_epi8(__A, __B),
                                         (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_shuffle_epi8(__mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                         (__v32qi)_mm256_shuffle_epi8(__A, __B),
                                         (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_subs_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_subs_epi8(__A, __B),
                                             (__v16qi)__W);
}
````
- **L1081 EN**: Executes a call or declaration centered on `statement`.
  **L1081 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1082 EN**: Closes the current lexical scope or compound statement.
  **L1082 CN**: 结束当前词法作用域或复合语句块。
- **L1083 EN**: Blank line separating nearby declarations or logic blocks.
  **L1083 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1084 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1084 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1085 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_shuffle_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`.
  **L1085 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_shuffle_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`。
- **L1086 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1086 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_shuffle_epi8(__A, __B),`.
  **L1087 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_shuffle_epi8(__A, __B),`。
- **L1088 EN**: Executes a call or declaration centered on `statement`.
  **L1088 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1089 EN**: Closes the current lexical scope or compound statement.
  **L1089 CN**: 结束当前词法作用域或复合语句块。
- **L1090 EN**: Blank line separating nearby declarations or logic blocks.
  **L1090 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1091 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1091 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1092 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_shuffle_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`.
  **L1092 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_shuffle_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`。
- **L1093 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1093 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_shuffle_epi8(__A, __B),`.
  **L1094 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_shuffle_epi8(__A, __B),`。
- **L1095 EN**: Executes a call or declaration centered on `statement`.
  **L1095 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1096 EN**: Closes the current lexical scope or compound statement.
  **L1096 CN**: 结束当前词法作用域或复合语句块。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1098 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1098 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1099 EN**: Continues logic associated with callable symbol `_mm_mask_subs_epi8`.
  **L1099 CN**: 继续与可调用符号 `_mm_mask_subs_epi8` 相关的逻辑。
- **L1100 EN**: Opens a new lexical scope or compound statement.
  **L1100 CN**: 打开一个新的词法作用域或复合语句块。
- **L1101 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1101 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_subs_epi8(__A, __B),`.
  **L1102 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_subs_epi8(__A, __B),`。
- **L1103 EN**: Executes a call or declaration centered on `statement`.
  **L1103 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1104 EN**: Closes the current lexical scope or compound statement.
  **L1104 CN**: 结束当前词法作用域或复合语句块。

### Lines 1105-1128

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_subs_epi8(__mmask16 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_subs_epi8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_subs_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                            (__v32qi)_mm256_subs_epi8(__A, __B),
                                            (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_subs_epi8(__mmask32 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                            (__v32qi)_mm256_subs_epi8(__A, __B),
                                            (__v32qi)_mm256_setzero_si256());
}
````
- **L1105 EN**: Blank line separating nearby declarations or logic blocks.
  **L1105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1106 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1106 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1107 EN**: Continues logic associated with callable symbol `_mm_maskz_subs_epi8`.
  **L1107 CN**: 继续与可调用符号 `_mm_maskz_subs_epi8` 相关的逻辑。
- **L1108 EN**: Opens a new lexical scope or compound statement.
  **L1108 CN**: 打开一个新的词法作用域或复合语句块。
- **L1109 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1109 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_subs_epi8(__A, __B),`.
  **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_subs_epi8(__A, __B),`。
- **L1111 EN**: Executes a call or declaration centered on `statement`.
  **L1111 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1112 EN**: Closes the current lexical scope or compound statement.
  **L1112 CN**: 结束当前词法作用域或复合语句块。
- **L1113 EN**: Blank line separating nearby declarations or logic blocks.
  **L1113 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1114 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1114 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1115 EN**: Continues logic associated with callable symbol `_mm256_mask_subs_epi8`.
  **L1115 CN**: 继续与可调用符号 `_mm256_mask_subs_epi8` 相关的逻辑。
- **L1116 EN**: Opens a new lexical scope or compound statement.
  **L1116 CN**: 打开一个新的词法作用域或复合语句块。
- **L1117 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1117 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_subs_epi8(__A, __B),`.
  **L1118 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_subs_epi8(__A, __B),`。
- **L1119 EN**: Executes a call or declaration centered on `statement`.
  **L1119 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1120 EN**: Closes the current lexical scope or compound statement.
  **L1120 CN**: 结束当前词法作用域或复合语句块。
- **L1121 EN**: Blank line separating nearby declarations or logic blocks.
  **L1121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1122 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1122 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1123 EN**: Continues logic associated with callable symbol `_mm256_maskz_subs_epi8`.
  **L1123 CN**: 继续与可调用符号 `_mm256_maskz_subs_epi8` 相关的逻辑。
- **L1124 EN**: Opens a new lexical scope or compound statement.
  **L1124 CN**: 打开一个新的词法作用域或复合语句块。
- **L1125 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1125 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_subs_epi8(__A, __B),`.
  **L1126 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_subs_epi8(__A, __B),`。
- **L1127 EN**: Executes a call or declaration centered on `statement`.
  **L1127 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1128 EN**: Closes the current lexical scope or compound statement.
  **L1128 CN**: 结束当前词法作用域或复合语句块。

### Lines 1129-1152

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_subs_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_subs_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_subs_epi16(__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_subs_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_subs_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_subs_epi16(__A, __B),
                                           (__v16hi)__W);
}
````
- **L1129 EN**: Blank line separating nearby declarations or logic blocks.
  **L1129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1130 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1130 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1131 EN**: Continues logic associated with callable symbol `_mm_mask_subs_epi16`.
  **L1131 CN**: 继续与可调用符号 `_mm_mask_subs_epi16` 相关的逻辑。
- **L1132 EN**: Opens a new lexical scope or compound statement.
  **L1132 CN**: 打开一个新的词法作用域或复合语句块。
- **L1133 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1133 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_subs_epi16(__A, __B),`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_subs_epi16(__A, __B),`。
- **L1135 EN**: Executes a call or declaration centered on `statement`.
  **L1135 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1136 EN**: Closes the current lexical scope or compound statement.
  **L1136 CN**: 结束当前词法作用域或复合语句块。
- **L1137 EN**: Blank line separating nearby declarations or logic blocks.
  **L1137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1138 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1138 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1139 EN**: Continues logic associated with callable symbol `_mm_maskz_subs_epi16`.
  **L1139 CN**: 继续与可调用符号 `_mm_maskz_subs_epi16` 相关的逻辑。
- **L1140 EN**: Opens a new lexical scope or compound statement.
  **L1140 CN**: 打开一个新的词法作用域或复合语句块。
- **L1141 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1141 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_subs_epi16(__A, __B),`.
  **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_subs_epi16(__A, __B),`。
- **L1143 EN**: Executes a call or declaration centered on `statement`.
  **L1143 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1144 EN**: Closes the current lexical scope or compound statement.
  **L1144 CN**: 结束当前词法作用域或复合语句块。
- **L1145 EN**: Blank line separating nearby declarations or logic blocks.
  **L1145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1146 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1146 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1147 EN**: Continues logic associated with callable symbol `_mm256_mask_subs_epi16`.
  **L1147 CN**: 继续与可调用符号 `_mm256_mask_subs_epi16` 相关的逻辑。
- **L1148 EN**: Opens a new lexical scope or compound statement.
  **L1148 CN**: 打开一个新的词法作用域或复合语句块。
- **L1149 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1149 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_subs_epi16(__A, __B),`.
  **L1150 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_subs_epi16(__A, __B),`。
- **L1151 EN**: Executes a call or declaration centered on `statement`.
  **L1151 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1152 EN**: Closes the current lexical scope or compound statement.
  **L1152 CN**: 结束当前词法作用域或复合语句块。

### Lines 1153-1176

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_subs_epi16(__mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_subs_epi16(__A, __B),
                                           (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_subs_epu8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_subs_epu8(__A, __B),
                                             (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_subs_epu8(__mmask16 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                             (__v16qi)_mm_subs_epu8(__A, __B),
                                             (__v16qi)_mm_setzero_si128());
}
````
- **L1153 EN**: Blank line separating nearby declarations or logic blocks.
  **L1153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1154 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1154 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1155 EN**: Continues logic associated with callable symbol `_mm256_maskz_subs_epi16`.
  **L1155 CN**: 继续与可调用符号 `_mm256_maskz_subs_epi16` 相关的逻辑。
- **L1156 EN**: Opens a new lexical scope or compound statement.
  **L1156 CN**: 打开一个新的词法作用域或复合语句块。
- **L1157 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1157 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_subs_epi16(__A, __B),`.
  **L1158 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_subs_epi16(__A, __B),`。
- **L1159 EN**: Executes a call or declaration centered on `statement`.
  **L1159 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1160 EN**: Closes the current lexical scope or compound statement.
  **L1160 CN**: 结束当前词法作用域或复合语句块。
- **L1161 EN**: Blank line separating nearby declarations or logic blocks.
  **L1161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1162 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1162 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1163 EN**: Continues logic associated with callable symbol `_mm_mask_subs_epu8`.
  **L1163 CN**: 继续与可调用符号 `_mm_mask_subs_epu8` 相关的逻辑。
- **L1164 EN**: Opens a new lexical scope or compound statement.
  **L1164 CN**: 打开一个新的词法作用域或复合语句块。
- **L1165 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1165 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_subs_epu8(__A, __B),`.
  **L1166 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_subs_epu8(__A, __B),`。
- **L1167 EN**: Executes a call or declaration centered on `statement`.
  **L1167 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1168 EN**: Closes the current lexical scope or compound statement.
  **L1168 CN**: 结束当前词法作用域或复合语句块。
- **L1169 EN**: Blank line separating nearby declarations or logic blocks.
  **L1169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1170 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1170 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1171 EN**: Continues logic associated with callable symbol `_mm_maskz_subs_epu8`.
  **L1171 CN**: 继续与可调用符号 `_mm_maskz_subs_epu8` 相关的逻辑。
- **L1172 EN**: Opens a new lexical scope or compound statement.
  **L1172 CN**: 打开一个新的词法作用域或复合语句块。
- **L1173 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1173 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_subs_epu8(__A, __B),`.
  **L1174 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_subs_epu8(__A, __B),`。
- **L1175 EN**: Executes a call or declaration centered on `statement`.
  **L1175 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1176 EN**: Closes the current lexical scope or compound statement.
  **L1176 CN**: 结束当前词法作用域或复合语句块。

### Lines 1177-1200

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_subs_epu8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                            (__v32qi)_mm256_subs_epu8(__A, __B),
                                            (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_subs_epu8(__mmask32 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                            (__v32qi)_mm256_subs_epu8(__A, __B),
                                            (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_subs_epu16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_subs_epu16(__A, __B),
                                             (__v8hi)__W);
}
````
- **L1177 EN**: Blank line separating nearby declarations or logic blocks.
  **L1177 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1178 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1178 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1179 EN**: Continues logic associated with callable symbol `_mm256_mask_subs_epu8`.
  **L1179 CN**: 继续与可调用符号 `_mm256_mask_subs_epu8` 相关的逻辑。
- **L1180 EN**: Opens a new lexical scope or compound statement.
  **L1180 CN**: 打开一个新的词法作用域或复合语句块。
- **L1181 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1181 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_subs_epu8(__A, __B),`.
  **L1182 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_subs_epu8(__A, __B),`。
- **L1183 EN**: Executes a call or declaration centered on `statement`.
  **L1183 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1184 EN**: Closes the current lexical scope or compound statement.
  **L1184 CN**: 结束当前词法作用域或复合语句块。
- **L1185 EN**: Blank line separating nearby declarations or logic blocks.
  **L1185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1186 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1186 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1187 EN**: Continues logic associated with callable symbol `_mm256_maskz_subs_epu8`.
  **L1187 CN**: 继续与可调用符号 `_mm256_maskz_subs_epu8` 相关的逻辑。
- **L1188 EN**: Opens a new lexical scope or compound statement.
  **L1188 CN**: 打开一个新的词法作用域或复合语句块。
- **L1189 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1189 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_subs_epu8(__A, __B),`.
  **L1190 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_subs_epu8(__A, __B),`。
- **L1191 EN**: Executes a call or declaration centered on `statement`.
  **L1191 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1192 EN**: Closes the current lexical scope or compound statement.
  **L1192 CN**: 结束当前词法作用域或复合语句块。
- **L1193 EN**: Blank line separating nearby declarations or logic blocks.
  **L1193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1194 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1194 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1195 EN**: Continues logic associated with callable symbol `_mm_mask_subs_epu16`.
  **L1195 CN**: 继续与可调用符号 `_mm_mask_subs_epu16` 相关的逻辑。
- **L1196 EN**: Opens a new lexical scope or compound statement.
  **L1196 CN**: 打开一个新的词法作用域或复合语句块。
- **L1197 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1197 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_subs_epu16(__A, __B),`.
  **L1198 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_subs_epu16(__A, __B),`。
- **L1199 EN**: Executes a call or declaration centered on `statement`.
  **L1199 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1200 EN**: Closes the current lexical scope or compound statement.
  **L1200 CN**: 结束当前词法作用域或复合语句块。

### Lines 1201-1224

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_subs_epu16(__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_subs_epu16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_subs_epu16(__m256i __W, __mmask16 __U, __m256i __A,
      __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_subs_epu16(__A, __B),
                                           (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_subs_epu16(__mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_subs_epu16(__A, __B),
                                           (__v16hi)_mm256_setzero_si256());
}
````
- **L1201 EN**: Blank line separating nearby declarations or logic blocks.
  **L1201 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1202 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1202 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1203 EN**: Continues logic associated with callable symbol `_mm_maskz_subs_epu16`.
  **L1203 CN**: 继续与可调用符号 `_mm_maskz_subs_epu16` 相关的逻辑。
- **L1204 EN**: Opens a new lexical scope or compound statement.
  **L1204 CN**: 打开一个新的词法作用域或复合语句块。
- **L1205 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1205 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_subs_epu16(__A, __B),`.
  **L1206 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_subs_epu16(__A, __B),`。
- **L1207 EN**: Executes a call or declaration centered on `statement`.
  **L1207 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1208 EN**: Closes the current lexical scope or compound statement.
  **L1208 CN**: 结束当前词法作用域或复合语句块。
- **L1209 EN**: Blank line separating nearby declarations or logic blocks.
  **L1209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1210 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1210 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_mask_subs_epu16(__m256i __W, __mmask16 __U, __m256i __A,`.
  **L1211 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_mask_subs_epu16(__m256i __W, __mmask16 __U, __m256i __A,`。
- **L1212 EN**: Continues the surrounding expression or declaration: `__m256i __B) {`.
  **L1212 CN**: 继续构造周围的表达式或声明：`__m256i __B) {`。
- **L1213 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1213 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_subs_epu16(__A, __B),`.
  **L1214 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_subs_epu16(__A, __B),`。
- **L1215 EN**: Executes a call or declaration centered on `statement`.
  **L1215 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1216 EN**: Closes the current lexical scope or compound statement.
  **L1216 CN**: 结束当前词法作用域或复合语句块。
- **L1217 EN**: Blank line separating nearby declarations or logic blocks.
  **L1217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1218 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1218 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1219 EN**: Continues logic associated with callable symbol `_mm256_maskz_subs_epu16`.
  **L1219 CN**: 继续与可调用符号 `_mm256_maskz_subs_epu16` 相关的逻辑。
- **L1220 EN**: Opens a new lexical scope or compound statement.
  **L1220 CN**: 打开一个新的词法作用域或复合语句块。
- **L1221 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1221 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_subs_epu16(__A, __B),`.
  **L1222 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_subs_epu16(__A, __B),`。
- **L1223 EN**: Executes a call or declaration centered on `statement`.
  **L1223 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1224 EN**: Closes the current lexical scope or compound statement.
  **L1224 CN**: 结束当前词法作用域或复合语句块。

### Lines 1225-1248

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_permutex2var_epi16(__m128i __A, __m128i __I, __m128i __B) {
  return (__m128i)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,
                                                 (__v8hi) __B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_permutex2var_epi16(__m128i __A, __mmask8 __U, __m128i __I,
                            __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128(__U,
                                  (__v8hi)_mm_permutex2var_epi16(__A, __I, __B),
                                  (__v8hi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask2_permutex2var_epi16(__m128i __A, __m128i __I, __mmask8 __U,
                             __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128(__U,
                                  (__v8hi)_mm_permutex2var_epi16(__A, __I, __B),
                                  (__v8hi)__I);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
````
- **L1225 EN**: Blank line separating nearby declarations or logic blocks.
  **L1225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1226 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1226 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1227 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_permutex2var_epi16(__m128i __A, __m128i __I, __m128i __B) {`.
  **L1227 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_permutex2var_epi16(__m128i __A, __m128i __I, __m128i __B) {`。
- **L1228 EN**: Returns from the current function with `(__m128i)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,`.
  **L1228 CN**: 以 `(__m128i)__builtin_ia32_vpermi2varhi128((__v8hi)__A, (__v8hi)__I,` 从当前函数返回。
- **L1229 EN**: Executes a call or declaration centered on `statement`.
  **L1229 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1230 EN**: Closes the current lexical scope or compound statement.
  **L1230 CN**: 结束当前词法作用域或复合语句块。
- **L1231 EN**: Blank line separating nearby declarations or logic blocks.
  **L1231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1232 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1232 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_mask_permutex2var_epi16(__m128i __A, __mmask8 __U, __m128i __I,`.
  **L1233 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_mask_permutex2var_epi16(__m128i __A, __mmask8 __U, __m128i __I,`。
- **L1234 EN**: Continues the surrounding expression or declaration: `__m128i __B) {`.
  **L1234 CN**: 继续构造周围的表达式或声明：`__m128i __B) {`。
- **L1235 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__U,`.
  **L1235 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__U,` 从当前函数返回。
- **L1236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_permutex2var_epi16(__A, __I, __B),`.
  **L1236 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_permutex2var_epi16(__A, __I, __B),`。
- **L1237 EN**: Executes a call or declaration centered on `statement`.
  **L1237 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1238 EN**: Closes the current lexical scope or compound statement.
  **L1238 CN**: 结束当前词法作用域或复合语句块。
- **L1239 EN**: Blank line separating nearby declarations or logic blocks.
  **L1239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1240 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1240 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_mask2_permutex2var_epi16(__m128i __A, __m128i __I, __mmask8 __U,`.
  **L1241 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_mask2_permutex2var_epi16(__m128i __A, __m128i __I, __mmask8 __U,`。
- **L1242 EN**: Continues the surrounding expression or declaration: `__m128i __B) {`.
  **L1242 CN**: 继续构造周围的表达式或声明：`__m128i __B) {`。
- **L1243 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__U,`.
  **L1243 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__U,` 从当前函数返回。
- **L1244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_permutex2var_epi16(__A, __I, __B),`.
  **L1244 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_permutex2var_epi16(__A, __I, __B),`。
- **L1245 EN**: Executes a call or declaration centered on `statement`.
  **L1245 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1246 EN**: Closes the current lexical scope or compound statement.
  **L1246 CN**: 结束当前词法作用域或复合语句块。
- **L1247 EN**: Blank line separating nearby declarations or logic blocks.
  **L1247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1248 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1248 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。

### Lines 1249-1272

````c
_mm_maskz_permutex2var_epi16(__mmask8 __U, __m128i __A, __m128i __I,
                             __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128(__U,
                                  (__v8hi)_mm_permutex2var_epi16(__A, __I, __B),
                                  (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_permutex2var_epi16(__m256i __A, __m256i __I, __m256i __B) {
  return (__m256i)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,
                                                 (__v16hi)__B);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_permutex2var_epi16(__m256i __A, __mmask16 __U, __m256i __I,
                               __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256(__U,
                              (__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),
                              (__v16hi)__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask2_permutex2var_epi16(__m256i __A, __m256i __I, __mmask16 __U,
                                __m256i __B) {
````
- **L1249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_maskz_permutex2var_epi16(__mmask8 __U, __m128i __A, __m128i __I,`.
  **L1249 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_maskz_permutex2var_epi16(__mmask8 __U, __m128i __A, __m128i __I,`。
- **L1250 EN**: Continues the surrounding expression or declaration: `__m128i __B) {`.
  **L1250 CN**: 继续构造周围的表达式或声明：`__m128i __B) {`。
- **L1251 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__U,`.
  **L1251 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__U,` 从当前函数返回。
- **L1252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_permutex2var_epi16(__A, __I, __B),`.
  **L1252 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_permutex2var_epi16(__A, __I, __B),`。
- **L1253 EN**: Executes a call or declaration centered on `statement`.
  **L1253 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1254 EN**: Closes the current lexical scope or compound statement.
  **L1254 CN**: 结束当前词法作用域或复合语句块。
- **L1255 EN**: Blank line separating nearby declarations or logic blocks.
  **L1255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1256 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1256 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1257 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_permutex2var_epi16(__m256i __A, __m256i __I, __m256i __B) {`.
  **L1257 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_permutex2var_epi16(__m256i __A, __m256i __I, __m256i __B) {`。
- **L1258 EN**: Returns from the current function with `(__m256i)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,`.
  **L1258 CN**: 以 `(__m256i)__builtin_ia32_vpermi2varhi256((__v16hi)__A, (__v16hi)__I,` 从当前函数返回。
- **L1259 EN**: Executes a call or declaration centered on `statement`.
  **L1259 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1260 EN**: Closes the current lexical scope or compound statement.
  **L1260 CN**: 结束当前词法作用域或复合语句块。
- **L1261 EN**: Blank line separating nearby declarations or logic blocks.
  **L1261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1262 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1262 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_mask_permutex2var_epi16(__m256i __A, __mmask16 __U, __m256i __I,`.
  **L1263 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_mask_permutex2var_epi16(__m256i __A, __mmask16 __U, __m256i __I,`。
- **L1264 EN**: Continues the surrounding expression or declaration: `__m256i __B) {`.
  **L1264 CN**: 继续构造周围的表达式或声明：`__m256i __B) {`。
- **L1265 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__U,`.
  **L1265 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__U,` 从当前函数返回。
- **L1266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),`.
  **L1266 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),`。
- **L1267 EN**: Executes a call or declaration centered on `statement`.
  **L1267 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1268 EN**: Closes the current lexical scope or compound statement.
  **L1268 CN**: 结束当前词法作用域或复合语句块。
- **L1269 EN**: Blank line separating nearby declarations or logic blocks.
  **L1269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1270 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1270 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_mask2_permutex2var_epi16(__m256i __A, __m256i __I, __mmask16 __U,`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_mask2_permutex2var_epi16(__m256i __A, __m256i __I, __mmask16 __U,`。
- **L1272 EN**: Continues the surrounding expression or declaration: `__m256i __B) {`.
  **L1272 CN**: 继续构造周围的表达式或声明：`__m256i __B) {`。

### Lines 1273-1296

````c
  return (__m256i)__builtin_ia32_selectw_256(__U,
                              (__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),
                              (__v16hi)__I);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_permutex2var_epi16(__mmask16 __U, __m256i __A, __m256i __I,
                                __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256(__U,
                              (__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),
                              (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_maddubs_epi16(__m128i __W, __mmask8 __U, __m128i __X, __m128i __Y) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                            (__v8hi)_mm_maddubs_epi16(__X, __Y),
                                            (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_maddubs_epi16(__mmask8 __U, __m128i __X, __m128i __Y) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                            (__v8hi)_mm_maddubs_epi16(__X, __Y),
````
- **L1273 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__U,`.
  **L1273 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__U,` 从当前函数返回。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),`。
- **L1275 EN**: Executes a call or declaration centered on `statement`.
  **L1275 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1276 EN**: Closes the current lexical scope or compound statement.
  **L1276 CN**: 结束当前词法作用域或复合语句块。
- **L1277 EN**: Blank line separating nearby declarations or logic blocks.
  **L1277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1278 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1278 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_maskz_permutex2var_epi16(__mmask16 __U, __m256i __A, __m256i __I,`.
  **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_maskz_permutex2var_epi16(__mmask16 __U, __m256i __A, __m256i __I,`。
- **L1280 EN**: Continues the surrounding expression or declaration: `__m256i __B) {`.
  **L1280 CN**: 继续构造周围的表达式或声明：`__m256i __B) {`。
- **L1281 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__U,`.
  **L1281 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__U,` 从当前函数返回。
- **L1282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),`.
  **L1282 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_permutex2var_epi16(__A, __I, __B),`。
- **L1283 EN**: Executes a call or declaration centered on `statement`.
  **L1283 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1284 EN**: Closes the current lexical scope or compound statement.
  **L1284 CN**: 结束当前词法作用域或复合语句块。
- **L1285 EN**: Blank line separating nearby declarations or logic blocks.
  **L1285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1286 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1286 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1287 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_maddubs_epi16(__m128i __W, __mmask8 __U, __m128i __X, __m128i __Y) {`.
  **L1287 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_maddubs_epi16(__m128i __W, __mmask8 __U, __m128i __X, __m128i __Y) {`。
- **L1288 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1288 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_maddubs_epi16(__X, __Y),`.
  **L1289 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_maddubs_epi16(__X, __Y),`。
- **L1290 EN**: Executes a call or declaration centered on `statement`.
  **L1290 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1291 EN**: Closes the current lexical scope or compound statement.
  **L1291 CN**: 结束当前词法作用域或复合语句块。
- **L1292 EN**: Blank line separating nearby declarations or logic blocks.
  **L1292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1293 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1293 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1294 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_maddubs_epi16(__mmask8 __U, __m128i __X, __m128i __Y) {`.
  **L1294 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_maddubs_epi16(__mmask8 __U, __m128i __X, __m128i __Y) {`。
- **L1295 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1295 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_maddubs_epi16(__X, __Y),`.
  **L1296 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_maddubs_epi16(__X, __Y),`。

### Lines 1297-1320

````c
                                            (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_maddubs_epi16(__m256i __W, __mmask16 __U, __m256i __X,
                          __m256i __Y) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                        (__v16hi)_mm256_maddubs_epi16(__X, __Y),
                                        (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_maddubs_epi16(__mmask16 __U, __m256i __X, __m256i __Y) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                        (__v16hi)_mm256_maddubs_epi16(__X, __Y),
                                        (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_madd_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectd_128((__mmask8)__U,
                                             (__v4si)_mm_madd_epi16(__A, __B),
                                             (__v4si)__W);
}
````
- **L1297 EN**: Executes a call or declaration centered on `statement`.
  **L1297 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1298 EN**: Closes the current lexical scope or compound statement.
  **L1298 CN**: 结束当前词法作用域或复合语句块。
- **L1299 EN**: Blank line separating nearby declarations or logic blocks.
  **L1299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1300 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1300 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_mask_maddubs_epi16(__m256i __W, __mmask16 __U, __m256i __X,`.
  **L1301 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_mask_maddubs_epi16(__m256i __W, __mmask16 __U, __m256i __X,`。
- **L1302 EN**: Continues the surrounding expression or declaration: `__m256i __Y) {`.
  **L1302 CN**: 继续构造周围的表达式或声明：`__m256i __Y) {`。
- **L1303 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1303 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_maddubs_epi16(__X, __Y),`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_maddubs_epi16(__X, __Y),`。
- **L1305 EN**: Executes a call or declaration centered on `statement`.
  **L1305 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1306 EN**: Closes the current lexical scope or compound statement.
  **L1306 CN**: 结束当前词法作用域或复合语句块。
- **L1307 EN**: Blank line separating nearby declarations or logic blocks.
  **L1307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1308 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1308 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1309 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_maddubs_epi16(__mmask16 __U, __m256i __X, __m256i __Y) {`.
  **L1309 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_maddubs_epi16(__mmask16 __U, __m256i __X, __m256i __Y) {`。
- **L1310 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1310 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_maddubs_epi16(__X, __Y),`.
  **L1311 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_maddubs_epi16(__X, __Y),`。
- **L1312 EN**: Executes a call or declaration centered on `statement`.
  **L1312 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1313 EN**: Closes the current lexical scope or compound statement.
  **L1313 CN**: 结束当前词法作用域或复合语句块。
- **L1314 EN**: Blank line separating nearby declarations or logic blocks.
  **L1314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1315 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1315 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1316 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_madd_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1316 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_madd_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1317 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectd_128((__mmask8)__U,`.
  **L1317 CN**: 以 `(__m128i)__builtin_ia32_selectd_128((__mmask8)__U,` 从当前函数返回。
- **L1318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4si)_mm_madd_epi16(__A, __B),`.
  **L1318 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4si)_mm_madd_epi16(__A, __B),`。
- **L1319 EN**: Executes a call or declaration centered on `statement`.
  **L1319 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1320 EN**: Closes the current lexical scope or compound statement.
  **L1320 CN**: 结束当前词法作用域或复合语句块。

### Lines 1321-1344

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_madd_epi16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectd_128((__mmask8)__U,
                                             (__v4si)_mm_madd_epi16(__A, __B),
                                             (__v4si)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_madd_epi16(__m256i __W, __mmask8 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectd_256((__mmask8)__U,
                                            (__v8si)_mm256_madd_epi16(__A, __B),
                                            (__v8si)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_madd_epi16(__mmask8 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectd_256((__mmask8)__U,
                                            (__v8si)_mm256_madd_epi16(__A, __B),
                                            (__v8si)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtsepi16_epi8 (__m128i __A) {
````
- **L1321 EN**: Blank line separating nearby declarations or logic blocks.
  **L1321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1322 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1322 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1323 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_madd_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1323 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_madd_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1324 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectd_128((__mmask8)__U,`.
  **L1324 CN**: 以 `(__m128i)__builtin_ia32_selectd_128((__mmask8)__U,` 从当前函数返回。
- **L1325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4si)_mm_madd_epi16(__A, __B),`.
  **L1325 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4si)_mm_madd_epi16(__A, __B),`。
- **L1326 EN**: Executes a call or declaration centered on `statement`.
  **L1326 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1327 EN**: Closes the current lexical scope or compound statement.
  **L1327 CN**: 结束当前词法作用域或复合语句块。
- **L1328 EN**: Blank line separating nearby declarations or logic blocks.
  **L1328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1329 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1329 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1330 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_madd_epi16(__m256i __W, __mmask8 __U, __m256i __A, __m256i __B) {`.
  **L1330 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_madd_epi16(__m256i __W, __mmask8 __U, __m256i __A, __m256i __B) {`。
- **L1331 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectd_256((__mmask8)__U,`.
  **L1331 CN**: 以 `(__m256i)__builtin_ia32_selectd_256((__mmask8)__U,` 从当前函数返回。
- **L1332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8si)_mm256_madd_epi16(__A, __B),`.
  **L1332 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8si)_mm256_madd_epi16(__A, __B),`。
- **L1333 EN**: Executes a call or declaration centered on `statement`.
  **L1333 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1334 EN**: Closes the current lexical scope or compound statement.
  **L1334 CN**: 结束当前词法作用域或复合语句块。
- **L1335 EN**: Blank line separating nearby declarations or logic blocks.
  **L1335 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1336 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1336 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1337 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_madd_epi16(__mmask8 __U, __m256i __A, __m256i __B) {`.
  **L1337 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_madd_epi16(__mmask8 __U, __m256i __A, __m256i __B) {`。
- **L1338 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectd_256((__mmask8)__U,`.
  **L1338 CN**: 以 `(__m256i)__builtin_ia32_selectd_256((__mmask8)__U,` 从当前函数返回。
- **L1339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8si)_mm256_madd_epi16(__A, __B),`.
  **L1339 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8si)_mm256_madd_epi16(__A, __B),`。
- **L1340 EN**: Executes a call or declaration centered on `statement`.
  **L1340 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1341 EN**: Closes the current lexical scope or compound statement.
  **L1341 CN**: 结束当前词法作用域或复合语句块。
- **L1342 EN**: Blank line separating nearby declarations or logic blocks.
  **L1342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1343 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1343 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1344 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtsepi16_epi8 (__m128i __A) {`.
  **L1344 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtsepi16_epi8 (__m128i __A) {`。

### Lines 1345-1368

````c
  return (__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,
               (__v16qi) _mm_setzero_si128(),
               (__mmask8) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtsepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {
  return (__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,
               (__v16qi) __O,
                __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtsepi16_epi8 (__mmask8 __M, __m128i __A) {
  return (__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,
               (__v16qi) _mm_setzero_si128(),
               __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvtsepi16_epi8 (__m256i __A) {
  return (__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,
               (__v16qi) _mm_setzero_si128(),
               (__mmask16) -1);
````
- **L1345 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,`.
  **L1345 CN**: 以 `(__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,` 从当前函数返回。
- **L1346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1346 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1347 EN**: Executes a call or declaration centered on `statement`.
  **L1347 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1348 EN**: Closes the current lexical scope or compound statement.
  **L1348 CN**: 结束当前词法作用域或复合语句块。
- **L1349 EN**: Blank line separating nearby declarations or logic blocks.
  **L1349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1350 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1350 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1351 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtsepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {`.
  **L1351 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtsepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {`。
- **L1352 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,`.
  **L1352 CN**: 以 `(__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,` 从当前函数返回。
- **L1353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __O,`.
  **L1353 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __O,`。
- **L1354 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1354 CN**: 添加一条独立语句或声明：`__M);`。
- **L1355 EN**: Closes the current lexical scope or compound statement.
  **L1355 CN**: 结束当前词法作用域或复合语句块。
- **L1356 EN**: Blank line separating nearby declarations or logic blocks.
  **L1356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1357 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1357 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1358 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtsepi16_epi8 (__mmask8 __M, __m128i __A) {`.
  **L1358 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtsepi16_epi8 (__mmask8 __M, __m128i __A) {`。
- **L1359 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,`.
  **L1359 CN**: 以 `(__m128i) __builtin_ia32_pmovswb128_mask ((__v8hi) __A,` 从当前函数返回。
- **L1360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1360 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1361 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1361 CN**: 添加一条独立语句或声明：`__M);`。
- **L1362 EN**: Closes the current lexical scope or compound statement.
  **L1362 CN**: 结束当前词法作用域或复合语句块。
- **L1363 EN**: Blank line separating nearby declarations or logic blocks.
  **L1363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1364 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L1364 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L1365 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtsepi16_epi8 (__m256i __A) {`.
  **L1365 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtsepi16_epi8 (__m256i __A) {`。
- **L1366 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,`.
  **L1366 CN**: 以 `(__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,` 从当前函数返回。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1368 EN**: Executes a call or declaration centered on `statement`.
  **L1368 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1369-1392

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtsepi16_epi8 (__m128i __O, __mmask16 __M, __m256i __A) {
  return (__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,
               (__v16qi) __O,
                __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtsepi16_epi8 (__mmask16 __M, __m256i __A) {
  return (__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,
               (__v16qi) _mm_setzero_si128(),
               __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_cvtusepi16_epi8 (__m128i __A) {
  return (__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,
                (__v16qi) _mm_setzero_si128(),
                (__mmask8) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
````
- **L1369 EN**: Closes the current lexical scope or compound statement.
  **L1369 CN**: 结束当前词法作用域或复合语句块。
- **L1370 EN**: Blank line separating nearby declarations or logic blocks.
  **L1370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1371 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L1371 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L1372 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtsepi16_epi8 (__m128i __O, __mmask16 __M, __m256i __A) {`.
  **L1372 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtsepi16_epi8 (__m128i __O, __mmask16 __M, __m256i __A) {`。
- **L1373 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,`.
  **L1373 CN**: 以 `(__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,` 从当前函数返回。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __O,`.
  **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __O,`。
- **L1375 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1375 CN**: 添加一条独立语句或声明：`__M);`。
- **L1376 EN**: Closes the current lexical scope or compound statement.
  **L1376 CN**: 结束当前词法作用域或复合语句块。
- **L1377 EN**: Blank line separating nearby declarations or logic blocks.
  **L1377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1378 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L1378 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L1379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtsepi16_epi8 (__mmask16 __M, __m256i __A) {`.
  **L1379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtsepi16_epi8 (__mmask16 __M, __m256i __A) {`。
- **L1380 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,`.
  **L1380 CN**: 以 `(__m128i) __builtin_ia32_pmovswb256_mask ((__v16hi) __A,` 从当前函数返回。
- **L1381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1381 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1382 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1382 CN**: 添加一条独立语句或声明：`__M);`。
- **L1383 EN**: Closes the current lexical scope or compound statement.
  **L1383 CN**: 结束当前词法作用域或复合语句块。
- **L1384 EN**: Blank line separating nearby declarations or logic blocks.
  **L1384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1385 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1385 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1386 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtusepi16_epi8 (__m128i __A) {`.
  **L1386 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtusepi16_epi8 (__m128i __A) {`。
- **L1387 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,`.
  **L1387 CN**: 以 `(__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,` 从当前函数返回。
- **L1388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1388 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1389 EN**: Executes a call or declaration centered on `statement`.
  **L1389 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1390 EN**: Closes the current lexical scope or compound statement.
  **L1390 CN**: 结束当前词法作用域或复合语句块。
- **L1391 EN**: Blank line separating nearby declarations or logic blocks.
  **L1391 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1392 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1392 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。

### Lines 1393-1416

````c
_mm_mask_cvtusepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {
  return (__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,
                (__v16qi) __O,
                __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtusepi16_epi8 (__mmask8 __M, __m128i __A) {
  return (__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,
                (__v16qi) _mm_setzero_si128(),
                __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_cvtusepi16_epi8 (__m256i __A) {
  return (__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,
                (__v16qi) _mm_setzero_si128(),
                (__mmask16) -1);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_mask_cvtusepi16_epi8 (__m128i __O, __mmask16 __M, __m256i __A) {
  return (__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,
                (__v16qi) __O,
````
- **L1393 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtusepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {`.
  **L1393 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtusepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {`。
- **L1394 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,`.
  **L1394 CN**: 以 `(__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,` 从当前函数返回。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __O,`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __O,`。
- **L1396 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1396 CN**: 添加一条独立语句或声明：`__M);`。
- **L1397 EN**: Closes the current lexical scope or compound statement.
  **L1397 CN**: 结束当前词法作用域或复合语句块。
- **L1398 EN**: Blank line separating nearby declarations or logic blocks.
  **L1398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1399 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1399 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1400 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtusepi16_epi8 (__mmask8 __M, __m128i __A) {`.
  **L1400 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtusepi16_epi8 (__mmask8 __M, __m128i __A) {`。
- **L1401 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,`.
  **L1401 CN**: 以 `(__m128i) __builtin_ia32_pmovuswb128_mask ((__v8hi) __A,` 从当前函数返回。
- **L1402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1402 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1403 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1403 CN**: 添加一条独立语句或声明：`__M);`。
- **L1404 EN**: Closes the current lexical scope or compound statement.
  **L1404 CN**: 结束当前词法作用域或复合语句块。
- **L1405 EN**: Blank line separating nearby declarations or logic blocks.
  **L1405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1406 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L1406 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L1407 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtusepi16_epi8 (__m256i __A) {`.
  **L1407 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtusepi16_epi8 (__m256i __A) {`。
- **L1408 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,`.
  **L1408 CN**: 以 `(__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,` 从当前函数返回。
- **L1409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1409 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1410 EN**: Executes a call or declaration centered on `statement`.
  **L1410 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1411 EN**: Closes the current lexical scope or compound statement.
  **L1411 CN**: 结束当前词法作用域或复合语句块。
- **L1412 EN**: Blank line separating nearby declarations or logic blocks.
  **L1412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1413 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L1413 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L1414 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtusepi16_epi8 (__m128i __O, __mmask16 __M, __m256i __A) {`.
  **L1414 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtusepi16_epi8 (__m128i __O, __mmask16 __M, __m256i __A) {`。
- **L1415 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,`.
  **L1415 CN**: 以 `(__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,` 从当前函数返回。
- **L1416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __O,`.
  **L1416 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __O,`。

### Lines 1417-1440

````c
                __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256
_mm256_maskz_cvtusepi16_epi8 (__mmask16 __M, __m256i __A) {
  return (__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,
                (__v16qi) _mm_setzero_si128(),
                __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_cvtepi16_epi8(__m128i __A) {
  return (__m128i)__builtin_shufflevector(
      __builtin_convertvector((__v8hi)__A, __v8qi),
      (__v8qi){0, 0, 0, 0, 0, 0, 0, 0}, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,
      12, 13, 14, 15);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_cvtepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {
  return (__m128i) __builtin_ia32_pmovwb128_mask ((__v8hi) __A,
               (__v16qi) __O,
               __M);
}
````
- **L1417 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1417 CN**: 添加一条独立语句或声明：`__M);`。
- **L1418 EN**: Closes the current lexical scope or compound statement.
  **L1418 CN**: 结束当前词法作用域或复合语句块。
- **L1419 EN**: Blank line separating nearby declarations or logic blocks.
  **L1419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1420 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256`.
  **L1420 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256`。
- **L1421 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtusepi16_epi8 (__mmask16 __M, __m256i __A) {`.
  **L1421 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtusepi16_epi8 (__mmask16 __M, __m256i __A) {`。
- **L1422 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,`.
  **L1422 CN**: 以 `(__m128i) __builtin_ia32_pmovuswb256_mask ((__v16hi) __A,` 从当前函数返回。
- **L1423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1423 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1424 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1424 CN**: 添加一条独立语句或声明：`__M);`。
- **L1425 EN**: Closes the current lexical scope or compound statement.
  **L1425 CN**: 结束当前词法作用域或复合语句块。
- **L1426 EN**: Blank line separating nearby declarations or logic blocks.
  **L1426 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1427 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1427 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1428 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_cvtepi16_epi8(__m128i __A) {`.
  **L1428 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_cvtepi16_epi8(__m128i __A) {`。
- **L1429 EN**: Returns from the current function with `(__m128i)__builtin_shufflevector(`.
  **L1429 CN**: 以 `(__m128i)__builtin_shufflevector(` 从当前函数返回。
- **L1430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_convertvector((__v8hi)__A, __v8qi),`.
  **L1430 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_convertvector((__v8hi)__A, __v8qi),`。
- **L1431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8qi){0, 0, 0, 0, 0, 0, 0, 0}, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,`.
  **L1431 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8qi){0, 0, 0, 0, 0, 0, 0, 0}, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,`。
- **L1432 EN**: Adds a standalone statement or declaration: `12, 13, 14, 15);`.
  **L1432 CN**: 添加一条独立语句或声明：`12, 13, 14, 15);`。
- **L1433 EN**: Closes the current lexical scope or compound statement.
  **L1433 CN**: 结束当前词法作用域或复合语句块。
- **L1434 EN**: Blank line separating nearby declarations or logic blocks.
  **L1434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1435 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1435 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1436 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_cvtepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {`.
  **L1436 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_cvtepi16_epi8 (__m128i __O, __mmask8 __M, __m128i __A) {`。
- **L1437 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovwb128_mask ((__v8hi) __A,`.
  **L1437 CN**: 以 `(__m128i) __builtin_ia32_pmovwb128_mask ((__v8hi) __A,` 从当前函数返回。
- **L1438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __O,`.
  **L1438 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __O,`。
- **L1439 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1439 CN**: 添加一条独立语句或声明：`__M);`。
- **L1440 EN**: Closes the current lexical scope or compound statement.
  **L1440 CN**: 结束当前词法作用域或复合语句块。

### Lines 1441-1464

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_cvtepi16_epi8 (__mmask8 __M, __m128i __A) {
  return (__m128i) __builtin_ia32_pmovwb128_mask ((__v8hi) __A,
               (__v16qi) _mm_setzero_si128(),
               __M);
}

static __inline__ void __DEFAULT_FN_ATTRS128
_mm_mask_cvtepi16_storeu_epi8 (void * __P, __mmask8 __M, __m128i __A)
{
  __builtin_ia32_pmovwb128mem_mask ((__v16qi *) __P, (__v8hi) __A, __M);
}


static __inline__ void __DEFAULT_FN_ATTRS128
_mm_mask_cvtsepi16_storeu_epi8 (void * __P, __mmask8 __M, __m128i __A)
{
  __builtin_ia32_pmovswb128mem_mask ((__v16qi *) __P, (__v8hi) __A, __M);
}

static __inline__ void __DEFAULT_FN_ATTRS128
_mm_mask_cvtusepi16_storeu_epi8 (void * __P, __mmask8 __M, __m128i __A)
{
````
- **L1441 EN**: Blank line separating nearby declarations or logic blocks.
  **L1441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1442 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1442 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1443 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_cvtepi16_epi8 (__mmask8 __M, __m128i __A) {`.
  **L1443 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_cvtepi16_epi8 (__mmask8 __M, __m128i __A) {`。
- **L1444 EN**: Returns from the current function with `(__m128i) __builtin_ia32_pmovwb128_mask ((__v8hi) __A,`.
  **L1444 CN**: 以 `(__m128i) __builtin_ia32_pmovwb128_mask ((__v8hi) __A,` 从当前函数返回。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L1446 EN**: Adds a standalone statement or declaration: `__M);`.
  **L1446 CN**: 添加一条独立语句或声明：`__M);`。
- **L1447 EN**: Closes the current lexical scope or compound statement.
  **L1447 CN**: 结束当前词法作用域或复合语句块。
- **L1448 EN**: Blank line separating nearby declarations or logic blocks.
  **L1448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1449 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS128`.
  **L1449 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS128`。
- **L1450 EN**: Continues logic associated with callable symbol `_mm_mask_cvtepi16_storeu_epi8`.
  **L1450 CN**: 继续与可调用符号 `_mm_mask_cvtepi16_storeu_epi8` 相关的逻辑。
- **L1451 EN**: Opens a new lexical scope or compound statement.
  **L1451 CN**: 打开一个新的词法作用域或复合语句块。
- **L1452 EN**: Executes a call or declaration centered on `__builtin_ia32_pmovwb128mem_mask`.
  **L1452 CN**: 执行以 `__builtin_ia32_pmovwb128mem_mask` 为核心的调用或声明。
- **L1453 EN**: Closes the current lexical scope or compound statement.
  **L1453 CN**: 结束当前词法作用域或复合语句块。
- **L1454 EN**: Blank line separating nearby declarations or logic blocks.
  **L1454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1455 EN**: Blank line separating nearby declarations or logic blocks.
  **L1455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1456 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS128`.
  **L1456 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS128`。
- **L1457 EN**: Continues logic associated with callable symbol `_mm_mask_cvtsepi16_storeu_epi8`.
  **L1457 CN**: 继续与可调用符号 `_mm_mask_cvtsepi16_storeu_epi8` 相关的逻辑。
- **L1458 EN**: Opens a new lexical scope or compound statement.
  **L1458 CN**: 打开一个新的词法作用域或复合语句块。
- **L1459 EN**: Executes a call or declaration centered on `__builtin_ia32_pmovswb128mem_mask`.
  **L1459 CN**: 执行以 `__builtin_ia32_pmovswb128mem_mask` 为核心的调用或声明。
- **L1460 EN**: Closes the current lexical scope or compound statement.
  **L1460 CN**: 结束当前词法作用域或复合语句块。
- **L1461 EN**: Blank line separating nearby declarations or logic blocks.
  **L1461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1462 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS128`.
  **L1462 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS128`。
- **L1463 EN**: Continues logic associated with callable symbol `_mm_mask_cvtusepi16_storeu_epi8`.
  **L1463 CN**: 继续与可调用符号 `_mm_mask_cvtusepi16_storeu_epi8` 相关的逻辑。
- **L1464 EN**: Opens a new lexical scope or compound statement.
  **L1464 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 1465-1488

````c
  __builtin_ia32_pmovuswb128mem_mask ((__v16qi *) __P, (__v8hi) __A, __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_cvtepi16_epi8(__m256i __A) {
  return (__m128i)__builtin_convertvector((__v16hi) __A, __v16qi);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepi16_epi8(__m128i __O, __mmask16 __M, __m256i __A) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm256_cvtepi16_epi8(__A),
                                             (__v16qi)__O);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepi16_epi8(__mmask16 __M, __m256i __A) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__M,
                                             (__v16qi)_mm256_cvtepi16_epi8(__A),
                                             (__v16qi)_mm_setzero_si128());
}

static __inline__ void __DEFAULT_FN_ATTRS256
_mm256_mask_cvtepi16_storeu_epi8 (void * __P, __mmask16 __M, __m256i __A)
````
- **L1465 EN**: Executes a call or declaration centered on `__builtin_ia32_pmovuswb128mem_mask`.
  **L1465 CN**: 执行以 `__builtin_ia32_pmovuswb128mem_mask` 为核心的调用或声明。
- **L1466 EN**: Closes the current lexical scope or compound statement.
  **L1466 CN**: 结束当前词法作用域或复合语句块。
- **L1467 EN**: Blank line separating nearby declarations or logic blocks.
  **L1467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1468 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1468 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1469 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_cvtepi16_epi8(__m256i __A) {`.
  **L1469 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_cvtepi16_epi8(__m256i __A) {`。
- **L1470 EN**: Returns from the current function with `(__m128i)__builtin_convertvector((__v16hi) __A, __v16qi)`.
  **L1470 CN**: 以 `(__m128i)__builtin_convertvector((__v16hi) __A, __v16qi)` 从当前函数返回。
- **L1471 EN**: Closes the current lexical scope or compound statement.
  **L1471 CN**: 结束当前词法作用域或复合语句块。
- **L1472 EN**: Blank line separating nearby declarations or logic blocks.
  **L1472 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1473 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1473 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1474 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_cvtepi16_epi8(__m128i __O, __mmask16 __M, __m256i __A) {`.
  **L1474 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_cvtepi16_epi8(__m128i __O, __mmask16 __M, __m256i __A) {`。
- **L1475 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L1475 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L1476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm256_cvtepi16_epi8(__A),`.
  **L1476 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm256_cvtepi16_epi8(__A),`。
- **L1477 EN**: Executes a call or declaration centered on `statement`.
  **L1477 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1478 EN**: Closes the current lexical scope or compound statement.
  **L1478 CN**: 结束当前词法作用域或复合语句块。
- **L1479 EN**: Blank line separating nearby declarations or logic blocks.
  **L1479 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1480 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1480 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1481 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_cvtepi16_epi8(__mmask16 __M, __m256i __A) {`.
  **L1481 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_cvtepi16_epi8(__mmask16 __M, __m256i __A) {`。
- **L1482 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,`.
  **L1482 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__M,` 从当前函数返回。
- **L1483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm256_cvtepi16_epi8(__A),`.
  **L1483 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm256_cvtepi16_epi8(__A),`。
- **L1484 EN**: Executes a call or declaration centered on `statement`.
  **L1484 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1485 EN**: Closes the current lexical scope or compound statement.
  **L1485 CN**: 结束当前词法作用域或复合语句块。
- **L1486 EN**: Blank line separating nearby declarations or logic blocks.
  **L1486 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1487 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS256`.
  **L1487 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS256`。
- **L1488 EN**: Continues logic associated with callable symbol `_mm256_mask_cvtepi16_storeu_epi8`.
  **L1488 CN**: 继续与可调用符号 `_mm256_mask_cvtepi16_storeu_epi8` 相关的逻辑。

### Lines 1489-1512

````c
{
  __builtin_ia32_pmovwb256mem_mask ((__v16qi *) __P, (__v16hi) __A, __M);
}

static __inline__ void __DEFAULT_FN_ATTRS256
_mm256_mask_cvtsepi16_storeu_epi8 (void * __P, __mmask16 __M, __m256i __A)
{
  __builtin_ia32_pmovswb256mem_mask ((__v16qi *) __P, (__v16hi) __A, __M);
}

static __inline__ void __DEFAULT_FN_ATTRS256
_mm256_mask_cvtusepi16_storeu_epi8 (void * __P, __mmask16 __M, __m256i __A)
{
  __builtin_ia32_pmovuswb256mem_mask ((__v16qi*) __P, (__v16hi) __A, __M);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_mulhrs_epi16(__m128i __W, __mmask8 __U, __m128i __X, __m128i __Y) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_mulhrs_epi16(__X, __Y),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
````
- **L1489 EN**: Opens a new lexical scope or compound statement.
  **L1489 CN**: 打开一个新的词法作用域或复合语句块。
- **L1490 EN**: Executes a call or declaration centered on `__builtin_ia32_pmovwb256mem_mask`.
  **L1490 CN**: 执行以 `__builtin_ia32_pmovwb256mem_mask` 为核心的调用或声明。
- **L1491 EN**: Closes the current lexical scope or compound statement.
  **L1491 CN**: 结束当前词法作用域或复合语句块。
- **L1492 EN**: Blank line separating nearby declarations or logic blocks.
  **L1492 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1493 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS256`.
  **L1493 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS256`。
- **L1494 EN**: Continues logic associated with callable symbol `_mm256_mask_cvtsepi16_storeu_epi8`.
  **L1494 CN**: 继续与可调用符号 `_mm256_mask_cvtsepi16_storeu_epi8` 相关的逻辑。
- **L1495 EN**: Opens a new lexical scope or compound statement.
  **L1495 CN**: 打开一个新的词法作用域或复合语句块。
- **L1496 EN**: Executes a call or declaration centered on `__builtin_ia32_pmovswb256mem_mask`.
  **L1496 CN**: 执行以 `__builtin_ia32_pmovswb256mem_mask` 为核心的调用或声明。
- **L1497 EN**: Closes the current lexical scope or compound statement.
  **L1497 CN**: 结束当前词法作用域或复合语句块。
- **L1498 EN**: Blank line separating nearby declarations or logic blocks.
  **L1498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1499 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS256`.
  **L1499 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS256`。
- **L1500 EN**: Continues logic associated with callable symbol `_mm256_mask_cvtusepi16_storeu_epi8`.
  **L1500 CN**: 继续与可调用符号 `_mm256_mask_cvtusepi16_storeu_epi8` 相关的逻辑。
- **L1501 EN**: Opens a new lexical scope or compound statement.
  **L1501 CN**: 打开一个新的词法作用域或复合语句块。
- **L1502 EN**: Executes a call or declaration centered on `__builtin_ia32_pmovuswb256mem_mask`.
  **L1502 CN**: 执行以 `__builtin_ia32_pmovuswb256mem_mask` 为核心的调用或声明。
- **L1503 EN**: Closes the current lexical scope or compound statement.
  **L1503 CN**: 结束当前词法作用域或复合语句块。
- **L1504 EN**: Blank line separating nearby declarations or logic blocks.
  **L1504 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1505 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1505 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1506 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mulhrs_epi16(__m128i __W, __mmask8 __U, __m128i __X, __m128i __Y) {`.
  **L1506 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mulhrs_epi16(__m128i __W, __mmask8 __U, __m128i __X, __m128i __Y) {`。
- **L1507 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1507 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_mulhrs_epi16(__X, __Y),`.
  **L1508 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_mulhrs_epi16(__X, __Y),`。
- **L1509 EN**: Executes a call or declaration centered on `statement`.
  **L1509 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1510 EN**: Closes the current lexical scope or compound statement.
  **L1510 CN**: 结束当前词法作用域或复合语句块。
- **L1511 EN**: Blank line separating nearby declarations or logic blocks.
  **L1511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1512 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1512 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。

### Lines 1513-1536

````c
_mm_maskz_mulhrs_epi16(__mmask8 __U, __m128i __X, __m128i __Y) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_mulhrs_epi16(__X, __Y),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_mulhrs_epi16(__m256i __W, __mmask16 __U, __m256i __X, __m256i __Y) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                         (__v16hi)_mm256_mulhrs_epi16(__X, __Y),
                                         (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_mulhrs_epi16(__mmask16 __U, __m256i __X, __m256i __Y) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                         (__v16hi)_mm256_mulhrs_epi16(__X, __Y),
                                         (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_mulhi_epu16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128(
      (__mmask8)__U, (__v8hi)_mm_mulhi_epu16(__A, __B), (__v8hi)__W);
````
- **L1513 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mulhrs_epi16(__mmask8 __U, __m128i __X, __m128i __Y) {`.
  **L1513 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mulhrs_epi16(__mmask8 __U, __m128i __X, __m128i __Y) {`。
- **L1514 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1514 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_mulhrs_epi16(__X, __Y),`.
  **L1515 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_mulhrs_epi16(__X, __Y),`。
- **L1516 EN**: Executes a call or declaration centered on `statement`.
  **L1516 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1517 EN**: Closes the current lexical scope or compound statement.
  **L1517 CN**: 结束当前词法作用域或复合语句块。
- **L1518 EN**: Blank line separating nearby declarations or logic blocks.
  **L1518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1519 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1519 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1520 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mulhrs_epi16(__m256i __W, __mmask16 __U, __m256i __X, __m256i __Y) {`.
  **L1520 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mulhrs_epi16(__m256i __W, __mmask16 __U, __m256i __X, __m256i __Y) {`。
- **L1521 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1521 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_mulhrs_epi16(__X, __Y),`.
  **L1522 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_mulhrs_epi16(__X, __Y),`。
- **L1523 EN**: Executes a call or declaration centered on `statement`.
  **L1523 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1524 EN**: Closes the current lexical scope or compound statement.
  **L1524 CN**: 结束当前词法作用域或复合语句块。
- **L1525 EN**: Blank line separating nearby declarations or logic blocks.
  **L1525 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1526 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1526 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1527 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mulhrs_epi16(__mmask16 __U, __m256i __X, __m256i __Y) {`.
  **L1527 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mulhrs_epi16(__mmask16 __U, __m256i __X, __m256i __Y) {`。
- **L1528 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1528 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_mulhrs_epi16(__X, __Y),`.
  **L1529 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_mulhrs_epi16(__X, __Y),`。
- **L1530 EN**: Executes a call or declaration centered on `statement`.
  **L1530 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1531 EN**: Closes the current lexical scope or compound statement.
  **L1531 CN**: 结束当前词法作用域或复合语句块。
- **L1532 EN**: Blank line separating nearby declarations or logic blocks.
  **L1532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1533 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1533 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1534 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mulhi_epu16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1534 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mulhi_epu16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1535 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(`.
  **L1535 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(` 从当前函数返回。
- **L1536 EN**: Executes a call or declaration centered on `statement`.
  **L1536 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1537-1560

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_mulhi_epu16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_mulhi_epu16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_mulhi_epu16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256(
      (__mmask16)__U, (__v16hi)_mm256_mulhi_epu16(__A, __B), (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_mulhi_epu16(__mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256(
      (__mmask16)__U, (__v16hi)_mm256_mulhi_epu16(__A, __B),
      (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_mulhi_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
````
- **L1537 EN**: Closes the current lexical scope or compound statement.
  **L1537 CN**: 结束当前词法作用域或复合语句块。
- **L1538 EN**: Blank line separating nearby declarations or logic blocks.
  **L1538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1539 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1539 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1540 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mulhi_epu16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1540 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mulhi_epu16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1541 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1541 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_mulhi_epu16(__A, __B),`.
  **L1542 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_mulhi_epu16(__A, __B),`。
- **L1543 EN**: Executes a call or declaration centered on `statement`.
  **L1543 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1544 EN**: Closes the current lexical scope or compound statement.
  **L1544 CN**: 结束当前词法作用域或复合语句块。
- **L1545 EN**: Blank line separating nearby declarations or logic blocks.
  **L1545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1546 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1546 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1547 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mulhi_epu16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`.
  **L1547 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mulhi_epu16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`。
- **L1548 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(`.
  **L1548 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(` 从当前函数返回。
- **L1549 EN**: Executes a call or declaration centered on `statement`.
  **L1549 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1550 EN**: Closes the current lexical scope or compound statement.
  **L1550 CN**: 结束当前词法作用域或复合语句块。
- **L1551 EN**: Blank line separating nearby declarations or logic blocks.
  **L1551 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1552 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1552 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1553 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mulhi_epu16(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L1553 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mulhi_epu16(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L1554 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(`.
  **L1554 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(` 从当前函数返回。
- **L1555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16hi)_mm256_mulhi_epu16(__A, __B),`.
  **L1555 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16hi)_mm256_mulhi_epu16(__A, __B),`。
- **L1556 EN**: Executes a call or declaration centered on `statement`.
  **L1556 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1557 EN**: Closes the current lexical scope or compound statement.
  **L1557 CN**: 结束当前词法作用域或复合语句块。
- **L1558 EN**: Blank line separating nearby declarations or logic blocks.
  **L1558 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1559 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1559 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1560 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mulhi_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1560 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mulhi_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。

### Lines 1561-1584

````c
  return (__m128i)__builtin_ia32_selectw_128(
      (__mmask8)__U, (__v8hi)_mm_mulhi_epi16(__A, __B), (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_mulhi_epi16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_mulhi_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_mulhi_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256(
      (__mmask16)__U, (__v16hi)_mm256_mulhi_epi16(__A, __B), (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_mulhi_epi16(__mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256(
      (__mmask16)__U, (__v16hi)_mm256_mulhi_epi16(__A, __B),
      (__v16hi)_mm256_setzero_si256());
}

````
- **L1561 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(`.
  **L1561 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(` 从当前函数返回。
- **L1562 EN**: Executes a call or declaration centered on `statement`.
  **L1562 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1563 EN**: Closes the current lexical scope or compound statement.
  **L1563 CN**: 结束当前词法作用域或复合语句块。
- **L1564 EN**: Blank line separating nearby declarations or logic blocks.
  **L1564 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1565 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1565 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1566 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mulhi_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1566 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mulhi_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1567 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1567 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_mulhi_epi16(__A, __B),`.
  **L1568 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_mulhi_epi16(__A, __B),`。
- **L1569 EN**: Executes a call or declaration centered on `statement`.
  **L1569 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1570 EN**: Closes the current lexical scope or compound statement.
  **L1570 CN**: 结束当前词法作用域或复合语句块。
- **L1571 EN**: Blank line separating nearby declarations or logic blocks.
  **L1571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1572 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1572 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1573 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mulhi_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`.
  **L1573 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mulhi_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`。
- **L1574 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(`.
  **L1574 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(` 从当前函数返回。
- **L1575 EN**: Executes a call or declaration centered on `statement`.
  **L1575 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1576 EN**: Closes the current lexical scope or compound statement.
  **L1576 CN**: 结束当前词法作用域或复合语句块。
- **L1577 EN**: Blank line separating nearby declarations or logic blocks.
  **L1577 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1578 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1578 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1579 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mulhi_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L1579 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mulhi_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L1580 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(`.
  **L1580 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(` 从当前函数返回。
- **L1581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__mmask16)__U, (__v16hi)_mm256_mulhi_epi16(__A, __B),`.
  **L1581 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__mmask16)__U, (__v16hi)_mm256_mulhi_epi16(__A, __B),`。
- **L1582 EN**: Executes a call or declaration centered on `statement`.
  **L1582 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1583 EN**: Closes the current lexical scope or compound statement.
  **L1583 CN**: 结束当前词法作用域或复合语句块。
- **L1584 EN**: Blank line separating nearby declarations or logic blocks.
  **L1584 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1585-1608

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_unpackhi_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128(
      (__mmask16)__U, (__v16qi)_mm_unpackhi_epi8(__A, __B), (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_unpackhi_epi8(__mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                           (__v16qi)_mm_unpackhi_epi8(__A, __B),
                                           (__v16qi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_unpackhi_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                        (__v32qi)_mm256_unpackhi_epi8(__A, __B),
                                        (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_unpackhi_epi8(__mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                        (__v32qi)_mm256_unpackhi_epi8(__A, __B),
````
- **L1585 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1585 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1586 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_unpackhi_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`.
  **L1586 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_unpackhi_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`。
- **L1587 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(`.
  **L1587 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(` 从当前函数返回。
- **L1588 EN**: Executes a call or declaration centered on `statement`.
  **L1588 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1589 EN**: Closes the current lexical scope or compound statement.
  **L1589 CN**: 结束当前词法作用域或复合语句块。
- **L1590 EN**: Blank line separating nearby declarations or logic blocks.
  **L1590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1591 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1591 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1592 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_unpackhi_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`.
  **L1592 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_unpackhi_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`。
- **L1593 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1593 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_unpackhi_epi8(__A, __B),`.
  **L1594 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_unpackhi_epi8(__A, __B),`。
- **L1595 EN**: Executes a call or declaration centered on `statement`.
  **L1595 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1596 EN**: Closes the current lexical scope or compound statement.
  **L1596 CN**: 结束当前词法作用域或复合语句块。
- **L1597 EN**: Blank line separating nearby declarations or logic blocks.
  **L1597 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1598 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1598 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1599 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_unpackhi_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`.
  **L1599 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_unpackhi_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`。
- **L1600 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1600 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_unpackhi_epi8(__A, __B),`.
  **L1601 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_unpackhi_epi8(__A, __B),`。
- **L1602 EN**: Executes a call or declaration centered on `statement`.
  **L1602 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1603 EN**: Closes the current lexical scope or compound statement.
  **L1603 CN**: 结束当前词法作用域或复合语句块。
- **L1604 EN**: Blank line separating nearby declarations or logic blocks.
  **L1604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1605 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1605 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1606 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_unpackhi_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`.
  **L1606 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_unpackhi_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`。
- **L1607 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1607 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_unpackhi_epi8(__A, __B),`.
  **L1608 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_unpackhi_epi8(__A, __B),`。

### Lines 1609-1632

````c
                                        (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_unpackhi_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                           (__v8hi)_mm_unpackhi_epi16(__A, __B),
                                           (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_unpackhi_epi16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                           (__v8hi)_mm_unpackhi_epi16(__A, __B),
                                           (__v8hi) _mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_unpackhi_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                       (__v16hi)_mm256_unpackhi_epi16(__A, __B),
                                       (__v16hi)__W);
}

````
- **L1609 EN**: Executes a call or declaration centered on `statement`.
  **L1609 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1610 EN**: Closes the current lexical scope or compound statement.
  **L1610 CN**: 结束当前词法作用域或复合语句块。
- **L1611 EN**: Blank line separating nearby declarations or logic blocks.
  **L1611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1612 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1612 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1613 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_unpackhi_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1613 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_unpackhi_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1614 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1614 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_unpackhi_epi16(__A, __B),`.
  **L1615 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_unpackhi_epi16(__A, __B),`。
- **L1616 EN**: Executes a call or declaration centered on `statement`.
  **L1616 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1617 EN**: Closes the current lexical scope or compound statement.
  **L1617 CN**: 结束当前词法作用域或复合语句块。
- **L1618 EN**: Blank line separating nearby declarations or logic blocks.
  **L1618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1619 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1619 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1620 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_unpackhi_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1620 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_unpackhi_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1621 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1621 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_unpackhi_epi16(__A, __B),`.
  **L1622 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_unpackhi_epi16(__A, __B),`。
- **L1623 EN**: Executes a call or declaration centered on `statement`.
  **L1623 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1624 EN**: Closes the current lexical scope or compound statement.
  **L1624 CN**: 结束当前词法作用域或复合语句块。
- **L1625 EN**: Blank line separating nearby declarations or logic blocks.
  **L1625 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1626 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1626 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1627 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_unpackhi_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`.
  **L1627 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_unpackhi_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`。
- **L1628 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1628 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_unpackhi_epi16(__A, __B),`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_unpackhi_epi16(__A, __B),`。
- **L1630 EN**: Executes a call or declaration centered on `statement`.
  **L1630 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1631 EN**: Closes the current lexical scope or compound statement.
  **L1631 CN**: 结束当前词法作用域或复合语句块。
- **L1632 EN**: Blank line separating nearby declarations or logic blocks.
  **L1632 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1633-1656

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_unpackhi_epi16(__mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                       (__v16hi)_mm256_unpackhi_epi16(__A, __B),
                                       (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_unpacklo_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                           (__v16qi)_mm_unpacklo_epi8(__A, __B),
                                           (__v16qi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_unpacklo_epi8(__mmask16 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectb_128((__mmask16)__U,
                                           (__v16qi)_mm_unpacklo_epi8(__A, __B),
                                           (__v16qi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_unpacklo_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
````
- **L1633 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1633 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1634 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_unpackhi_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L1634 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_unpackhi_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L1635 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1635 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_unpackhi_epi16(__A, __B),`.
  **L1636 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_unpackhi_epi16(__A, __B),`。
- **L1637 EN**: Executes a call or declaration centered on `statement`.
  **L1637 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1638 EN**: Closes the current lexical scope or compound statement.
  **L1638 CN**: 结束当前词法作用域或复合语句块。
- **L1639 EN**: Blank line separating nearby declarations or logic blocks.
  **L1639 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1640 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1640 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1641 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_unpacklo_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`.
  **L1641 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_unpacklo_epi8(__m128i __W, __mmask16 __U, __m128i __A, __m128i __B) {`。
- **L1642 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1642 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_unpacklo_epi8(__A, __B),`.
  **L1643 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_unpacklo_epi8(__A, __B),`。
- **L1644 EN**: Executes a call or declaration centered on `statement`.
  **L1644 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1645 EN**: Closes the current lexical scope or compound statement.
  **L1645 CN**: 结束当前词法作用域或复合语句块。
- **L1646 EN**: Blank line separating nearby declarations or logic blocks.
  **L1646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1647 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1647 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1648 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_unpacklo_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`.
  **L1648 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_unpacklo_epi8(__mmask16 __U, __m128i __A, __m128i __B) {`。
- **L1649 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,`.
  **L1649 CN**: 以 `(__m128i)__builtin_ia32_selectb_128((__mmask16)__U,` 从当前函数返回。
- **L1650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi)_mm_unpacklo_epi8(__A, __B),`.
  **L1650 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi)_mm_unpacklo_epi8(__A, __B),`。
- **L1651 EN**: Executes a call or declaration centered on `statement`.
  **L1651 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1652 EN**: Closes the current lexical scope or compound statement.
  **L1652 CN**: 结束当前词法作用域或复合语句块。
- **L1653 EN**: Blank line separating nearby declarations or logic blocks.
  **L1653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1654 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1654 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1655 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_unpacklo_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`.
  **L1655 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_unpacklo_epi8(__m256i __W, __mmask32 __U, __m256i __A, __m256i __B) {`。
- **L1656 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1656 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。

### Lines 1657-1680

````c
                                        (__v32qi)_mm256_unpacklo_epi8(__A, __B),
                                        (__v32qi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_unpacklo_epi8(__mmask32 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectb_256((__mmask32)__U,
                                        (__v32qi)_mm256_unpacklo_epi8(__A, __B),
                                        (__v32qi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_unpacklo_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                           (__v8hi)_mm_unpacklo_epi16(__A, __B),
                                           (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_unpacklo_epi16(__mmask8 __U, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                           (__v8hi)_mm_unpacklo_epi16(__A, __B),
                                           (__v8hi) _mm_setzero_si128());
}
````
- **L1657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_unpacklo_epi8(__A, __B),`.
  **L1657 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_unpacklo_epi8(__A, __B),`。
- **L1658 EN**: Executes a call or declaration centered on `statement`.
  **L1658 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1659 EN**: Closes the current lexical scope or compound statement.
  **L1659 CN**: 结束当前词法作用域或复合语句块。
- **L1660 EN**: Blank line separating nearby declarations or logic blocks.
  **L1660 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1661 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1661 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1662 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_unpacklo_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`.
  **L1662 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_unpacklo_epi8(__mmask32 __U, __m256i __A, __m256i __B) {`。
- **L1663 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,`.
  **L1663 CN**: 以 `(__m256i)__builtin_ia32_selectb_256((__mmask32)__U,` 从当前函数返回。
- **L1664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi)_mm256_unpacklo_epi8(__A, __B),`.
  **L1664 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi)_mm256_unpacklo_epi8(__A, __B),`。
- **L1665 EN**: Executes a call or declaration centered on `statement`.
  **L1665 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1666 EN**: Closes the current lexical scope or compound statement.
  **L1666 CN**: 结束当前词法作用域或复合语句块。
- **L1667 EN**: Blank line separating nearby declarations or logic blocks.
  **L1667 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1668 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1668 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1669 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_unpacklo_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1669 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_unpacklo_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1670 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1670 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_unpacklo_epi16(__A, __B),`.
  **L1671 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_unpacklo_epi16(__A, __B),`。
- **L1672 EN**: Executes a call or declaration centered on `statement`.
  **L1672 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1673 EN**: Closes the current lexical scope or compound statement.
  **L1673 CN**: 结束当前词法作用域或复合语句块。
- **L1674 EN**: Blank line separating nearby declarations or logic blocks.
  **L1674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1675 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1675 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1676 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_unpacklo_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L1676 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_unpacklo_epi16(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L1677 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1677 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_unpacklo_epi16(__A, __B),`.
  **L1678 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_unpacklo_epi16(__A, __B),`。
- **L1679 EN**: Executes a call or declaration centered on `statement`.
  **L1679 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1680 EN**: Closes the current lexical scope or compound statement.
  **L1680 CN**: 结束当前词法作用域或复合语句块。

### Lines 1681-1704

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_unpacklo_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                       (__v16hi)_mm256_unpacklo_epi16(__A, __B),
                                       (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_unpacklo_epi16(__mmask16 __U, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                       (__v16hi)_mm256_unpacklo_epi16(__A, __B),
                                       (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_cvtepi8_epi16(__m128i __W, __mmask8 __U, __m128i __A)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_cvtepi8_epi16(__A),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
````
- **L1681 EN**: Blank line separating nearby declarations or logic blocks.
  **L1681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1682 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1682 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1683 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_unpacklo_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`.
  **L1683 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_unpacklo_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B) {`。
- **L1684 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1684 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_unpacklo_epi16(__A, __B),`.
  **L1685 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_unpacklo_epi16(__A, __B),`。
- **L1686 EN**: Executes a call or declaration centered on `statement`.
  **L1686 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1687 EN**: Closes the current lexical scope or compound statement.
  **L1687 CN**: 结束当前词法作用域或复合语句块。
- **L1688 EN**: Blank line separating nearby declarations or logic blocks.
  **L1688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1689 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1689 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1690 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_unpacklo_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L1690 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_unpacklo_epi16(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L1691 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1691 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_unpacklo_epi16(__A, __B),`.
  **L1692 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_unpacklo_epi16(__A, __B),`。
- **L1693 EN**: Executes a call or declaration centered on `statement`.
  **L1693 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1694 EN**: Closes the current lexical scope or compound statement.
  **L1694 CN**: 结束当前词法作用域或复合语句块。
- **L1695 EN**: Blank line separating nearby declarations or logic blocks.
  **L1695 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1696 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1696 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1697 EN**: Continues logic associated with callable symbol `_mm_mask_cvtepi8_epi16`.
  **L1697 CN**: 继续与可调用符号 `_mm_mask_cvtepi8_epi16` 相关的逻辑。
- **L1698 EN**: Opens a new lexical scope or compound statement.
  **L1698 CN**: 打开一个新的词法作用域或复合语句块。
- **L1699 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1699 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_cvtepi8_epi16(__A),`.
  **L1700 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_cvtepi8_epi16(__A),`。
- **L1701 EN**: Executes a call or declaration centered on `statement`.
  **L1701 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1702 EN**: Closes the current lexical scope or compound statement.
  **L1702 CN**: 结束当前词法作用域或复合语句块。
- **L1703 EN**: Blank line separating nearby declarations or logic blocks.
  **L1703 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1704 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1704 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。

### Lines 1705-1728

````c
_mm_maskz_cvtepi8_epi16(__mmask8 __U, __m128i __A)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_cvtepi8_epi16(__A),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepi8_epi16(__m256i __W, __mmask16 __U, __m128i __A)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_cvtepi8_epi16(__A),
                                             (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepi8_epi16(__mmask16 __U, __m128i __A)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_cvtepi8_epi16(__A),
                                             (__v16hi)_mm256_setzero_si256());
}


````
- **L1705 EN**: Continues logic associated with callable symbol `_mm_maskz_cvtepi8_epi16`.
  **L1705 CN**: 继续与可调用符号 `_mm_maskz_cvtepi8_epi16` 相关的逻辑。
- **L1706 EN**: Opens a new lexical scope or compound statement.
  **L1706 CN**: 打开一个新的词法作用域或复合语句块。
- **L1707 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1707 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_cvtepi8_epi16(__A),`.
  **L1708 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_cvtepi8_epi16(__A),`。
- **L1709 EN**: Executes a call or declaration centered on `statement`.
  **L1709 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1710 EN**: Closes the current lexical scope or compound statement.
  **L1710 CN**: 结束当前词法作用域或复合语句块。
- **L1711 EN**: Blank line separating nearby declarations or logic blocks.
  **L1711 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1712 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1712 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1713 EN**: Continues logic associated with callable symbol `_mm256_mask_cvtepi8_epi16`.
  **L1713 CN**: 继续与可调用符号 `_mm256_mask_cvtepi8_epi16` 相关的逻辑。
- **L1714 EN**: Opens a new lexical scope or compound statement.
  **L1714 CN**: 打开一个新的词法作用域或复合语句块。
- **L1715 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1715 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_cvtepi8_epi16(__A),`.
  **L1716 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_cvtepi8_epi16(__A),`。
- **L1717 EN**: Executes a call or declaration centered on `statement`.
  **L1717 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1718 EN**: Closes the current lexical scope or compound statement.
  **L1718 CN**: 结束当前词法作用域或复合语句块。
- **L1719 EN**: Blank line separating nearby declarations or logic blocks.
  **L1719 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1720 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1720 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1721 EN**: Continues logic associated with callable symbol `_mm256_maskz_cvtepi8_epi16`.
  **L1721 CN**: 继续与可调用符号 `_mm256_maskz_cvtepi8_epi16` 相关的逻辑。
- **L1722 EN**: Opens a new lexical scope or compound statement.
  **L1722 CN**: 打开一个新的词法作用域或复合语句块。
- **L1723 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1723 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_cvtepi8_epi16(__A),`.
  **L1724 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_cvtepi8_epi16(__A),`。
- **L1725 EN**: Executes a call or declaration centered on `statement`.
  **L1725 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1726 EN**: Closes the current lexical scope or compound statement.
  **L1726 CN**: 结束当前词法作用域或复合语句块。
- **L1727 EN**: Blank line separating nearby declarations or logic blocks.
  **L1727 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1728 EN**: Blank line separating nearby declarations or logic blocks.
  **L1728 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1729-1752

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_cvtepu8_epi16(__m128i __W, __mmask8 __U, __m128i __A)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_cvtepu8_epi16(__A),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_cvtepu8_epi16(__mmask8 __U, __m128i __A)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_cvtepu8_epi16(__A),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_cvtepu8_epi16(__m256i __W, __mmask16 __U, __m128i __A)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_cvtepu8_epi16(__A),
                                             (__v16hi)__W);
}

````
- **L1729 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1729 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1730 EN**: Continues logic associated with callable symbol `_mm_mask_cvtepu8_epi16`.
  **L1730 CN**: 继续与可调用符号 `_mm_mask_cvtepu8_epi16` 相关的逻辑。
- **L1731 EN**: Opens a new lexical scope or compound statement.
  **L1731 CN**: 打开一个新的词法作用域或复合语句块。
- **L1732 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1732 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_cvtepu8_epi16(__A),`.
  **L1733 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_cvtepu8_epi16(__A),`。
- **L1734 EN**: Executes a call or declaration centered on `statement`.
  **L1734 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1735 EN**: Closes the current lexical scope or compound statement.
  **L1735 CN**: 结束当前词法作用域或复合语句块。
- **L1736 EN**: Blank line separating nearby declarations or logic blocks.
  **L1736 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1737 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1737 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1738 EN**: Continues logic associated with callable symbol `_mm_maskz_cvtepu8_epi16`.
  **L1738 CN**: 继续与可调用符号 `_mm_maskz_cvtepu8_epi16` 相关的逻辑。
- **L1739 EN**: Opens a new lexical scope or compound statement.
  **L1739 CN**: 打开一个新的词法作用域或复合语句块。
- **L1740 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1740 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_cvtepu8_epi16(__A),`.
  **L1741 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_cvtepu8_epi16(__A),`。
- **L1742 EN**: Executes a call or declaration centered on `statement`.
  **L1742 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1743 EN**: Closes the current lexical scope or compound statement.
  **L1743 CN**: 结束当前词法作用域或复合语句块。
- **L1744 EN**: Blank line separating nearby declarations or logic blocks.
  **L1744 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1745 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1745 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1746 EN**: Continues logic associated with callable symbol `_mm256_mask_cvtepu8_epi16`.
  **L1746 CN**: 继续与可调用符号 `_mm256_mask_cvtepu8_epi16` 相关的逻辑。
- **L1747 EN**: Opens a new lexical scope or compound statement.
  **L1747 CN**: 打开一个新的词法作用域或复合语句块。
- **L1748 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1748 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_cvtepu8_epi16(__A),`.
  **L1749 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_cvtepu8_epi16(__A),`。
- **L1750 EN**: Executes a call or declaration centered on `statement`.
  **L1750 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1751 EN**: Closes the current lexical scope or compound statement.
  **L1751 CN**: 结束当前词法作用域或复合语句块。
- **L1752 EN**: Blank line separating nearby declarations or logic blocks.
  **L1752 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1753-1776

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_cvtepu8_epi16 (__mmask16 __U, __m128i __A)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                             (__v16hi)_mm256_cvtepu8_epi16(__A),
                                             (__v16hi)_mm256_setzero_si256());
}


#define _mm_mask_shufflehi_epi16(W, U, A, imm) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                       (__v8hi)_mm_shufflehi_epi16((A), (imm)), \
                                       (__v8hi)(__m128i)(W)))

#define _mm_maskz_shufflehi_epi16(U, A, imm) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                       (__v8hi)_mm_shufflehi_epi16((A), (imm)), \
                                       (__v8hi)_mm_setzero_si128()))

#define _mm256_mask_shufflehi_epi16(W, U, A, imm) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                       (__v16hi)_mm256_shufflehi_epi16((A), (imm)), \
                                       (__v16hi)(__m256i)(W)))

````
- **L1753 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1753 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1754 EN**: Continues logic associated with callable symbol `_mm256_maskz_cvtepu8_epi16`.
  **L1754 CN**: 继续与可调用符号 `_mm256_maskz_cvtepu8_epi16` 相关的逻辑。
- **L1755 EN**: Opens a new lexical scope or compound statement.
  **L1755 CN**: 打开一个新的词法作用域或复合语句块。
- **L1756 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1756 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_cvtepu8_epi16(__A),`.
  **L1757 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_cvtepu8_epi16(__A),`。
- **L1758 EN**: Executes a call or declaration centered on `statement`.
  **L1758 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1759 EN**: Closes the current lexical scope or compound statement.
  **L1759 CN**: 结束当前词法作用域或复合语句块。
- **L1760 EN**: Blank line separating nearby declarations or logic blocks.
  **L1760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1761 EN**: Blank line separating nearby declarations or logic blocks.
  **L1761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1762 EN**: Defines macro `_mm_mask_shufflehi_epi16(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1762 CN**: 定义宏 `_mm_mask_shufflehi_epi16(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1763 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L1763 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L1764 EN**: Continues logic associated with callable symbol `_mm_shufflehi_epi16`.
  **L1764 CN**: 继续与可调用符号 `_mm_shufflehi_epi16` 相关的逻辑。
- **L1765 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(W)))`.
  **L1765 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(W)))`。
- **L1766 EN**: Blank line separating nearby declarations or logic blocks.
  **L1766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1767 EN**: Defines macro `_mm_maskz_shufflehi_epi16(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1767 CN**: 定义宏 `_mm_maskz_shufflehi_epi16(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1768 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L1768 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L1769 EN**: Continues logic associated with callable symbol `_mm_shufflehi_epi16`.
  **L1769 CN**: 继续与可调用符号 `_mm_shufflehi_epi16` 相关的逻辑。
- **L1770 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L1770 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L1771 EN**: Blank line separating nearby declarations or logic blocks.
  **L1771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1772 EN**: Defines macro `_mm256_mask_shufflehi_epi16(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1772 CN**: 定义宏 `_mm256_mask_shufflehi_epi16(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1773 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L1773 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L1774 EN**: Continues logic associated with callable symbol `_mm256_shufflehi_epi16`.
  **L1774 CN**: 继续与可调用符号 `_mm256_shufflehi_epi16` 相关的逻辑。
- **L1775 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(W)))`.
  **L1775 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(W)))`。
- **L1776 EN**: Blank line separating nearby declarations or logic blocks.
  **L1776 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1777-1800

````c
#define _mm256_maskz_shufflehi_epi16(U, A, imm) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                       (__v16hi)_mm256_shufflehi_epi16((A), (imm)), \
                                       (__v16hi)_mm256_setzero_si256()))

#define _mm_mask_shufflelo_epi16(W, U, A, imm) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                       (__v8hi)_mm_shufflelo_epi16((A), (imm)), \
                                       (__v8hi)(__m128i)(W)))

#define _mm_maskz_shufflelo_epi16(U, A, imm) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                       (__v8hi)_mm_shufflelo_epi16((A), (imm)), \
                                       (__v8hi)_mm_setzero_si128()))

#define _mm256_mask_shufflelo_epi16(W, U, A, imm) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                       (__v16hi)_mm256_shufflelo_epi16((A), \
                                                                       (imm)), \
                                       (__v16hi)(__m256i)(W)))

#define _mm256_maskz_shufflelo_epi16(U, A, imm) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                       (__v16hi)_mm256_shufflelo_epi16((A), \
````
- **L1777 EN**: Defines macro `_mm256_maskz_shufflehi_epi16(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1777 CN**: 定义宏 `_mm256_maskz_shufflehi_epi16(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1778 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L1778 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L1779 EN**: Continues logic associated with callable symbol `_mm256_shufflehi_epi16`.
  **L1779 CN**: 继续与可调用符号 `_mm256_shufflehi_epi16` 相关的逻辑。
- **L1780 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L1780 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L1781 EN**: Blank line separating nearby declarations or logic blocks.
  **L1781 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1782 EN**: Defines macro `_mm_mask_shufflelo_epi16(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1782 CN**: 定义宏 `_mm_mask_shufflelo_epi16(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1783 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L1783 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L1784 EN**: Continues logic associated with callable symbol `_mm_shufflelo_epi16`.
  **L1784 CN**: 继续与可调用符号 `_mm_shufflelo_epi16` 相关的逻辑。
- **L1785 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(W)))`.
  **L1785 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(W)))`。
- **L1786 EN**: Blank line separating nearby declarations or logic blocks.
  **L1786 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1787 EN**: Defines macro `_mm_maskz_shufflelo_epi16(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1787 CN**: 定义宏 `_mm_maskz_shufflelo_epi16(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1788 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L1788 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L1789 EN**: Continues logic associated with callable symbol `_mm_shufflelo_epi16`.
  **L1789 CN**: 继续与可调用符号 `_mm_shufflelo_epi16` 相关的逻辑。
- **L1790 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L1790 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L1791 EN**: Blank line separating nearby declarations or logic blocks.
  **L1791 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1792 EN**: Defines macro `_mm256_mask_shufflelo_epi16(W, U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1792 CN**: 定义宏 `_mm256_mask_shufflelo_epi16(W, U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1793 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L1793 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L1794 EN**: Continues logic associated with callable symbol `_mm256_shufflelo_epi16`.
  **L1794 CN**: 继续与可调用符号 `_mm256_shufflelo_epi16` 相关的逻辑。
- **L1795 EN**: Continues the surrounding expression or declaration: `(imm)), \`.
  **L1795 CN**: 继续构造周围的表达式或声明：`(imm)), \`。
- **L1796 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(W)))`.
  **L1796 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(W)))`。
- **L1797 EN**: Blank line separating nearby declarations or logic blocks.
  **L1797 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1798 EN**: Defines macro `_mm256_maskz_shufflelo_epi16(U, A, imm)` for conditional compilation, shorthand, or API generation.
  **L1798 CN**: 定义宏 `_mm256_maskz_shufflelo_epi16(U, A, imm)`，用于条件编译、简写或 API 生成。
- **L1799 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L1799 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L1800 EN**: Continues logic associated with callable symbol `_mm256_shufflelo_epi16`.
  **L1800 CN**: 继续与可调用符号 `_mm256_shufflelo_epi16` 相关的逻辑。

### Lines 1801-1824

````c
                                                                       (imm)), \
                                       (__v16hi)_mm256_setzero_si256()))

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_sllv_epi16(__m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_psllv16hi((__v16hi)__A, (__v16hi)__B);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_sllv_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_sllv_epi16(__A, __B),
                                           (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_sllv_epi16(__mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_sllv_epi16(__A, __B),
                                           (__v16hi)_mm256_setzero_si256());
}
````
- **L1801 EN**: Continues the surrounding expression or declaration: `(imm)), \`.
  **L1801 CN**: 继续构造周围的表达式或声明：`(imm)), \`。
- **L1802 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L1802 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L1803 EN**: Blank line separating nearby declarations or logic blocks.
  **L1803 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1804 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1804 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1805 EN**: Continues logic associated with callable symbol `_mm256_sllv_epi16`.
  **L1805 CN**: 继续与可调用符号 `_mm256_sllv_epi16` 相关的逻辑。
- **L1806 EN**: Opens a new lexical scope or compound statement.
  **L1806 CN**: 打开一个新的词法作用域或复合语句块。
- **L1807 EN**: Returns from the current function with `(__m256i)__builtin_ia32_psllv16hi((__v16hi)__A, (__v16hi)__B)`.
  **L1807 CN**: 以 `(__m256i)__builtin_ia32_psllv16hi((__v16hi)__A, (__v16hi)__B)` 从当前函数返回。
- **L1808 EN**: Closes the current lexical scope or compound statement.
  **L1808 CN**: 结束当前词法作用域或复合语句块。
- **L1809 EN**: Blank line separating nearby declarations or logic blocks.
  **L1809 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1810 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1810 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1811 EN**: Continues logic associated with callable symbol `_mm256_mask_sllv_epi16`.
  **L1811 CN**: 继续与可调用符号 `_mm256_mask_sllv_epi16` 相关的逻辑。
- **L1812 EN**: Opens a new lexical scope or compound statement.
  **L1812 CN**: 打开一个新的词法作用域或复合语句块。
- **L1813 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1813 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_sllv_epi16(__A, __B),`.
  **L1814 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_sllv_epi16(__A, __B),`。
- **L1815 EN**: Executes a call or declaration centered on `statement`.
  **L1815 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1816 EN**: Closes the current lexical scope or compound statement.
  **L1816 CN**: 结束当前词法作用域或复合语句块。
- **L1817 EN**: Blank line separating nearby declarations or logic blocks.
  **L1817 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1818 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1818 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1819 EN**: Continues logic associated with callable symbol `_mm256_maskz_sllv_epi16`.
  **L1819 CN**: 继续与可调用符号 `_mm256_maskz_sllv_epi16` 相关的逻辑。
- **L1820 EN**: Opens a new lexical scope or compound statement.
  **L1820 CN**: 打开一个新的词法作用域或复合语句块。
- **L1821 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1821 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_sllv_epi16(__A, __B),`.
  **L1822 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_sllv_epi16(__A, __B),`。
- **L1823 EN**: Executes a call or declaration centered on `statement`.
  **L1823 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1824 EN**: Closes the current lexical scope or compound statement.
  **L1824 CN**: 结束当前词法作用域或复合语句块。

### Lines 1825-1848

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_sllv_epi16(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_psllv8hi((__v8hi)__A, (__v8hi)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_sllv_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_sllv_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_sllv_epi16(__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_sllv_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
````
- **L1825 EN**: Blank line separating nearby declarations or logic blocks.
  **L1825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1826 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1826 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1827 EN**: Continues logic associated with callable symbol `_mm_sllv_epi16`.
  **L1827 CN**: 继续与可调用符号 `_mm_sllv_epi16` 相关的逻辑。
- **L1828 EN**: Opens a new lexical scope or compound statement.
  **L1828 CN**: 打开一个新的词法作用域或复合语句块。
- **L1829 EN**: Returns from the current function with `(__m128i)__builtin_ia32_psllv8hi((__v8hi)__A, (__v8hi)__B)`.
  **L1829 CN**: 以 `(__m128i)__builtin_ia32_psllv8hi((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1830 EN**: Closes the current lexical scope or compound statement.
  **L1830 CN**: 结束当前词法作用域或复合语句块。
- **L1831 EN**: Blank line separating nearby declarations or logic blocks.
  **L1831 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1832 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1832 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1833 EN**: Continues logic associated with callable symbol `_mm_mask_sllv_epi16`.
  **L1833 CN**: 继续与可调用符号 `_mm_mask_sllv_epi16` 相关的逻辑。
- **L1834 EN**: Opens a new lexical scope or compound statement.
  **L1834 CN**: 打开一个新的词法作用域或复合语句块。
- **L1835 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1835 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_sllv_epi16(__A, __B),`.
  **L1836 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_sllv_epi16(__A, __B),`。
- **L1837 EN**: Executes a call or declaration centered on `statement`.
  **L1837 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1838 EN**: Closes the current lexical scope or compound statement.
  **L1838 CN**: 结束当前词法作用域或复合语句块。
- **L1839 EN**: Blank line separating nearby declarations or logic blocks.
  **L1839 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1840 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1840 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1841 EN**: Continues logic associated with callable symbol `_mm_maskz_sllv_epi16`.
  **L1841 CN**: 继续与可调用符号 `_mm_maskz_sllv_epi16` 相关的逻辑。
- **L1842 EN**: Opens a new lexical scope or compound statement.
  **L1842 CN**: 打开一个新的词法作用域或复合语句块。
- **L1843 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1843 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_sllv_epi16(__A, __B),`.
  **L1844 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_sllv_epi16(__A, __B),`。
- **L1845 EN**: Executes a call or declaration centered on `statement`.
  **L1845 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1846 EN**: Closes the current lexical scope or compound statement.
  **L1846 CN**: 结束当前词法作用域或复合语句块。
- **L1847 EN**: Blank line separating nearby declarations or logic blocks.
  **L1847 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1848 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1848 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。

### Lines 1849-1872

````c
_mm_mask_sll_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_sll_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_sll_epi16 (__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_sll_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_sll_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m128i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                          (__v16hi)_mm256_sll_epi16(__A, __B),
                                          (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
````
- **L1849 EN**: Continues logic associated with callable symbol `_mm_mask_sll_epi16`.
  **L1849 CN**: 继续与可调用符号 `_mm_mask_sll_epi16` 相关的逻辑。
- **L1850 EN**: Opens a new lexical scope or compound statement.
  **L1850 CN**: 打开一个新的词法作用域或复合语句块。
- **L1851 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1851 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_sll_epi16(__A, __B),`.
  **L1852 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_sll_epi16(__A, __B),`。
- **L1853 EN**: Executes a call or declaration centered on `statement`.
  **L1853 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1854 EN**: Closes the current lexical scope or compound statement.
  **L1854 CN**: 结束当前词法作用域或复合语句块。
- **L1855 EN**: Blank line separating nearby declarations or logic blocks.
  **L1855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1856 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1856 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L1857 EN**: Continues logic associated with callable symbol `_mm_maskz_sll_epi16`.
  **L1857 CN**: 继续与可调用符号 `_mm_maskz_sll_epi16` 相关的逻辑。
- **L1858 EN**: Opens a new lexical scope or compound statement.
  **L1858 CN**: 打开一个新的词法作用域或复合语句块。
- **L1859 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1859 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_sll_epi16(__A, __B),`.
  **L1860 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_sll_epi16(__A, __B),`。
- **L1861 EN**: Executes a call or declaration centered on `statement`.
  **L1861 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1862 EN**: Closes the current lexical scope or compound statement.
  **L1862 CN**: 结束当前词法作用域或复合语句块。
- **L1863 EN**: Blank line separating nearby declarations or logic blocks.
  **L1863 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1864 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1864 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L1865 EN**: Continues logic associated with callable symbol `_mm256_mask_sll_epi16`.
  **L1865 CN**: 继续与可调用符号 `_mm256_mask_sll_epi16` 相关的逻辑。
- **L1866 EN**: Opens a new lexical scope or compound statement.
  **L1866 CN**: 打开一个新的词法作用域或复合语句块。
- **L1867 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1867 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_sll_epi16(__A, __B),`.
  **L1868 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_sll_epi16(__A, __B),`。
- **L1869 EN**: Executes a call or declaration centered on `statement`.
  **L1869 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1870 EN**: Closes the current lexical scope or compound statement.
  **L1870 CN**: 结束当前词法作用域或复合语句块。
- **L1871 EN**: Blank line separating nearby declarations or logic blocks.
  **L1871 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1872 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L1872 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。

### Lines 1873-1896

````c
_mm256_maskz_sll_epi16(__mmask16 __U, __m256i __A, __m128i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                          (__v16hi)_mm256_sll_epi16(__A, __B),
                                          (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_slli_epi16(__m128i __W, __mmask8 __U, __m128i __A, unsigned int __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_slli_epi16(__A, (int)__B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_slli_epi16 (__mmask8 __U, __m128i __A, unsigned int __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_slli_epi16(__A, (int)__B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L1873 EN**: Continues logic associated with callable symbol `_mm256_maskz_sll_epi16`.
  **L1873 CN**: 继续与可调用符号 `_mm256_maskz_sll_epi16` 相关的逻辑。
- **L1874 EN**: Opens a new lexical scope or compound statement.
  **L1874 CN**: 打开一个新的词法作用域或复合语句块。
- **L1875 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1875 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_sll_epi16(__A, __B),`.
  **L1876 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_sll_epi16(__A, __B),`。
- **L1877 EN**: Executes a call or declaration centered on `statement`.
  **L1877 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1878 EN**: Closes the current lexical scope or compound statement.
  **L1878 CN**: 结束当前词法作用域或复合语句块。
- **L1879 EN**: Blank line separating nearby declarations or logic blocks.
  **L1879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1880 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1880 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1881 EN**: Continues logic associated with callable symbol `_mm_mask_slli_epi16`.
  **L1881 CN**: 继续与可调用符号 `_mm_mask_slli_epi16` 相关的逻辑。
- **L1882 EN**: Opens a new lexical scope or compound statement.
  **L1882 CN**: 打开一个新的词法作用域或复合语句块。
- **L1883 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1883 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_slli_epi16(__A, (int)__B),`.
  **L1884 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_slli_epi16(__A, (int)__B),`。
- **L1885 EN**: Executes a call or declaration centered on `statement`.
  **L1885 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1886 EN**: Closes the current lexical scope or compound statement.
  **L1886 CN**: 结束当前词法作用域或复合语句块。
- **L1887 EN**: Blank line separating nearby declarations or logic blocks.
  **L1887 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1888 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1888 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1889 EN**: Continues logic associated with callable symbol `_mm_maskz_slli_epi16`.
  **L1889 CN**: 继续与可调用符号 `_mm_maskz_slli_epi16` 相关的逻辑。
- **L1890 EN**: Opens a new lexical scope or compound statement.
  **L1890 CN**: 打开一个新的词法作用域或复合语句块。
- **L1891 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1891 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_slli_epi16(__A, (int)__B),`.
  **L1892 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_slli_epi16(__A, (int)__B),`。
- **L1893 EN**: Executes a call or declaration centered on `statement`.
  **L1893 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1894 EN**: Closes the current lexical scope or compound statement.
  **L1894 CN**: 结束当前词法作用域或复合语句块。
- **L1895 EN**: Blank line separating nearby declarations or logic blocks.
  **L1895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1896 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1896 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 1897-1920

````c
_mm256_mask_slli_epi16(__m256i __W, __mmask16 __U, __m256i __A,
                       unsigned int __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                         (__v16hi)_mm256_slli_epi16(__A, (int)__B),
                                         (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_slli_epi16(__mmask16 __U, __m256i __A, unsigned int __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                         (__v16hi)_mm256_slli_epi16(__A, (int)__B),
                                         (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_srlv_epi16(__m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_psrlv16hi((__v16hi)__A, (__v16hi)__B);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_srlv_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
````
- **L1897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_mask_slli_epi16(__m256i __W, __mmask16 __U, __m256i __A,`.
  **L1897 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_mask_slli_epi16(__m256i __W, __mmask16 __U, __m256i __A,`。
- **L1898 EN**: Continues the surrounding expression or declaration: `unsigned int __B) {`.
  **L1898 CN**: 继续构造周围的表达式或声明：`unsigned int __B) {`。
- **L1899 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1899 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_slli_epi16(__A, (int)__B),`.
  **L1900 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_slli_epi16(__A, (int)__B),`。
- **L1901 EN**: Executes a call or declaration centered on `statement`.
  **L1901 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1902 EN**: Closes the current lexical scope or compound statement.
  **L1902 CN**: 结束当前词法作用域或复合语句块。
- **L1903 EN**: Blank line separating nearby declarations or logic blocks.
  **L1903 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1904 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1904 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1905 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_slli_epi16(__mmask16 __U, __m256i __A, unsigned int __B) {`.
  **L1905 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_slli_epi16(__mmask16 __U, __m256i __A, unsigned int __B) {`。
- **L1906 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1906 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_slli_epi16(__A, (int)__B),`.
  **L1907 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_slli_epi16(__A, (int)__B),`。
- **L1908 EN**: Executes a call or declaration centered on `statement`.
  **L1908 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1909 EN**: Closes the current lexical scope or compound statement.
  **L1909 CN**: 结束当前词法作用域或复合语句块。
- **L1910 EN**: Blank line separating nearby declarations or logic blocks.
  **L1910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1911 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1911 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1912 EN**: Continues logic associated with callable symbol `_mm256_srlv_epi16`.
  **L1912 CN**: 继续与可调用符号 `_mm256_srlv_epi16` 相关的逻辑。
- **L1913 EN**: Opens a new lexical scope or compound statement.
  **L1913 CN**: 打开一个新的词法作用域或复合语句块。
- **L1914 EN**: Returns from the current function with `(__m256i)__builtin_ia32_psrlv16hi((__v16hi)__A, (__v16hi)__B)`.
  **L1914 CN**: 以 `(__m256i)__builtin_ia32_psrlv16hi((__v16hi)__A, (__v16hi)__B)` 从当前函数返回。
- **L1915 EN**: Closes the current lexical scope or compound statement.
  **L1915 CN**: 结束当前词法作用域或复合语句块。
- **L1916 EN**: Blank line separating nearby declarations or logic blocks.
  **L1916 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1917 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1917 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1918 EN**: Continues logic associated with callable symbol `_mm256_mask_srlv_epi16`.
  **L1918 CN**: 继续与可调用符号 `_mm256_mask_srlv_epi16` 相关的逻辑。
- **L1919 EN**: Opens a new lexical scope or compound statement.
  **L1919 CN**: 打开一个新的词法作用域或复合语句块。
- **L1920 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1920 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。

### Lines 1921-1944

````c
                                           (__v16hi)_mm256_srlv_epi16(__A, __B),
                                           (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_srlv_epi16(__mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_srlv_epi16(__A, __B),
                                           (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_srlv_epi16(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_psrlv8hi((__v8hi)__A, (__v8hi)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_srlv_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srlv_epi16(__A, __B),
                                             (__v8hi)__W);
````
- **L1921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srlv_epi16(__A, __B),`.
  **L1921 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srlv_epi16(__A, __B),`。
- **L1922 EN**: Executes a call or declaration centered on `statement`.
  **L1922 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1923 EN**: Closes the current lexical scope or compound statement.
  **L1923 CN**: 结束当前词法作用域或复合语句块。
- **L1924 EN**: Blank line separating nearby declarations or logic blocks.
  **L1924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1925 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1925 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1926 EN**: Continues logic associated with callable symbol `_mm256_maskz_srlv_epi16`.
  **L1926 CN**: 继续与可调用符号 `_mm256_maskz_srlv_epi16` 相关的逻辑。
- **L1927 EN**: Opens a new lexical scope or compound statement.
  **L1927 CN**: 打开一个新的词法作用域或复合语句块。
- **L1928 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1928 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srlv_epi16(__A, __B),`.
  **L1929 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srlv_epi16(__A, __B),`。
- **L1930 EN**: Executes a call or declaration centered on `statement`.
  **L1930 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1931 EN**: Closes the current lexical scope or compound statement.
  **L1931 CN**: 结束当前词法作用域或复合语句块。
- **L1932 EN**: Blank line separating nearby declarations or logic blocks.
  **L1932 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1933 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1933 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1934 EN**: Continues logic associated with callable symbol `_mm_srlv_epi16`.
  **L1934 CN**: 继续与可调用符号 `_mm_srlv_epi16` 相关的逻辑。
- **L1935 EN**: Opens a new lexical scope or compound statement.
  **L1935 CN**: 打开一个新的词法作用域或复合语句块。
- **L1936 EN**: Returns from the current function with `(__m128i)__builtin_ia32_psrlv8hi((__v8hi)__A, (__v8hi)__B)`.
  **L1936 CN**: 以 `(__m128i)__builtin_ia32_psrlv8hi((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1937 EN**: Closes the current lexical scope or compound statement.
  **L1937 CN**: 结束当前词法作用域或复合语句块。
- **L1938 EN**: Blank line separating nearby declarations or logic blocks.
  **L1938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1939 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1939 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1940 EN**: Continues logic associated with callable symbol `_mm_mask_srlv_epi16`.
  **L1940 CN**: 继续与可调用符号 `_mm_mask_srlv_epi16` 相关的逻辑。
- **L1941 EN**: Opens a new lexical scope or compound statement.
  **L1941 CN**: 打开一个新的词法作用域或复合语句块。
- **L1942 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1942 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1943 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srlv_epi16(__A, __B),`.
  **L1943 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srlv_epi16(__A, __B),`。
- **L1944 EN**: Executes a call or declaration centered on `statement`.
  **L1944 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1945-1968

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_srlv_epi16(__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srlv_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_srav_epi16(__m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_psrav16hi((__v16hi)__A, (__v16hi)__B);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_srav_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_srav_epi16(__A, __B),
                                           (__v16hi)__W);
}

````
- **L1945 EN**: Closes the current lexical scope or compound statement.
  **L1945 CN**: 结束当前词法作用域或复合语句块。
- **L1946 EN**: Blank line separating nearby declarations or logic blocks.
  **L1946 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1947 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1947 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1948 EN**: Continues logic associated with callable symbol `_mm_maskz_srlv_epi16`.
  **L1948 CN**: 继续与可调用符号 `_mm_maskz_srlv_epi16` 相关的逻辑。
- **L1949 EN**: Opens a new lexical scope or compound statement.
  **L1949 CN**: 打开一个新的词法作用域或复合语句块。
- **L1950 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1950 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srlv_epi16(__A, __B),`.
  **L1951 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srlv_epi16(__A, __B),`。
- **L1952 EN**: Executes a call or declaration centered on `statement`.
  **L1952 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1953 EN**: Closes the current lexical scope or compound statement.
  **L1953 CN**: 结束当前词法作用域或复合语句块。
- **L1954 EN**: Blank line separating nearby declarations or logic blocks.
  **L1954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1955 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1955 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1956 EN**: Continues logic associated with callable symbol `_mm256_srav_epi16`.
  **L1956 CN**: 继续与可调用符号 `_mm256_srav_epi16` 相关的逻辑。
- **L1957 EN**: Opens a new lexical scope or compound statement.
  **L1957 CN**: 打开一个新的词法作用域或复合语句块。
- **L1958 EN**: Returns from the current function with `(__m256i)__builtin_ia32_psrav16hi((__v16hi)__A, (__v16hi)__B)`.
  **L1958 CN**: 以 `(__m256i)__builtin_ia32_psrav16hi((__v16hi)__A, (__v16hi)__B)` 从当前函数返回。
- **L1959 EN**: Closes the current lexical scope or compound statement.
  **L1959 CN**: 结束当前词法作用域或复合语句块。
- **L1960 EN**: Blank line separating nearby declarations or logic blocks.
  **L1960 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1961 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1961 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1962 EN**: Continues logic associated with callable symbol `_mm256_mask_srav_epi16`.
  **L1962 CN**: 继续与可调用符号 `_mm256_mask_srav_epi16` 相关的逻辑。
- **L1963 EN**: Opens a new lexical scope or compound statement.
  **L1963 CN**: 打开一个新的词法作用域或复合语句块。
- **L1964 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1964 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srav_epi16(__A, __B),`.
  **L1965 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srav_epi16(__A, __B),`。
- **L1966 EN**: Executes a call or declaration centered on `statement`.
  **L1966 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1967 EN**: Closes the current lexical scope or compound statement.
  **L1967 CN**: 结束当前词法作用域或复合语句块。
- **L1968 EN**: Blank line separating nearby declarations or logic blocks.
  **L1968 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1969-1992

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_srav_epi16(__mmask16 __U, __m256i __A, __m256i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                           (__v16hi)_mm256_srav_epi16(__A, __B),
                                           (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_srav_epi16(__m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_psrav8hi((__v8hi)__A, (__v8hi)__B);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_srav_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srav_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_srav_epi16(__mmask8 __U, __m128i __A, __m128i __B)
````
- **L1969 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L1969 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L1970 EN**: Continues logic associated with callable symbol `_mm256_maskz_srav_epi16`.
  **L1970 CN**: 继续与可调用符号 `_mm256_maskz_srav_epi16` 相关的逻辑。
- **L1971 EN**: Opens a new lexical scope or compound statement.
  **L1971 CN**: 打开一个新的词法作用域或复合语句块。
- **L1972 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L1972 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L1973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srav_epi16(__A, __B),`.
  **L1973 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srav_epi16(__A, __B),`。
- **L1974 EN**: Executes a call or declaration centered on `statement`.
  **L1974 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1975 EN**: Closes the current lexical scope or compound statement.
  **L1975 CN**: 结束当前词法作用域或复合语句块。
- **L1976 EN**: Blank line separating nearby declarations or logic blocks.
  **L1976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1977 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1977 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1978 EN**: Continues logic associated with callable symbol `_mm_srav_epi16`.
  **L1978 CN**: 继续与可调用符号 `_mm_srav_epi16` 相关的逻辑。
- **L1979 EN**: Opens a new lexical scope or compound statement.
  **L1979 CN**: 打开一个新的词法作用域或复合语句块。
- **L1980 EN**: Returns from the current function with `(__m128i)__builtin_ia32_psrav8hi((__v8hi)__A, (__v8hi)__B)`.
  **L1980 CN**: 以 `(__m128i)__builtin_ia32_psrav8hi((__v8hi)__A, (__v8hi)__B)` 从当前函数返回。
- **L1981 EN**: Closes the current lexical scope or compound statement.
  **L1981 CN**: 结束当前词法作用域或复合语句块。
- **L1982 EN**: Blank line separating nearby declarations or logic blocks.
  **L1982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1983 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1983 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1984 EN**: Continues logic associated with callable symbol `_mm_mask_srav_epi16`.
  **L1984 CN**: 继续与可调用符号 `_mm_mask_srav_epi16` 相关的逻辑。
- **L1985 EN**: Opens a new lexical scope or compound statement.
  **L1985 CN**: 打开一个新的词法作用域或复合语句块。
- **L1986 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1986 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srav_epi16(__A, __B),`.
  **L1987 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srav_epi16(__A, __B),`。
- **L1988 EN**: Executes a call or declaration centered on `statement`.
  **L1988 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1989 EN**: Closes the current lexical scope or compound statement.
  **L1989 CN**: 结束当前词法作用域或复合语句块。
- **L1990 EN**: Blank line separating nearby declarations or logic blocks.
  **L1990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1991 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L1991 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L1992 EN**: Continues logic associated with callable symbol `_mm_maskz_srav_epi16`.
  **L1992 CN**: 继续与可调用符号 `_mm_maskz_srav_epi16` 相关的逻辑。

### Lines 1993-2016

````c
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srav_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_sra_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_sra_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_sra_epi16(__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_sra_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_sra_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m128i __B)
````
- **L1993 EN**: Opens a new lexical scope or compound statement.
  **L1993 CN**: 打开一个新的词法作用域或复合语句块。
- **L1994 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L1994 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L1995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srav_epi16(__A, __B),`.
  **L1995 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srav_epi16(__A, __B),`。
- **L1996 EN**: Executes a call or declaration centered on `statement`.
  **L1996 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1997 EN**: Closes the current lexical scope or compound statement.
  **L1997 CN**: 结束当前词法作用域或复合语句块。
- **L1998 EN**: Blank line separating nearby declarations or logic blocks.
  **L1998 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1999 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L1999 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2000 EN**: Continues logic associated with callable symbol `_mm_mask_sra_epi16`.
  **L2000 CN**: 继续与可调用符号 `_mm_mask_sra_epi16` 相关的逻辑。
- **L2001 EN**: Opens a new lexical scope or compound statement.
  **L2001 CN**: 打开一个新的词法作用域或复合语句块。
- **L2002 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L2002 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L2003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_sra_epi16(__A, __B),`.
  **L2003 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_sra_epi16(__A, __B),`。
- **L2004 EN**: Executes a call or declaration centered on `statement`.
  **L2004 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2005 EN**: Closes the current lexical scope or compound statement.
  **L2005 CN**: 结束当前词法作用域或复合语句块。
- **L2006 EN**: Blank line separating nearby declarations or logic blocks.
  **L2006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2007 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2007 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2008 EN**: Continues logic associated with callable symbol `_mm_maskz_sra_epi16`.
  **L2008 CN**: 继续与可调用符号 `_mm_maskz_sra_epi16` 相关的逻辑。
- **L2009 EN**: Opens a new lexical scope or compound statement.
  **L2009 CN**: 打开一个新的词法作用域或复合语句块。
- **L2010 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L2010 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L2011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_sra_epi16(__A, __B),`.
  **L2011 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_sra_epi16(__A, __B),`。
- **L2012 EN**: Executes a call or declaration centered on `statement`.
  **L2012 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2013 EN**: Closes the current lexical scope or compound statement.
  **L2013 CN**: 结束当前词法作用域或复合语句块。
- **L2014 EN**: Blank line separating nearby declarations or logic blocks.
  **L2014 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2015 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2015 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2016 EN**: Continues logic associated with callable symbol `_mm256_mask_sra_epi16`.
  **L2016 CN**: 继续与可调用符号 `_mm256_mask_sra_epi16` 相关的逻辑。

### Lines 2017-2040

````c
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                          (__v16hi)_mm256_sra_epi16(__A, __B),
                                          (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_sra_epi16(__mmask16 __U, __m256i __A, __m128i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                          (__v16hi)_mm256_sra_epi16(__A, __B),
                                          (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_srai_epi16(__m128i __W, __mmask8 __U, __m128i __A, unsigned int __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srai_epi16(__A, (int)__B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_srai_epi16(__mmask8 __U, __m128i __A, unsigned int __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
````
- **L2017 EN**: Opens a new lexical scope or compound statement.
  **L2017 CN**: 打开一个新的词法作用域或复合语句块。
- **L2018 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L2018 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L2019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_sra_epi16(__A, __B),`.
  **L2019 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_sra_epi16(__A, __B),`。
- **L2020 EN**: Executes a call or declaration centered on `statement`.
  **L2020 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2021 EN**: Closes the current lexical scope or compound statement.
  **L2021 CN**: 结束当前词法作用域或复合语句块。
- **L2022 EN**: Blank line separating nearby declarations or logic blocks.
  **L2022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2023 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2023 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2024 EN**: Continues logic associated with callable symbol `_mm256_maskz_sra_epi16`.
  **L2024 CN**: 继续与可调用符号 `_mm256_maskz_sra_epi16` 相关的逻辑。
- **L2025 EN**: Opens a new lexical scope or compound statement.
  **L2025 CN**: 打开一个新的词法作用域或复合语句块。
- **L2026 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L2026 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L2027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_sra_epi16(__A, __B),`.
  **L2027 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_sra_epi16(__A, __B),`。
- **L2028 EN**: Executes a call or declaration centered on `statement`.
  **L2028 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2029 EN**: Closes the current lexical scope or compound statement.
  **L2029 CN**: 结束当前词法作用域或复合语句块。
- **L2030 EN**: Blank line separating nearby declarations or logic blocks.
  **L2030 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2031 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2031 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2032 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_srai_epi16(__m128i __W, __mmask8 __U, __m128i __A, unsigned int __B) {`.
  **L2032 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_srai_epi16(__m128i __W, __mmask8 __U, __m128i __A, unsigned int __B) {`。
- **L2033 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L2033 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L2034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srai_epi16(__A, (int)__B),`.
  **L2034 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srai_epi16(__A, (int)__B),`。
- **L2035 EN**: Executes a call or declaration centered on `statement`.
  **L2035 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2036 EN**: Closes the current lexical scope or compound statement.
  **L2036 CN**: 结束当前词法作用域或复合语句块。
- **L2037 EN**: Blank line separating nearby declarations or logic blocks.
  **L2037 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2038 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2038 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2039 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_srai_epi16(__mmask8 __U, __m128i __A, unsigned int __B) {`.
  **L2039 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_srai_epi16(__mmask8 __U, __m128i __A, unsigned int __B) {`。
- **L2040 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L2040 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。

### Lines 2041-2064

````c
                                             (__v8hi)_mm_srai_epi16(__A, (int)__B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_srai_epi16(__m256i __W, __mmask16 __U, __m256i __A,
                       unsigned int __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                         (__v16hi)_mm256_srai_epi16(__A, (int)__B),
                                         (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_srai_epi16(__mmask16 __U, __m256i __A, unsigned int __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                         (__v16hi)_mm256_srai_epi16(__A, (int)__B),
                                         (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_srl_epi16(__m128i __W, __mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srl_epi16(__A, __B),
````
- **L2041 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srai_epi16(__A, (int)__B),`.
  **L2041 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srai_epi16(__A, (int)__B),`。
- **L2042 EN**: Executes a call or declaration centered on `statement`.
  **L2042 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2043 EN**: Closes the current lexical scope or compound statement.
  **L2043 CN**: 结束当前词法作用域或复合语句块。
- **L2044 EN**: Blank line separating nearby declarations or logic blocks.
  **L2044 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2045 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2045 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_mask_srai_epi16(__m256i __W, __mmask16 __U, __m256i __A,`.
  **L2046 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_mask_srai_epi16(__m256i __W, __mmask16 __U, __m256i __A,`。
- **L2047 EN**: Continues the surrounding expression or declaration: `unsigned int __B) {`.
  **L2047 CN**: 继续构造周围的表达式或声明：`unsigned int __B) {`。
- **L2048 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L2048 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L2049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srai_epi16(__A, (int)__B),`.
  **L2049 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srai_epi16(__A, (int)__B),`。
- **L2050 EN**: Executes a call or declaration centered on `statement`.
  **L2050 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2051 EN**: Closes the current lexical scope or compound statement.
  **L2051 CN**: 结束当前词法作用域或复合语句块。
- **L2052 EN**: Blank line separating nearby declarations or logic blocks.
  **L2052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2053 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2053 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2054 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_srai_epi16(__mmask16 __U, __m256i __A, unsigned int __B) {`.
  **L2054 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_srai_epi16(__mmask16 __U, __m256i __A, unsigned int __B) {`。
- **L2055 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L2055 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L2056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srai_epi16(__A, (int)__B),`.
  **L2056 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srai_epi16(__A, (int)__B),`。
- **L2057 EN**: Executes a call or declaration centered on `statement`.
  **L2057 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2058 EN**: Closes the current lexical scope or compound statement.
  **L2058 CN**: 结束当前词法作用域或复合语句块。
- **L2059 EN**: Blank line separating nearby declarations or logic blocks.
  **L2059 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2060 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2060 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2061 EN**: Continues logic associated with callable symbol `_mm_mask_srl_epi16`.
  **L2061 CN**: 继续与可调用符号 `_mm_mask_srl_epi16` 相关的逻辑。
- **L2062 EN**: Opens a new lexical scope or compound statement.
  **L2062 CN**: 打开一个新的词法作用域或复合语句块。
- **L2063 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L2063 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L2064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srl_epi16(__A, __B),`.
  **L2064 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srl_epi16(__A, __B),`。

### Lines 2065-2088

````c
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_srl_epi16 (__mmask8 __U, __m128i __A, __m128i __B)
{
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srl_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_srl_epi16(__m256i __W, __mmask16 __U, __m256i __A, __m128i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                          (__v16hi)_mm256_srl_epi16(__A, __B),
                                          (__v16hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_srl_epi16(__mmask16 __U, __m256i __A, __m128i __B)
{
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                          (__v16hi)_mm256_srl_epi16(__A, __B),
````
- **L2065 EN**: Executes a call or declaration centered on `statement`.
  **L2065 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2066 EN**: Closes the current lexical scope or compound statement.
  **L2066 CN**: 结束当前词法作用域或复合语句块。
- **L2067 EN**: Blank line separating nearby declarations or logic blocks.
  **L2067 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2068 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2068 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2069 EN**: Continues logic associated with callable symbol `_mm_maskz_srl_epi16`.
  **L2069 CN**: 继续与可调用符号 `_mm_maskz_srl_epi16` 相关的逻辑。
- **L2070 EN**: Opens a new lexical scope or compound statement.
  **L2070 CN**: 打开一个新的词法作用域或复合语句块。
- **L2071 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L2071 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L2072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srl_epi16(__A, __B),`.
  **L2072 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srl_epi16(__A, __B),`。
- **L2073 EN**: Executes a call or declaration centered on `statement`.
  **L2073 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2074 EN**: Closes the current lexical scope or compound statement.
  **L2074 CN**: 结束当前词法作用域或复合语句块。
- **L2075 EN**: Blank line separating nearby declarations or logic blocks.
  **L2075 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2076 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2076 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2077 EN**: Continues logic associated with callable symbol `_mm256_mask_srl_epi16`.
  **L2077 CN**: 继续与可调用符号 `_mm256_mask_srl_epi16` 相关的逻辑。
- **L2078 EN**: Opens a new lexical scope or compound statement.
  **L2078 CN**: 打开一个新的词法作用域或复合语句块。
- **L2079 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L2079 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L2080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srl_epi16(__A, __B),`.
  **L2080 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srl_epi16(__A, __B),`。
- **L2081 EN**: Executes a call or declaration centered on `statement`.
  **L2081 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2082 EN**: Closes the current lexical scope or compound statement.
  **L2082 CN**: 结束当前词法作用域或复合语句块。
- **L2083 EN**: Blank line separating nearby declarations or logic blocks.
  **L2083 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2084 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2084 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2085 EN**: Continues logic associated with callable symbol `_mm256_maskz_srl_epi16`.
  **L2085 CN**: 继续与可调用符号 `_mm256_maskz_srl_epi16` 相关的逻辑。
- **L2086 EN**: Opens a new lexical scope or compound statement.
  **L2086 CN**: 打开一个新的词法作用域或复合语句块。
- **L2087 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L2087 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L2088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srl_epi16(__A, __B),`.
  **L2088 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srl_epi16(__A, __B),`。

### Lines 2089-2112

````c
                                          (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_srli_epi16(__m128i __W, __mmask8 __U, __m128i __A, int __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srli_epi16(__A, __B),
                                             (__v8hi)__W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_srli_epi16(__mmask8 __U, __m128i __A, int __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__U,
                                             (__v8hi)_mm_srli_epi16(__A, __B),
                                             (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_srli_epi16(__m256i __W, __mmask16 __U, __m256i __A, int __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                         (__v16hi)_mm256_srli_epi16(__A, __B),
                                         (__v16hi)__W);
}

````
- **L2089 EN**: Executes a call or declaration centered on `statement`.
  **L2089 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2090 EN**: Closes the current lexical scope or compound statement.
  **L2090 CN**: 结束当前词法作用域或复合语句块。
- **L2091 EN**: Blank line separating nearby declarations or logic blocks.
  **L2091 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2092 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2092 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2093 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_srli_epi16(__m128i __W, __mmask8 __U, __m128i __A, int __B) {`.
  **L2093 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_srli_epi16(__m128i __W, __mmask8 __U, __m128i __A, int __B) {`。
- **L2094 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L2094 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L2095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srli_epi16(__A, __B),`.
  **L2095 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srli_epi16(__A, __B),`。
- **L2096 EN**: Executes a call or declaration centered on `statement`.
  **L2096 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2097 EN**: Closes the current lexical scope or compound statement.
  **L2097 CN**: 结束当前词法作用域或复合语句块。
- **L2098 EN**: Blank line separating nearby declarations or logic blocks.
  **L2098 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2099 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2099 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2100 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_srli_epi16(__mmask8 __U, __m128i __A, int __B) {`.
  **L2100 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_srli_epi16(__mmask8 __U, __m128i __A, int __B) {`。
- **L2101 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,`.
  **L2101 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__U,` 从当前函数返回。
- **L2102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_srli_epi16(__A, __B),`.
  **L2102 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_srli_epi16(__A, __B),`。
- **L2103 EN**: Executes a call or declaration centered on `statement`.
  **L2103 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2104 EN**: Closes the current lexical scope or compound statement.
  **L2104 CN**: 结束当前词法作用域或复合语句块。
- **L2105 EN**: Blank line separating nearby declarations or logic blocks.
  **L2105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2106 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2106 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2107 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_srli_epi16(__m256i __W, __mmask16 __U, __m256i __A, int __B) {`.
  **L2107 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_srli_epi16(__m256i __W, __mmask16 __U, __m256i __A, int __B) {`。
- **L2108 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L2108 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L2109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srli_epi16(__A, __B),`.
  **L2109 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srli_epi16(__A, __B),`。
- **L2110 EN**: Executes a call or declaration centered on `statement`.
  **L2110 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2111 EN**: Closes the current lexical scope or compound statement.
  **L2111 CN**: 结束当前词法作用域或复合语句块。
- **L2112 EN**: Blank line separating nearby declarations or logic blocks.
  **L2112 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2113-2136

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_srli_epi16(__mmask16 __U, __m256i __A, int __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__U,
                                         (__v16hi)_mm256_srli_epi16(__A, __B),
                                         (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_mov_epi16(__m128i __W, __mmask8 __U, __m128i __A) {
  return (__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,
                (__v8hi) __A,
                (__v8hi) __W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_mov_epi16(__mmask8 __U, __m128i __A) {
  return (__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,
                (__v8hi) __A,
                (__v8hi) _mm_setzero_si128 ());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_mov_epi16(__m256i __W, __mmask16 __U, __m256i __A) {
  return (__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,
````
- **L2113 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2113 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2114 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_srli_epi16(__mmask16 __U, __m256i __A, int __B) {`.
  **L2114 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_srli_epi16(__mmask16 __U, __m256i __A, int __B) {`。
- **L2115 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,`.
  **L2115 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__U,` 从当前函数返回。
- **L2116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_srli_epi16(__A, __B),`.
  **L2116 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_srli_epi16(__A, __B),`。
- **L2117 EN**: Executes a call or declaration centered on `statement`.
  **L2117 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2118 EN**: Closes the current lexical scope or compound statement.
  **L2118 CN**: 结束当前词法作用域或复合语句块。
- **L2119 EN**: Blank line separating nearby declarations or logic blocks.
  **L2119 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2120 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2120 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2121 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mov_epi16(__m128i __W, __mmask8 __U, __m128i __A) {`.
  **L2121 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mov_epi16(__m128i __W, __mmask8 __U, __m128i __A) {`。
- **L2122 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,`.
  **L2122 CN**: 以 `(__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,` 从当前函数返回。
- **L2123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) __A,`.
  **L2123 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) __A,`。
- **L2124 EN**: Executes a call or declaration centered on `statement`.
  **L2124 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2125 EN**: Closes the current lexical scope or compound statement.
  **L2125 CN**: 结束当前词法作用域或复合语句块。
- **L2126 EN**: Blank line separating nearby declarations or logic blocks.
  **L2126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2127 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2127 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2128 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mov_epi16(__mmask8 __U, __m128i __A) {`.
  **L2128 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mov_epi16(__mmask8 __U, __m128i __A) {`。
- **L2129 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,`.
  **L2129 CN**: 以 `(__m128i) __builtin_ia32_selectw_128 ((__mmask8) __U,` 从当前函数返回。
- **L2130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) __A,`.
  **L2130 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) __A,`。
- **L2131 EN**: Executes a call or declaration centered on `statement`.
  **L2131 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2132 EN**: Closes the current lexical scope or compound statement.
  **L2132 CN**: 结束当前词法作用域或复合语句块。
- **L2133 EN**: Blank line separating nearby declarations or logic blocks.
  **L2133 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2134 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2134 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2135 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mov_epi16(__m256i __W, __mmask16 __U, __m256i __A) {`.
  **L2135 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mov_epi16(__m256i __W, __mmask16 __U, __m256i __A) {`。
- **L2136 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,`.
  **L2136 CN**: 以 `(__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,` 从当前函数返回。

### Lines 2137-2160

````c
                (__v16hi) __A,
                (__v16hi) __W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_mov_epi16(__mmask16 __U, __m256i __A) {
  return (__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,
                (__v16hi) __A,
                (__v16hi) _mm256_setzero_si256 ());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_mov_epi8(__m128i __W, __mmask16 __U, __m128i __A) {
  return (__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,
                (__v16qi) __A,
                (__v16qi) __W);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_mov_epi8(__mmask16 __U, __m128i __A) {
  return (__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,
                (__v16qi) __A,
                (__v16qi) _mm_setzero_si128 ());
}
````
- **L2137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) __A,`.
  **L2137 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) __A,`。
- **L2138 EN**: Executes a call or declaration centered on `statement`.
  **L2138 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2139 EN**: Closes the current lexical scope or compound statement.
  **L2139 CN**: 结束当前词法作用域或复合语句块。
- **L2140 EN**: Blank line separating nearby declarations or logic blocks.
  **L2140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2141 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2141 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mov_epi16(__mmask16 __U, __m256i __A) {`.
  **L2142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mov_epi16(__mmask16 __U, __m256i __A) {`。
- **L2143 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,`.
  **L2143 CN**: 以 `(__m256i) __builtin_ia32_selectw_256 ((__mmask16) __U,` 从当前函数返回。
- **L2144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) __A,`.
  **L2144 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) __A,`。
- **L2145 EN**: Executes a call or declaration centered on `statement`.
  **L2145 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2146 EN**: Closes the current lexical scope or compound statement.
  **L2146 CN**: 结束当前词法作用域或复合语句块。
- **L2147 EN**: Blank line separating nearby declarations or logic blocks.
  **L2147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2148 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2148 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_mov_epi8(__m128i __W, __mmask16 __U, __m128i __A) {`.
  **L2149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_mov_epi8(__m128i __W, __mmask16 __U, __m128i __A) {`。
- **L2150 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,`.
  **L2150 CN**: 以 `(__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,` 从当前函数返回。
- **L2151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __A,`.
  **L2151 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __A,`。
- **L2152 EN**: Executes a call or declaration centered on `statement`.
  **L2152 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2153 EN**: Closes the current lexical scope or compound statement.
  **L2153 CN**: 结束当前词法作用域或复合语句块。
- **L2154 EN**: Blank line separating nearby declarations or logic blocks.
  **L2154 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2155 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2155 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_mov_epi8(__mmask16 __U, __m128i __A) {`.
  **L2156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_mov_epi8(__mmask16 __U, __m128i __A) {`。
- **L2157 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,`.
  **L2157 CN**: 以 `(__m128i) __builtin_ia32_selectb_128 ((__mmask16) __U,` 从当前函数返回。
- **L2158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __A,`.
  **L2158 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __A,`。
- **L2159 EN**: Executes a call or declaration centered on `statement`.
  **L2159 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2160 EN**: Closes the current lexical scope or compound statement.
  **L2160 CN**: 结束当前词法作用域或复合语句块。

### Lines 2161-2184

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_mov_epi8(__m256i __W, __mmask32 __U, __m256i __A) {
  return (__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,
                (__v32qi) __A,
                (__v32qi) __W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_mov_epi8(__mmask32 __U, __m256i __A) {
  return (__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,
                (__v32qi) __A,
                (__v32qi) _mm256_setzero_si256 ());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_set1_epi8 (__m128i __O, __mmask16 __M, char __A)
{
  return (__m128i) __builtin_ia32_selectb_128(__M,
                                              (__v16qi) _mm_set1_epi8(__A),
                                              (__v16qi) __O);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
````
- **L2161 EN**: Blank line separating nearby declarations or logic blocks.
  **L2161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2162 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2162 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_mov_epi8(__m256i __W, __mmask32 __U, __m256i __A) {`.
  **L2163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_mov_epi8(__m256i __W, __mmask32 __U, __m256i __A) {`。
- **L2164 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,`.
  **L2164 CN**: 以 `(__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,` 从当前函数返回。
- **L2165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) __A,`.
  **L2165 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) __A,`。
- **L2166 EN**: Executes a call or declaration centered on `statement`.
  **L2166 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2167 EN**: Closes the current lexical scope or compound statement.
  **L2167 CN**: 结束当前词法作用域或复合语句块。
- **L2168 EN**: Blank line separating nearby declarations or logic blocks.
  **L2168 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2169 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2169 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2170 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_mov_epi8(__mmask32 __U, __m256i __A) {`.
  **L2170 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_mov_epi8(__mmask32 __U, __m256i __A) {`。
- **L2171 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,`.
  **L2171 CN**: 以 `(__m256i) __builtin_ia32_selectb_256 ((__mmask32) __U,` 从当前函数返回。
- **L2172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) __A,`.
  **L2172 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) __A,`。
- **L2173 EN**: Executes a call or declaration centered on `statement`.
  **L2173 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2174 EN**: Closes the current lexical scope or compound statement.
  **L2174 CN**: 结束当前词法作用域或复合语句块。
- **L2175 EN**: Blank line separating nearby declarations or logic blocks.
  **L2175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2176 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2176 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2177 EN**: Continues logic associated with callable symbol `_mm_mask_set1_epi8`.
  **L2177 CN**: 继续与可调用符号 `_mm_mask_set1_epi8` 相关的逻辑。
- **L2178 EN**: Opens a new lexical scope or compound statement.
  **L2178 CN**: 打开一个新的词法作用域或复合语句块。
- **L2179 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectb_128(__M,`.
  **L2179 CN**: 以 `(__m128i) __builtin_ia32_selectb_128(__M,` 从当前函数返回。
- **L2180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_set1_epi8(__A),`.
  **L2180 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_set1_epi8(__A),`。
- **L2181 EN**: Executes a call or declaration centered on `statement`.
  **L2181 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2182 EN**: Closes the current lexical scope or compound statement.
  **L2182 CN**: 结束当前词法作用域或复合语句块。
- **L2183 EN**: Blank line separating nearby declarations or logic blocks.
  **L2183 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2184 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2184 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。

### Lines 2185-2208

````c
_mm_maskz_set1_epi8 (__mmask16 __M, char __A)
{
 return (__m128i) __builtin_ia32_selectb_128(__M,
                                             (__v16qi) _mm_set1_epi8(__A),
                                             (__v16qi) _mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_set1_epi8 (__m256i __O, __mmask32 __M, char __A)
{
  return (__m256i) __builtin_ia32_selectb_256(__M,
                                              (__v32qi) _mm256_set1_epi8(__A),
                                              (__v32qi) __O);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_set1_epi8 (__mmask32 __M, char __A)
{
  return (__m256i) __builtin_ia32_selectb_256(__M,
                                              (__v32qi) _mm256_set1_epi8(__A),
                                              (__v32qi) _mm256_setzero_si256());
}

static __inline __m128i __DEFAULT_FN_ATTRS128
````
- **L2185 EN**: Continues logic associated with callable symbol `_mm_maskz_set1_epi8`.
  **L2185 CN**: 继续与可调用符号 `_mm_maskz_set1_epi8` 相关的逻辑。
- **L2186 EN**: Opens a new lexical scope or compound statement.
  **L2186 CN**: 打开一个新的词法作用域或复合语句块。
- **L2187 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectb_128(__M,`.
  **L2187 CN**: 以 `(__m128i) __builtin_ia32_selectb_128(__M,` 从当前函数返回。
- **L2188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_set1_epi8(__A),`.
  **L2188 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_set1_epi8(__A),`。
- **L2189 EN**: Executes a call or declaration centered on `statement`.
  **L2189 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2190 EN**: Closes the current lexical scope or compound statement.
  **L2190 CN**: 结束当前词法作用域或复合语句块。
- **L2191 EN**: Blank line separating nearby declarations or logic blocks.
  **L2191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2192 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2192 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2193 EN**: Continues logic associated with callable symbol `_mm256_mask_set1_epi8`.
  **L2193 CN**: 继续与可调用符号 `_mm256_mask_set1_epi8` 相关的逻辑。
- **L2194 EN**: Opens a new lexical scope or compound statement.
  **L2194 CN**: 打开一个新的词法作用域或复合语句块。
- **L2195 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectb_256(__M,`.
  **L2195 CN**: 以 `(__m256i) __builtin_ia32_selectb_256(__M,` 从当前函数返回。
- **L2196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) _mm256_set1_epi8(__A),`.
  **L2196 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) _mm256_set1_epi8(__A),`。
- **L2197 EN**: Executes a call or declaration centered on `statement`.
  **L2197 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2198 EN**: Closes the current lexical scope or compound statement.
  **L2198 CN**: 结束当前词法作用域或复合语句块。
- **L2199 EN**: Blank line separating nearby declarations or logic blocks.
  **L2199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2200 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2200 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2201 EN**: Continues logic associated with callable symbol `_mm256_maskz_set1_epi8`.
  **L2201 CN**: 继续与可调用符号 `_mm256_maskz_set1_epi8` 相关的逻辑。
- **L2202 EN**: Opens a new lexical scope or compound statement.
  **L2202 CN**: 打开一个新的词法作用域或复合语句块。
- **L2203 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectb_256(__M,`.
  **L2203 CN**: 以 `(__m256i) __builtin_ia32_selectb_256(__M,` 从当前函数返回。
- **L2204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) _mm256_set1_epi8(__A),`.
  **L2204 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) _mm256_set1_epi8(__A),`。
- **L2205 EN**: Executes a call or declaration centered on `statement`.
  **L2205 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2206 EN**: Closes the current lexical scope or compound statement.
  **L2206 CN**: 结束当前词法作用域或复合语句块。
- **L2207 EN**: Blank line separating nearby declarations or logic blocks.
  **L2207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2208 EN**: Continues the surrounding expression or declaration: `static __inline __m128i __DEFAULT_FN_ATTRS128`.
  **L2208 CN**: 继续构造周围的表达式或声明：`static __inline __m128i __DEFAULT_FN_ATTRS128`。

### Lines 2209-2232

````c
_mm_loadu_epi16 (void const *__P)
{
  struct __loadu_epi16 {
    __m128i_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_epi16*)__P)->__v;
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_loadu_epi16 (__m128i __W, __mmask8 __U, void const *__P)
{
  return (__m128i) __builtin_ia32_loaddquhi128_mask ((const __v8hi *) __P,
                 (__v8hi) __W,
                 (__mmask8) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_loadu_epi16 (__mmask8 __U, void const *__P)
{
  return (__m128i) __builtin_ia32_loaddquhi128_mask ((const __v8hi *) __P,
                 (__v8hi)
                 _mm_setzero_si128 (),
                 (__mmask8) __U);
}
````
- **L2209 EN**: Continues logic associated with callable symbol `_mm_loadu_epi16`.
  **L2209 CN**: 继续与可调用符号 `_mm_loadu_epi16` 相关的逻辑。
- **L2210 EN**: Opens a new lexical scope or compound statement.
  **L2210 CN**: 打开一个新的词法作用域或复合语句块。
- **L2211 EN**: Declares struct `__loadu_epi16`.
  **L2211 CN**: 声明 struct `__loadu_epi16`。
- **L2212 EN**: Adds a standalone statement or declaration: `__m128i_u __v;`.
  **L2212 CN**: 添加一条独立语句或声明：`__m128i_u __v;`。
- **L2213 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L2213 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L2214 EN**: Returns from the current function with `((const struct __loadu_epi16*)__P)->__v`.
  **L2214 CN**: 以 `((const struct __loadu_epi16*)__P)->__v` 从当前函数返回。
- **L2215 EN**: Closes the current lexical scope or compound statement.
  **L2215 CN**: 结束当前词法作用域或复合语句块。
- **L2216 EN**: Blank line separating nearby declarations or logic blocks.
  **L2216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2217 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2217 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2218 EN**: Continues logic associated with callable symbol `_mm_mask_loadu_epi16`.
  **L2218 CN**: 继续与可调用符号 `_mm_mask_loadu_epi16` 相关的逻辑。
- **L2219 EN**: Opens a new lexical scope or compound statement.
  **L2219 CN**: 打开一个新的词法作用域或复合语句块。
- **L2220 EN**: Returns from the current function with `(__m128i) __builtin_ia32_loaddquhi128_mask ((const __v8hi *) __P,`.
  **L2220 CN**: 以 `(__m128i) __builtin_ia32_loaddquhi128_mask ((const __v8hi *) __P,` 从当前函数返回。
- **L2221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) __W,`.
  **L2221 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) __W,`。
- **L2222 EN**: Executes a call or declaration centered on `statement`.
  **L2222 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2223 EN**: Closes the current lexical scope or compound statement.
  **L2223 CN**: 结束当前词法作用域或复合语句块。
- **L2224 EN**: Blank line separating nearby declarations or logic blocks.
  **L2224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2225 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2225 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2226 EN**: Continues logic associated with callable symbol `_mm_maskz_loadu_epi16`.
  **L2226 CN**: 继续与可调用符号 `_mm_maskz_loadu_epi16` 相关的逻辑。
- **L2227 EN**: Opens a new lexical scope or compound statement.
  **L2227 CN**: 打开一个新的词法作用域或复合语句块。
- **L2228 EN**: Returns from the current function with `(__m128i) __builtin_ia32_loaddquhi128_mask ((const __v8hi *) __P,`.
  **L2228 CN**: 以 `(__m128i) __builtin_ia32_loaddquhi128_mask ((const __v8hi *) __P,` 从当前函数返回。
- **L2229 EN**: Continues the surrounding expression or declaration: `(__v8hi)`.
  **L2229 CN**: 继续构造周围的表达式或声明：`(__v8hi)`。
- **L2230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_setzero_si128 (),`.
  **L2230 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_setzero_si128 (),`。
- **L2231 EN**: Executes a call or declaration centered on `statement`.
  **L2231 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2232 EN**: Closes the current lexical scope or compound statement.
  **L2232 CN**: 结束当前词法作用域或复合语句块。

### Lines 2233-2256

````c

static __inline __m256i __DEFAULT_FN_ATTRS256
_mm256_loadu_epi16 (void const *__P)
{
  struct __loadu_epi16 {
    __m256i_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_epi16*)__P)->__v;
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_loadu_epi16 (__m256i __W, __mmask16 __U, void const *__P)
{
  return (__m256i) __builtin_ia32_loaddquhi256_mask ((const __v16hi *) __P,
                 (__v16hi) __W,
                 (__mmask16) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_loadu_epi16 (__mmask16 __U, void const *__P)
{
  return (__m256i) __builtin_ia32_loaddquhi256_mask ((const __v16hi *) __P,
                 (__v16hi)
                 _mm256_setzero_si256 (),
````
- **L2233 EN**: Blank line separating nearby declarations or logic blocks.
  **L2233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2234 EN**: Continues the surrounding expression or declaration: `static __inline __m256i __DEFAULT_FN_ATTRS256`.
  **L2234 CN**: 继续构造周围的表达式或声明：`static __inline __m256i __DEFAULT_FN_ATTRS256`。
- **L2235 EN**: Continues logic associated with callable symbol `_mm256_loadu_epi16`.
  **L2235 CN**: 继续与可调用符号 `_mm256_loadu_epi16` 相关的逻辑。
- **L2236 EN**: Opens a new lexical scope or compound statement.
  **L2236 CN**: 打开一个新的词法作用域或复合语句块。
- **L2237 EN**: Declares struct `__loadu_epi16`.
  **L2237 CN**: 声明 struct `__loadu_epi16`。
- **L2238 EN**: Adds a standalone statement or declaration: `__m256i_u __v;`.
  **L2238 CN**: 添加一条独立语句或声明：`__m256i_u __v;`。
- **L2239 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L2239 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L2240 EN**: Returns from the current function with `((const struct __loadu_epi16*)__P)->__v`.
  **L2240 CN**: 以 `((const struct __loadu_epi16*)__P)->__v` 从当前函数返回。
- **L2241 EN**: Closes the current lexical scope or compound statement.
  **L2241 CN**: 结束当前词法作用域或复合语句块。
- **L2242 EN**: Blank line separating nearby declarations or logic blocks.
  **L2242 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2243 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2243 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2244 EN**: Continues logic associated with callable symbol `_mm256_mask_loadu_epi16`.
  **L2244 CN**: 继续与可调用符号 `_mm256_mask_loadu_epi16` 相关的逻辑。
- **L2245 EN**: Opens a new lexical scope or compound statement.
  **L2245 CN**: 打开一个新的词法作用域或复合语句块。
- **L2246 EN**: Returns from the current function with `(__m256i) __builtin_ia32_loaddquhi256_mask ((const __v16hi *) __P,`.
  **L2246 CN**: 以 `(__m256i) __builtin_ia32_loaddquhi256_mask ((const __v16hi *) __P,` 从当前函数返回。
- **L2247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) __W,`.
  **L2247 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) __W,`。
- **L2248 EN**: Executes a call or declaration centered on `statement`.
  **L2248 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2249 EN**: Closes the current lexical scope or compound statement.
  **L2249 CN**: 结束当前词法作用域或复合语句块。
- **L2250 EN**: Blank line separating nearby declarations or logic blocks.
  **L2250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2251 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2251 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2252 EN**: Continues logic associated with callable symbol `_mm256_maskz_loadu_epi16`.
  **L2252 CN**: 继续与可调用符号 `_mm256_maskz_loadu_epi16` 相关的逻辑。
- **L2253 EN**: Opens a new lexical scope or compound statement.
  **L2253 CN**: 打开一个新的词法作用域或复合语句块。
- **L2254 EN**: Returns from the current function with `(__m256i) __builtin_ia32_loaddquhi256_mask ((const __v16hi *) __P,`.
  **L2254 CN**: 以 `(__m256i) __builtin_ia32_loaddquhi256_mask ((const __v16hi *) __P,` 从当前函数返回。
- **L2255 EN**: Continues the surrounding expression or declaration: `(__v16hi)`.
  **L2255 CN**: 继续构造周围的表达式或声明：`(__v16hi)`。
- **L2256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_setzero_si256 (),`.
  **L2256 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_setzero_si256 (),`。

### Lines 2257-2280

````c
                 (__mmask16) __U);
}

static __inline __m128i __DEFAULT_FN_ATTRS128
_mm_loadu_epi8 (void const *__P)
{
  struct __loadu_epi8 {
    __m128i_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_epi8*)__P)->__v;
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_loadu_epi8 (__m128i __W, __mmask16 __U, void const *__P)
{
  return (__m128i) __builtin_ia32_loaddquqi128_mask ((const __v16qi *) __P,
                 (__v16qi) __W,
                 (__mmask16) __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_loadu_epi8 (__mmask16 __U, void const *__P)
{
  return (__m128i) __builtin_ia32_loaddquqi128_mask ((const __v16qi *) __P,
````
- **L2257 EN**: Executes a call or declaration centered on `statement`.
  **L2257 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2258 EN**: Closes the current lexical scope or compound statement.
  **L2258 CN**: 结束当前词法作用域或复合语句块。
- **L2259 EN**: Blank line separating nearby declarations or logic blocks.
  **L2259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2260 EN**: Continues the surrounding expression or declaration: `static __inline __m128i __DEFAULT_FN_ATTRS128`.
  **L2260 CN**: 继续构造周围的表达式或声明：`static __inline __m128i __DEFAULT_FN_ATTRS128`。
- **L2261 EN**: Continues logic associated with callable symbol `_mm_loadu_epi8`.
  **L2261 CN**: 继续与可调用符号 `_mm_loadu_epi8` 相关的逻辑。
- **L2262 EN**: Opens a new lexical scope or compound statement.
  **L2262 CN**: 打开一个新的词法作用域或复合语句块。
- **L2263 EN**: Declares struct `__loadu_epi8`.
  **L2263 CN**: 声明 struct `__loadu_epi8`。
- **L2264 EN**: Adds a standalone statement or declaration: `__m128i_u __v;`.
  **L2264 CN**: 添加一条独立语句或声明：`__m128i_u __v;`。
- **L2265 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L2265 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L2266 EN**: Returns from the current function with `((const struct __loadu_epi8*)__P)->__v`.
  **L2266 CN**: 以 `((const struct __loadu_epi8*)__P)->__v` 从当前函数返回。
- **L2267 EN**: Closes the current lexical scope or compound statement.
  **L2267 CN**: 结束当前词法作用域或复合语句块。
- **L2268 EN**: Blank line separating nearby declarations or logic blocks.
  **L2268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2269 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2269 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2270 EN**: Continues logic associated with callable symbol `_mm_mask_loadu_epi8`.
  **L2270 CN**: 继续与可调用符号 `_mm_mask_loadu_epi8` 相关的逻辑。
- **L2271 EN**: Opens a new lexical scope or compound statement.
  **L2271 CN**: 打开一个新的词法作用域或复合语句块。
- **L2272 EN**: Returns from the current function with `(__m128i) __builtin_ia32_loaddquqi128_mask ((const __v16qi *) __P,`.
  **L2272 CN**: 以 `(__m128i) __builtin_ia32_loaddquqi128_mask ((const __v16qi *) __P,` 从当前函数返回。
- **L2273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __W,`.
  **L2273 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __W,`。
- **L2274 EN**: Executes a call or declaration centered on `statement`.
  **L2274 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2275 EN**: Closes the current lexical scope or compound statement.
  **L2275 CN**: 结束当前词法作用域或复合语句块。
- **L2276 EN**: Blank line separating nearby declarations or logic blocks.
  **L2276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2277 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L2277 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L2278 EN**: Continues logic associated with callable symbol `_mm_maskz_loadu_epi8`.
  **L2278 CN**: 继续与可调用符号 `_mm_maskz_loadu_epi8` 相关的逻辑。
- **L2279 EN**: Opens a new lexical scope or compound statement.
  **L2279 CN**: 打开一个新的词法作用域或复合语句块。
- **L2280 EN**: Returns from the current function with `(__m128i) __builtin_ia32_loaddquqi128_mask ((const __v16qi *) __P,`.
  **L2280 CN**: 以 `(__m128i) __builtin_ia32_loaddquqi128_mask ((const __v16qi *) __P,` 从当前函数返回。

### Lines 2281-2304

````c
                 (__v16qi)
                 _mm_setzero_si128 (),
                 (__mmask16) __U);
}

static __inline __m256i __DEFAULT_FN_ATTRS256
_mm256_loadu_epi8 (void const *__P)
{
  struct __loadu_epi8 {
    __m256i_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __loadu_epi8*)__P)->__v;
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_loadu_epi8 (__m256i __W, __mmask32 __U, void const *__P)
{
  return (__m256i) __builtin_ia32_loaddquqi256_mask ((const __v32qi *) __P,
                 (__v32qi) __W,
                 (__mmask32) __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_loadu_epi8 (__mmask32 __U, void const *__P)
````
- **L2281 EN**: Continues the surrounding expression or declaration: `(__v16qi)`.
  **L2281 CN**: 继续构造周围的表达式或声明：`(__v16qi)`。
- **L2282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_setzero_si128 (),`.
  **L2282 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_setzero_si128 (),`。
- **L2283 EN**: Executes a call or declaration centered on `statement`.
  **L2283 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2284 EN**: Closes the current lexical scope or compound statement.
  **L2284 CN**: 结束当前词法作用域或复合语句块。
- **L2285 EN**: Blank line separating nearby declarations or logic blocks.
  **L2285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2286 EN**: Continues the surrounding expression or declaration: `static __inline __m256i __DEFAULT_FN_ATTRS256`.
  **L2286 CN**: 继续构造周围的表达式或声明：`static __inline __m256i __DEFAULT_FN_ATTRS256`。
- **L2287 EN**: Continues logic associated with callable symbol `_mm256_loadu_epi8`.
  **L2287 CN**: 继续与可调用符号 `_mm256_loadu_epi8` 相关的逻辑。
- **L2288 EN**: Opens a new lexical scope or compound statement.
  **L2288 CN**: 打开一个新的词法作用域或复合语句块。
- **L2289 EN**: Declares struct `__loadu_epi8`.
  **L2289 CN**: 声明 struct `__loadu_epi8`。
- **L2290 EN**: Adds a standalone statement or declaration: `__m256i_u __v;`.
  **L2290 CN**: 添加一条独立语句或声明：`__m256i_u __v;`。
- **L2291 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L2291 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L2292 EN**: Returns from the current function with `((const struct __loadu_epi8*)__P)->__v`.
  **L2292 CN**: 以 `((const struct __loadu_epi8*)__P)->__v` 从当前函数返回。
- **L2293 EN**: Closes the current lexical scope or compound statement.
  **L2293 CN**: 结束当前词法作用域或复合语句块。
- **L2294 EN**: Blank line separating nearby declarations or logic blocks.
  **L2294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2295 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2295 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2296 EN**: Continues logic associated with callable symbol `_mm256_mask_loadu_epi8`.
  **L2296 CN**: 继续与可调用符号 `_mm256_mask_loadu_epi8` 相关的逻辑。
- **L2297 EN**: Opens a new lexical scope or compound statement.
  **L2297 CN**: 打开一个新的词法作用域或复合语句块。
- **L2298 EN**: Returns from the current function with `(__m256i) __builtin_ia32_loaddquqi256_mask ((const __v32qi *) __P,`.
  **L2298 CN**: 以 `(__m256i) __builtin_ia32_loaddquqi256_mask ((const __v32qi *) __P,` 从当前函数返回。
- **L2299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) __W,`.
  **L2299 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) __W,`。
- **L2300 EN**: Executes a call or declaration centered on `statement`.
  **L2300 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2301 EN**: Closes the current lexical scope or compound statement.
  **L2301 CN**: 结束当前词法作用域或复合语句块。
- **L2302 EN**: Blank line separating nearby declarations or logic blocks.
  **L2302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2303 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L2303 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L2304 EN**: Continues logic associated with callable symbol `_mm256_maskz_loadu_epi8`.
  **L2304 CN**: 继续与可调用符号 `_mm256_maskz_loadu_epi8` 相关的逻辑。

### Lines 2305-2328

````c
{
  return (__m256i) __builtin_ia32_loaddquqi256_mask ((const __v32qi *) __P,
                 (__v32qi)
                 _mm256_setzero_si256 (),
                 (__mmask32) __U);
}

static __inline void __DEFAULT_FN_ATTRS128
_mm_storeu_epi16 (void *__P, __m128i __A)
{
  struct __storeu_epi16 {
    __m128i_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_epi16*)__P)->__v = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS128
_mm_mask_storeu_epi16 (void *__P, __mmask8 __U, __m128i __A)
{
  __builtin_ia32_storedquhi128_mask ((__v8hi *) __P,
             (__v8hi) __A,
             (__mmask8) __U);
}

````
- **L2305 EN**: Opens a new lexical scope or compound statement.
  **L2305 CN**: 打开一个新的词法作用域或复合语句块。
- **L2306 EN**: Returns from the current function with `(__m256i) __builtin_ia32_loaddquqi256_mask ((const __v32qi *) __P,`.
  **L2306 CN**: 以 `(__m256i) __builtin_ia32_loaddquqi256_mask ((const __v32qi *) __P,` 从当前函数返回。
- **L2307 EN**: Continues the surrounding expression or declaration: `(__v32qi)`.
  **L2307 CN**: 继续构造周围的表达式或声明：`(__v32qi)`。
- **L2308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_setzero_si256 (),`.
  **L2308 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_setzero_si256 (),`。
- **L2309 EN**: Executes a call or declaration centered on `statement`.
  **L2309 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2310 EN**: Closes the current lexical scope or compound statement.
  **L2310 CN**: 结束当前词法作用域或复合语句块。
- **L2311 EN**: Blank line separating nearby declarations or logic blocks.
  **L2311 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2312 EN**: Continues the surrounding expression or declaration: `static __inline void __DEFAULT_FN_ATTRS128`.
  **L2312 CN**: 继续构造周围的表达式或声明：`static __inline void __DEFAULT_FN_ATTRS128`。
- **L2313 EN**: Continues logic associated with callable symbol `_mm_storeu_epi16`.
  **L2313 CN**: 继续与可调用符号 `_mm_storeu_epi16` 相关的逻辑。
- **L2314 EN**: Opens a new lexical scope or compound statement.
  **L2314 CN**: 打开一个新的词法作用域或复合语句块。
- **L2315 EN**: Declares struct `__storeu_epi16`.
  **L2315 CN**: 声明 struct `__storeu_epi16`。
- **L2316 EN**: Adds a standalone statement or declaration: `__m128i_u __v;`.
  **L2316 CN**: 添加一条独立语句或声明：`__m128i_u __v;`。
- **L2317 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L2317 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L2318 EN**: Executes a call or declaration centered on `statement`.
  **L2318 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2319 EN**: Closes the current lexical scope or compound statement.
  **L2319 CN**: 结束当前词法作用域或复合语句块。
- **L2320 EN**: Blank line separating nearby declarations or logic blocks.
  **L2320 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2321 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS128`.
  **L2321 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS128`。
- **L2322 EN**: Continues logic associated with callable symbol `_mm_mask_storeu_epi16`.
  **L2322 CN**: 继续与可调用符号 `_mm_mask_storeu_epi16` 相关的逻辑。
- **L2323 EN**: Opens a new lexical scope or compound statement.
  **L2323 CN**: 打开一个新的词法作用域或复合语句块。
- **L2324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_storedquhi128_mask ((__v8hi *) __P,`.
  **L2324 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_storedquhi128_mask ((__v8hi *) __P,`。
- **L2325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) __A,`.
  **L2325 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) __A,`。
- **L2326 EN**: Executes a call or declaration centered on `statement`.
  **L2326 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2327 EN**: Closes the current lexical scope or compound statement.
  **L2327 CN**: 结束当前词法作用域或复合语句块。
- **L2328 EN**: Blank line separating nearby declarations or logic blocks.
  **L2328 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2329-2352

````c
static __inline void __DEFAULT_FN_ATTRS256
_mm256_storeu_epi16 (void *__P, __m256i __A)
{
  struct __storeu_epi16 {
    __m256i_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_epi16*)__P)->__v = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS256
_mm256_mask_storeu_epi16 (void *__P, __mmask16 __U, __m256i __A)
{
  __builtin_ia32_storedquhi256_mask ((__v16hi *) __P,
             (__v16hi) __A,
             (__mmask16) __U);
}

static __inline void __DEFAULT_FN_ATTRS128
_mm_storeu_epi8 (void *__P, __m128i __A)
{
  struct __storeu_epi8 {
    __m128i_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_epi8*)__P)->__v = __A;
````
- **L2329 EN**: Continues the surrounding expression or declaration: `static __inline void __DEFAULT_FN_ATTRS256`.
  **L2329 CN**: 继续构造周围的表达式或声明：`static __inline void __DEFAULT_FN_ATTRS256`。
- **L2330 EN**: Continues logic associated with callable symbol `_mm256_storeu_epi16`.
  **L2330 CN**: 继续与可调用符号 `_mm256_storeu_epi16` 相关的逻辑。
- **L2331 EN**: Opens a new lexical scope or compound statement.
  **L2331 CN**: 打开一个新的词法作用域或复合语句块。
- **L2332 EN**: Declares struct `__storeu_epi16`.
  **L2332 CN**: 声明 struct `__storeu_epi16`。
- **L2333 EN**: Adds a standalone statement or declaration: `__m256i_u __v;`.
  **L2333 CN**: 添加一条独立语句或声明：`__m256i_u __v;`。
- **L2334 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L2334 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L2335 EN**: Executes a call or declaration centered on `statement`.
  **L2335 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2336 EN**: Closes the current lexical scope or compound statement.
  **L2336 CN**: 结束当前词法作用域或复合语句块。
- **L2337 EN**: Blank line separating nearby declarations or logic blocks.
  **L2337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2338 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS256`.
  **L2338 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS256`。
- **L2339 EN**: Continues logic associated with callable symbol `_mm256_mask_storeu_epi16`.
  **L2339 CN**: 继续与可调用符号 `_mm256_mask_storeu_epi16` 相关的逻辑。
- **L2340 EN**: Opens a new lexical scope or compound statement.
  **L2340 CN**: 打开一个新的词法作用域或复合语句块。
- **L2341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_storedquhi256_mask ((__v16hi *) __P,`.
  **L2341 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_storedquhi256_mask ((__v16hi *) __P,`。
- **L2342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) __A,`.
  **L2342 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) __A,`。
- **L2343 EN**: Executes a call or declaration centered on `statement`.
  **L2343 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2344 EN**: Closes the current lexical scope or compound statement.
  **L2344 CN**: 结束当前词法作用域或复合语句块。
- **L2345 EN**: Blank line separating nearby declarations or logic blocks.
  **L2345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2346 EN**: Continues the surrounding expression or declaration: `static __inline void __DEFAULT_FN_ATTRS128`.
  **L2346 CN**: 继续构造周围的表达式或声明：`static __inline void __DEFAULT_FN_ATTRS128`。
- **L2347 EN**: Continues logic associated with callable symbol `_mm_storeu_epi8`.
  **L2347 CN**: 继续与可调用符号 `_mm_storeu_epi8` 相关的逻辑。
- **L2348 EN**: Opens a new lexical scope or compound statement.
  **L2348 CN**: 打开一个新的词法作用域或复合语句块。
- **L2349 EN**: Declares struct `__storeu_epi8`.
  **L2349 CN**: 声明 struct `__storeu_epi8`。
- **L2350 EN**: Adds a standalone statement or declaration: `__m128i_u __v;`.
  **L2350 CN**: 添加一条独立语句或声明：`__m128i_u __v;`。
- **L2351 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L2351 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L2352 EN**: Executes a call or declaration centered on `statement`.
  **L2352 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 2353-2376

````c
}

static __inline__ void __DEFAULT_FN_ATTRS128
_mm_mask_storeu_epi8 (void *__P, __mmask16 __U, __m128i __A)
{
  __builtin_ia32_storedquqi128_mask ((__v16qi *) __P,
             (__v16qi) __A,
             (__mmask16) __U);
}

static __inline void __DEFAULT_FN_ATTRS256
_mm256_storeu_epi8 (void *__P, __m256i __A)
{
  struct __storeu_epi8 {
    __m256i_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __storeu_epi8*)__P)->__v = __A;
}

static __inline__ void __DEFAULT_FN_ATTRS256
_mm256_mask_storeu_epi8 (void *__P, __mmask32 __U, __m256i __A)
{
  __builtin_ia32_storedquqi256_mask ((__v32qi *) __P,
             (__v32qi) __A,
````
- **L2353 EN**: Closes the current lexical scope or compound statement.
  **L2353 CN**: 结束当前词法作用域或复合语句块。
- **L2354 EN**: Blank line separating nearby declarations or logic blocks.
  **L2354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2355 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS128`.
  **L2355 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS128`。
- **L2356 EN**: Continues logic associated with callable symbol `_mm_mask_storeu_epi8`.
  **L2356 CN**: 继续与可调用符号 `_mm_mask_storeu_epi8` 相关的逻辑。
- **L2357 EN**: Opens a new lexical scope or compound statement.
  **L2357 CN**: 打开一个新的词法作用域或复合语句块。
- **L2358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_storedquqi128_mask ((__v16qi *) __P,`.
  **L2358 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_storedquqi128_mask ((__v16qi *) __P,`。
- **L2359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __A,`.
  **L2359 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __A,`。
- **L2360 EN**: Executes a call or declaration centered on `statement`.
  **L2360 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2361 EN**: Closes the current lexical scope or compound statement.
  **L2361 CN**: 结束当前词法作用域或复合语句块。
- **L2362 EN**: Blank line separating nearby declarations or logic blocks.
  **L2362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2363 EN**: Continues the surrounding expression or declaration: `static __inline void __DEFAULT_FN_ATTRS256`.
  **L2363 CN**: 继续构造周围的表达式或声明：`static __inline void __DEFAULT_FN_ATTRS256`。
- **L2364 EN**: Continues logic associated with callable symbol `_mm256_storeu_epi8`.
  **L2364 CN**: 继续与可调用符号 `_mm256_storeu_epi8` 相关的逻辑。
- **L2365 EN**: Opens a new lexical scope or compound statement.
  **L2365 CN**: 打开一个新的词法作用域或复合语句块。
- **L2366 EN**: Declares struct `__storeu_epi8`.
  **L2366 CN**: 声明 struct `__storeu_epi8`。
- **L2367 EN**: Adds a standalone statement or declaration: `__m256i_u __v;`.
  **L2367 CN**: 添加一条独立语句或声明：`__m256i_u __v;`。
- **L2368 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L2368 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L2369 EN**: Executes a call or declaration centered on `statement`.
  **L2369 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2370 EN**: Closes the current lexical scope or compound statement.
  **L2370 CN**: 结束当前词法作用域或复合语句块。
- **L2371 EN**: Blank line separating nearby declarations or logic blocks.
  **L2371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2372 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS256`.
  **L2372 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS256`。
- **L2373 EN**: Continues logic associated with callable symbol `_mm256_mask_storeu_epi8`.
  **L2373 CN**: 继续与可调用符号 `_mm256_mask_storeu_epi8` 相关的逻辑。
- **L2374 EN**: Opens a new lexical scope or compound statement.
  **L2374 CN**: 打开一个新的词法作用域或复合语句块。
- **L2375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_storedquqi256_mask ((__v32qi *) __P,`.
  **L2375 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_storedquqi256_mask ((__v32qi *) __P,`。
- **L2376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) __A,`.
  **L2376 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) __A,`。

### Lines 2377-2400

````c
             (__mmask32) __U);
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_test_epi8_mask(__m128i __A, __m128i __B) {
  return _mm_cmpneq_epi8_mask (_mm_and_si128(__A, __B), _mm_setzero_si128());
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_test_epi8_mask(__mmask16 __U, __m128i __A, __m128i __B) {
  return _mm_mask_cmpneq_epi8_mask (__U, _mm_and_si128 (__A, __B),
                                    _mm_setzero_si128());
}

static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_test_epi8_mask(__m256i __A, __m256i __B) {
  return _mm256_cmpneq_epi8_mask (_mm256_and_si256(__A, __B),
                                  _mm256_setzero_si256());
}

static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_test_epi8_mask(__mmask32 __U, __m256i __A, __m256i __B) {
  return _mm256_mask_cmpneq_epi8_mask (__U, _mm256_and_si256(__A, __B),
                                       _mm256_setzero_si256());
````
- **L2377 EN**: Executes a call or declaration centered on `statement`.
  **L2377 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2378 EN**: Closes the current lexical scope or compound statement.
  **L2378 CN**: 结束当前词法作用域或复合语句块。
- **L2379 EN**: Blank line separating nearby declarations or logic blocks.
  **L2379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2380 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2380 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2381 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_test_epi8_mask(__m128i __A, __m128i __B) {`.
  **L2381 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_test_epi8_mask(__m128i __A, __m128i __B) {`。
- **L2382 EN**: Returns from the current function with `_mm_cmpneq_epi8_mask (_mm_and_si128(__A, __B), _mm_setzero_si128())`.
  **L2382 CN**: 以 `_mm_cmpneq_epi8_mask (_mm_and_si128(__A, __B), _mm_setzero_si128())` 从当前函数返回。
- **L2383 EN**: Closes the current lexical scope or compound statement.
  **L2383 CN**: 结束当前词法作用域或复合语句块。
- **L2384 EN**: Blank line separating nearby declarations or logic blocks.
  **L2384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2385 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2385 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2386 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_test_epi8_mask(__mmask16 __U, __m128i __A, __m128i __B) {`.
  **L2386 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_test_epi8_mask(__mmask16 __U, __m128i __A, __m128i __B) {`。
- **L2387 EN**: Returns from the current function with `_mm_mask_cmpneq_epi8_mask (__U, _mm_and_si128 (__A, __B),`.
  **L2387 CN**: 以 `_mm_mask_cmpneq_epi8_mask (__U, _mm_and_si128 (__A, __B),` 从当前函数返回。
- **L2388 EN**: Executes a call or declaration centered on `_mm_setzero_si128`.
  **L2388 CN**: 执行以 `_mm_setzero_si128` 为核心的调用或声明。
- **L2389 EN**: Closes the current lexical scope or compound statement.
  **L2389 CN**: 结束当前词法作用域或复合语句块。
- **L2390 EN**: Blank line separating nearby declarations or logic blocks.
  **L2390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2391 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2391 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2392 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_test_epi8_mask(__m256i __A, __m256i __B) {`.
  **L2392 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_test_epi8_mask(__m256i __A, __m256i __B) {`。
- **L2393 EN**: Returns from the current function with `_mm256_cmpneq_epi8_mask (_mm256_and_si256(__A, __B),`.
  **L2393 CN**: 以 `_mm256_cmpneq_epi8_mask (_mm256_and_si256(__A, __B),` 从当前函数返回。
- **L2394 EN**: Executes a call or declaration centered on `_mm256_setzero_si256`.
  **L2394 CN**: 执行以 `_mm256_setzero_si256` 为核心的调用或声明。
- **L2395 EN**: Closes the current lexical scope or compound statement.
  **L2395 CN**: 结束当前词法作用域或复合语句块。
- **L2396 EN**: Blank line separating nearby declarations or logic blocks.
  **L2396 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2397 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2397 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2398 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_test_epi8_mask(__mmask32 __U, __m256i __A, __m256i __B) {`.
  **L2398 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_test_epi8_mask(__mmask32 __U, __m256i __A, __m256i __B) {`。
- **L2399 EN**: Returns from the current function with `_mm256_mask_cmpneq_epi8_mask (__U, _mm256_and_si256(__A, __B),`.
  **L2399 CN**: 以 `_mm256_mask_cmpneq_epi8_mask (__U, _mm256_and_si256(__A, __B),` 从当前函数返回。
- **L2400 EN**: Executes a call or declaration centered on `_mm256_setzero_si256`.
  **L2400 CN**: 执行以 `_mm256_setzero_si256` 为核心的调用或声明。

### Lines 2401-2424

````c
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_test_epi16_mask(__m128i __A, __m128i __B) {
  return _mm_cmpneq_epi16_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128());
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_test_epi16_mask(__mmask8 __U, __m128i __A, __m128i __B) {
  return _mm_mask_cmpneq_epi16_mask (__U, _mm_and_si128 (__A, __B),
                                     _mm_setzero_si128());
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_test_epi16_mask(__m256i __A, __m256i __B) {
  return _mm256_cmpneq_epi16_mask (_mm256_and_si256 (__A, __B),
                                   _mm256_setzero_si256 ());
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_test_epi16_mask(__mmask16 __U, __m256i __A, __m256i __B) {
  return _mm256_mask_cmpneq_epi16_mask (__U, _mm256_and_si256(__A, __B),
                                        _mm256_setzero_si256());
}
````
- **L2401 EN**: Closes the current lexical scope or compound statement.
  **L2401 CN**: 结束当前词法作用域或复合语句块。
- **L2402 EN**: Blank line separating nearby declarations or logic blocks.
  **L2402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2403 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2403 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2404 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_test_epi16_mask(__m128i __A, __m128i __B) {`.
  **L2404 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_test_epi16_mask(__m128i __A, __m128i __B) {`。
- **L2405 EN**: Returns from the current function with `_mm_cmpneq_epi16_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128())`.
  **L2405 CN**: 以 `_mm_cmpneq_epi16_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128())` 从当前函数返回。
- **L2406 EN**: Closes the current lexical scope or compound statement.
  **L2406 CN**: 结束当前词法作用域或复合语句块。
- **L2407 EN**: Blank line separating nearby declarations or logic blocks.
  **L2407 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2408 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2408 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2409 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_test_epi16_mask(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L2409 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_test_epi16_mask(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L2410 EN**: Returns from the current function with `_mm_mask_cmpneq_epi16_mask (__U, _mm_and_si128 (__A, __B),`.
  **L2410 CN**: 以 `_mm_mask_cmpneq_epi16_mask (__U, _mm_and_si128 (__A, __B),` 从当前函数返回。
- **L2411 EN**: Executes a call or declaration centered on `_mm_setzero_si128`.
  **L2411 CN**: 执行以 `_mm_setzero_si128` 为核心的调用或声明。
- **L2412 EN**: Closes the current lexical scope or compound statement.
  **L2412 CN**: 结束当前词法作用域或复合语句块。
- **L2413 EN**: Blank line separating nearby declarations or logic blocks.
  **L2413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2414 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2414 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2415 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_test_epi16_mask(__m256i __A, __m256i __B) {`.
  **L2415 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_test_epi16_mask(__m256i __A, __m256i __B) {`。
- **L2416 EN**: Returns from the current function with `_mm256_cmpneq_epi16_mask (_mm256_and_si256 (__A, __B),`.
  **L2416 CN**: 以 `_mm256_cmpneq_epi16_mask (_mm256_and_si256 (__A, __B),` 从当前函数返回。
- **L2417 EN**: Executes a call or declaration centered on `_mm256_setzero_si256`.
  **L2417 CN**: 执行以 `_mm256_setzero_si256` 为核心的调用或声明。
- **L2418 EN**: Closes the current lexical scope or compound statement.
  **L2418 CN**: 结束当前词法作用域或复合语句块。
- **L2419 EN**: Blank line separating nearby declarations or logic blocks.
  **L2419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2420 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2420 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2421 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_test_epi16_mask(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L2421 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_test_epi16_mask(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L2422 EN**: Returns from the current function with `_mm256_mask_cmpneq_epi16_mask (__U, _mm256_and_si256(__A, __B),`.
  **L2422 CN**: 以 `_mm256_mask_cmpneq_epi16_mask (__U, _mm256_and_si256(__A, __B),` 从当前函数返回。
- **L2423 EN**: Executes a call or declaration centered on `_mm256_setzero_si256`.
  **L2423 CN**: 执行以 `_mm256_setzero_si256` 为核心的调用或声明。
- **L2424 EN**: Closes the current lexical scope or compound statement.
  **L2424 CN**: 结束当前词法作用域或复合语句块。

### Lines 2425-2448

````c

static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_testn_epi8_mask(__m128i __A, __m128i __B) {
  return _mm_cmpeq_epi8_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128());
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_testn_epi8_mask(__mmask16 __U, __m128i __A, __m128i __B) {
  return _mm_mask_cmpeq_epi8_mask (__U, _mm_and_si128 (__A, __B),
                                  _mm_setzero_si128());
}

static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_testn_epi8_mask(__m256i __A, __m256i __B) {
  return _mm256_cmpeq_epi8_mask (_mm256_and_si256 (__A, __B),
                                 _mm256_setzero_si256());
}

static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_testn_epi8_mask(__mmask32 __U, __m256i __A, __m256i __B) {
  return _mm256_mask_cmpeq_epi8_mask (__U, _mm256_and_si256 (__A, __B),
                                      _mm256_setzero_si256());
}

````
- **L2425 EN**: Blank line separating nearby declarations or logic blocks.
  **L2425 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2426 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2426 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2427 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_testn_epi8_mask(__m128i __A, __m128i __B) {`.
  **L2427 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_testn_epi8_mask(__m128i __A, __m128i __B) {`。
- **L2428 EN**: Returns from the current function with `_mm_cmpeq_epi8_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128())`.
  **L2428 CN**: 以 `_mm_cmpeq_epi8_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128())` 从当前函数返回。
- **L2429 EN**: Closes the current lexical scope or compound statement.
  **L2429 CN**: 结束当前词法作用域或复合语句块。
- **L2430 EN**: Blank line separating nearby declarations or logic blocks.
  **L2430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2431 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2431 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2432 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_testn_epi8_mask(__mmask16 __U, __m128i __A, __m128i __B) {`.
  **L2432 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_testn_epi8_mask(__mmask16 __U, __m128i __A, __m128i __B) {`。
- **L2433 EN**: Returns from the current function with `_mm_mask_cmpeq_epi8_mask (__U, _mm_and_si128 (__A, __B),`.
  **L2433 CN**: 以 `_mm_mask_cmpeq_epi8_mask (__U, _mm_and_si128 (__A, __B),` 从当前函数返回。
- **L2434 EN**: Executes a call or declaration centered on `_mm_setzero_si128`.
  **L2434 CN**: 执行以 `_mm_setzero_si128` 为核心的调用或声明。
- **L2435 EN**: Closes the current lexical scope or compound statement.
  **L2435 CN**: 结束当前词法作用域或复合语句块。
- **L2436 EN**: Blank line separating nearby declarations or logic blocks.
  **L2436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2437 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2437 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2438 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_testn_epi8_mask(__m256i __A, __m256i __B) {`.
  **L2438 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_testn_epi8_mask(__m256i __A, __m256i __B) {`。
- **L2439 EN**: Returns from the current function with `_mm256_cmpeq_epi8_mask (_mm256_and_si256 (__A, __B),`.
  **L2439 CN**: 以 `_mm256_cmpeq_epi8_mask (_mm256_and_si256 (__A, __B),` 从当前函数返回。
- **L2440 EN**: Executes a call or declaration centered on `_mm256_setzero_si256`.
  **L2440 CN**: 执行以 `_mm256_setzero_si256` 为核心的调用或声明。
- **L2441 EN**: Closes the current lexical scope or compound statement.
  **L2441 CN**: 结束当前词法作用域或复合语句块。
- **L2442 EN**: Blank line separating nearby declarations or logic blocks.
  **L2442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2443 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2443 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2444 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_testn_epi8_mask(__mmask32 __U, __m256i __A, __m256i __B) {`.
  **L2444 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_testn_epi8_mask(__mmask32 __U, __m256i __A, __m256i __B) {`。
- **L2445 EN**: Returns from the current function with `_mm256_mask_cmpeq_epi8_mask (__U, _mm256_and_si256 (__A, __B),`.
  **L2445 CN**: 以 `_mm256_mask_cmpeq_epi8_mask (__U, _mm256_and_si256 (__A, __B),` 从当前函数返回。
- **L2446 EN**: Executes a call or declaration centered on `_mm256_setzero_si256`.
  **L2446 CN**: 执行以 `_mm256_setzero_si256` 为核心的调用或声明。
- **L2447 EN**: Closes the current lexical scope or compound statement.
  **L2447 CN**: 结束当前词法作用域或复合语句块。
- **L2448 EN**: Blank line separating nearby declarations or logic blocks.
  **L2448 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2449-2472

````c
static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_testn_epi16_mask(__m128i __A, __m128i __B) {
  return _mm_cmpeq_epi16_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128());
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_testn_epi16_mask(__mmask8 __U, __m128i __A, __m128i __B) {
  return _mm_mask_cmpeq_epi16_mask (__U, _mm_and_si128(__A, __B), _mm_setzero_si128());
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_testn_epi16_mask(__m256i __A, __m256i __B) {
  return _mm256_cmpeq_epi16_mask (_mm256_and_si256(__A, __B),
                                  _mm256_setzero_si256());
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_testn_epi16_mask(__mmask16 __U, __m256i __A, __m256i __B) {
  return _mm256_mask_cmpeq_epi16_mask (__U, _mm256_and_si256 (__A, __B),
                                       _mm256_setzero_si256());
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_movepi8_mask(__m128i __A) {
````
- **L2449 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2449 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2450 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_testn_epi16_mask(__m128i __A, __m128i __B) {`.
  **L2450 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_testn_epi16_mask(__m128i __A, __m128i __B) {`。
- **L2451 EN**: Returns from the current function with `_mm_cmpeq_epi16_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128())`.
  **L2451 CN**: 以 `_mm_cmpeq_epi16_mask (_mm_and_si128 (__A, __B), _mm_setzero_si128())` 从当前函数返回。
- **L2452 EN**: Closes the current lexical scope or compound statement.
  **L2452 CN**: 结束当前词法作用域或复合语句块。
- **L2453 EN**: Blank line separating nearby declarations or logic blocks.
  **L2453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2454 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2454 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2455 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_testn_epi16_mask(__mmask8 __U, __m128i __A, __m128i __B) {`.
  **L2455 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_testn_epi16_mask(__mmask8 __U, __m128i __A, __m128i __B) {`。
- **L2456 EN**: Returns from the current function with `_mm_mask_cmpeq_epi16_mask (__U, _mm_and_si128(__A, __B), _mm_setzero_si128())`.
  **L2456 CN**: 以 `_mm_mask_cmpeq_epi16_mask (__U, _mm_and_si128(__A, __B), _mm_setzero_si128())` 从当前函数返回。
- **L2457 EN**: Closes the current lexical scope or compound statement.
  **L2457 CN**: 结束当前词法作用域或复合语句块。
- **L2458 EN**: Blank line separating nearby declarations or logic blocks.
  **L2458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2459 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2459 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2460 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_testn_epi16_mask(__m256i __A, __m256i __B) {`.
  **L2460 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_testn_epi16_mask(__m256i __A, __m256i __B) {`。
- **L2461 EN**: Returns from the current function with `_mm256_cmpeq_epi16_mask (_mm256_and_si256(__A, __B),`.
  **L2461 CN**: 以 `_mm256_cmpeq_epi16_mask (_mm256_and_si256(__A, __B),` 从当前函数返回。
- **L2462 EN**: Executes a call or declaration centered on `_mm256_setzero_si256`.
  **L2462 CN**: 执行以 `_mm256_setzero_si256` 为核心的调用或声明。
- **L2463 EN**: Closes the current lexical scope or compound statement.
  **L2463 CN**: 结束当前词法作用域或复合语句块。
- **L2464 EN**: Blank line separating nearby declarations or logic blocks.
  **L2464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2465 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2465 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2466 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_testn_epi16_mask(__mmask16 __U, __m256i __A, __m256i __B) {`.
  **L2466 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_testn_epi16_mask(__mmask16 __U, __m256i __A, __m256i __B) {`。
- **L2467 EN**: Returns from the current function with `_mm256_mask_cmpeq_epi16_mask (__U, _mm256_and_si256 (__A, __B),`.
  **L2467 CN**: 以 `_mm256_mask_cmpeq_epi16_mask (__U, _mm256_and_si256 (__A, __B),` 从当前函数返回。
- **L2468 EN**: Executes a call or declaration centered on `_mm256_setzero_si256`.
  **L2468 CN**: 执行以 `_mm256_setzero_si256` 为核心的调用或声明。
- **L2469 EN**: Closes the current lexical scope or compound statement.
  **L2469 CN**: 结束当前词法作用域或复合语句块。
- **L2470 EN**: Blank line separating nearby declarations or logic blocks.
  **L2470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2471 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2471 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2472 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movepi8_mask(__m128i __A) {`.
  **L2472 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movepi8_mask(__m128i __A) {`。

### Lines 2473-2496

````c
  return (__mmask16) __builtin_ia32_cvtb2mask128 ((__v16qi) __A);
}

static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_movepi8_mask(__m256i __A) {
  return (__mmask32) __builtin_ia32_cvtb2mask256 ((__v32qi) __A);
}

static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_movepi16_mask(__m128i __A) {
  return (__mmask8) __builtin_ia32_cvtw2mask128 ((__v8hi) __A);
}

static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_movepi16_mask(__m256i __A) {
  return (__mmask16) __builtin_ia32_cvtw2mask256 ((__v16hi) __A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_movm_epi8(__mmask16 __A) {
  return (__m128i) __builtin_ia32_cvtmask2b128 (__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L2473 EN**: Returns from the current function with `(__mmask16) __builtin_ia32_cvtb2mask128 ((__v16qi) __A)`.
  **L2473 CN**: 以 `(__mmask16) __builtin_ia32_cvtb2mask128 ((__v16qi) __A)` 从当前函数返回。
- **L2474 EN**: Closes the current lexical scope or compound statement.
  **L2474 CN**: 结束当前词法作用域或复合语句块。
- **L2475 EN**: Blank line separating nearby declarations or logic blocks.
  **L2475 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2476 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2476 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask32 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2477 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_movepi8_mask(__m256i __A) {`.
  **L2477 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_movepi8_mask(__m256i __A) {`。
- **L2478 EN**: Returns from the current function with `(__mmask32) __builtin_ia32_cvtb2mask256 ((__v32qi) __A)`.
  **L2478 CN**: 以 `(__mmask32) __builtin_ia32_cvtb2mask256 ((__v32qi) __A)` 从当前函数返回。
- **L2479 EN**: Closes the current lexical scope or compound statement.
  **L2479 CN**: 结束当前词法作用域或复合语句块。
- **L2480 EN**: Blank line separating nearby declarations or logic blocks.
  **L2480 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2481 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2481 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask8 __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2482 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movepi16_mask(__m128i __A) {`.
  **L2482 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movepi16_mask(__m128i __A) {`。
- **L2483 EN**: Returns from the current function with `(__mmask8) __builtin_ia32_cvtw2mask128 ((__v8hi) __A)`.
  **L2483 CN**: 以 `(__mmask8) __builtin_ia32_cvtw2mask128 ((__v8hi) __A)` 从当前函数返回。
- **L2484 EN**: Closes the current lexical scope or compound statement.
  **L2484 CN**: 结束当前词法作用域或复合语句块。
- **L2485 EN**: Blank line separating nearby declarations or logic blocks.
  **L2485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2486 EN**: Continues the surrounding expression or declaration: `static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2486 CN**: 继续构造周围的表达式或声明：`static __inline__ __mmask16 __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2487 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_movepi16_mask(__m256i __A) {`.
  **L2487 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_movepi16_mask(__m256i __A) {`。
- **L2488 EN**: Returns from the current function with `(__mmask16) __builtin_ia32_cvtw2mask256 ((__v16hi) __A)`.
  **L2488 CN**: 以 `(__mmask16) __builtin_ia32_cvtw2mask256 ((__v16hi) __A)` 从当前函数返回。
- **L2489 EN**: Closes the current lexical scope or compound statement.
  **L2489 CN**: 结束当前词法作用域或复合语句块。
- **L2490 EN**: Blank line separating nearby declarations or logic blocks.
  **L2490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2491 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2491 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2492 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movm_epi8(__mmask16 __A) {`.
  **L2492 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movm_epi8(__mmask16 __A) {`。
- **L2493 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtmask2b128 (__A)`.
  **L2493 CN**: 以 `(__m128i) __builtin_ia32_cvtmask2b128 (__A)` 从当前函数返回。
- **L2494 EN**: Closes the current lexical scope or compound statement.
  **L2494 CN**: 结束当前词法作用域或复合语句块。
- **L2495 EN**: Blank line separating nearby declarations or logic blocks.
  **L2495 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2496 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2496 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 2497-2520

````c
_mm256_movm_epi8(__mmask32 __A) {
  return (__m256i) __builtin_ia32_cvtmask2b256 (__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_movm_epi16(__mmask8 __A) {
  return (__m128i) __builtin_ia32_cvtmask2w128 (__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_movm_epi16(__mmask16 __A) {
  return (__m256i) __builtin_ia32_cvtmask2w256 (__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_broadcastb_epi8 (__m128i __O, __mmask16 __M, __m128i __A)
{
  return (__m128i)__builtin_ia32_selectb_128(__M,
                                             (__v16qi) _mm_broadcastb_epi8(__A),
                                             (__v16qi) __O);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_broadcastb_epi8 (__mmask16 __M, __m128i __A)
````
- **L2497 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_movm_epi8(__mmask32 __A) {`.
  **L2497 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_movm_epi8(__mmask32 __A) {`。
- **L2498 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtmask2b256 (__A)`.
  **L2498 CN**: 以 `(__m256i) __builtin_ia32_cvtmask2b256 (__A)` 从当前函数返回。
- **L2499 EN**: Closes the current lexical scope or compound statement.
  **L2499 CN**: 结束当前词法作用域或复合语句块。
- **L2500 EN**: Blank line separating nearby declarations or logic blocks.
  **L2500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2501 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2501 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2502 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_movm_epi16(__mmask8 __A) {`.
  **L2502 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_movm_epi16(__mmask8 __A) {`。
- **L2503 EN**: Returns from the current function with `(__m128i) __builtin_ia32_cvtmask2w128 (__A)`.
  **L2503 CN**: 以 `(__m128i) __builtin_ia32_cvtmask2w128 (__A)` 从当前函数返回。
- **L2504 EN**: Closes the current lexical scope or compound statement.
  **L2504 CN**: 结束当前词法作用域或复合语句块。
- **L2505 EN**: Blank line separating nearby declarations or logic blocks.
  **L2505 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2506 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2506 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2507 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_movm_epi16(__mmask16 __A) {`.
  **L2507 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_movm_epi16(__mmask16 __A) {`。
- **L2508 EN**: Returns from the current function with `(__m256i) __builtin_ia32_cvtmask2w256 (__A)`.
  **L2508 CN**: 以 `(__m256i) __builtin_ia32_cvtmask2w256 (__A)` 从当前函数返回。
- **L2509 EN**: Closes the current lexical scope or compound statement.
  **L2509 CN**: 结束当前词法作用域或复合语句块。
- **L2510 EN**: Blank line separating nearby declarations or logic blocks.
  **L2510 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2511 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2511 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2512 EN**: Continues logic associated with callable symbol `_mm_mask_broadcastb_epi8`.
  **L2512 CN**: 继续与可调用符号 `_mm_mask_broadcastb_epi8` 相关的逻辑。
- **L2513 EN**: Opens a new lexical scope or compound statement.
  **L2513 CN**: 打开一个新的词法作用域或复合语句块。
- **L2514 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(__M,`.
  **L2514 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(__M,` 从当前函数返回。
- **L2515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_broadcastb_epi8(__A),`.
  **L2515 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_broadcastb_epi8(__A),`。
- **L2516 EN**: Executes a call or declaration centered on `statement`.
  **L2516 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2517 EN**: Closes the current lexical scope or compound statement.
  **L2517 CN**: 结束当前词法作用域或复合语句块。
- **L2518 EN**: Blank line separating nearby declarations or logic blocks.
  **L2518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2519 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2519 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2520 EN**: Continues logic associated with callable symbol `_mm_maskz_broadcastb_epi8`.
  **L2520 CN**: 继续与可调用符号 `_mm_maskz_broadcastb_epi8` 相关的逻辑。

### Lines 2521-2544

````c
{
  return (__m128i)__builtin_ia32_selectb_128(__M,
                                             (__v16qi) _mm_broadcastb_epi8(__A),
                                             (__v16qi) _mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_broadcastb_epi8 (__m256i __O, __mmask32 __M, __m128i __A)
{
  return (__m256i)__builtin_ia32_selectb_256(__M,
                                             (__v32qi) _mm256_broadcastb_epi8(__A),
                                             (__v32qi) __O);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_broadcastb_epi8 (__mmask32 __M, __m128i __A)
{
  return (__m256i)__builtin_ia32_selectb_256(__M,
                                             (__v32qi) _mm256_broadcastb_epi8(__A),
                                             (__v32qi) _mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_broadcastw_epi16 (__m128i __O, __mmask8 __M, __m128i __A)
````
- **L2521 EN**: Opens a new lexical scope or compound statement.
  **L2521 CN**: 打开一个新的词法作用域或复合语句块。
- **L2522 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectb_128(__M,`.
  **L2522 CN**: 以 `(__m128i)__builtin_ia32_selectb_128(__M,` 从当前函数返回。
- **L2523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_broadcastb_epi8(__A),`.
  **L2523 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_broadcastb_epi8(__A),`。
- **L2524 EN**: Executes a call or declaration centered on `statement`.
  **L2524 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2525 EN**: Closes the current lexical scope or compound statement.
  **L2525 CN**: 结束当前词法作用域或复合语句块。
- **L2526 EN**: Blank line separating nearby declarations or logic blocks.
  **L2526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2527 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2527 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2528 EN**: Continues logic associated with callable symbol `_mm256_mask_broadcastb_epi8`.
  **L2528 CN**: 继续与可调用符号 `_mm256_mask_broadcastb_epi8` 相关的逻辑。
- **L2529 EN**: Opens a new lexical scope or compound statement.
  **L2529 CN**: 打开一个新的词法作用域或复合语句块。
- **L2530 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(__M,`.
  **L2530 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(__M,` 从当前函数返回。
- **L2531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) _mm256_broadcastb_epi8(__A),`.
  **L2531 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) _mm256_broadcastb_epi8(__A),`。
- **L2532 EN**: Executes a call or declaration centered on `statement`.
  **L2532 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2533 EN**: Closes the current lexical scope or compound statement.
  **L2533 CN**: 结束当前词法作用域或复合语句块。
- **L2534 EN**: Blank line separating nearby declarations or logic blocks.
  **L2534 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2535 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2535 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2536 EN**: Continues logic associated with callable symbol `_mm256_maskz_broadcastb_epi8`.
  **L2536 CN**: 继续与可调用符号 `_mm256_maskz_broadcastb_epi8` 相关的逻辑。
- **L2537 EN**: Opens a new lexical scope or compound statement.
  **L2537 CN**: 打开一个新的词法作用域或复合语句块。
- **L2538 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectb_256(__M,`.
  **L2538 CN**: 以 `(__m256i)__builtin_ia32_selectb_256(__M,` 从当前函数返回。
- **L2539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) _mm256_broadcastb_epi8(__A),`.
  **L2539 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) _mm256_broadcastb_epi8(__A),`。
- **L2540 EN**: Executes a call or declaration centered on `statement`.
  **L2540 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2541 EN**: Closes the current lexical scope or compound statement.
  **L2541 CN**: 结束当前词法作用域或复合语句块。
- **L2542 EN**: Blank line separating nearby declarations or logic blocks.
  **L2542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2543 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2543 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2544 EN**: Continues logic associated with callable symbol `_mm_mask_broadcastw_epi16`.
  **L2544 CN**: 继续与可调用符号 `_mm_mask_broadcastw_epi16` 相关的逻辑。

### Lines 2545-2568

````c
{
  return (__m128i)__builtin_ia32_selectw_128(__M,
                                             (__v8hi) _mm_broadcastw_epi16(__A),
                                             (__v8hi) __O);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_broadcastw_epi16 (__mmask8 __M, __m128i __A)
{
  return (__m128i)__builtin_ia32_selectw_128(__M,
                                             (__v8hi) _mm_broadcastw_epi16(__A),
                                             (__v8hi) _mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_broadcastw_epi16 (__m256i __O, __mmask16 __M, __m128i __A)
{
  return (__m256i)__builtin_ia32_selectw_256(__M,
                                             (__v16hi) _mm256_broadcastw_epi16(__A),
                                             (__v16hi) __O);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_broadcastw_epi16 (__mmask16 __M, __m128i __A)
````
- **L2545 EN**: Opens a new lexical scope or compound statement.
  **L2545 CN**: 打开一个新的词法作用域或复合语句块。
- **L2546 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__M,`.
  **L2546 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__M,` 从当前函数返回。
- **L2547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) _mm_broadcastw_epi16(__A),`.
  **L2547 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) _mm_broadcastw_epi16(__A),`。
- **L2548 EN**: Executes a call or declaration centered on `statement`.
  **L2548 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2549 EN**: Closes the current lexical scope or compound statement.
  **L2549 CN**: 结束当前词法作用域或复合语句块。
- **L2550 EN**: Blank line separating nearby declarations or logic blocks.
  **L2550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2551 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2551 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2552 EN**: Continues logic associated with callable symbol `_mm_maskz_broadcastw_epi16`.
  **L2552 CN**: 继续与可调用符号 `_mm_maskz_broadcastw_epi16` 相关的逻辑。
- **L2553 EN**: Opens a new lexical scope or compound statement.
  **L2553 CN**: 打开一个新的词法作用域或复合语句块。
- **L2554 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__M,`.
  **L2554 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__M,` 从当前函数返回。
- **L2555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) _mm_broadcastw_epi16(__A),`.
  **L2555 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) _mm_broadcastw_epi16(__A),`。
- **L2556 EN**: Executes a call or declaration centered on `statement`.
  **L2556 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2557 EN**: Closes the current lexical scope or compound statement.
  **L2557 CN**: 结束当前词法作用域或复合语句块。
- **L2558 EN**: Blank line separating nearby declarations or logic blocks.
  **L2558 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2559 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2559 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2560 EN**: Continues logic associated with callable symbol `_mm256_mask_broadcastw_epi16`.
  **L2560 CN**: 继续与可调用符号 `_mm256_mask_broadcastw_epi16` 相关的逻辑。
- **L2561 EN**: Opens a new lexical scope or compound statement.
  **L2561 CN**: 打开一个新的词法作用域或复合语句块。
- **L2562 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__M,`.
  **L2562 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__M,` 从当前函数返回。
- **L2563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) _mm256_broadcastw_epi16(__A),`.
  **L2563 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) _mm256_broadcastw_epi16(__A),`。
- **L2564 EN**: Executes a call or declaration centered on `statement`.
  **L2564 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2565 EN**: Closes the current lexical scope or compound statement.
  **L2565 CN**: 结束当前词法作用域或复合语句块。
- **L2566 EN**: Blank line separating nearby declarations or logic blocks.
  **L2566 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2567 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2567 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2568 EN**: Continues logic associated with callable symbol `_mm256_maskz_broadcastw_epi16`.
  **L2568 CN**: 继续与可调用符号 `_mm256_maskz_broadcastw_epi16` 相关的逻辑。

### Lines 2569-2592

````c
{
  return (__m256i)__builtin_ia32_selectw_256(__M,
                                             (__v16hi) _mm256_broadcastw_epi16(__A),
                                             (__v16hi) _mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_set1_epi16 (__m256i __O, __mmask16 __M, short __A)
{
  return (__m256i) __builtin_ia32_selectw_256 (__M,
                                               (__v16hi) _mm256_set1_epi16(__A),
                                               (__v16hi) __O);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_set1_epi16 (__mmask16 __M, short __A)
{
  return (__m256i) __builtin_ia32_selectw_256(__M,
                                              (__v16hi)_mm256_set1_epi16(__A),
                                              (__v16hi) _mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_set1_epi16 (__m128i __O, __mmask8 __M, short __A)
````
- **L2569 EN**: Opens a new lexical scope or compound statement.
  **L2569 CN**: 打开一个新的词法作用域或复合语句块。
- **L2570 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__M,`.
  **L2570 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__M,` 从当前函数返回。
- **L2571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) _mm256_broadcastw_epi16(__A),`.
  **L2571 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) _mm256_broadcastw_epi16(__A),`。
- **L2572 EN**: Executes a call or declaration centered on `statement`.
  **L2572 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2573 EN**: Closes the current lexical scope or compound statement.
  **L2573 CN**: 结束当前词法作用域或复合语句块。
- **L2574 EN**: Blank line separating nearby declarations or logic blocks.
  **L2574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2575 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2575 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2576 EN**: Continues logic associated with callable symbol `_mm256_mask_set1_epi16`.
  **L2576 CN**: 继续与可调用符号 `_mm256_mask_set1_epi16` 相关的逻辑。
- **L2577 EN**: Opens a new lexical scope or compound statement.
  **L2577 CN**: 打开一个新的词法作用域或复合语句块。
- **L2578 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectw_256 (__M,`.
  **L2578 CN**: 以 `(__m256i) __builtin_ia32_selectw_256 (__M,` 从当前函数返回。
- **L2579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) _mm256_set1_epi16(__A),`.
  **L2579 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) _mm256_set1_epi16(__A),`。
- **L2580 EN**: Executes a call or declaration centered on `statement`.
  **L2580 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2581 EN**: Closes the current lexical scope or compound statement.
  **L2581 CN**: 结束当前词法作用域或复合语句块。
- **L2582 EN**: Blank line separating nearby declarations or logic blocks.
  **L2582 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2583 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2583 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2584 EN**: Continues logic associated with callable symbol `_mm256_maskz_set1_epi16`.
  **L2584 CN**: 继续与可调用符号 `_mm256_maskz_set1_epi16` 相关的逻辑。
- **L2585 EN**: Opens a new lexical scope or compound statement.
  **L2585 CN**: 打开一个新的词法作用域或复合语句块。
- **L2586 EN**: Returns from the current function with `(__m256i) __builtin_ia32_selectw_256(__M,`.
  **L2586 CN**: 以 `(__m256i) __builtin_ia32_selectw_256(__M,` 从当前函数返回。
- **L2587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_set1_epi16(__A),`.
  **L2587 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_set1_epi16(__A),`。
- **L2588 EN**: Executes a call or declaration centered on `statement`.
  **L2588 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2589 EN**: Closes the current lexical scope or compound statement.
  **L2589 CN**: 结束当前词法作用域或复合语句块。
- **L2590 EN**: Blank line separating nearby declarations or logic blocks.
  **L2590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2591 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2591 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2592 EN**: Continues logic associated with callable symbol `_mm_mask_set1_epi16`.
  **L2592 CN**: 继续与可调用符号 `_mm_mask_set1_epi16` 相关的逻辑。

### Lines 2593-2616

````c
{
  return (__m128i) __builtin_ia32_selectw_128(__M,
                                              (__v8hi) _mm_set1_epi16(__A),
                                              (__v8hi) __O);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_set1_epi16 (__mmask8 __M, short __A)
{
  return (__m128i) __builtin_ia32_selectw_128(__M,
                                              (__v8hi) _mm_set1_epi16(__A),
                                              (__v8hi) _mm_setzero_si128());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_permutexvar_epi16(__m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_permvarhi128((__v8hi) __B, (__v8hi) __A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_permutexvar_epi16(__mmask8 __M, __m128i __A, __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                        (__v8hi)_mm_permutexvar_epi16(__A, __B),
                                        (__v8hi) _mm_setzero_si128());
````
- **L2593 EN**: Opens a new lexical scope or compound statement.
  **L2593 CN**: 打开一个新的词法作用域或复合语句块。
- **L2594 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectw_128(__M,`.
  **L2594 CN**: 以 `(__m128i) __builtin_ia32_selectw_128(__M,` 从当前函数返回。
- **L2595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) _mm_set1_epi16(__A),`.
  **L2595 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) _mm_set1_epi16(__A),`。
- **L2596 EN**: Executes a call or declaration centered on `statement`.
  **L2596 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2597 EN**: Closes the current lexical scope or compound statement.
  **L2597 CN**: 结束当前词法作用域或复合语句块。
- **L2598 EN**: Blank line separating nearby declarations or logic blocks.
  **L2598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2599 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2599 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2600 EN**: Continues logic associated with callable symbol `_mm_maskz_set1_epi16`.
  **L2600 CN**: 继续与可调用符号 `_mm_maskz_set1_epi16` 相关的逻辑。
- **L2601 EN**: Opens a new lexical scope or compound statement.
  **L2601 CN**: 打开一个新的词法作用域或复合语句块。
- **L2602 EN**: Returns from the current function with `(__m128i) __builtin_ia32_selectw_128(__M,`.
  **L2602 CN**: 以 `(__m128i) __builtin_ia32_selectw_128(__M,` 从当前函数返回。
- **L2603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) _mm_set1_epi16(__A),`.
  **L2603 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) _mm_set1_epi16(__A),`。
- **L2604 EN**: Executes a call or declaration centered on `statement`.
  **L2604 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2605 EN**: Closes the current lexical scope or compound statement.
  **L2605 CN**: 结束当前词法作用域或复合语句块。
- **L2606 EN**: Blank line separating nearby declarations or logic blocks.
  **L2606 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2607 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2607 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2608 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_permutexvar_epi16(__m128i __A, __m128i __B) {`.
  **L2608 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_permutexvar_epi16(__m128i __A, __m128i __B) {`。
- **L2609 EN**: Returns from the current function with `(__m128i)__builtin_ia32_permvarhi128((__v8hi) __B, (__v8hi) __A)`.
  **L2609 CN**: 以 `(__m128i)__builtin_ia32_permvarhi128((__v8hi) __B, (__v8hi) __A)` 从当前函数返回。
- **L2610 EN**: Closes the current lexical scope or compound statement.
  **L2610 CN**: 结束当前词法作用域或复合语句块。
- **L2611 EN**: Blank line separating nearby declarations or logic blocks.
  **L2611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2612 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2612 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2613 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_permutexvar_epi16(__mmask8 __M, __m128i __A, __m128i __B) {`.
  **L2613 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_permutexvar_epi16(__mmask8 __M, __m128i __A, __m128i __B) {`。
- **L2614 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L2614 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L2615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_permutexvar_epi16(__A, __B),`.
  **L2615 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_permutexvar_epi16(__A, __B),`。
- **L2616 EN**: Executes a call or declaration centered on `statement`.
  **L2616 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 2617-2640

````c
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_permutexvar_epi16(__m128i __W, __mmask8 __M, __m128i __A,
                           __m128i __B) {
  return (__m128i)__builtin_ia32_selectw_128((__mmask8)__M,
                                        (__v8hi)_mm_permutexvar_epi16(__A, __B),
                                        (__v8hi)__W);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_permutexvar_epi16(__m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_permvarhi256((__v16hi) __B, (__v16hi) __A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_permutexvar_epi16(__mmask16 __M, __m256i __A, __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                    (__v16hi)_mm256_permutexvar_epi16(__A, __B),
                                    (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_permutexvar_epi16(__m256i __W, __mmask16 __M, __m256i __A,
````
- **L2617 EN**: Closes the current lexical scope or compound statement.
  **L2617 CN**: 结束当前词法作用域或复合语句块。
- **L2618 EN**: Blank line separating nearby declarations or logic blocks.
  **L2618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2619 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2619 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm_mask_permutexvar_epi16(__m128i __W, __mmask8 __M, __m128i __A,`.
  **L2620 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm_mask_permutexvar_epi16(__m128i __W, __mmask8 __M, __m128i __A,`。
- **L2621 EN**: Continues the surrounding expression or declaration: `__m128i __B) {`.
  **L2621 CN**: 继续构造周围的表达式或声明：`__m128i __B) {`。
- **L2622 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,`.
  **L2622 CN**: 以 `(__m128i)__builtin_ia32_selectw_128((__mmask8)__M,` 从当前函数返回。
- **L2623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_permutexvar_epi16(__A, __B),`.
  **L2623 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_permutexvar_epi16(__A, __B),`。
- **L2624 EN**: Executes a call or declaration centered on `statement`.
  **L2624 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2625 EN**: Closes the current lexical scope or compound statement.
  **L2625 CN**: 结束当前词法作用域或复合语句块。
- **L2626 EN**: Blank line separating nearby declarations or logic blocks.
  **L2626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2627 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2627 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2628 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_permutexvar_epi16(__m256i __A, __m256i __B) {`.
  **L2628 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_permutexvar_epi16(__m256i __A, __m256i __B) {`。
- **L2629 EN**: Returns from the current function with `(__m256i)__builtin_ia32_permvarhi256((__v16hi) __B, (__v16hi) __A)`.
  **L2629 CN**: 以 `(__m256i)__builtin_ia32_permvarhi256((__v16hi) __B, (__v16hi) __A)` 从当前函数返回。
- **L2630 EN**: Closes the current lexical scope or compound statement.
  **L2630 CN**: 结束当前词法作用域或复合语句块。
- **L2631 EN**: Blank line separating nearby declarations or logic blocks.
  **L2631 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2632 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2632 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2633 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_permutexvar_epi16(__mmask16 __M, __m256i __A, __m256i __B) {`.
  **L2633 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_permutexvar_epi16(__mmask16 __M, __m256i __A, __m256i __B) {`。
- **L2634 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L2634 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L2635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_permutexvar_epi16(__A, __B),`.
  **L2635 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_permutexvar_epi16(__A, __B),`。
- **L2636 EN**: Executes a call or declaration centered on `statement`.
  **L2636 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2637 EN**: Closes the current lexical scope or compound statement.
  **L2637 CN**: 结束当前词法作用域或复合语句块。
- **L2638 EN**: Blank line separating nearby declarations or logic blocks.
  **L2638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2639 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2639 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `_mm256_mask_permutexvar_epi16(__m256i __W, __mmask16 __M, __m256i __A,`.
  **L2640 CN**: 继续一个多行参数列表、初始化器或聚合项：`_mm256_mask_permutexvar_epi16(__m256i __W, __mmask16 __M, __m256i __A,`。

### Lines 2641-2664

````c
                              __m256i __B) {
  return (__m256i)__builtin_ia32_selectw_256((__mmask16)__M,
                                    (__v16hi)_mm256_permutexvar_epi16(__A, __B),
                                    (__v16hi)__W);
}

#define _mm_mask_alignr_epi8(W, U, A, B, N) \
  ((__m128i)__builtin_ia32_selectb_128((__mmask16)(U), \
                                 (__v16qi)_mm_alignr_epi8((A), (B), (int)(N)), \
                                 (__v16qi)(__m128i)(W)))

#define _mm_maskz_alignr_epi8(U, A, B, N) \
  ((__m128i)__builtin_ia32_selectb_128((__mmask16)(U), \
                                 (__v16qi)_mm_alignr_epi8((A), (B), (int)(N)), \
                                 (__v16qi)_mm_setzero_si128()))

#define _mm256_mask_alignr_epi8(W, U, A, B, N) \
  ((__m256i)__builtin_ia32_selectb_256((__mmask32)(U), \
                              (__v32qi)_mm256_alignr_epi8((A), (B), (int)(N)), \
                              (__v32qi)(__m256i)(W)))

#define _mm256_maskz_alignr_epi8(U, A, B, N) \
  ((__m256i)__builtin_ia32_selectb_256((__mmask32)(U), \
                              (__v32qi)_mm256_alignr_epi8((A), (B), (int)(N)), \
````
- **L2641 EN**: Continues the surrounding expression or declaration: `__m256i __B) {`.
  **L2641 CN**: 继续构造周围的表达式或声明：`__m256i __B) {`。
- **L2642 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,`.
  **L2642 CN**: 以 `(__m256i)__builtin_ia32_selectw_256((__mmask16)__M,` 从当前函数返回。
- **L2643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_permutexvar_epi16(__A, __B),`.
  **L2643 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_permutexvar_epi16(__A, __B),`。
- **L2644 EN**: Executes a call or declaration centered on `statement`.
  **L2644 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2645 EN**: Closes the current lexical scope or compound statement.
  **L2645 CN**: 结束当前词法作用域或复合语句块。
- **L2646 EN**: Blank line separating nearby declarations or logic blocks.
  **L2646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2647 EN**: Defines macro `_mm_mask_alignr_epi8(W, U, A, B, N)` for conditional compilation, shorthand, or API generation.
  **L2647 CN**: 定义宏 `_mm_mask_alignr_epi8(W, U, A, B, N)`，用于条件编译、简写或 API 生成。
- **L2648 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectb_128`.
  **L2648 CN**: 继续与可调用符号 `__builtin_ia32_selectb_128` 相关的逻辑。
- **L2649 EN**: Continues logic associated with callable symbol `_mm_alignr_epi8`.
  **L2649 CN**: 继续与可调用符号 `_mm_alignr_epi8` 相关的逻辑。
- **L2650 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(W)))`.
  **L2650 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(W)))`。
- **L2651 EN**: Blank line separating nearby declarations or logic blocks.
  **L2651 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2652 EN**: Defines macro `_mm_maskz_alignr_epi8(U, A, B, N)` for conditional compilation, shorthand, or API generation.
  **L2652 CN**: 定义宏 `_mm_maskz_alignr_epi8(U, A, B, N)`，用于条件编译、简写或 API 生成。
- **L2653 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectb_128`.
  **L2653 CN**: 继续与可调用符号 `__builtin_ia32_selectb_128` 相关的逻辑。
- **L2654 EN**: Continues logic associated with callable symbol `_mm_alignr_epi8`.
  **L2654 CN**: 继续与可调用符号 `_mm_alignr_epi8` 相关的逻辑。
- **L2655 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L2655 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L2656 EN**: Blank line separating nearby declarations or logic blocks.
  **L2656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2657 EN**: Defines macro `_mm256_mask_alignr_epi8(W, U, A, B, N)` for conditional compilation, shorthand, or API generation.
  **L2657 CN**: 定义宏 `_mm256_mask_alignr_epi8(W, U, A, B, N)`，用于条件编译、简写或 API 生成。
- **L2658 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectb_256`.
  **L2658 CN**: 继续与可调用符号 `__builtin_ia32_selectb_256` 相关的逻辑。
- **L2659 EN**: Continues logic associated with callable symbol `_mm256_alignr_epi8`.
  **L2659 CN**: 继续与可调用符号 `_mm256_alignr_epi8` 相关的逻辑。
- **L2660 EN**: Continues the surrounding expression or declaration: `(__v32qi)(__m256i)(W)))`.
  **L2660 CN**: 继续构造周围的表达式或声明：`(__v32qi)(__m256i)(W)))`。
- **L2661 EN**: Blank line separating nearby declarations or logic blocks.
  **L2661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2662 EN**: Defines macro `_mm256_maskz_alignr_epi8(U, A, B, N)` for conditional compilation, shorthand, or API generation.
  **L2662 CN**: 定义宏 `_mm256_maskz_alignr_epi8(U, A, B, N)`，用于条件编译、简写或 API 生成。
- **L2663 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectb_256`.
  **L2663 CN**: 继续与可调用符号 `__builtin_ia32_selectb_256` 相关的逻辑。
- **L2664 EN**: Continues logic associated with callable symbol `_mm256_alignr_epi8`.
  **L2664 CN**: 继续与可调用符号 `_mm256_alignr_epi8` 相关的逻辑。

### Lines 2665-2688

````c
                              (__v32qi)_mm256_setzero_si256()))

#define _mm_dbsad_epu8(A, B, imm) \
  ((__m128i)__builtin_ia32_dbpsadbw128((__v16qi)(__m128i)(A), \
                                       (__v16qi)(__m128i)(B), (int)(imm)))

#define _mm_mask_dbsad_epu8(W, U, A, B, imm) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                      (__v8hi)_mm_dbsad_epu8((A), (B), (imm)), \
                                      (__v8hi)(__m128i)(W)))

#define _mm_maskz_dbsad_epu8(U, A, B, imm) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                      (__v8hi)_mm_dbsad_epu8((A), (B), (imm)), \
                                      (__v8hi)_mm_setzero_si128()))

#define _mm256_dbsad_epu8(A, B, imm) \
  ((__m256i)__builtin_ia32_dbpsadbw256((__v32qi)(__m256i)(A), \
                                       (__v32qi)(__m256i)(B), (int)(imm)))

#define _mm256_mask_dbsad_epu8(W, U, A, B, imm) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                  (__v16hi)_mm256_dbsad_epu8((A), (B), (imm)), \
                                  (__v16hi)(__m256i)(W)))
````
- **L2665 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L2665 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L2666 EN**: Blank line separating nearby declarations or logic blocks.
  **L2666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2667 EN**: Defines macro `_mm_dbsad_epu8(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L2667 CN**: 定义宏 `_mm_dbsad_epu8(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L2668 EN**: Continues logic associated with callable symbol `__builtin_ia32_dbpsadbw128`.
  **L2668 CN**: 继续与可调用符号 `__builtin_ia32_dbpsadbw128` 相关的逻辑。
- **L2669 EN**: Continues the surrounding expression or declaration: `(__v16qi)(__m128i)(B), (int)(imm)))`.
  **L2669 CN**: 继续构造周围的表达式或声明：`(__v16qi)(__m128i)(B), (int)(imm)))`。
- **L2670 EN**: Blank line separating nearby declarations or logic blocks.
  **L2670 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2671 EN**: Defines macro `_mm_mask_dbsad_epu8(W, U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L2671 CN**: 定义宏 `_mm_mask_dbsad_epu8(W, U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L2672 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L2672 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L2673 EN**: Continues logic associated with callable symbol `_mm_dbsad_epu8`.
  **L2673 CN**: 继续与可调用符号 `_mm_dbsad_epu8` 相关的逻辑。
- **L2674 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(W)))`.
  **L2674 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(W)))`。
- **L2675 EN**: Blank line separating nearby declarations or logic blocks.
  **L2675 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2676 EN**: Defines macro `_mm_maskz_dbsad_epu8(U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L2676 CN**: 定义宏 `_mm_maskz_dbsad_epu8(U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L2677 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L2677 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L2678 EN**: Continues logic associated with callable symbol `_mm_dbsad_epu8`.
  **L2678 CN**: 继续与可调用符号 `_mm_dbsad_epu8` 相关的逻辑。
- **L2679 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L2679 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L2680 EN**: Blank line separating nearby declarations or logic blocks.
  **L2680 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2681 EN**: Defines macro `_mm256_dbsad_epu8(A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L2681 CN**: 定义宏 `_mm256_dbsad_epu8(A, B, imm)`，用于条件编译、简写或 API 生成。
- **L2682 EN**: Continues logic associated with callable symbol `__builtin_ia32_dbpsadbw256`.
  **L2682 CN**: 继续与可调用符号 `__builtin_ia32_dbpsadbw256` 相关的逻辑。
- **L2683 EN**: Continues the surrounding expression or declaration: `(__v32qi)(__m256i)(B), (int)(imm)))`.
  **L2683 CN**: 继续构造周围的表达式或声明：`(__v32qi)(__m256i)(B), (int)(imm)))`。
- **L2684 EN**: Blank line separating nearby declarations or logic blocks.
  **L2684 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2685 EN**: Defines macro `_mm256_mask_dbsad_epu8(W, U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L2685 CN**: 定义宏 `_mm256_mask_dbsad_epu8(W, U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L2686 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L2686 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L2687 EN**: Continues logic associated with callable symbol `_mm256_dbsad_epu8`.
  **L2687 CN**: 继续与可调用符号 `_mm256_dbsad_epu8` 相关的逻辑。
- **L2688 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(W)))`.
  **L2688 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(W)))`。

### Lines 2689-2712

````c

#define _mm256_maskz_dbsad_epu8(U, A, B, imm) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                  (__v16hi)_mm256_dbsad_epu8((A), (B), (imm)), \
                                  (__v16hi)_mm256_setzero_si256()))

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_add_epi16(__m128i __W) {
  return __builtin_reduce_add((__v8hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_mul_epi16(__m128i __W) {
  return __builtin_reduce_mul((__v8hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_and_epi16(__m128i __W) {
  return __builtin_reduce_and((__v8hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_or_epi16(__m128i __W) {
  return __builtin_reduce_or((__v8hi)__W);
````
- **L2689 EN**: Blank line separating nearby declarations or logic blocks.
  **L2689 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2690 EN**: Defines macro `_mm256_maskz_dbsad_epu8(U, A, B, imm)` for conditional compilation, shorthand, or API generation.
  **L2690 CN**: 定义宏 `_mm256_maskz_dbsad_epu8(U, A, B, imm)`，用于条件编译、简写或 API 生成。
- **L2691 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L2691 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L2692 EN**: Continues logic associated with callable symbol `_mm256_dbsad_epu8`.
  **L2692 CN**: 继续与可调用符号 `_mm256_dbsad_epu8` 相关的逻辑。
- **L2693 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L2693 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L2694 EN**: Blank line separating nearby declarations or logic blocks.
  **L2694 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2695 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2695 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2696 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_add_epi16(__m128i __W) {`.
  **L2696 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_add_epi16(__m128i __W) {`。
- **L2697 EN**: Returns from the current function with `__builtin_reduce_add((__v8hi)__W)`.
  **L2697 CN**: 以 `__builtin_reduce_add((__v8hi)__W)` 从当前函数返回。
- **L2698 EN**: Closes the current lexical scope or compound statement.
  **L2698 CN**: 结束当前词法作用域或复合语句块。
- **L2699 EN**: Blank line separating nearby declarations or logic blocks.
  **L2699 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2700 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2700 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2701 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_mul_epi16(__m128i __W) {`.
  **L2701 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_mul_epi16(__m128i __W) {`。
- **L2702 EN**: Returns from the current function with `__builtin_reduce_mul((__v8hi)__W)`.
  **L2702 CN**: 以 `__builtin_reduce_mul((__v8hi)__W)` 从当前函数返回。
- **L2703 EN**: Closes the current lexical scope or compound statement.
  **L2703 CN**: 结束当前词法作用域或复合语句块。
- **L2704 EN**: Blank line separating nearby declarations or logic blocks.
  **L2704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2705 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2705 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2706 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_and_epi16(__m128i __W) {`.
  **L2706 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_and_epi16(__m128i __W) {`。
- **L2707 EN**: Returns from the current function with `__builtin_reduce_and((__v8hi)__W)`.
  **L2707 CN**: 以 `__builtin_reduce_and((__v8hi)__W)` 从当前函数返回。
- **L2708 EN**: Closes the current lexical scope or compound statement.
  **L2708 CN**: 结束当前词法作用域或复合语句块。
- **L2709 EN**: Blank line separating nearby declarations or logic blocks.
  **L2709 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2710 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2710 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2711 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_or_epi16(__m128i __W) {`.
  **L2711 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_or_epi16(__m128i __W) {`。
- **L2712 EN**: Returns from the current function with `__builtin_reduce_or((__v8hi)__W)`.
  **L2712 CN**: 以 `__builtin_reduce_or((__v8hi)__W)` 从当前函数返回。

### Lines 2713-2736

````c
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_add_epi16(__mmask8 __M, __m128i __W) {
  __W = _mm_maskz_mov_epi16(__M, __W);
  return __builtin_reduce_add((__v8hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_mul_epi16(__mmask8 __M, __m128i __W) {
  __W = _mm_mask_mov_epi16(_mm_set1_epi16(1), __M, __W);
  return __builtin_reduce_mul((__v8hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_and_epi16(__mmask8 __M, __m128i __W) {
  __W = _mm_mask_mov_epi16(_mm_set1_epi16(-1), __M, __W);
  return __builtin_reduce_and((__v8hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_or_epi16(__mmask8 __M, __m128i __W) {
  __W = _mm_maskz_mov_epi16(__M, __W);
  return __builtin_reduce_or((__v8hi)__W);
````
- **L2713 EN**: Closes the current lexical scope or compound statement.
  **L2713 CN**: 结束当前词法作用域或复合语句块。
- **L2714 EN**: Blank line separating nearby declarations or logic blocks.
  **L2714 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2715 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2715 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2716 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_add_epi16(__mmask8 __M, __m128i __W) {`.
  **L2716 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_add_epi16(__mmask8 __M, __m128i __W) {`。
- **L2717 EN**: Executes a call or declaration centered on `_mm_maskz_mov_epi16`.
  **L2717 CN**: 执行以 `_mm_maskz_mov_epi16` 为核心的调用或声明。
- **L2718 EN**: Returns from the current function with `__builtin_reduce_add((__v8hi)__W)`.
  **L2718 CN**: 以 `__builtin_reduce_add((__v8hi)__W)` 从当前函数返回。
- **L2719 EN**: Closes the current lexical scope or compound statement.
  **L2719 CN**: 结束当前词法作用域或复合语句块。
- **L2720 EN**: Blank line separating nearby declarations or logic blocks.
  **L2720 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2721 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2721 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2722 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_mul_epi16(__mmask8 __M, __m128i __W) {`.
  **L2722 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_mul_epi16(__mmask8 __M, __m128i __W) {`。
- **L2723 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi16`.
  **L2723 CN**: 执行以 `_mm_mask_mov_epi16` 为核心的调用或声明。
- **L2724 EN**: Returns from the current function with `__builtin_reduce_mul((__v8hi)__W)`.
  **L2724 CN**: 以 `__builtin_reduce_mul((__v8hi)__W)` 从当前函数返回。
- **L2725 EN**: Closes the current lexical scope or compound statement.
  **L2725 CN**: 结束当前词法作用域或复合语句块。
- **L2726 EN**: Blank line separating nearby declarations or logic blocks.
  **L2726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2727 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2727 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2728 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_and_epi16(__mmask8 __M, __m128i __W) {`.
  **L2728 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_and_epi16(__mmask8 __M, __m128i __W) {`。
- **L2729 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi16`.
  **L2729 CN**: 执行以 `_mm_mask_mov_epi16` 为核心的调用或声明。
- **L2730 EN**: Returns from the current function with `__builtin_reduce_and((__v8hi)__W)`.
  **L2730 CN**: 以 `__builtin_reduce_and((__v8hi)__W)` 从当前函数返回。
- **L2731 EN**: Closes the current lexical scope or compound statement.
  **L2731 CN**: 结束当前词法作用域或复合语句块。
- **L2732 EN**: Blank line separating nearby declarations or logic blocks.
  **L2732 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2733 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2733 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2734 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_or_epi16(__mmask8 __M, __m128i __W) {`.
  **L2734 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_or_epi16(__mmask8 __M, __m128i __W) {`。
- **L2735 EN**: Executes a call or declaration centered on `_mm_maskz_mov_epi16`.
  **L2735 CN**: 执行以 `_mm_maskz_mov_epi16` 为核心的调用或声明。
- **L2736 EN**: Returns from the current function with `__builtin_reduce_or((__v8hi)__W)`.
  **L2736 CN**: 以 `__builtin_reduce_or((__v8hi)__W)` 从当前函数返回。

### Lines 2737-2760

````c
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_max_epi16(__m128i __V) {
  return __builtin_reduce_max((__v8hi)__V);
}

static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_max_epu16(__m128i __V) {
  return __builtin_reduce_max((__v8hu)__V);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_min_epi16(__m128i __V) {
  return __builtin_reduce_min((__v8hi)__V);
}

static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_min_epu16(__m128i __V) {
  return __builtin_reduce_min((__v8hu)__V);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_max_epi16(__mmask16 __M, __m128i __V) {
````
- **L2737 EN**: Closes the current lexical scope or compound statement.
  **L2737 CN**: 结束当前词法作用域或复合语句块。
- **L2738 EN**: Blank line separating nearby declarations or logic blocks.
  **L2738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2739 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2739 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2740 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_max_epi16(__m128i __V) {`.
  **L2740 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_max_epi16(__m128i __V) {`。
- **L2741 EN**: Returns from the current function with `__builtin_reduce_max((__v8hi)__V)`.
  **L2741 CN**: 以 `__builtin_reduce_max((__v8hi)__V)` 从当前函数返回。
- **L2742 EN**: Closes the current lexical scope or compound statement.
  **L2742 CN**: 结束当前词法作用域或复合语句块。
- **L2743 EN**: Blank line separating nearby declarations or logic blocks.
  **L2743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2744 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2744 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2745 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_max_epu16(__m128i __V) {`.
  **L2745 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_max_epu16(__m128i __V) {`。
- **L2746 EN**: Returns from the current function with `__builtin_reduce_max((__v8hu)__V)`.
  **L2746 CN**: 以 `__builtin_reduce_max((__v8hu)__V)` 从当前函数返回。
- **L2747 EN**: Closes the current lexical scope or compound statement.
  **L2747 CN**: 结束当前词法作用域或复合语句块。
- **L2748 EN**: Blank line separating nearby declarations or logic blocks.
  **L2748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2749 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2749 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2750 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_min_epi16(__m128i __V) {`.
  **L2750 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_min_epi16(__m128i __V) {`。
- **L2751 EN**: Returns from the current function with `__builtin_reduce_min((__v8hi)__V)`.
  **L2751 CN**: 以 `__builtin_reduce_min((__v8hi)__V)` 从当前函数返回。
- **L2752 EN**: Closes the current lexical scope or compound statement.
  **L2752 CN**: 结束当前词法作用域或复合语句块。
- **L2753 EN**: Blank line separating nearby declarations or logic blocks.
  **L2753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2754 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2754 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2755 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_min_epu16(__m128i __V) {`.
  **L2755 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_min_epu16(__m128i __V) {`。
- **L2756 EN**: Returns from the current function with `__builtin_reduce_min((__v8hu)__V)`.
  **L2756 CN**: 以 `__builtin_reduce_min((__v8hu)__V)` 从当前函数返回。
- **L2757 EN**: Closes the current lexical scope or compound statement.
  **L2757 CN**: 结束当前词法作用域或复合语句块。
- **L2758 EN**: Blank line separating nearby declarations or logic blocks.
  **L2758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2759 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2759 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2760 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_max_epi16(__mmask16 __M, __m128i __V) {`.
  **L2760 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_max_epi16(__mmask16 __M, __m128i __V) {`。

### Lines 2761-2784

````c
  __V = _mm_mask_mov_epi16(_mm_set1_epi16(-32767-1), __M, __V);
  return __builtin_reduce_max((__v8hi)__V);
}

static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_max_epu16(__mmask16 __M, __m128i __V) {
  __V = _mm_maskz_mov_epi16(__M, __V);
  return __builtin_reduce_max((__v8hu)__V);
}

static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_min_epi16(__mmask16 __M, __m128i __V) {
  __V = _mm_mask_mov_epi16(_mm_set1_epi16(32767), __M, __V);
  return __builtin_reduce_min((__v8hi)__V);
}

static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_min_epu16(__mmask16 __M, __m128i __V) {
  __V = _mm_mask_mov_epi16(_mm_set1_epi16(-1), __M, __V);
  return __builtin_reduce_min((__v8hu)__V);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_add_epi16(__m256i __W) {
````
- **L2761 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi16`.
  **L2761 CN**: 执行以 `_mm_mask_mov_epi16` 为核心的调用或声明。
- **L2762 EN**: Returns from the current function with `__builtin_reduce_max((__v8hi)__V)`.
  **L2762 CN**: 以 `__builtin_reduce_max((__v8hi)__V)` 从当前函数返回。
- **L2763 EN**: Closes the current lexical scope or compound statement.
  **L2763 CN**: 结束当前词法作用域或复合语句块。
- **L2764 EN**: Blank line separating nearby declarations or logic blocks.
  **L2764 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2765 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2765 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2766 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_max_epu16(__mmask16 __M, __m128i __V) {`.
  **L2766 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_max_epu16(__mmask16 __M, __m128i __V) {`。
- **L2767 EN**: Executes a call or declaration centered on `_mm_maskz_mov_epi16`.
  **L2767 CN**: 执行以 `_mm_maskz_mov_epi16` 为核心的调用或声明。
- **L2768 EN**: Returns from the current function with `__builtin_reduce_max((__v8hu)__V)`.
  **L2768 CN**: 以 `__builtin_reduce_max((__v8hu)__V)` 从当前函数返回。
- **L2769 EN**: Closes the current lexical scope or compound statement.
  **L2769 CN**: 结束当前词法作用域或复合语句块。
- **L2770 EN**: Blank line separating nearby declarations or logic blocks.
  **L2770 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2771 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2771 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2772 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_min_epi16(__mmask16 __M, __m128i __V) {`.
  **L2772 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_min_epi16(__mmask16 __M, __m128i __V) {`。
- **L2773 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi16`.
  **L2773 CN**: 执行以 `_mm_mask_mov_epi16` 为核心的调用或声明。
- **L2774 EN**: Returns from the current function with `__builtin_reduce_min((__v8hi)__V)`.
  **L2774 CN**: 以 `__builtin_reduce_min((__v8hi)__V)` 从当前函数返回。
- **L2775 EN**: Closes the current lexical scope or compound statement.
  **L2775 CN**: 结束当前词法作用域或复合语句块。
- **L2776 EN**: Blank line separating nearby declarations or logic blocks.
  **L2776 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2777 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2777 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned short __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2778 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_min_epu16(__mmask16 __M, __m128i __V) {`.
  **L2778 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_min_epu16(__mmask16 __M, __m128i __V) {`。
- **L2779 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi16`.
  **L2779 CN**: 执行以 `_mm_mask_mov_epi16` 为核心的调用或声明。
- **L2780 EN**: Returns from the current function with `__builtin_reduce_min((__v8hu)__V)`.
  **L2780 CN**: 以 `__builtin_reduce_min((__v8hu)__V)` 从当前函数返回。
- **L2781 EN**: Closes the current lexical scope or compound statement.
  **L2781 CN**: 结束当前词法作用域或复合语句块。
- **L2782 EN**: Blank line separating nearby declarations or logic blocks.
  **L2782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2783 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2783 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2784 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_add_epi16(__m256i __W) {`.
  **L2784 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_add_epi16(__m256i __W) {`。

### Lines 2785-2808

````c
  return __builtin_reduce_add((__v16hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_mul_epi16(__m256i __W) {
  return __builtin_reduce_mul((__v16hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_and_epi16(__m256i __W) {
  return __builtin_reduce_and((__v16hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_or_epi16(__m256i __W) {
  return __builtin_reduce_or((__v16hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_add_epi16(__mmask16 __M, __m256i __W) {
  __W = _mm256_maskz_mov_epi16(__M, __W);
  return __builtin_reduce_add((__v16hi)__W);
}

````
- **L2785 EN**: Returns from the current function with `__builtin_reduce_add((__v16hi)__W)`.
  **L2785 CN**: 以 `__builtin_reduce_add((__v16hi)__W)` 从当前函数返回。
- **L2786 EN**: Closes the current lexical scope or compound statement.
  **L2786 CN**: 结束当前词法作用域或复合语句块。
- **L2787 EN**: Blank line separating nearby declarations or logic blocks.
  **L2787 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2788 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2788 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2789 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_mul_epi16(__m256i __W) {`.
  **L2789 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_mul_epi16(__m256i __W) {`。
- **L2790 EN**: Returns from the current function with `__builtin_reduce_mul((__v16hi)__W)`.
  **L2790 CN**: 以 `__builtin_reduce_mul((__v16hi)__W)` 从当前函数返回。
- **L2791 EN**: Closes the current lexical scope or compound statement.
  **L2791 CN**: 结束当前词法作用域或复合语句块。
- **L2792 EN**: Blank line separating nearby declarations or logic blocks.
  **L2792 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2793 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2793 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2794 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_and_epi16(__m256i __W) {`.
  **L2794 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_and_epi16(__m256i __W) {`。
- **L2795 EN**: Returns from the current function with `__builtin_reduce_and((__v16hi)__W)`.
  **L2795 CN**: 以 `__builtin_reduce_and((__v16hi)__W)` 从当前函数返回。
- **L2796 EN**: Closes the current lexical scope or compound statement.
  **L2796 CN**: 结束当前词法作用域或复合语句块。
- **L2797 EN**: Blank line separating nearby declarations or logic blocks.
  **L2797 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2798 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2798 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2799 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_or_epi16(__m256i __W) {`.
  **L2799 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_or_epi16(__m256i __W) {`。
- **L2800 EN**: Returns from the current function with `__builtin_reduce_or((__v16hi)__W)`.
  **L2800 CN**: 以 `__builtin_reduce_or((__v16hi)__W)` 从当前函数返回。
- **L2801 EN**: Closes the current lexical scope or compound statement.
  **L2801 CN**: 结束当前词法作用域或复合语句块。
- **L2802 EN**: Blank line separating nearby declarations or logic blocks.
  **L2802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2803 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2803 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2804 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_add_epi16(__mmask16 __M, __m256i __W) {`.
  **L2804 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_add_epi16(__mmask16 __M, __m256i __W) {`。
- **L2805 EN**: Executes a call or declaration centered on `_mm256_maskz_mov_epi16`.
  **L2805 CN**: 执行以 `_mm256_maskz_mov_epi16` 为核心的调用或声明。
- **L2806 EN**: Returns from the current function with `__builtin_reduce_add((__v16hi)__W)`.
  **L2806 CN**: 以 `__builtin_reduce_add((__v16hi)__W)` 从当前函数返回。
- **L2807 EN**: Closes the current lexical scope or compound statement.
  **L2807 CN**: 结束当前词法作用域或复合语句块。
- **L2808 EN**: Blank line separating nearby declarations or logic blocks.
  **L2808 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2809-2832

````c
static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_mul_epi16(__mmask16 __M, __m256i __W) {
  __W = _mm256_mask_mov_epi16(_mm256_set1_epi16(1), __M, __W);
  return __builtin_reduce_mul((__v16hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_and_epi16(__mmask16 __M, __m256i __W) {
  __W = _mm256_mask_mov_epi16(_mm256_set1_epi16(-1), __M, __W);
  return __builtin_reduce_and((__v16hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_or_epi16(__mmask16 __M, __m256i __W) {
  __W = _mm256_maskz_mov_epi16(__M, __W);
  return __builtin_reduce_or((__v16hi)__W);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_max_epi16(__m256i __V) {
  return __builtin_reduce_max((__v16hi)__V);
}

static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L2809 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2809 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2810 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_mul_epi16(__mmask16 __M, __m256i __W) {`.
  **L2810 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_mul_epi16(__mmask16 __M, __m256i __W) {`。
- **L2811 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi16`.
  **L2811 CN**: 执行以 `_mm256_mask_mov_epi16` 为核心的调用或声明。
- **L2812 EN**: Returns from the current function with `__builtin_reduce_mul((__v16hi)__W)`.
  **L2812 CN**: 以 `__builtin_reduce_mul((__v16hi)__W)` 从当前函数返回。
- **L2813 EN**: Closes the current lexical scope or compound statement.
  **L2813 CN**: 结束当前词法作用域或复合语句块。
- **L2814 EN**: Blank line separating nearby declarations or logic blocks.
  **L2814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2815 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2815 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2816 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_and_epi16(__mmask16 __M, __m256i __W) {`.
  **L2816 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_and_epi16(__mmask16 __M, __m256i __W) {`。
- **L2817 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi16`.
  **L2817 CN**: 执行以 `_mm256_mask_mov_epi16` 为核心的调用或声明。
- **L2818 EN**: Returns from the current function with `__builtin_reduce_and((__v16hi)__W)`.
  **L2818 CN**: 以 `__builtin_reduce_and((__v16hi)__W)` 从当前函数返回。
- **L2819 EN**: Closes the current lexical scope or compound statement.
  **L2819 CN**: 结束当前词法作用域或复合语句块。
- **L2820 EN**: Blank line separating nearby declarations or logic blocks.
  **L2820 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2821 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2821 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2822 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_or_epi16(__mmask16 __M, __m256i __W) {`.
  **L2822 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_or_epi16(__mmask16 __M, __m256i __W) {`。
- **L2823 EN**: Executes a call or declaration centered on `_mm256_maskz_mov_epi16`.
  **L2823 CN**: 执行以 `_mm256_maskz_mov_epi16` 为核心的调用或声明。
- **L2824 EN**: Returns from the current function with `__builtin_reduce_or((__v16hi)__W)`.
  **L2824 CN**: 以 `__builtin_reduce_or((__v16hi)__W)` 从当前函数返回。
- **L2825 EN**: Closes the current lexical scope or compound statement.
  **L2825 CN**: 结束当前词法作用域或复合语句块。
- **L2826 EN**: Blank line separating nearby declarations or logic blocks.
  **L2826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2827 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2827 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2828 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_max_epi16(__m256i __V) {`.
  **L2828 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_max_epi16(__m256i __V) {`。
- **L2829 EN**: Returns from the current function with `__builtin_reduce_max((__v16hi)__V)`.
  **L2829 CN**: 以 `__builtin_reduce_max((__v16hi)__V)` 从当前函数返回。
- **L2830 EN**: Closes the current lexical scope or compound statement.
  **L2830 CN**: 结束当前词法作用域或复合语句块。
- **L2831 EN**: Blank line separating nearby declarations or logic blocks.
  **L2831 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2832 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2832 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 2833-2856

````c
_mm256_reduce_max_epu16(__m256i __V) {
  return __builtin_reduce_max((__v16hu)__V);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_min_epi16(__m256i __V) {
  return __builtin_reduce_min((__v16hi)__V);
}

static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_min_epu16(__m256i __V) {
  return __builtin_reduce_min((__v16hu)__V);
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_max_epi16(__mmask16 __M, __m256i __V) {
  __V = _mm256_mask_mov_epi16(_mm256_set1_epi16(-32767-1), __M, __V);
  return __builtin_reduce_max((__v16hi)__V);
}

static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_max_epu16(__mmask16 __M, __m256i __V) {
  __V = _mm256_maskz_mov_epi16(__M, __V);
  return __builtin_reduce_max((__v16hu)__V);
````
- **L2833 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_max_epu16(__m256i __V) {`.
  **L2833 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_max_epu16(__m256i __V) {`。
- **L2834 EN**: Returns from the current function with `__builtin_reduce_max((__v16hu)__V)`.
  **L2834 CN**: 以 `__builtin_reduce_max((__v16hu)__V)` 从当前函数返回。
- **L2835 EN**: Closes the current lexical scope or compound statement.
  **L2835 CN**: 结束当前词法作用域或复合语句块。
- **L2836 EN**: Blank line separating nearby declarations or logic blocks.
  **L2836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2837 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2837 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2838 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_min_epi16(__m256i __V) {`.
  **L2838 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_min_epi16(__m256i __V) {`。
- **L2839 EN**: Returns from the current function with `__builtin_reduce_min((__v16hi)__V)`.
  **L2839 CN**: 以 `__builtin_reduce_min((__v16hi)__V)` 从当前函数返回。
- **L2840 EN**: Closes the current lexical scope or compound statement.
  **L2840 CN**: 结束当前词法作用域或复合语句块。
- **L2841 EN**: Blank line separating nearby declarations or logic blocks.
  **L2841 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2842 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2842 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2843 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_min_epu16(__m256i __V) {`.
  **L2843 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_min_epu16(__m256i __V) {`。
- **L2844 EN**: Returns from the current function with `__builtin_reduce_min((__v16hu)__V)`.
  **L2844 CN**: 以 `__builtin_reduce_min((__v16hu)__V)` 从当前函数返回。
- **L2845 EN**: Closes the current lexical scope or compound statement.
  **L2845 CN**: 结束当前词法作用域或复合语句块。
- **L2846 EN**: Blank line separating nearby declarations or logic blocks.
  **L2846 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2847 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2847 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2848 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_max_epi16(__mmask16 __M, __m256i __V) {`.
  **L2848 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_max_epi16(__mmask16 __M, __m256i __V) {`。
- **L2849 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi16`.
  **L2849 CN**: 执行以 `_mm256_mask_mov_epi16` 为核心的调用或声明。
- **L2850 EN**: Returns from the current function with `__builtin_reduce_max((__v16hi)__V)`.
  **L2850 CN**: 以 `__builtin_reduce_max((__v16hi)__V)` 从当前函数返回。
- **L2851 EN**: Closes the current lexical scope or compound statement.
  **L2851 CN**: 结束当前词法作用域或复合语句块。
- **L2852 EN**: Blank line separating nearby declarations or logic blocks.
  **L2852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2853 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2853 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2854 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_max_epu16(__mmask16 __M, __m256i __V) {`.
  **L2854 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_max_epu16(__mmask16 __M, __m256i __V) {`。
- **L2855 EN**: Executes a call or declaration centered on `_mm256_maskz_mov_epi16`.
  **L2855 CN**: 执行以 `_mm256_maskz_mov_epi16` 为核心的调用或声明。
- **L2856 EN**: Returns from the current function with `__builtin_reduce_max((__v16hu)__V)`.
  **L2856 CN**: 以 `__builtin_reduce_max((__v16hu)__V)` 从当前函数返回。

### Lines 2857-2880

````c
}

static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_min_epi16(__mmask16 __M, __m256i __V) {
  __V = _mm256_mask_mov_epi16(_mm256_set1_epi16(32767), __M, __V);
  return __builtin_reduce_min((__v16hi)__V);
}

static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_min_epu16(__mmask16 __M, __m256i __V) {
  __V = _mm256_mask_mov_epi16(_mm256_set1_epi16(-1), __M, __V);
  return __builtin_reduce_min((__v16hu)__V);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_add_epi8(__m128i __W) {
  return __builtin_reduce_add((__v16qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_mul_epi8(__m128i __W) {
  return __builtin_reduce_mul((__v16qs)__W);
}

````
- **L2857 EN**: Closes the current lexical scope or compound statement.
  **L2857 CN**: 结束当前词法作用域或复合语句块。
- **L2858 EN**: Blank line separating nearby declarations or logic blocks.
  **L2858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2859 EN**: Continues the surrounding expression or declaration: `static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2859 CN**: 继续构造周围的表达式或声明：`static __inline__ short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2860 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_min_epi16(__mmask16 __M, __m256i __V) {`.
  **L2860 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_min_epi16(__mmask16 __M, __m256i __V) {`。
- **L2861 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi16`.
  **L2861 CN**: 执行以 `_mm256_mask_mov_epi16` 为核心的调用或声明。
- **L2862 EN**: Returns from the current function with `__builtin_reduce_min((__v16hi)__V)`.
  **L2862 CN**: 以 `__builtin_reduce_min((__v16hi)__V)` 从当前函数返回。
- **L2863 EN**: Closes the current lexical scope or compound statement.
  **L2863 CN**: 结束当前词法作用域或复合语句块。
- **L2864 EN**: Blank line separating nearby declarations or logic blocks.
  **L2864 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2865 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2865 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned short __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2866 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_min_epu16(__mmask16 __M, __m256i __V) {`.
  **L2866 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_min_epu16(__mmask16 __M, __m256i __V) {`。
- **L2867 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi16`.
  **L2867 CN**: 执行以 `_mm256_mask_mov_epi16` 为核心的调用或声明。
- **L2868 EN**: Returns from the current function with `__builtin_reduce_min((__v16hu)__V)`.
  **L2868 CN**: 以 `__builtin_reduce_min((__v16hu)__V)` 从当前函数返回。
- **L2869 EN**: Closes the current lexical scope or compound statement.
  **L2869 CN**: 结束当前词法作用域或复合语句块。
- **L2870 EN**: Blank line separating nearby declarations or logic blocks.
  **L2870 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2871 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2871 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2872 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_add_epi8(__m128i __W) {`.
  **L2872 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_add_epi8(__m128i __W) {`。
- **L2873 EN**: Returns from the current function with `__builtin_reduce_add((__v16qs)__W)`.
  **L2873 CN**: 以 `__builtin_reduce_add((__v16qs)__W)` 从当前函数返回。
- **L2874 EN**: Closes the current lexical scope or compound statement.
  **L2874 CN**: 结束当前词法作用域或复合语句块。
- **L2875 EN**: Blank line separating nearby declarations or logic blocks.
  **L2875 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2876 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2876 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2877 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_mul_epi8(__m128i __W) {`.
  **L2877 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_mul_epi8(__m128i __W) {`。
- **L2878 EN**: Returns from the current function with `__builtin_reduce_mul((__v16qs)__W)`.
  **L2878 CN**: 以 `__builtin_reduce_mul((__v16qs)__W)` 从当前函数返回。
- **L2879 EN**: Closes the current lexical scope or compound statement.
  **L2879 CN**: 结束当前词法作用域或复合语句块。
- **L2880 EN**: Blank line separating nearby declarations or logic blocks.
  **L2880 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2881-2904

````c
static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_and_epi8(__m128i __W) {
  return __builtin_reduce_and((__v16qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_or_epi8(__m128i __W) {
  return __builtin_reduce_or((__v16qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_add_epi8(__mmask16 __M, __m128i __W) {
  __W = _mm_maskz_mov_epi8(__M, __W);
  return __builtin_reduce_add((__v16qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_mul_epi8(__mmask16 __M, __m128i __W) {
  __W = _mm_mask_mov_epi8(_mm_set1_epi8(1), __M, __W);
  return __builtin_reduce_mul((__v16qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_and_epi8(__mmask16 __M, __m128i __W) {
````
- **L2881 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2881 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2882 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_and_epi8(__m128i __W) {`.
  **L2882 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_and_epi8(__m128i __W) {`。
- **L2883 EN**: Returns from the current function with `__builtin_reduce_and((__v16qs)__W)`.
  **L2883 CN**: 以 `__builtin_reduce_and((__v16qs)__W)` 从当前函数返回。
- **L2884 EN**: Closes the current lexical scope or compound statement.
  **L2884 CN**: 结束当前词法作用域或复合语句块。
- **L2885 EN**: Blank line separating nearby declarations or logic blocks.
  **L2885 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2886 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2886 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2887 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_or_epi8(__m128i __W) {`.
  **L2887 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_or_epi8(__m128i __W) {`。
- **L2888 EN**: Returns from the current function with `__builtin_reduce_or((__v16qs)__W)`.
  **L2888 CN**: 以 `__builtin_reduce_or((__v16qs)__W)` 从当前函数返回。
- **L2889 EN**: Closes the current lexical scope or compound statement.
  **L2889 CN**: 结束当前词法作用域或复合语句块。
- **L2890 EN**: Blank line separating nearby declarations or logic blocks.
  **L2890 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2891 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2891 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2892 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_add_epi8(__mmask16 __M, __m128i __W) {`.
  **L2892 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_add_epi8(__mmask16 __M, __m128i __W) {`。
- **L2893 EN**: Executes a call or declaration centered on `_mm_maskz_mov_epi8`.
  **L2893 CN**: 执行以 `_mm_maskz_mov_epi8` 为核心的调用或声明。
- **L2894 EN**: Returns from the current function with `__builtin_reduce_add((__v16qs)__W)`.
  **L2894 CN**: 以 `__builtin_reduce_add((__v16qs)__W)` 从当前函数返回。
- **L2895 EN**: Closes the current lexical scope or compound statement.
  **L2895 CN**: 结束当前词法作用域或复合语句块。
- **L2896 EN**: Blank line separating nearby declarations or logic blocks.
  **L2896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2897 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2897 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2898 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_mul_epi8(__mmask16 __M, __m128i __W) {`.
  **L2898 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_mul_epi8(__mmask16 __M, __m128i __W) {`。
- **L2899 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi8`.
  **L2899 CN**: 执行以 `_mm_mask_mov_epi8` 为核心的调用或声明。
- **L2900 EN**: Returns from the current function with `__builtin_reduce_mul((__v16qs)__W)`.
  **L2900 CN**: 以 `__builtin_reduce_mul((__v16qs)__W)` 从当前函数返回。
- **L2901 EN**: Closes the current lexical scope or compound statement.
  **L2901 CN**: 结束当前词法作用域或复合语句块。
- **L2902 EN**: Blank line separating nearby declarations or logic blocks.
  **L2902 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2903 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2903 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2904 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_and_epi8(__mmask16 __M, __m128i __W) {`.
  **L2904 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_and_epi8(__mmask16 __M, __m128i __W) {`。

### Lines 2905-2928

````c
  __W = _mm_mask_mov_epi8(_mm_set1_epi8(-1), __M, __W);
  return __builtin_reduce_and((__v16qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_or_epi8(__mmask16 __M, __m128i __W) {
  __W = _mm_maskz_mov_epi8(__M, __W);
  return __builtin_reduce_or((__v16qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_max_epi8(__m128i __V) {
  return __builtin_reduce_max((__v16qs)__V);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_max_epu8(__m128i __V) {
  return __builtin_reduce_max((__v16qu)__V);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_min_epi8(__m128i __V) {
  return __builtin_reduce_min((__v16qs)__V);
}
````
- **L2905 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi8`.
  **L2905 CN**: 执行以 `_mm_mask_mov_epi8` 为核心的调用或声明。
- **L2906 EN**: Returns from the current function with `__builtin_reduce_and((__v16qs)__W)`.
  **L2906 CN**: 以 `__builtin_reduce_and((__v16qs)__W)` 从当前函数返回。
- **L2907 EN**: Closes the current lexical scope or compound statement.
  **L2907 CN**: 结束当前词法作用域或复合语句块。
- **L2908 EN**: Blank line separating nearby declarations or logic blocks.
  **L2908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2909 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2909 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2910 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_or_epi8(__mmask16 __M, __m128i __W) {`.
  **L2910 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_or_epi8(__mmask16 __M, __m128i __W) {`。
- **L2911 EN**: Executes a call or declaration centered on `_mm_maskz_mov_epi8`.
  **L2911 CN**: 执行以 `_mm_maskz_mov_epi8` 为核心的调用或声明。
- **L2912 EN**: Returns from the current function with `__builtin_reduce_or((__v16qs)__W)`.
  **L2912 CN**: 以 `__builtin_reduce_or((__v16qs)__W)` 从当前函数返回。
- **L2913 EN**: Closes the current lexical scope or compound statement.
  **L2913 CN**: 结束当前词法作用域或复合语句块。
- **L2914 EN**: Blank line separating nearby declarations or logic blocks.
  **L2914 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2915 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2915 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2916 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_max_epi8(__m128i __V) {`.
  **L2916 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_max_epi8(__m128i __V) {`。
- **L2917 EN**: Returns from the current function with `__builtin_reduce_max((__v16qs)__V)`.
  **L2917 CN**: 以 `__builtin_reduce_max((__v16qs)__V)` 从当前函数返回。
- **L2918 EN**: Closes the current lexical scope or compound statement.
  **L2918 CN**: 结束当前词法作用域或复合语句块。
- **L2919 EN**: Blank line separating nearby declarations or logic blocks.
  **L2919 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2920 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2920 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2921 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_max_epu8(__m128i __V) {`.
  **L2921 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_max_epu8(__m128i __V) {`。
- **L2922 EN**: Returns from the current function with `__builtin_reduce_max((__v16qu)__V)`.
  **L2922 CN**: 以 `__builtin_reduce_max((__v16qu)__V)` 从当前函数返回。
- **L2923 EN**: Closes the current lexical scope or compound statement.
  **L2923 CN**: 结束当前词法作用域或复合语句块。
- **L2924 EN**: Blank line separating nearby declarations or logic blocks.
  **L2924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2925 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2925 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2926 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_min_epi8(__m128i __V) {`.
  **L2926 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_min_epi8(__m128i __V) {`。
- **L2927 EN**: Returns from the current function with `__builtin_reduce_min((__v16qs)__V)`.
  **L2927 CN**: 以 `__builtin_reduce_min((__v16qs)__V)` 从当前函数返回。
- **L2928 EN**: Closes the current lexical scope or compound statement.
  **L2928 CN**: 结束当前词法作用域或复合语句块。

### Lines 2929-2952

````c

static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_reduce_min_epu8(__m128i __V) {
  return __builtin_reduce_min((__v16qu)__V);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_max_epi8(__mmask16 __M, __m128i __V) {
  __V = _mm_mask_mov_epi8(_mm_set1_epi8(-127-1), __M, __V);
  return __builtin_reduce_max((__v16qs)__V);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_max_epu8(__mmask16 __M, __m128i __V) {
  __V = _mm_maskz_mov_epi8(__M, __V);
  return __builtin_reduce_max((__v16qu)__V);
}

static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_min_epi8(__mmask16 __M, __m128i __V) {
  __V = _mm_mask_mov_epi8(_mm_set1_epi8(127), __M, __V);
  return __builtin_reduce_min((__v16qs)__V);
}

````
- **L2929 EN**: Blank line separating nearby declarations or logic blocks.
  **L2929 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2930 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2930 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2931 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_reduce_min_epu8(__m128i __V) {`.
  **L2931 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_reduce_min_epu8(__m128i __V) {`。
- **L2932 EN**: Returns from the current function with `__builtin_reduce_min((__v16qu)__V)`.
  **L2932 CN**: 以 `__builtin_reduce_min((__v16qu)__V)` 从当前函数返回。
- **L2933 EN**: Closes the current lexical scope or compound statement.
  **L2933 CN**: 结束当前词法作用域或复合语句块。
- **L2934 EN**: Blank line separating nearby declarations or logic blocks.
  **L2934 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2935 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2935 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2936 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_max_epi8(__mmask16 __M, __m128i __V) {`.
  **L2936 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_max_epi8(__mmask16 __M, __m128i __V) {`。
- **L2937 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi8`.
  **L2937 CN**: 执行以 `_mm_mask_mov_epi8` 为核心的调用或声明。
- **L2938 EN**: Returns from the current function with `__builtin_reduce_max((__v16qs)__V)`.
  **L2938 CN**: 以 `__builtin_reduce_max((__v16qs)__V)` 从当前函数返回。
- **L2939 EN**: Closes the current lexical scope or compound statement.
  **L2939 CN**: 结束当前词法作用域或复合语句块。
- **L2940 EN**: Blank line separating nearby declarations or logic blocks.
  **L2940 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2941 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2941 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2942 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_max_epu8(__mmask16 __M, __m128i __V) {`.
  **L2942 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_max_epu8(__mmask16 __M, __m128i __V) {`。
- **L2943 EN**: Executes a call or declaration centered on `_mm_maskz_mov_epi8`.
  **L2943 CN**: 执行以 `_mm_maskz_mov_epi8` 为核心的调用或声明。
- **L2944 EN**: Returns from the current function with `__builtin_reduce_max((__v16qu)__V)`.
  **L2944 CN**: 以 `__builtin_reduce_max((__v16qu)__V)` 从当前函数返回。
- **L2945 EN**: Closes the current lexical scope or compound statement.
  **L2945 CN**: 结束当前词法作用域或复合语句块。
- **L2946 EN**: Blank line separating nearby declarations or logic blocks.
  **L2946 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2947 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2947 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2948 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_min_epi8(__mmask16 __M, __m128i __V) {`.
  **L2948 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_min_epi8(__mmask16 __M, __m128i __V) {`。
- **L2949 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi8`.
  **L2949 CN**: 执行以 `_mm_mask_mov_epi8` 为核心的调用或声明。
- **L2950 EN**: Returns from the current function with `__builtin_reduce_min((__v16qs)__V)`.
  **L2950 CN**: 以 `__builtin_reduce_min((__v16qs)__V)` 从当前函数返回。
- **L2951 EN**: Closes the current lexical scope or compound statement.
  **L2951 CN**: 结束当前词法作用域或复合语句块。
- **L2952 EN**: Blank line separating nearby declarations or logic blocks.
  **L2952 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2953-2976

````c
static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_reduce_min_epu8(__mmask16 __M, __m128i __V) {
  __V = _mm_mask_mov_epi8(_mm_set1_epi8(-1), __M, __V);
  return __builtin_reduce_min((__v16qu)__V);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_add_epi8(__m256i __W) {
  return __builtin_reduce_add((__v32qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_mul_epi8(__m256i __W) {
  return __builtin_reduce_mul((__v32qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_and_epi8(__m256i __W) {
  return __builtin_reduce_and((__v32qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_or_epi8(__m256i __W) {
  return __builtin_reduce_or((__v32qs)__W);
````
- **L2953 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L2953 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L2954 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_reduce_min_epu8(__mmask16 __M, __m128i __V) {`.
  **L2954 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_reduce_min_epu8(__mmask16 __M, __m128i __V) {`。
- **L2955 EN**: Executes a call or declaration centered on `_mm_mask_mov_epi8`.
  **L2955 CN**: 执行以 `_mm_mask_mov_epi8` 为核心的调用或声明。
- **L2956 EN**: Returns from the current function with `__builtin_reduce_min((__v16qu)__V)`.
  **L2956 CN**: 以 `__builtin_reduce_min((__v16qu)__V)` 从当前函数返回。
- **L2957 EN**: Closes the current lexical scope or compound statement.
  **L2957 CN**: 结束当前词法作用域或复合语句块。
- **L2958 EN**: Blank line separating nearby declarations or logic blocks.
  **L2958 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2959 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2959 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2960 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_add_epi8(__m256i __W) {`.
  **L2960 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_add_epi8(__m256i __W) {`。
- **L2961 EN**: Returns from the current function with `__builtin_reduce_add((__v32qs)__W)`.
  **L2961 CN**: 以 `__builtin_reduce_add((__v32qs)__W)` 从当前函数返回。
- **L2962 EN**: Closes the current lexical scope or compound statement.
  **L2962 CN**: 结束当前词法作用域或复合语句块。
- **L2963 EN**: Blank line separating nearby declarations or logic blocks.
  **L2963 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2964 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2964 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2965 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_mul_epi8(__m256i __W) {`.
  **L2965 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_mul_epi8(__m256i __W) {`。
- **L2966 EN**: Returns from the current function with `__builtin_reduce_mul((__v32qs)__W)`.
  **L2966 CN**: 以 `__builtin_reduce_mul((__v32qs)__W)` 从当前函数返回。
- **L2967 EN**: Closes the current lexical scope or compound statement.
  **L2967 CN**: 结束当前词法作用域或复合语句块。
- **L2968 EN**: Blank line separating nearby declarations or logic blocks.
  **L2968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2969 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2969 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2970 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_and_epi8(__m256i __W) {`.
  **L2970 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_and_epi8(__m256i __W) {`。
- **L2971 EN**: Returns from the current function with `__builtin_reduce_and((__v32qs)__W)`.
  **L2971 CN**: 以 `__builtin_reduce_and((__v32qs)__W)` 从当前函数返回。
- **L2972 EN**: Closes the current lexical scope or compound statement.
  **L2972 CN**: 结束当前词法作用域或复合语句块。
- **L2973 EN**: Blank line separating nearby declarations or logic blocks.
  **L2973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2974 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2974 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2975 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_or_epi8(__m256i __W) {`.
  **L2975 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_or_epi8(__m256i __W) {`。
- **L2976 EN**: Returns from the current function with `__builtin_reduce_or((__v32qs)__W)`.
  **L2976 CN**: 以 `__builtin_reduce_or((__v32qs)__W)` 从当前函数返回。

### Lines 2977-3000

````c
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_add_epi8(__mmask32 __M, __m256i __W) {
  __W = _mm256_maskz_mov_epi8(__M, __W);
  return __builtin_reduce_add((__v32qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_mul_epi8(__mmask32 __M, __m256i __W) {
  __W = _mm256_mask_mov_epi8(_mm256_set1_epi8(1), __M, __W);
  return __builtin_reduce_mul((__v32qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_and_epi8(__mmask32 __M, __m256i __W) {
  __W = _mm256_mask_mov_epi8(_mm256_set1_epi8(-1), __M, __W);
  return __builtin_reduce_and((__v32qs)__W);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_or_epi8(__mmask32 __M, __m256i __W) {
  __W = _mm256_maskz_mov_epi8(__M, __W);
  return __builtin_reduce_or((__v32qs)__W);
````
- **L2977 EN**: Closes the current lexical scope or compound statement.
  **L2977 CN**: 结束当前词法作用域或复合语句块。
- **L2978 EN**: Blank line separating nearby declarations or logic blocks.
  **L2978 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2979 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2979 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2980 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_add_epi8(__mmask32 __M, __m256i __W) {`.
  **L2980 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_add_epi8(__mmask32 __M, __m256i __W) {`。
- **L2981 EN**: Executes a call or declaration centered on `_mm256_maskz_mov_epi8`.
  **L2981 CN**: 执行以 `_mm256_maskz_mov_epi8` 为核心的调用或声明。
- **L2982 EN**: Returns from the current function with `__builtin_reduce_add((__v32qs)__W)`.
  **L2982 CN**: 以 `__builtin_reduce_add((__v32qs)__W)` 从当前函数返回。
- **L2983 EN**: Closes the current lexical scope or compound statement.
  **L2983 CN**: 结束当前词法作用域或复合语句块。
- **L2984 EN**: Blank line separating nearby declarations or logic blocks.
  **L2984 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2985 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2985 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2986 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_mul_epi8(__mmask32 __M, __m256i __W) {`.
  **L2986 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_mul_epi8(__mmask32 __M, __m256i __W) {`。
- **L2987 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi8`.
  **L2987 CN**: 执行以 `_mm256_mask_mov_epi8` 为核心的调用或声明。
- **L2988 EN**: Returns from the current function with `__builtin_reduce_mul((__v32qs)__W)`.
  **L2988 CN**: 以 `__builtin_reduce_mul((__v32qs)__W)` 从当前函数返回。
- **L2989 EN**: Closes the current lexical scope or compound statement.
  **L2989 CN**: 结束当前词法作用域或复合语句块。
- **L2990 EN**: Blank line separating nearby declarations or logic blocks.
  **L2990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2991 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2991 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2992 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_and_epi8(__mmask32 __M, __m256i __W) {`.
  **L2992 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_and_epi8(__mmask32 __M, __m256i __W) {`。
- **L2993 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi8`.
  **L2993 CN**: 执行以 `_mm256_mask_mov_epi8` 为核心的调用或声明。
- **L2994 EN**: Returns from the current function with `__builtin_reduce_and((__v32qs)__W)`.
  **L2994 CN**: 以 `__builtin_reduce_and((__v32qs)__W)` 从当前函数返回。
- **L2995 EN**: Closes the current lexical scope or compound statement.
  **L2995 CN**: 结束当前词法作用域或复合语句块。
- **L2996 EN**: Blank line separating nearby declarations or logic blocks.
  **L2996 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2997 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L2997 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L2998 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_or_epi8(__mmask32 __M, __m256i __W) {`.
  **L2998 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_or_epi8(__mmask32 __M, __m256i __W) {`。
- **L2999 EN**: Executes a call or declaration centered on `_mm256_maskz_mov_epi8`.
  **L2999 CN**: 执行以 `_mm256_maskz_mov_epi8` 为核心的调用或声明。
- **L3000 EN**: Returns from the current function with `__builtin_reduce_or((__v32qs)__W)`.
  **L3000 CN**: 以 `__builtin_reduce_or((__v32qs)__W)` 从当前函数返回。

### Lines 3001-3024

````c
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_max_epi8(__m256i __V) {
  return __builtin_reduce_max((__v32qs)__V);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_max_epu8(__m256i __V) {
  return __builtin_reduce_max((__v32qu)__V);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_min_epi8(__m256i __V) {
  return __builtin_reduce_min((__v32qs)__V);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_reduce_min_epu8(__m256i __V) {
  return __builtin_reduce_min((__v32qu)__V);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_max_epi8(__mmask32 __M, __m256i __V) {
````
- **L3001 EN**: Closes the current lexical scope or compound statement.
  **L3001 CN**: 结束当前词法作用域或复合语句块。
- **L3002 EN**: Blank line separating nearby declarations or logic blocks.
  **L3002 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3003 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3003 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3004 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_max_epi8(__m256i __V) {`.
  **L3004 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_max_epi8(__m256i __V) {`。
- **L3005 EN**: Returns from the current function with `__builtin_reduce_max((__v32qs)__V)`.
  **L3005 CN**: 以 `__builtin_reduce_max((__v32qs)__V)` 从当前函数返回。
- **L3006 EN**: Closes the current lexical scope or compound statement.
  **L3006 CN**: 结束当前词法作用域或复合语句块。
- **L3007 EN**: Blank line separating nearby declarations or logic blocks.
  **L3007 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3008 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3008 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3009 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_max_epu8(__m256i __V) {`.
  **L3009 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_max_epu8(__m256i __V) {`。
- **L3010 EN**: Returns from the current function with `__builtin_reduce_max((__v32qu)__V)`.
  **L3010 CN**: 以 `__builtin_reduce_max((__v32qu)__V)` 从当前函数返回。
- **L3011 EN**: Closes the current lexical scope or compound statement.
  **L3011 CN**: 结束当前词法作用域或复合语句块。
- **L3012 EN**: Blank line separating nearby declarations or logic blocks.
  **L3012 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3013 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3013 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3014 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_min_epi8(__m256i __V) {`.
  **L3014 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_min_epi8(__m256i __V) {`。
- **L3015 EN**: Returns from the current function with `__builtin_reduce_min((__v32qs)__V)`.
  **L3015 CN**: 以 `__builtin_reduce_min((__v32qs)__V)` 从当前函数返回。
- **L3016 EN**: Closes the current lexical scope or compound statement.
  **L3016 CN**: 结束当前词法作用域或复合语句块。
- **L3017 EN**: Blank line separating nearby declarations or logic blocks.
  **L3017 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3018 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3018 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3019 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_reduce_min_epu8(__m256i __V) {`.
  **L3019 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_reduce_min_epu8(__m256i __V) {`。
- **L3020 EN**: Returns from the current function with `__builtin_reduce_min((__v32qu)__V)`.
  **L3020 CN**: 以 `__builtin_reduce_min((__v32qu)__V)` 从当前函数返回。
- **L3021 EN**: Closes the current lexical scope or compound statement.
  **L3021 CN**: 结束当前词法作用域或复合语句块。
- **L3022 EN**: Blank line separating nearby declarations or logic blocks.
  **L3022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3023 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3023 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3024 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_max_epi8(__mmask32 __M, __m256i __V) {`.
  **L3024 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_max_epi8(__mmask32 __M, __m256i __V) {`。

### Lines 3025-3048

````c
  __V = _mm256_mask_mov_epi8(_mm256_set1_epi8(-127-1), __M, __V);
  return __builtin_reduce_max((__v32qs)__V);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_max_epu8(__mmask32 __M, __m256i __V) {
  __V = _mm256_maskz_mov_epi8(__M, __V);
  return __builtin_reduce_max((__v32qu)__V);
}

static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_min_epi8(__mmask32 __M, __m256i __V) {
  __V = _mm256_mask_mov_epi8(_mm256_set1_epi8(127), __M, __V);
  return __builtin_reduce_min((__v32qs)__V);
}

static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_reduce_min_epu8(__mmask32 __M, __m256i __V) {
  __V = _mm256_mask_mov_epi8(_mm256_set1_epi8(-1), __M, __V);
  return __builtin_reduce_min((__v32qu)__V);
}

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256
````
- **L3025 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi8`.
  **L3025 CN**: 执行以 `_mm256_mask_mov_epi8` 为核心的调用或声明。
- **L3026 EN**: Returns from the current function with `__builtin_reduce_max((__v32qs)__V)`.
  **L3026 CN**: 以 `__builtin_reduce_max((__v32qs)__V)` 从当前函数返回。
- **L3027 EN**: Closes the current lexical scope or compound statement.
  **L3027 CN**: 结束当前词法作用域或复合语句块。
- **L3028 EN**: Blank line separating nearby declarations or logic blocks.
  **L3028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3029 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3029 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3030 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_max_epu8(__mmask32 __M, __m256i __V) {`.
  **L3030 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_max_epu8(__mmask32 __M, __m256i __V) {`。
- **L3031 EN**: Executes a call or declaration centered on `_mm256_maskz_mov_epi8`.
  **L3031 CN**: 执行以 `_mm256_maskz_mov_epi8` 为核心的调用或声明。
- **L3032 EN**: Returns from the current function with `__builtin_reduce_max((__v32qu)__V)`.
  **L3032 CN**: 以 `__builtin_reduce_max((__v32qu)__V)` 从当前函数返回。
- **L3033 EN**: Closes the current lexical scope or compound statement.
  **L3033 CN**: 结束当前词法作用域或复合语句块。
- **L3034 EN**: Blank line separating nearby declarations or logic blocks.
  **L3034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3035 EN**: Continues the surrounding expression or declaration: `static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3035 CN**: 继续构造周围的表达式或声明：`static __inline__ signed char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3036 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_min_epi8(__mmask32 __M, __m256i __V) {`.
  **L3036 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_min_epi8(__mmask32 __M, __m256i __V) {`。
- **L3037 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi8`.
  **L3037 CN**: 执行以 `_mm256_mask_mov_epi8` 为核心的调用或声明。
- **L3038 EN**: Returns from the current function with `__builtin_reduce_min((__v32qs)__V)`.
  **L3038 CN**: 以 `__builtin_reduce_min((__v32qs)__V)` 从当前函数返回。
- **L3039 EN**: Closes the current lexical scope or compound statement.
  **L3039 CN**: 结束当前词法作用域或复合语句块。
- **L3040 EN**: Blank line separating nearby declarations or logic blocks.
  **L3040 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3041 EN**: Continues the surrounding expression or declaration: `static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3041 CN**: 继续构造周围的表达式或声明：`static __inline__ unsigned char __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3042 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_reduce_min_epu8(__mmask32 __M, __m256i __V) {`.
  **L3042 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_reduce_min_epu8(__mmask32 __M, __m256i __V) {`。
- **L3043 EN**: Executes a call or declaration centered on `_mm256_mask_mov_epi8`.
  **L3043 CN**: 执行以 `_mm256_mask_mov_epi8` 为核心的调用或声明。
- **L3044 EN**: Returns from the current function with `__builtin_reduce_min((__v32qu)__V)`.
  **L3044 CN**: 以 `__builtin_reduce_min((__v32qu)__V)` 从当前函数返回。
- **L3045 EN**: Closes the current lexical scope or compound statement.
  **L3045 CN**: 结束当前词法作用域或复合语句块。
- **L3046 EN**: Blank line separating nearby declarations or logic blocks.
  **L3046 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3047 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L3047 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L3048 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L3048 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。

### Lines 3049-3052

````c
#undef __DEFAULT_FN_ATTRS128_CONSTEXPR
#undef __DEFAULT_FN_ATTRS256_CONSTEXPR

#endif /* __AVX512VLBWINTRIN_H */
````
- **L3049 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L3049 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L3050 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L3050 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L3051 EN**: Blank line separating nearby declarations or logic blocks.
  **L3051 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L3052 EN**: Closes the current preprocessor conditional block.
  **L3052 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **x86 SIMD or system intrinsics / x86 SIMD 或系统 intrinsic 接口**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Inline wrapper functions / 内联包装函数**
- **Compiler-specific attributes / 编译器特定属性**
- **Structured data declarations / 结构化数据声明**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__AVX512VLBWINTRIN_H`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_ia32_cmpb128_mask`, `__builtin_ia32_ucmpb128_mask`, `__builtin_ia32_cmpb256_mask`, `__builtin_ia32_ucmpb256_mask`, `__builtin_ia32_cmpw128_mask`, `__builtin_ia32_ucmpw128_mask`, `__builtin_ia32_cmpw256_mask`, `__builtin_ia32_ucmpw256_mask`, `__builtin_ia32_selectb_256`, `__builtin_ia32_selectw_256`, `__builtin_ia32_selectb_128`, `__builtin_ia32_selectw_128`
