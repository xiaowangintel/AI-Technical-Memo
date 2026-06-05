# avx512vlvbmi2intrin.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/avx512vlvbmi2intrin.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: VBMI2 intrinsics.
- **Purpose (CN)**: 提供 VBMI2 intrinsic 接口。
- **Line Count / 行数**: 702

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*===------------- avx512vlvbmi2intrin.h - VBMI2 intrinsics -----------------===
 *
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */
#ifndef __IMMINTRIN_H
#error "Never use <avx512vlvbmi2intrin.h> directly; include <immintrin.h> instead."
#endif

#ifndef __AVX512VLVBMI2INTRIN_H
#define __AVX512VLVBMI2INTRIN_H

/* Define the default attributes for the functions in this file. */
#define __DEFAULT_FN_ATTRS128                                                  \
  __attribute__((__always_inline__, __nodebug__,                               \
                 __target__("avx512vl,avx512vbmi2"),                           \
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Separator comment used for visual grouping.
  **L3 CN**: 用于视觉分组的分隔注释。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L6 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L6 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L7 EN**: Separator comment used for visual grouping.
  **L7 CN**: 用于视觉分组的分隔注释。
- **L8 EN**: Banner comment marking a file or section boundary.
  **L8 CN**: 横幅注释，用于标记文件或章节边界。
- **L9 EN**: Separator comment used for visual grouping.
  **L9 CN**: 用于视觉分组的分隔注释。
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __IMMINTRIN_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __IMMINTRIN_H`。
- **L11 EN**: Emits a compilation error for an unsupported configuration: `#error "Never use <avx512vlvbmi2intrin.h> directly; include <immintrin.h> instead."`.
  **L11 CN**: 为不受支持的配置触发编译错误：`#error "Never use <avx512vlvbmi2intrin.h> directly; include <immintrin.h> instead."`。
- **L12 EN**: Closes the current preprocessor conditional block.
  **L12 CN**: 结束当前预处理条件块。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Starts a preprocessor conditional block: `#ifndef __AVX512VLVBMI2INTRIN_H`.
  **L14 CN**: 开始一个预处理条件块：`#ifndef __AVX512VLVBMI2INTRIN_H`。
- **L15 EN**: Defines macro `__AVX512VLVBMI2INTRIN_H` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `__AVX512VLVBMI2INTRIN_H`，用于条件编译、简写或 API 生成。
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

### Lines 21-40

````c
                 __min_vector_width__(128)))
#define __DEFAULT_FN_ATTRS256                                                  \
  __attribute__((__always_inline__, __nodebug__,                               \
                 __target__("avx512vl,avx512vbmi2"),                           \
                 __min_vector_width__(256)))

#if defined(__cplusplus) && (__cplusplus >= 201103L)
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128 constexpr
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256 constexpr
#else
#define __DEFAULT_FN_ATTRS128_CONSTEXPR __DEFAULT_FN_ATTRS128
#define __DEFAULT_FN_ATTRS256_CONSTEXPR __DEFAULT_FN_ATTRS256
#endif

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_compress_epi16(__m128i __S, __mmask8 __U, __m128i __D) {
  return (__m128i) __builtin_ia32_compresshi128_mask ((__v8hi) __D,
              (__v8hi) __S,
              __U);
}
````
- **L21 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L21 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L22 EN**: Defines macro `__DEFAULT_FN_ATTRS256` for conditional compilation, shorthand, or API generation.
  **L22 CN**: 定义宏 `__DEFAULT_FN_ATTRS256`，用于条件编译、简写或 API 生成。
- **L23 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__,                               \`.
  **L23 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__,                               \`。
- **L24 EN**: Continues logic associated with callable symbol `__target__`.
  **L24 CN**: 继续与可调用符号 `__target__` 相关的逻辑。
- **L25 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L25 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L27 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus) && (__cplusplus >= 201103L)`.
  **L27 CN**: 开始一个预处理条件块：`#if defined(__cplusplus) && (__cplusplus >= 201103L)`。
- **L28 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L28 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L29 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L29 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L30 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L30 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L31 EN**: Defines macro `__DEFAULT_FN_ATTRS128_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L31 CN**: 定义宏 `__DEFAULT_FN_ATTRS128_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L32 EN**: Defines macro `__DEFAULT_FN_ATTRS256_CONSTEXPR` for conditional compilation, shorthand, or API generation.
  **L32 CN**: 定义宏 `__DEFAULT_FN_ATTRS256_CONSTEXPR`，用于条件编译、简写或 API 生成。
- **L33 EN**: Closes the current preprocessor conditional block.
  **L33 CN**: 结束当前预处理条件块。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L35 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L36 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_compress_epi16(__m128i __S, __mmask8 __U, __m128i __D) {`.
  **L36 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_compress_epi16(__m128i __S, __mmask8 __U, __m128i __D) {`。
- **L37 EN**: Returns from the current function with `(__m128i) __builtin_ia32_compresshi128_mask ((__v8hi) __D,`.
  **L37 CN**: 以 `(__m128i) __builtin_ia32_compresshi128_mask ((__v8hi) __D,` 从当前函数返回。
- **L38 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) __S,`.
  **L38 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) __S,`。
- **L39 EN**: Adds a standalone statement or declaration: `__U);`.
  **L39 CN**: 添加一条独立语句或声明：`__U);`。
- **L40 EN**: Closes the current lexical scope or compound statement.
  **L40 CN**: 结束当前词法作用域或复合语句块。

### Lines 41-60

````c

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_compress_epi16(__mmask8 __U, __m128i __D) {
  return (__m128i) __builtin_ia32_compresshi128_mask ((__v8hi) __D,
              (__v8hi) _mm_setzero_si128(),
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_compress_epi8(__m128i __S, __mmask16 __U, __m128i __D) {
  return (__m128i) __builtin_ia32_compressqi128_mask ((__v16qi) __D,
              (__v16qi) __S,
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_compress_epi8(__mmask16 __U, __m128i __D) {
  return (__m128i) __builtin_ia32_compressqi128_mask ((__v16qi) __D,
              (__v16qi) _mm_setzero_si128(),
              __U);
````
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L42 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L43 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_compress_epi16(__mmask8 __U, __m128i __D) {`.
  **L43 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_compress_epi16(__mmask8 __U, __m128i __D) {`。
- **L44 EN**: Returns from the current function with `(__m128i) __builtin_ia32_compresshi128_mask ((__v8hi) __D,`.
  **L44 CN**: 以 `(__m128i) __builtin_ia32_compresshi128_mask ((__v8hi) __D,` 从当前函数返回。
- **L45 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) _mm_setzero_si128(),`.
  **L45 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) _mm_setzero_si128(),`。
- **L46 EN**: Adds a standalone statement or declaration: `__U);`.
  **L46 CN**: 添加一条独立语句或声明：`__U);`。
- **L47 EN**: Closes the current lexical scope or compound statement.
  **L47 CN**: 结束当前词法作用域或复合语句块。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L49 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L49 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L50 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_mask_compress_epi8(__m128i __S, __mmask16 __U, __m128i __D) {`.
  **L50 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_mask_compress_epi8(__m128i __S, __mmask16 __U, __m128i __D) {`。
- **L51 EN**: Returns from the current function with `(__m128i) __builtin_ia32_compressqi128_mask ((__v16qi) __D,`.
  **L51 CN**: 以 `(__m128i) __builtin_ia32_compressqi128_mask ((__v16qi) __D,` 从当前函数返回。
- **L52 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __S,`.
  **L52 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __S,`。
- **L53 EN**: Adds a standalone statement or declaration: `__U);`.
  **L53 CN**: 添加一条独立语句或声明：`__U);`。
- **L54 EN**: Closes the current lexical scope or compound statement.
  **L54 CN**: 结束当前词法作用域或复合语句块。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L56 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L56 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L57 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm_maskz_compress_epi8(__mmask16 __U, __m128i __D) {`.
  **L57 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm_maskz_compress_epi8(__mmask16 __U, __m128i __D) {`。
- **L58 EN**: Returns from the current function with `(__m128i) __builtin_ia32_compressqi128_mask ((__v16qi) __D,`.
  **L58 CN**: 以 `(__m128i) __builtin_ia32_compressqi128_mask ((__v16qi) __D,` 从当前函数返回。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L60 EN**: Adds a standalone statement or declaration: `__U);`.
  **L60 CN**: 添加一条独立语句或声明：`__U);`。

### Lines 61-80

````c
}

static __inline__ void __DEFAULT_FN_ATTRS128
_mm_mask_compressstoreu_epi16(void *__P, __mmask8 __U, __m128i __D)
{
  __builtin_ia32_compressstorehi128_mask ((__v8hi *) __P, (__v8hi) __D,
              __U);
}

static __inline__ void __DEFAULT_FN_ATTRS128
_mm_mask_compressstoreu_epi8(void *__P, __mmask16 __U, __m128i __D)
{
  __builtin_ia32_compressstoreqi128_mask ((__v16qi *) __P, (__v16qi) __D,
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_expand_epi16(__m128i __S, __mmask8 __U, __m128i __D)
{
  return (__m128i) __builtin_ia32_expandhi128_mask ((__v8hi) __D,
````
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS128`.
  **L63 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS128`。
- **L64 EN**: Continues logic associated with callable symbol `_mm_mask_compressstoreu_epi16`.
  **L64 CN**: 继续与可调用符号 `_mm_mask_compressstoreu_epi16` 相关的逻辑。
- **L65 EN**: Opens a new lexical scope or compound statement.
  **L65 CN**: 打开一个新的词法作用域或复合语句块。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_compressstorehi128_mask ((__v8hi *) __P, (__v8hi) __D,`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_compressstorehi128_mask ((__v8hi *) __P, (__v8hi) __D,`。
- **L67 EN**: Adds a standalone statement or declaration: `__U);`.
  **L67 CN**: 添加一条独立语句或声明：`__U);`。
- **L68 EN**: Closes the current lexical scope or compound statement.
  **L68 CN**: 结束当前词法作用域或复合语句块。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS128`.
  **L70 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS128`。
- **L71 EN**: Continues logic associated with callable symbol `_mm_mask_compressstoreu_epi8`.
  **L71 CN**: 继续与可调用符号 `_mm_mask_compressstoreu_epi8` 相关的逻辑。
- **L72 EN**: Opens a new lexical scope or compound statement.
  **L72 CN**: 打开一个新的词法作用域或复合语句块。
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_compressstoreqi128_mask ((__v16qi *) __P, (__v16qi) __D,`.
  **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_compressstoreqi128_mask ((__v16qi *) __P, (__v16qi) __D,`。
- **L74 EN**: Adds a standalone statement or declaration: `__U);`.
  **L74 CN**: 添加一条独立语句或声明：`__U);`。
- **L75 EN**: Closes the current lexical scope or compound statement.
  **L75 CN**: 结束当前词法作用域或复合语句块。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L77 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L77 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L78 EN**: Continues logic associated with callable symbol `_mm_mask_expand_epi16`.
  **L78 CN**: 继续与可调用符号 `_mm_mask_expand_epi16` 相关的逻辑。
- **L79 EN**: Opens a new lexical scope or compound statement.
  **L79 CN**: 打开一个新的词法作用域或复合语句块。
- **L80 EN**: Returns from the current function with `(__m128i) __builtin_ia32_expandhi128_mask ((__v8hi) __D,`.
  **L80 CN**: 以 `(__m128i) __builtin_ia32_expandhi128_mask ((__v8hi) __D,` 从当前函数返回。

### Lines 81-100

````c
              (__v8hi) __S,
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_expand_epi16(__mmask8 __U, __m128i __D)
{
  return (__m128i) __builtin_ia32_expandhi128_mask ((__v8hi) __D,
              (__v8hi) _mm_setzero_si128(),
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_expand_epi8(__m128i __S, __mmask16 __U, __m128i __D)
{
  return (__m128i) __builtin_ia32_expandqi128_mask ((__v16qi) __D,
              (__v16qi) __S,
              __U);
}

````
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) __S,`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) __S,`。
- **L82 EN**: Adds a standalone statement or declaration: `__U);`.
  **L82 CN**: 添加一条独立语句或声明：`__U);`。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L85 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L86 EN**: Continues logic associated with callable symbol `_mm_maskz_expand_epi16`.
  **L86 CN**: 继续与可调用符号 `_mm_maskz_expand_epi16` 相关的逻辑。
- **L87 EN**: Opens a new lexical scope or compound statement.
  **L87 CN**: 打开一个新的词法作用域或复合语句块。
- **L88 EN**: Returns from the current function with `(__m128i) __builtin_ia32_expandhi128_mask ((__v8hi) __D,`.
  **L88 CN**: 以 `(__m128i) __builtin_ia32_expandhi128_mask ((__v8hi) __D,` 从当前函数返回。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) _mm_setzero_si128(),`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) _mm_setzero_si128(),`。
- **L90 EN**: Adds a standalone statement or declaration: `__U);`.
  **L90 CN**: 添加一条独立语句或声明：`__U);`。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L93 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L93 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L94 EN**: Continues logic associated with callable symbol `_mm_mask_expand_epi8`.
  **L94 CN**: 继续与可调用符号 `_mm_mask_expand_epi8` 相关的逻辑。
- **L95 EN**: Opens a new lexical scope or compound statement.
  **L95 CN**: 打开一个新的词法作用域或复合语句块。
- **L96 EN**: Returns from the current function with `(__m128i) __builtin_ia32_expandqi128_mask ((__v16qi) __D,`.
  **L96 CN**: 以 `(__m128i) __builtin_ia32_expandqi128_mask ((__v16qi) __D,` 从当前函数返回。
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __S,`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __S,`。
- **L98 EN**: Adds a standalone statement or declaration: `__U);`.
  **L98 CN**: 添加一条独立语句或声明：`__U);`。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 101-120

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_expand_epi8(__mmask16 __U, __m128i __D)
{
  return (__m128i) __builtin_ia32_expandqi128_mask ((__v16qi) __D,
              (__v16qi) _mm_setzero_si128(),
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_expandloadu_epi16(__m128i __S, __mmask8 __U, void const *__P)
{
  return (__m128i) __builtin_ia32_expandloadhi128_mask ((const __v8hi *)__P,
              (__v8hi) __S,
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_expandloadu_epi16(__mmask8 __U, void const *__P)
{
  return (__m128i) __builtin_ia32_expandloadhi128_mask ((const __v8hi *)__P,
````
- **L101 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L101 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L102 EN**: Continues logic associated with callable symbol `_mm_maskz_expand_epi8`.
  **L102 CN**: 继续与可调用符号 `_mm_maskz_expand_epi8` 相关的逻辑。
- **L103 EN**: Opens a new lexical scope or compound statement.
  **L103 CN**: 打开一个新的词法作用域或复合语句块。
- **L104 EN**: Returns from the current function with `(__m128i) __builtin_ia32_expandqi128_mask ((__v16qi) __D,`.
  **L104 CN**: 以 `(__m128i) __builtin_ia32_expandqi128_mask ((__v16qi) __D,` 从当前函数返回。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L106 EN**: Adds a standalone statement or declaration: `__U);`.
  **L106 CN**: 添加一条独立语句或声明：`__U);`。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L109 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L109 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L110 EN**: Continues logic associated with callable symbol `_mm_mask_expandloadu_epi16`.
  **L110 CN**: 继续与可调用符号 `_mm_mask_expandloadu_epi16` 相关的逻辑。
- **L111 EN**: Opens a new lexical scope or compound statement.
  **L111 CN**: 打开一个新的词法作用域或复合语句块。
- **L112 EN**: Returns from the current function with `(__m128i) __builtin_ia32_expandloadhi128_mask ((const __v8hi *)__P,`.
  **L112 CN**: 以 `(__m128i) __builtin_ia32_expandloadhi128_mask ((const __v8hi *)__P,` 从当前函数返回。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) __S,`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) __S,`。
- **L114 EN**: Adds a standalone statement or declaration: `__U);`.
  **L114 CN**: 添加一条独立语句或声明：`__U);`。
- **L115 EN**: Closes the current lexical scope or compound statement.
  **L115 CN**: 结束当前词法作用域或复合语句块。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L117 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L117 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L118 EN**: Continues logic associated with callable symbol `_mm_maskz_expandloadu_epi16`.
  **L118 CN**: 继续与可调用符号 `_mm_maskz_expandloadu_epi16` 相关的逻辑。
- **L119 EN**: Opens a new lexical scope or compound statement.
  **L119 CN**: 打开一个新的词法作用域或复合语句块。
- **L120 EN**: Returns from the current function with `(__m128i) __builtin_ia32_expandloadhi128_mask ((const __v8hi *)__P,`.
  **L120 CN**: 以 `(__m128i) __builtin_ia32_expandloadhi128_mask ((const __v8hi *)__P,` 从当前函数返回。

### Lines 121-140

````c
              (__v8hi) _mm_setzero_si128(),
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_mask_expandloadu_epi8(__m128i __S, __mmask16 __U, void const *__P)
{
  return (__m128i) __builtin_ia32_expandloadqi128_mask ((const __v16qi *)__P,
              (__v16qi) __S,
              __U);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128
_mm_maskz_expandloadu_epi8(__mmask16 __U, void const *__P)
{
  return (__m128i) __builtin_ia32_expandloadqi128_mask ((const __v16qi *)__P,
              (__v16qi) _mm_setzero_si128(),
              __U);
}

````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi) _mm_setzero_si128(),`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi) _mm_setzero_si128(),`。
- **L122 EN**: Adds a standalone statement or declaration: `__U);`.
  **L122 CN**: 添加一条独立语句或声明：`__U);`。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L125 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L125 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L126 EN**: Continues logic associated with callable symbol `_mm_mask_expandloadu_epi8`.
  **L126 CN**: 继续与可调用符号 `_mm_mask_expandloadu_epi8` 相关的逻辑。
- **L127 EN**: Opens a new lexical scope or compound statement.
  **L127 CN**: 打开一个新的词法作用域或复合语句块。
- **L128 EN**: Returns from the current function with `(__m128i) __builtin_ia32_expandloadqi128_mask ((const __v16qi *)__P,`.
  **L128 CN**: 以 `(__m128i) __builtin_ia32_expandloadqi128_mask ((const __v16qi *)__P,` 从当前函数返回。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) __S,`.
  **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) __S,`。
- **L130 EN**: Adds a standalone statement or declaration: `__U);`.
  **L130 CN**: 添加一条独立语句或声明：`__U);`。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L133 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128`.
  **L133 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128`。
- **L134 EN**: Continues logic associated with callable symbol `_mm_maskz_expandloadu_epi8`.
  **L134 CN**: 继续与可调用符号 `_mm_maskz_expandloadu_epi8` 相关的逻辑。
- **L135 EN**: Opens a new lexical scope or compound statement.
  **L135 CN**: 打开一个新的词法作用域或复合语句块。
- **L136 EN**: Returns from the current function with `(__m128i) __builtin_ia32_expandloadqi128_mask ((const __v16qi *)__P,`.
  **L136 CN**: 以 `(__m128i) __builtin_ia32_expandloadqi128_mask ((const __v16qi *)__P,` 从当前函数返回。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16qi) _mm_setzero_si128(),`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16qi) _mm_setzero_si128(),`。
- **L138 EN**: Adds a standalone statement or declaration: `__U);`.
  **L138 CN**: 添加一条独立语句或声明：`__U);`。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 141-160

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_compress_epi16(__m256i __S, __mmask16 __U, __m256i __D) {
  return (__m256i) __builtin_ia32_compresshi256_mask ((__v16hi) __D,
              (__v16hi) __S,
              __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_compress_epi16(__mmask16 __U, __m256i __D) {
  return (__m256i) __builtin_ia32_compresshi256_mask ((__v16hi) __D,
              (__v16hi) _mm256_setzero_si256(),
              __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_compress_epi8(__m256i __S, __mmask32 __U, __m256i __D) {
  return (__m256i) __builtin_ia32_compressqi256_mask ((__v32qi) __D,
              (__v32qi) __S,
              __U);
}
````
- **L141 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L141 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_compress_epi16(__m256i __S, __mmask16 __U, __m256i __D) {`.
  **L142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_compress_epi16(__m256i __S, __mmask16 __U, __m256i __D) {`。
- **L143 EN**: Returns from the current function with `(__m256i) __builtin_ia32_compresshi256_mask ((__v16hi) __D,`.
  **L143 CN**: 以 `(__m256i) __builtin_ia32_compresshi256_mask ((__v16hi) __D,` 从当前函数返回。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) __S,`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) __S,`。
- **L145 EN**: Adds a standalone statement or declaration: `__U);`.
  **L145 CN**: 添加一条独立语句或声明：`__U);`。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L148 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_compress_epi16(__mmask16 __U, __m256i __D) {`.
  **L149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_compress_epi16(__mmask16 __U, __m256i __D) {`。
- **L150 EN**: Returns from the current function with `(__m256i) __builtin_ia32_compresshi256_mask ((__v16hi) __D,`.
  **L150 CN**: 以 `(__m256i) __builtin_ia32_compresshi256_mask ((__v16hi) __D,` 从当前函数返回。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) _mm256_setzero_si256(),`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) _mm256_setzero_si256(),`。
- **L152 EN**: Adds a standalone statement or declaration: `__U);`.
  **L152 CN**: 添加一条独立语句或声明：`__U);`。
- **L153 EN**: Closes the current lexical scope or compound statement.
  **L153 CN**: 结束当前词法作用域或复合语句块。
- **L154 EN**: Blank line separating nearby declarations or logic blocks.
  **L154 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L155 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L155 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_mask_compress_epi8(__m256i __S, __mmask32 __U, __m256i __D) {`.
  **L156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_mask_compress_epi8(__m256i __S, __mmask32 __U, __m256i __D) {`。
- **L157 EN**: Returns from the current function with `(__m256i) __builtin_ia32_compressqi256_mask ((__v32qi) __D,`.
  **L157 CN**: 以 `(__m256i) __builtin_ia32_compressqi256_mask ((__v32qi) __D,` 从当前函数返回。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) __S,`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) __S,`。
- **L159 EN**: Adds a standalone statement or declaration: `__U);`.
  **L159 CN**: 添加一条独立语句或声明：`__U);`。
- **L160 EN**: Closes the current lexical scope or compound statement.
  **L160 CN**: 结束当前词法作用域或复合语句块。

### Lines 161-180

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_compress_epi8(__mmask32 __U, __m256i __D) {
  return (__m256i) __builtin_ia32_compressqi256_mask ((__v32qi) __D,
              (__v32qi) _mm256_setzero_si256(),
              __U);
}

static __inline__ void __DEFAULT_FN_ATTRS256
_mm256_mask_compressstoreu_epi16(void *__P, __mmask16 __U, __m256i __D)
{
  __builtin_ia32_compressstorehi256_mask ((__v16hi *) __P, (__v16hi) __D,
              __U);
}

static __inline__ void __DEFAULT_FN_ATTRS256
_mm256_mask_compressstoreu_epi8(void *__P, __mmask32 __U, __m256i __D)
{
  __builtin_ia32_compressstoreqi256_mask ((__v32qi *) __P, (__v32qi) __D,
              __U);
````
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L162 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L163 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `_mm256_maskz_compress_epi8(__mmask32 __U, __m256i __D) {`.
  **L163 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`_mm256_maskz_compress_epi8(__mmask32 __U, __m256i __D) {`。
- **L164 EN**: Returns from the current function with `(__m256i) __builtin_ia32_compressqi256_mask ((__v32qi) __D,`.
  **L164 CN**: 以 `(__m256i) __builtin_ia32_compressqi256_mask ((__v32qi) __D,` 从当前函数返回。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) _mm256_setzero_si256(),`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) _mm256_setzero_si256(),`。
- **L166 EN**: Adds a standalone statement or declaration: `__U);`.
  **L166 CN**: 添加一条独立语句或声明：`__U);`。
- **L167 EN**: Closes the current lexical scope or compound statement.
  **L167 CN**: 结束当前词法作用域或复合语句块。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L169 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS256`.
  **L169 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS256`。
- **L170 EN**: Continues logic associated with callable symbol `_mm256_mask_compressstoreu_epi16`.
  **L170 CN**: 继续与可调用符号 `_mm256_mask_compressstoreu_epi16` 相关的逻辑。
- **L171 EN**: Opens a new lexical scope or compound statement.
  **L171 CN**: 打开一个新的词法作用域或复合语句块。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_compressstorehi256_mask ((__v16hi *) __P, (__v16hi) __D,`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_compressstorehi256_mask ((__v16hi *) __P, (__v16hi) __D,`。
- **L173 EN**: Adds a standalone statement or declaration: `__U);`.
  **L173 CN**: 添加一条独立语句或声明：`__U);`。
- **L174 EN**: Closes the current lexical scope or compound statement.
  **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Blank line separating nearby declarations or logic blocks.
  **L175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L176 EN**: Continues the surrounding expression or declaration: `static __inline__ void __DEFAULT_FN_ATTRS256`.
  **L176 CN**: 继续构造周围的表达式或声明：`static __inline__ void __DEFAULT_FN_ATTRS256`。
- **L177 EN**: Continues logic associated with callable symbol `_mm256_mask_compressstoreu_epi8`.
  **L177 CN**: 继续与可调用符号 `_mm256_mask_compressstoreu_epi8` 相关的逻辑。
- **L178 EN**: Opens a new lexical scope or compound statement.
  **L178 CN**: 打开一个新的词法作用域或复合语句块。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_ia32_compressstoreqi256_mask ((__v32qi *) __P, (__v32qi) __D,`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_ia32_compressstoreqi256_mask ((__v32qi *) __P, (__v32qi) __D,`。
- **L180 EN**: Adds a standalone statement or declaration: `__U);`.
  **L180 CN**: 添加一条独立语句或声明：`__U);`。

### Lines 181-200

````c
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_expand_epi16(__m256i __S, __mmask16 __U, __m256i __D)
{
  return (__m256i) __builtin_ia32_expandhi256_mask ((__v16hi) __D,
              (__v16hi) __S,
              __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_expand_epi16(__mmask16 __U, __m256i __D)
{
  return (__m256i) __builtin_ia32_expandhi256_mask ((__v16hi) __D,
              (__v16hi) _mm256_setzero_si256(),
              __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_expand_epi8(__m256i __S, __mmask32 __U, __m256i __D)
````
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L183 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L183 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L184 EN**: Continues logic associated with callable symbol `_mm256_mask_expand_epi16`.
  **L184 CN**: 继续与可调用符号 `_mm256_mask_expand_epi16` 相关的逻辑。
- **L185 EN**: Opens a new lexical scope or compound statement.
  **L185 CN**: 打开一个新的词法作用域或复合语句块。
- **L186 EN**: Returns from the current function with `(__m256i) __builtin_ia32_expandhi256_mask ((__v16hi) __D,`.
  **L186 CN**: 以 `(__m256i) __builtin_ia32_expandhi256_mask ((__v16hi) __D,` 从当前函数返回。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) __S,`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) __S,`。
- **L188 EN**: Adds a standalone statement or declaration: `__U);`.
  **L188 CN**: 添加一条独立语句或声明：`__U);`。
- **L189 EN**: Closes the current lexical scope or compound statement.
  **L189 CN**: 结束当前词法作用域或复合语句块。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L191 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L191 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L192 EN**: Continues logic associated with callable symbol `_mm256_maskz_expand_epi16`.
  **L192 CN**: 继续与可调用符号 `_mm256_maskz_expand_epi16` 相关的逻辑。
- **L193 EN**: Opens a new lexical scope or compound statement.
  **L193 CN**: 打开一个新的词法作用域或复合语句块。
- **L194 EN**: Returns from the current function with `(__m256i) __builtin_ia32_expandhi256_mask ((__v16hi) __D,`.
  **L194 CN**: 以 `(__m256i) __builtin_ia32_expandhi256_mask ((__v16hi) __D,` 从当前函数返回。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) _mm256_setzero_si256(),`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) _mm256_setzero_si256(),`。
- **L196 EN**: Adds a standalone statement or declaration: `__U);`.
  **L196 CN**: 添加一条独立语句或声明：`__U);`。
- **L197 EN**: Closes the current lexical scope or compound statement.
  **L197 CN**: 结束当前词法作用域或复合语句块。
- **L198 EN**: Blank line separating nearby declarations or logic blocks.
  **L198 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L199 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L199 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L200 EN**: Continues logic associated with callable symbol `_mm256_mask_expand_epi8`.
  **L200 CN**: 继续与可调用符号 `_mm256_mask_expand_epi8` 相关的逻辑。

### Lines 201-220

````c
{
  return (__m256i) __builtin_ia32_expandqi256_mask ((__v32qi) __D,
              (__v32qi) __S,
              __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_expand_epi8(__mmask32 __U, __m256i __D)
{
  return (__m256i) __builtin_ia32_expandqi256_mask ((__v32qi) __D,
              (__v32qi) _mm256_setzero_si256(),
              __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_expandloadu_epi16(__m256i __S, __mmask16 __U, void const *__P)
{
  return (__m256i) __builtin_ia32_expandloadhi256_mask ((const __v16hi *)__P,
              (__v16hi) __S,
              __U);
````
- **L201 EN**: Opens a new lexical scope or compound statement.
  **L201 CN**: 打开一个新的词法作用域或复合语句块。
- **L202 EN**: Returns from the current function with `(__m256i) __builtin_ia32_expandqi256_mask ((__v32qi) __D,`.
  **L202 CN**: 以 `(__m256i) __builtin_ia32_expandqi256_mask ((__v32qi) __D,` 从当前函数返回。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) __S,`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) __S,`。
- **L204 EN**: Adds a standalone statement or declaration: `__U);`.
  **L204 CN**: 添加一条独立语句或声明：`__U);`。
- **L205 EN**: Closes the current lexical scope or compound statement.
  **L205 CN**: 结束当前词法作用域或复合语句块。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L207 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L207 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L208 EN**: Continues logic associated with callable symbol `_mm256_maskz_expand_epi8`.
  **L208 CN**: 继续与可调用符号 `_mm256_maskz_expand_epi8` 相关的逻辑。
- **L209 EN**: Opens a new lexical scope or compound statement.
  **L209 CN**: 打开一个新的词法作用域或复合语句块。
- **L210 EN**: Returns from the current function with `(__m256i) __builtin_ia32_expandqi256_mask ((__v32qi) __D,`.
  **L210 CN**: 以 `(__m256i) __builtin_ia32_expandqi256_mask ((__v32qi) __D,` 从当前函数返回。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) _mm256_setzero_si256(),`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) _mm256_setzero_si256(),`。
- **L212 EN**: Adds a standalone statement or declaration: `__U);`.
  **L212 CN**: 添加一条独立语句或声明：`__U);`。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L215 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L215 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L216 EN**: Continues logic associated with callable symbol `_mm256_mask_expandloadu_epi16`.
  **L216 CN**: 继续与可调用符号 `_mm256_mask_expandloadu_epi16` 相关的逻辑。
- **L217 EN**: Opens a new lexical scope or compound statement.
  **L217 CN**: 打开一个新的词法作用域或复合语句块。
- **L218 EN**: Returns from the current function with `(__m256i) __builtin_ia32_expandloadhi256_mask ((const __v16hi *)__P,`.
  **L218 CN**: 以 `(__m256i) __builtin_ia32_expandloadhi256_mask ((const __v16hi *)__P,` 从当前函数返回。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) __S,`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) __S,`。
- **L220 EN**: Adds a standalone statement or declaration: `__U);`.
  **L220 CN**: 添加一条独立语句或声明：`__U);`。

### Lines 221-240

````c
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_expandloadu_epi16(__mmask16 __U, void const *__P)
{
  return (__m256i) __builtin_ia32_expandloadhi256_mask ((const __v16hi *)__P,
              (__v16hi) _mm256_setzero_si256(),
              __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_mask_expandloadu_epi8(__m256i __S, __mmask32 __U, void const *__P)
{
  return (__m256i) __builtin_ia32_expandloadqi256_mask ((const __v32qi *)__P,
              (__v32qi) __S,
              __U);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256
_mm256_maskz_expandloadu_epi8(__mmask32 __U, void const *__P)
````
- **L221 EN**: Closes the current lexical scope or compound statement.
  **L221 CN**: 结束当前词法作用域或复合语句块。
- **L222 EN**: Blank line separating nearby declarations or logic blocks.
  **L222 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L223 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L223 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L224 EN**: Continues logic associated with callable symbol `_mm256_maskz_expandloadu_epi16`.
  **L224 CN**: 继续与可调用符号 `_mm256_maskz_expandloadu_epi16` 相关的逻辑。
- **L225 EN**: Opens a new lexical scope or compound statement.
  **L225 CN**: 打开一个新的词法作用域或复合语句块。
- **L226 EN**: Returns from the current function with `(__m256i) __builtin_ia32_expandloadhi256_mask ((const __v16hi *)__P,`.
  **L226 CN**: 以 `(__m256i) __builtin_ia32_expandloadhi256_mask ((const __v16hi *)__P,` 从当前函数返回。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi) _mm256_setzero_si256(),`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi) _mm256_setzero_si256(),`。
- **L228 EN**: Adds a standalone statement or declaration: `__U);`.
  **L228 CN**: 添加一条独立语句或声明：`__U);`。
- **L229 EN**: Closes the current lexical scope or compound statement.
  **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L231 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L231 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L232 EN**: Continues logic associated with callable symbol `_mm256_mask_expandloadu_epi8`.
  **L232 CN**: 继续与可调用符号 `_mm256_mask_expandloadu_epi8` 相关的逻辑。
- **L233 EN**: Opens a new lexical scope or compound statement.
  **L233 CN**: 打开一个新的词法作用域或复合语句块。
- **L234 EN**: Returns from the current function with `(__m256i) __builtin_ia32_expandloadqi256_mask ((const __v32qi *)__P,`.
  **L234 CN**: 以 `(__m256i) __builtin_ia32_expandloadqi256_mask ((const __v32qi *)__P,` 从当前函数返回。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) __S,`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) __S,`。
- **L236 EN**: Adds a standalone statement or declaration: `__U);`.
  **L236 CN**: 添加一条独立语句或声明：`__U);`。
- **L237 EN**: Closes the current lexical scope or compound statement.
  **L237 CN**: 结束当前词法作用域或复合语句块。
- **L238 EN**: Blank line separating nearby declarations or logic blocks.
  **L238 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L239 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256`.
  **L239 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256`。
- **L240 EN**: Continues logic associated with callable symbol `_mm256_maskz_expandloadu_epi8`.
  **L240 CN**: 继续与可调用符号 `_mm256_maskz_expandloadu_epi8` 相关的逻辑。

### Lines 241-260

````c
{
  return (__m256i) __builtin_ia32_expandloadqi256_mask ((const __v32qi *)__P,
              (__v32qi) _mm256_setzero_si256(),
              __U);
}

#define _mm256_shldi_epi64(A, B, I) \
  ((__m256i)__builtin_ia32_vpshldq256((__v4di)(__m256i)(A), \
                                      (__v4di)(__m256i)(B), (int)(I)))

#define _mm256_mask_shldi_epi64(S, U, A, B, I) \
  ((__m256i)__builtin_ia32_selectq_256((__mmask8)(U), \
                                     (__v4di)_mm256_shldi_epi64((A), (B), (I)), \
                                     (__v4di)(__m256i)(S)))

#define _mm256_maskz_shldi_epi64(U, A, B, I) \
  ((__m256i)__builtin_ia32_selectq_256((__mmask8)(U), \
                                     (__v4di)_mm256_shldi_epi64((A), (B), (I)), \
                                     (__v4di)_mm256_setzero_si256()))

````
- **L241 EN**: Opens a new lexical scope or compound statement.
  **L241 CN**: 打开一个新的词法作用域或复合语句块。
- **L242 EN**: Returns from the current function with `(__m256i) __builtin_ia32_expandloadqi256_mask ((const __v32qi *)__P,`.
  **L242 CN**: 以 `(__m256i) __builtin_ia32_expandloadqi256_mask ((const __v32qi *)__P,` 从当前函数返回。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v32qi) _mm256_setzero_si256(),`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v32qi) _mm256_setzero_si256(),`。
- **L244 EN**: Adds a standalone statement or declaration: `__U);`.
  **L244 CN**: 添加一条独立语句或声明：`__U);`。
- **L245 EN**: Closes the current lexical scope or compound statement.
  **L245 CN**: 结束当前词法作用域或复合语句块。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L247 EN**: Defines macro `_mm256_shldi_epi64(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L247 CN**: 定义宏 `_mm256_shldi_epi64(A, B, I)`，用于条件编译、简写或 API 生成。
- **L248 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshldq256`.
  **L248 CN**: 继续与可调用符号 `__builtin_ia32_vpshldq256` 相关的逻辑。
- **L249 EN**: Continues the surrounding expression or declaration: `(__v4di)(__m256i)(B), (int)(I)))`.
  **L249 CN**: 继续构造周围的表达式或声明：`(__v4di)(__m256i)(B), (int)(I)))`。
- **L250 EN**: Blank line separating nearby declarations or logic blocks.
  **L250 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L251 EN**: Defines macro `_mm256_mask_shldi_epi64(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L251 CN**: 定义宏 `_mm256_mask_shldi_epi64(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L252 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_256`.
  **L252 CN**: 继续与可调用符号 `__builtin_ia32_selectq_256` 相关的逻辑。
- **L253 EN**: Continues logic associated with callable symbol `_mm256_shldi_epi64`.
  **L253 CN**: 继续与可调用符号 `_mm256_shldi_epi64` 相关的逻辑。
- **L254 EN**: Continues the surrounding expression or declaration: `(__v4di)(__m256i)(S)))`.
  **L254 CN**: 继续构造周围的表达式或声明：`(__v4di)(__m256i)(S)))`。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L256 EN**: Defines macro `_mm256_maskz_shldi_epi64(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L256 CN**: 定义宏 `_mm256_maskz_shldi_epi64(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L257 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_256`.
  **L257 CN**: 继续与可调用符号 `__builtin_ia32_selectq_256` 相关的逻辑。
- **L258 EN**: Continues logic associated with callable symbol `_mm256_shldi_epi64`.
  **L258 CN**: 继续与可调用符号 `_mm256_shldi_epi64` 相关的逻辑。
- **L259 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L259 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 261-280

````c
#define _mm_shldi_epi64(A, B, I) \
  ((__m128i)__builtin_ia32_vpshldq128((__v2di)(__m128i)(A), \
                                      (__v2di)(__m128i)(B), (int)(I)))

#define _mm_mask_shldi_epi64(S, U, A, B, I) \
  ((__m128i)__builtin_ia32_selectq_128((__mmask8)(U), \
                                       (__v2di)_mm_shldi_epi64((A), (B), (I)), \
                                       (__v2di)(__m128i)(S)))

#define _mm_maskz_shldi_epi64(U, A, B, I) \
  ((__m128i)__builtin_ia32_selectq_128((__mmask8)(U), \
                                       (__v2di)_mm_shldi_epi64((A), (B), (I)), \
                                       (__v2di)_mm_setzero_si128()))

#define _mm256_shldi_epi32(A, B, I) \
  ((__m256i)__builtin_ia32_vpshldd256((__v8si)(__m256i)(A), \
                                      (__v8si)(__m256i)(B), (int)(I)))

#define _mm256_mask_shldi_epi32(S, U, A, B, I) \
  ((__m256i)__builtin_ia32_selectd_256((__mmask8)(U), \
````
- **L261 EN**: Defines macro `_mm_shldi_epi64(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L261 CN**: 定义宏 `_mm_shldi_epi64(A, B, I)`，用于条件编译、简写或 API 生成。
- **L262 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshldq128`.
  **L262 CN**: 继续与可调用符号 `__builtin_ia32_vpshldq128` 相关的逻辑。
- **L263 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(B), (int)(I)))`.
  **L263 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(B), (int)(I)))`。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L265 EN**: Defines macro `_mm_mask_shldi_epi64(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L265 CN**: 定义宏 `_mm_mask_shldi_epi64(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L266 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_128`.
  **L266 CN**: 继续与可调用符号 `__builtin_ia32_selectq_128` 相关的逻辑。
- **L267 EN**: Continues logic associated with callable symbol `_mm_shldi_epi64`.
  **L267 CN**: 继续与可调用符号 `_mm_shldi_epi64` 相关的逻辑。
- **L268 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(S)))`.
  **L268 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(S)))`。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L270 EN**: Defines macro `_mm_maskz_shldi_epi64(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L270 CN**: 定义宏 `_mm_maskz_shldi_epi64(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L271 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_128`.
  **L271 CN**: 继续与可调用符号 `__builtin_ia32_selectq_128` 相关的逻辑。
- **L272 EN**: Continues logic associated with callable symbol `_mm_shldi_epi64`.
  **L272 CN**: 继续与可调用符号 `_mm_shldi_epi64` 相关的逻辑。
- **L273 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L273 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L275 EN**: Defines macro `_mm256_shldi_epi32(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L275 CN**: 定义宏 `_mm256_shldi_epi32(A, B, I)`，用于条件编译、简写或 API 生成。
- **L276 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshldd256`.
  **L276 CN**: 继续与可调用符号 `__builtin_ia32_vpshldd256` 相关的逻辑。
- **L277 EN**: Continues the surrounding expression or declaration: `(__v8si)(__m256i)(B), (int)(I)))`.
  **L277 CN**: 继续构造周围的表达式或声明：`(__v8si)(__m256i)(B), (int)(I)))`。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L279 EN**: Defines macro `_mm256_mask_shldi_epi32(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L279 CN**: 定义宏 `_mm256_mask_shldi_epi32(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L280 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_256`.
  **L280 CN**: 继续与可调用符号 `__builtin_ia32_selectd_256` 相关的逻辑。

### Lines 281-300

````c
                                     (__v8si)_mm256_shldi_epi32((A), (B), (I)), \
                                     (__v8si)(__m256i)(S)))

#define _mm256_maskz_shldi_epi32(U, A, B, I) \
  ((__m256i)__builtin_ia32_selectd_256((__mmask8)(U), \
                                     (__v8si)_mm256_shldi_epi32((A), (B), (I)), \
                                     (__v8si)_mm256_setzero_si256()))

#define _mm_shldi_epi32(A, B, I) \
  ((__m128i)__builtin_ia32_vpshldd128((__v4si)(__m128i)(A), \
                                      (__v4si)(__m128i)(B), (int)(I)))

#define _mm_mask_shldi_epi32(S, U, A, B, I) \
  ((__m128i)__builtin_ia32_selectd_128((__mmask8)(U), \
                                       (__v4si)_mm_shldi_epi32((A), (B), (I)), \
                                       (__v4si)(__m128i)(S)))

#define _mm_maskz_shldi_epi32(U, A, B, I) \
  ((__m128i)__builtin_ia32_selectd_128((__mmask8)(U), \
                                       (__v4si)_mm_shldi_epi32((A), (B), (I)), \
````
- **L281 EN**: Continues logic associated with callable symbol `_mm256_shldi_epi32`.
  **L281 CN**: 继续与可调用符号 `_mm256_shldi_epi32` 相关的逻辑。
- **L282 EN**: Continues the surrounding expression or declaration: `(__v8si)(__m256i)(S)))`.
  **L282 CN**: 继续构造周围的表达式或声明：`(__v8si)(__m256i)(S)))`。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L284 EN**: Defines macro `_mm256_maskz_shldi_epi32(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L284 CN**: 定义宏 `_mm256_maskz_shldi_epi32(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L285 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_256`.
  **L285 CN**: 继续与可调用符号 `__builtin_ia32_selectd_256` 相关的逻辑。
- **L286 EN**: Continues logic associated with callable symbol `_mm256_shldi_epi32`.
  **L286 CN**: 继续与可调用符号 `_mm256_shldi_epi32` 相关的逻辑。
- **L287 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L287 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L289 EN**: Defines macro `_mm_shldi_epi32(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L289 CN**: 定义宏 `_mm_shldi_epi32(A, B, I)`，用于条件编译、简写或 API 生成。
- **L290 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshldd128`.
  **L290 CN**: 继续与可调用符号 `__builtin_ia32_vpshldd128` 相关的逻辑。
- **L291 EN**: Continues the surrounding expression or declaration: `(__v4si)(__m128i)(B), (int)(I)))`.
  **L291 CN**: 继续构造周围的表达式或声明：`(__v4si)(__m128i)(B), (int)(I)))`。
- **L292 EN**: Blank line separating nearby declarations or logic blocks.
  **L292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L293 EN**: Defines macro `_mm_mask_shldi_epi32(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L293 CN**: 定义宏 `_mm_mask_shldi_epi32(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L294 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_128`.
  **L294 CN**: 继续与可调用符号 `__builtin_ia32_selectd_128` 相关的逻辑。
- **L295 EN**: Continues logic associated with callable symbol `_mm_shldi_epi32`.
  **L295 CN**: 继续与可调用符号 `_mm_shldi_epi32` 相关的逻辑。
- **L296 EN**: Continues the surrounding expression or declaration: `(__v4si)(__m128i)(S)))`.
  **L296 CN**: 继续构造周围的表达式或声明：`(__v4si)(__m128i)(S)))`。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L298 EN**: Defines macro `_mm_maskz_shldi_epi32(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L298 CN**: 定义宏 `_mm_maskz_shldi_epi32(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L299 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_128`.
  **L299 CN**: 继续与可调用符号 `__builtin_ia32_selectd_128` 相关的逻辑。
- **L300 EN**: Continues logic associated with callable symbol `_mm_shldi_epi32`.
  **L300 CN**: 继续与可调用符号 `_mm_shldi_epi32` 相关的逻辑。

### Lines 301-320

````c
                                       (__v4si)_mm_setzero_si128()))

#define _mm256_shldi_epi16(A, B, I) \
  ((__m256i)__builtin_ia32_vpshldw256((__v16hi)(__m256i)(A), \
                                      (__v16hi)(__m256i)(B), (int)(I)))

#define _mm256_mask_shldi_epi16(S, U, A, B, I) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                    (__v16hi)_mm256_shldi_epi16((A), (B), (I)), \
                                    (__v16hi)(__m256i)(S)))

#define _mm256_maskz_shldi_epi16(U, A, B, I) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                    (__v16hi)_mm256_shldi_epi16((A), (B), (I)), \
                                    (__v16hi)_mm256_setzero_si256()))

#define _mm_shldi_epi16(A, B, I) \
  ((__m128i)__builtin_ia32_vpshldw128((__v8hi)(__m128i)(A), \
                                      (__v8hi)(__m128i)(B), (int)(I)))

````
- **L301 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L301 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L303 EN**: Defines macro `_mm256_shldi_epi16(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L303 CN**: 定义宏 `_mm256_shldi_epi16(A, B, I)`，用于条件编译、简写或 API 生成。
- **L304 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshldw256`.
  **L304 CN**: 继续与可调用符号 `__builtin_ia32_vpshldw256` 相关的逻辑。
- **L305 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(B), (int)(I)))`.
  **L305 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(B), (int)(I)))`。
- **L306 EN**: Blank line separating nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L307 EN**: Defines macro `_mm256_mask_shldi_epi16(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L307 CN**: 定义宏 `_mm256_mask_shldi_epi16(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L308 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L308 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L309 EN**: Continues logic associated with callable symbol `_mm256_shldi_epi16`.
  **L309 CN**: 继续与可调用符号 `_mm256_shldi_epi16` 相关的逻辑。
- **L310 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(S)))`.
  **L310 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(S)))`。
- **L311 EN**: Blank line separating nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L312 EN**: Defines macro `_mm256_maskz_shldi_epi16(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L312 CN**: 定义宏 `_mm256_maskz_shldi_epi16(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L313 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L313 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L314 EN**: Continues logic associated with callable symbol `_mm256_shldi_epi16`.
  **L314 CN**: 继续与可调用符号 `_mm256_shldi_epi16` 相关的逻辑。
- **L315 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L315 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L317 EN**: Defines macro `_mm_shldi_epi16(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L317 CN**: 定义宏 `_mm_shldi_epi16(A, B, I)`，用于条件编译、简写或 API 生成。
- **L318 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshldw128`.
  **L318 CN**: 继续与可调用符号 `__builtin_ia32_vpshldw128` 相关的逻辑。
- **L319 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(B), (int)(I)))`.
  **L319 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(B), (int)(I)))`。
- **L320 EN**: Blank line separating nearby declarations or logic blocks.
  **L320 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 321-340

````c
#define _mm_mask_shldi_epi16(S, U, A, B, I) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                       (__v8hi)_mm_shldi_epi16((A), (B), (I)), \
                                       (__v8hi)(__m128i)(S)))

#define _mm_maskz_shldi_epi16(U, A, B, I) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                       (__v8hi)_mm_shldi_epi16((A), (B), (I)), \
                                       (__v8hi)_mm_setzero_si128()))

#define _mm256_shrdi_epi64(A, B, I) \
  ((__m256i)__builtin_ia32_vpshrdq256((__v4di)(__m256i)(A), \
                                      (__v4di)(__m256i)(B), (int)(I)))

#define _mm256_mask_shrdi_epi64(S, U, A, B, I) \
  ((__m256i)__builtin_ia32_selectq_256((__mmask8)(U), \
                                     (__v4di)_mm256_shrdi_epi64((A), (B), (I)), \
                                     (__v4di)(__m256i)(S)))

#define _mm256_maskz_shrdi_epi64(U, A, B, I) \
````
- **L321 EN**: Defines macro `_mm_mask_shldi_epi16(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L321 CN**: 定义宏 `_mm_mask_shldi_epi16(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L322 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L322 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L323 EN**: Continues logic associated with callable symbol `_mm_shldi_epi16`.
  **L323 CN**: 继续与可调用符号 `_mm_shldi_epi16` 相关的逻辑。
- **L324 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(S)))`.
  **L324 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(S)))`。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L326 EN**: Defines macro `_mm_maskz_shldi_epi16(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L326 CN**: 定义宏 `_mm_maskz_shldi_epi16(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L327 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L327 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L328 EN**: Continues logic associated with callable symbol `_mm_shldi_epi16`.
  **L328 CN**: 继续与可调用符号 `_mm_shldi_epi16` 相关的逻辑。
- **L329 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L329 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L331 EN**: Defines macro `_mm256_shrdi_epi64(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L331 CN**: 定义宏 `_mm256_shrdi_epi64(A, B, I)`，用于条件编译、简写或 API 生成。
- **L332 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshrdq256`.
  **L332 CN**: 继续与可调用符号 `__builtin_ia32_vpshrdq256` 相关的逻辑。
- **L333 EN**: Continues the surrounding expression or declaration: `(__v4di)(__m256i)(B), (int)(I)))`.
  **L333 CN**: 继续构造周围的表达式或声明：`(__v4di)(__m256i)(B), (int)(I)))`。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L335 EN**: Defines macro `_mm256_mask_shrdi_epi64(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L335 CN**: 定义宏 `_mm256_mask_shrdi_epi64(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L336 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_256`.
  **L336 CN**: 继续与可调用符号 `__builtin_ia32_selectq_256` 相关的逻辑。
- **L337 EN**: Continues logic associated with callable symbol `_mm256_shrdi_epi64`.
  **L337 CN**: 继续与可调用符号 `_mm256_shrdi_epi64` 相关的逻辑。
- **L338 EN**: Continues the surrounding expression or declaration: `(__v4di)(__m256i)(S)))`.
  **L338 CN**: 继续构造周围的表达式或声明：`(__v4di)(__m256i)(S)))`。
- **L339 EN**: Blank line separating nearby declarations or logic blocks.
  **L339 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L340 EN**: Defines macro `_mm256_maskz_shrdi_epi64(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L340 CN**: 定义宏 `_mm256_maskz_shrdi_epi64(U, A, B, I)`，用于条件编译、简写或 API 生成。

### Lines 341-360

````c
  ((__m256i)__builtin_ia32_selectq_256((__mmask8)(U), \
                                     (__v4di)_mm256_shrdi_epi64((A), (B), (I)), \
                                     (__v4di)_mm256_setzero_si256()))

#define _mm_shrdi_epi64(A, B, I) \
  ((__m128i)__builtin_ia32_vpshrdq128((__v2di)(__m128i)(A), \
                                      (__v2di)(__m128i)(B), (int)(I)))

#define _mm_mask_shrdi_epi64(S, U, A, B, I) \
  ((__m128i)__builtin_ia32_selectq_128((__mmask8)(U), \
                                       (__v2di)_mm_shrdi_epi64((A), (B), (I)), \
                                       (__v2di)(__m128i)(S)))

#define _mm_maskz_shrdi_epi64(U, A, B, I) \
  ((__m128i)__builtin_ia32_selectq_128((__mmask8)(U), \
                                       (__v2di)_mm_shrdi_epi64((A), (B), (I)), \
                                       (__v2di)_mm_setzero_si128()))

#define _mm256_shrdi_epi32(A, B, I) \
  ((__m256i)__builtin_ia32_vpshrdd256((__v8si)(__m256i)(A), \
````
- **L341 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_256`.
  **L341 CN**: 继续与可调用符号 `__builtin_ia32_selectq_256` 相关的逻辑。
- **L342 EN**: Continues logic associated with callable symbol `_mm256_shrdi_epi64`.
  **L342 CN**: 继续与可调用符号 `_mm256_shrdi_epi64` 相关的逻辑。
- **L343 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L343 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L344 EN**: Blank line separating nearby declarations or logic blocks.
  **L344 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L345 EN**: Defines macro `_mm_shrdi_epi64(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L345 CN**: 定义宏 `_mm_shrdi_epi64(A, B, I)`，用于条件编译、简写或 API 生成。
- **L346 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshrdq128`.
  **L346 CN**: 继续与可调用符号 `__builtin_ia32_vpshrdq128` 相关的逻辑。
- **L347 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(B), (int)(I)))`.
  **L347 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(B), (int)(I)))`。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Defines macro `_mm_mask_shrdi_epi64(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L349 CN**: 定义宏 `_mm_mask_shrdi_epi64(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L350 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_128`.
  **L350 CN**: 继续与可调用符号 `__builtin_ia32_selectq_128` 相关的逻辑。
- **L351 EN**: Continues logic associated with callable symbol `_mm_shrdi_epi64`.
  **L351 CN**: 继续与可调用符号 `_mm_shrdi_epi64` 相关的逻辑。
- **L352 EN**: Continues the surrounding expression or declaration: `(__v2di)(__m128i)(S)))`.
  **L352 CN**: 继续构造周围的表达式或声明：`(__v2di)(__m128i)(S)))`。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L354 EN**: Defines macro `_mm_maskz_shrdi_epi64(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L354 CN**: 定义宏 `_mm_maskz_shrdi_epi64(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L355 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectq_128`.
  **L355 CN**: 继续与可调用符号 `__builtin_ia32_selectq_128` 相关的逻辑。
- **L356 EN**: Continues logic associated with callable symbol `_mm_shrdi_epi64`.
  **L356 CN**: 继续与可调用符号 `_mm_shrdi_epi64` 相关的逻辑。
- **L357 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L357 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L358 EN**: Blank line separating nearby declarations or logic blocks.
  **L358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L359 EN**: Defines macro `_mm256_shrdi_epi32(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L359 CN**: 定义宏 `_mm256_shrdi_epi32(A, B, I)`，用于条件编译、简写或 API 生成。
- **L360 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshrdd256`.
  **L360 CN**: 继续与可调用符号 `__builtin_ia32_vpshrdd256` 相关的逻辑。

### Lines 361-380

````c
                                      (__v8si)(__m256i)(B), (int)(I)))

#define _mm256_mask_shrdi_epi32(S, U, A, B, I) \
  ((__m256i)__builtin_ia32_selectd_256((__mmask8)(U), \
                                     (__v8si)_mm256_shrdi_epi32((A), (B), (I)), \
                                     (__v8si)(__m256i)(S)))

#define _mm256_maskz_shrdi_epi32(U, A, B, I) \
  ((__m256i)__builtin_ia32_selectd_256((__mmask8)(U), \
                                     (__v8si)_mm256_shrdi_epi32((A), (B), (I)), \
                                     (__v8si)_mm256_setzero_si256()))

#define _mm_shrdi_epi32(A, B, I) \
  ((__m128i)__builtin_ia32_vpshrdd128((__v4si)(__m128i)(A), \
                                      (__v4si)(__m128i)(B), (int)(I)))

#define _mm_mask_shrdi_epi32(S, U, A, B, I) \
  ((__m128i)__builtin_ia32_selectd_128((__mmask8)(U), \
                                       (__v4si)_mm_shrdi_epi32((A), (B), (I)), \
                                       (__v4si)(__m128i)(S)))
````
- **L361 EN**: Continues the surrounding expression or declaration: `(__v8si)(__m256i)(B), (int)(I)))`.
  **L361 CN**: 继续构造周围的表达式或声明：`(__v8si)(__m256i)(B), (int)(I)))`。
- **L362 EN**: Blank line separating nearby declarations or logic blocks.
  **L362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L363 EN**: Defines macro `_mm256_mask_shrdi_epi32(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L363 CN**: 定义宏 `_mm256_mask_shrdi_epi32(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L364 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_256`.
  **L364 CN**: 继续与可调用符号 `__builtin_ia32_selectd_256` 相关的逻辑。
- **L365 EN**: Continues logic associated with callable symbol `_mm256_shrdi_epi32`.
  **L365 CN**: 继续与可调用符号 `_mm256_shrdi_epi32` 相关的逻辑。
- **L366 EN**: Continues the surrounding expression or declaration: `(__v8si)(__m256i)(S)))`.
  **L366 CN**: 继续构造周围的表达式或声明：`(__v8si)(__m256i)(S)))`。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L368 EN**: Defines macro `_mm256_maskz_shrdi_epi32(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L368 CN**: 定义宏 `_mm256_maskz_shrdi_epi32(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L369 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_256`.
  **L369 CN**: 继续与可调用符号 `__builtin_ia32_selectd_256` 相关的逻辑。
- **L370 EN**: Continues logic associated with callable symbol `_mm256_shrdi_epi32`.
  **L370 CN**: 继续与可调用符号 `_mm256_shrdi_epi32` 相关的逻辑。
- **L371 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L371 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L373 EN**: Defines macro `_mm_shrdi_epi32(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L373 CN**: 定义宏 `_mm_shrdi_epi32(A, B, I)`，用于条件编译、简写或 API 生成。
- **L374 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshrdd128`.
  **L374 CN**: 继续与可调用符号 `__builtin_ia32_vpshrdd128` 相关的逻辑。
- **L375 EN**: Continues the surrounding expression or declaration: `(__v4si)(__m128i)(B), (int)(I)))`.
  **L375 CN**: 继续构造周围的表达式或声明：`(__v4si)(__m128i)(B), (int)(I)))`。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L377 EN**: Defines macro `_mm_mask_shrdi_epi32(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L377 CN**: 定义宏 `_mm_mask_shrdi_epi32(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L378 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_128`.
  **L378 CN**: 继续与可调用符号 `__builtin_ia32_selectd_128` 相关的逻辑。
- **L379 EN**: Continues logic associated with callable symbol `_mm_shrdi_epi32`.
  **L379 CN**: 继续与可调用符号 `_mm_shrdi_epi32` 相关的逻辑。
- **L380 EN**: Continues the surrounding expression or declaration: `(__v4si)(__m128i)(S)))`.
  **L380 CN**: 继续构造周围的表达式或声明：`(__v4si)(__m128i)(S)))`。

### Lines 381-400

````c

#define _mm_maskz_shrdi_epi32(U, A, B, I) \
  ((__m128i)__builtin_ia32_selectd_128((__mmask8)(U), \
                                       (__v4si)_mm_shrdi_epi32((A), (B), (I)), \
                                       (__v4si)_mm_setzero_si128()))

#define _mm256_shrdi_epi16(A, B, I) \
  ((__m256i)__builtin_ia32_vpshrdw256((__v16hi)(__m256i)(A), \
                                      (__v16hi)(__m256i)(B), (int)(I)))

#define _mm256_mask_shrdi_epi16(S, U, A, B, I) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                    (__v16hi)_mm256_shrdi_epi16((A), (B), (I)), \
                                    (__v16hi)(__m256i)(S)))

#define _mm256_maskz_shrdi_epi16(U, A, B, I) \
  ((__m256i)__builtin_ia32_selectw_256((__mmask16)(U), \
                                    (__v16hi)_mm256_shrdi_epi16((A), (B), (I)), \
                                    (__v16hi)_mm256_setzero_si256()))

````
- **L381 EN**: Blank line separating nearby declarations or logic blocks.
  **L381 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L382 EN**: Defines macro `_mm_maskz_shrdi_epi32(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L382 CN**: 定义宏 `_mm_maskz_shrdi_epi32(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L383 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectd_128`.
  **L383 CN**: 继续与可调用符号 `__builtin_ia32_selectd_128` 相关的逻辑。
- **L384 EN**: Continues logic associated with callable symbol `_mm_shrdi_epi32`.
  **L384 CN**: 继续与可调用符号 `_mm_shrdi_epi32` 相关的逻辑。
- **L385 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L385 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L387 EN**: Defines macro `_mm256_shrdi_epi16(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L387 CN**: 定义宏 `_mm256_shrdi_epi16(A, B, I)`，用于条件编译、简写或 API 生成。
- **L388 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshrdw256`.
  **L388 CN**: 继续与可调用符号 `__builtin_ia32_vpshrdw256` 相关的逻辑。
- **L389 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(B), (int)(I)))`.
  **L389 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(B), (int)(I)))`。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L391 EN**: Defines macro `_mm256_mask_shrdi_epi16(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L391 CN**: 定义宏 `_mm256_mask_shrdi_epi16(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L392 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L392 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L393 EN**: Continues logic associated with callable symbol `_mm256_shrdi_epi16`.
  **L393 CN**: 继续与可调用符号 `_mm256_shrdi_epi16` 相关的逻辑。
- **L394 EN**: Continues the surrounding expression or declaration: `(__v16hi)(__m256i)(S)))`.
  **L394 CN**: 继续构造周围的表达式或声明：`(__v16hi)(__m256i)(S)))`。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Defines macro `_mm256_maskz_shrdi_epi16(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L396 CN**: 定义宏 `_mm256_maskz_shrdi_epi16(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L397 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_256`.
  **L397 CN**: 继续与可调用符号 `__builtin_ia32_selectw_256` 相关的逻辑。
- **L398 EN**: Continues logic associated with callable symbol `_mm256_shrdi_epi16`.
  **L398 CN**: 继续与可调用符号 `_mm256_shrdi_epi16` 相关的逻辑。
- **L399 EN**: Continues logic associated with callable symbol `_mm256_setzero_si256`.
  **L399 CN**: 继续与可调用符号 `_mm256_setzero_si256` 相关的逻辑。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 401-420

````c
#define _mm_shrdi_epi16(A, B, I) \
  ((__m128i)__builtin_ia32_vpshrdw128((__v8hi)(__m128i)(A), \
                                      (__v8hi)(__m128i)(B), (int)(I)))

#define _mm_mask_shrdi_epi16(S, U, A, B, I) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                       (__v8hi)_mm_shrdi_epi16((A), (B), (I)), \
                                       (__v8hi)(__m128i)(S)))

#define _mm_maskz_shrdi_epi16(U, A, B, I) \
  ((__m128i)__builtin_ia32_selectw_128((__mmask8)(U), \
                                       (__v8hi)_mm_shrdi_epi16((A), (B), (I)), \
                                       (__v8hi)_mm_setzero_si128()))

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_shldv_epi64(__m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_elementwise_fshl((__v4du)__A, (__v4du)__B,
                                             (__v4du)__C);
}
````
- **L401 EN**: Defines macro `_mm_shrdi_epi16(A, B, I)` for conditional compilation, shorthand, or API generation.
  **L401 CN**: 定义宏 `_mm_shrdi_epi16(A, B, I)`，用于条件编译、简写或 API 生成。
- **L402 EN**: Continues logic associated with callable symbol `__builtin_ia32_vpshrdw128`.
  **L402 CN**: 继续与可调用符号 `__builtin_ia32_vpshrdw128` 相关的逻辑。
- **L403 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(B), (int)(I)))`.
  **L403 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(B), (int)(I)))`。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L405 EN**: Defines macro `_mm_mask_shrdi_epi16(S, U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L405 CN**: 定义宏 `_mm_mask_shrdi_epi16(S, U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L406 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L406 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L407 EN**: Continues logic associated with callable symbol `_mm_shrdi_epi16`.
  **L407 CN**: 继续与可调用符号 `_mm_shrdi_epi16` 相关的逻辑。
- **L408 EN**: Continues the surrounding expression or declaration: `(__v8hi)(__m128i)(S)))`.
  **L408 CN**: 继续构造周围的表达式或声明：`(__v8hi)(__m128i)(S)))`。
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L410 EN**: Defines macro `_mm_maskz_shrdi_epi16(U, A, B, I)` for conditional compilation, shorthand, or API generation.
  **L410 CN**: 定义宏 `_mm_maskz_shrdi_epi16(U, A, B, I)`，用于条件编译、简写或 API 生成。
- **L411 EN**: Continues logic associated with callable symbol `__builtin_ia32_selectw_128`.
  **L411 CN**: 继续与可调用符号 `__builtin_ia32_selectw_128` 相关的逻辑。
- **L412 EN**: Continues logic associated with callable symbol `_mm_shrdi_epi16`.
  **L412 CN**: 继续与可调用符号 `_mm_shrdi_epi16` 相关的逻辑。
- **L413 EN**: Continues logic associated with callable symbol `_mm_setzero_si128`.
  **L413 CN**: 继续与可调用符号 `_mm_setzero_si128` 相关的逻辑。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L415 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L415 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L416 EN**: Continues logic associated with callable symbol `_mm256_shldv_epi64`.
  **L416 CN**: 继续与可调用符号 `_mm256_shldv_epi64` 相关的逻辑。
- **L417 EN**: Opens a new lexical scope or compound statement.
  **L417 CN**: 打开一个新的词法作用域或复合语句块。
- **L418 EN**: Returns from the current function with `(__m256i)__builtin_elementwise_fshl((__v4du)__A, (__v4du)__B,`.
  **L418 CN**: 以 `(__m256i)__builtin_elementwise_fshl((__v4du)__A, (__v4du)__B,` 从当前函数返回。
- **L419 EN**: Executes a call or declaration centered on `statement`.
  **L419 CN**: 执行以 `statement` 为核心的调用或声明。
- **L420 EN**: Closes the current lexical scope or compound statement.
  **L420 CN**: 结束当前词法作用域或复合语句块。

### Lines 421-440

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_shldv_epi64(__m256i __A, __mmask8 __U, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectq_256(__U,
                                      (__v4di)_mm256_shldv_epi64(__A, __B, __C),
                                      (__v4di)__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_shldv_epi64(__mmask8 __U, __m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectq_256(__U,
                                      (__v4di)_mm256_shldv_epi64(__A, __B, __C),
                                      (__v4di)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_shldv_epi64(__m128i __A, __m128i __B, __m128i __C)
{
````
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L422 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L422 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L423 EN**: Continues logic associated with callable symbol `_mm256_mask_shldv_epi64`.
  **L423 CN**: 继续与可调用符号 `_mm256_mask_shldv_epi64` 相关的逻辑。
- **L424 EN**: Opens a new lexical scope or compound statement.
  **L424 CN**: 打开一个新的词法作用域或复合语句块。
- **L425 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectq_256(__U,`.
  **L425 CN**: 以 `(__m256i)__builtin_ia32_selectq_256(__U,` 从当前函数返回。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di)_mm256_shldv_epi64(__A, __B, __C),`.
  **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di)_mm256_shldv_epi64(__A, __B, __C),`。
- **L427 EN**: Executes a call or declaration centered on `statement`.
  **L427 CN**: 执行以 `statement` 为核心的调用或声明。
- **L428 EN**: Closes the current lexical scope or compound statement.
  **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L430 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L430 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L431 EN**: Continues logic associated with callable symbol `_mm256_maskz_shldv_epi64`.
  **L431 CN**: 继续与可调用符号 `_mm256_maskz_shldv_epi64` 相关的逻辑。
- **L432 EN**: Opens a new lexical scope or compound statement.
  **L432 CN**: 打开一个新的词法作用域或复合语句块。
- **L433 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectq_256(__U,`.
  **L433 CN**: 以 `(__m256i)__builtin_ia32_selectq_256(__U,` 从当前函数返回。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di)_mm256_shldv_epi64(__A, __B, __C),`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di)_mm256_shldv_epi64(__A, __B, __C),`。
- **L435 EN**: Executes a call or declaration centered on `statement`.
  **L435 CN**: 执行以 `statement` 为核心的调用或声明。
- **L436 EN**: Closes the current lexical scope or compound statement.
  **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L438 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L439 EN**: Continues logic associated with callable symbol `_mm_shldv_epi64`.
  **L439 CN**: 继续与可调用符号 `_mm_shldv_epi64` 相关的逻辑。
- **L440 EN**: Opens a new lexical scope or compound statement.
  **L440 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 441-460

````c
  return (__m128i)__builtin_elementwise_fshl((__v2du)__A, (__v2du)__B,
                                             (__v2du)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_shldv_epi64(__m128i __A, __mmask8 __U, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectq_128(__U,
                                         (__v2di)_mm_shldv_epi64(__A, __B, __C),
                                         (__v2di)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_shldv_epi64(__mmask8 __U, __m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectq_128(__U,
                                         (__v2di)_mm_shldv_epi64(__A, __B, __C),
                                         (__v2di)_mm_setzero_si128());
}

````
- **L441 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshl((__v2du)__A, (__v2du)__B,`.
  **L441 CN**: 以 `(__m128i)__builtin_elementwise_fshl((__v2du)__A, (__v2du)__B,` 从当前函数返回。
- **L442 EN**: Executes a call or declaration centered on `statement`.
  **L442 CN**: 执行以 `statement` 为核心的调用或声明。
- **L443 EN**: Closes the current lexical scope or compound statement.
  **L443 CN**: 结束当前词法作用域或复合语句块。
- **L444 EN**: Blank line separating nearby declarations or logic blocks.
  **L444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L445 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L445 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L446 EN**: Continues logic associated with callable symbol `_mm_mask_shldv_epi64`.
  **L446 CN**: 继续与可调用符号 `_mm_mask_shldv_epi64` 相关的逻辑。
- **L447 EN**: Opens a new lexical scope or compound statement.
  **L447 CN**: 打开一个新的词法作用域或复合语句块。
- **L448 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectq_128(__U,`.
  **L448 CN**: 以 `(__m128i)__builtin_ia32_selectq_128(__U,` 从当前函数返回。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di)_mm_shldv_epi64(__A, __B, __C),`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di)_mm_shldv_epi64(__A, __B, __C),`。
- **L450 EN**: Executes a call or declaration centered on `statement`.
  **L450 CN**: 执行以 `statement` 为核心的调用或声明。
- **L451 EN**: Closes the current lexical scope or compound statement.
  **L451 CN**: 结束当前词法作用域或复合语句块。
- **L452 EN**: Blank line separating nearby declarations or logic blocks.
  **L452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L453 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L453 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L454 EN**: Continues logic associated with callable symbol `_mm_maskz_shldv_epi64`.
  **L454 CN**: 继续与可调用符号 `_mm_maskz_shldv_epi64` 相关的逻辑。
- **L455 EN**: Opens a new lexical scope or compound statement.
  **L455 CN**: 打开一个新的词法作用域或复合语句块。
- **L456 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectq_128(__U,`.
  **L456 CN**: 以 `(__m128i)__builtin_ia32_selectq_128(__U,` 从当前函数返回。
- **L457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di)_mm_shldv_epi64(__A, __B, __C),`.
  **L457 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di)_mm_shldv_epi64(__A, __B, __C),`。
- **L458 EN**: Executes a call or declaration centered on `statement`.
  **L458 CN**: 执行以 `statement` 为核心的调用或声明。
- **L459 EN**: Closes the current lexical scope or compound statement.
  **L459 CN**: 结束当前词法作用域或复合语句块。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 461-480

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_shldv_epi32(__m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_elementwise_fshl((__v8su)__A, (__v8su)__B,
                                             (__v8su)__C);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_shldv_epi32(__m256i __A, __mmask8 __U, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectd_256(__U,
                                      (__v8si)_mm256_shldv_epi32(__A, __B, __C),
                                      (__v8si)__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_shldv_epi32(__mmask8 __U, __m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectd_256(__U,
                                      (__v8si)_mm256_shldv_epi32(__A, __B, __C),
````
- **L461 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L461 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L462 EN**: Continues logic associated with callable symbol `_mm256_shldv_epi32`.
  **L462 CN**: 继续与可调用符号 `_mm256_shldv_epi32` 相关的逻辑。
- **L463 EN**: Opens a new lexical scope or compound statement.
  **L463 CN**: 打开一个新的词法作用域或复合语句块。
- **L464 EN**: Returns from the current function with `(__m256i)__builtin_elementwise_fshl((__v8su)__A, (__v8su)__B,`.
  **L464 CN**: 以 `(__m256i)__builtin_elementwise_fshl((__v8su)__A, (__v8su)__B,` 从当前函数返回。
- **L465 EN**: Executes a call or declaration centered on `statement`.
  **L465 CN**: 执行以 `statement` 为核心的调用或声明。
- **L466 EN**: Closes the current lexical scope or compound statement.
  **L466 CN**: 结束当前词法作用域或复合语句块。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L468 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L469 EN**: Continues logic associated with callable symbol `_mm256_mask_shldv_epi32`.
  **L469 CN**: 继续与可调用符号 `_mm256_mask_shldv_epi32` 相关的逻辑。
- **L470 EN**: Opens a new lexical scope or compound statement.
  **L470 CN**: 打开一个新的词法作用域或复合语句块。
- **L471 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectd_256(__U,`.
  **L471 CN**: 以 `(__m256i)__builtin_ia32_selectd_256(__U,` 从当前函数返回。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8si)_mm256_shldv_epi32(__A, __B, __C),`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8si)_mm256_shldv_epi32(__A, __B, __C),`。
- **L473 EN**: Executes a call or declaration centered on `statement`.
  **L473 CN**: 执行以 `statement` 为核心的调用或声明。
- **L474 EN**: Closes the current lexical scope or compound statement.
  **L474 CN**: 结束当前词法作用域或复合语句块。
- **L475 EN**: Blank line separating nearby declarations or logic blocks.
  **L475 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L476 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L476 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L477 EN**: Continues logic associated with callable symbol `_mm256_maskz_shldv_epi32`.
  **L477 CN**: 继续与可调用符号 `_mm256_maskz_shldv_epi32` 相关的逻辑。
- **L478 EN**: Opens a new lexical scope or compound statement.
  **L478 CN**: 打开一个新的词法作用域或复合语句块。
- **L479 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectd_256(__U,`.
  **L479 CN**: 以 `(__m256i)__builtin_ia32_selectd_256(__U,` 从当前函数返回。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8si)_mm256_shldv_epi32(__A, __B, __C),`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8si)_mm256_shldv_epi32(__A, __B, __C),`。

### Lines 481-500

````c
                                      (__v8si)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_shldv_epi32(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_elementwise_fshl((__v4su)__A, (__v4su)__B,
                                             (__v4su)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_shldv_epi32(__m128i __A, __mmask8 __U, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectd_128(__U,
                                         (__v4si)_mm_shldv_epi32(__A, __B, __C),
                                         (__v4si)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_shldv_epi32(__mmask8 __U, __m128i __A, __m128i __B, __m128i __C)
````
- **L481 EN**: Executes a call or declaration centered on `statement`.
  **L481 CN**: 执行以 `statement` 为核心的调用或声明。
- **L482 EN**: Closes the current lexical scope or compound statement.
  **L482 CN**: 结束当前词法作用域或复合语句块。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L484 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L484 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L485 EN**: Continues logic associated with callable symbol `_mm_shldv_epi32`.
  **L485 CN**: 继续与可调用符号 `_mm_shldv_epi32` 相关的逻辑。
- **L486 EN**: Opens a new lexical scope or compound statement.
  **L486 CN**: 打开一个新的词法作用域或复合语句块。
- **L487 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshl((__v4su)__A, (__v4su)__B,`.
  **L487 CN**: 以 `(__m128i)__builtin_elementwise_fshl((__v4su)__A, (__v4su)__B,` 从当前函数返回。
- **L488 EN**: Executes a call or declaration centered on `statement`.
  **L488 CN**: 执行以 `statement` 为核心的调用或声明。
- **L489 EN**: Closes the current lexical scope or compound statement.
  **L489 CN**: 结束当前词法作用域或复合语句块。
- **L490 EN**: Blank line separating nearby declarations or logic blocks.
  **L490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L491 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L491 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L492 EN**: Continues logic associated with callable symbol `_mm_mask_shldv_epi32`.
  **L492 CN**: 继续与可调用符号 `_mm_mask_shldv_epi32` 相关的逻辑。
- **L493 EN**: Opens a new lexical scope or compound statement.
  **L493 CN**: 打开一个新的词法作用域或复合语句块。
- **L494 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectd_128(__U,`.
  **L494 CN**: 以 `(__m128i)__builtin_ia32_selectd_128(__U,` 从当前函数返回。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4si)_mm_shldv_epi32(__A, __B, __C),`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4si)_mm_shldv_epi32(__A, __B, __C),`。
- **L496 EN**: Executes a call or declaration centered on `statement`.
  **L496 CN**: 执行以 `statement` 为核心的调用或声明。
- **L497 EN**: Closes the current lexical scope or compound statement.
  **L497 CN**: 结束当前词法作用域或复合语句块。
- **L498 EN**: Blank line separating nearby declarations or logic blocks.
  **L498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L499 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L499 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L500 EN**: Continues logic associated with callable symbol `_mm_maskz_shldv_epi32`.
  **L500 CN**: 继续与可调用符号 `_mm_maskz_shldv_epi32` 相关的逻辑。

### Lines 501-520

````c
{
  return (__m128i)__builtin_ia32_selectd_128(__U,
                                         (__v4si)_mm_shldv_epi32(__A, __B, __C),
                                         (__v4si)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_shldv_epi16(__m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_elementwise_fshl((__v16hu)__A, (__v16hu)__B,
                                             (__v16hu)__C);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_shldv_epi16(__m256i __A, __mmask16 __U, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectw_256(__U,
                                      (__v16hi)_mm256_shldv_epi16(__A, __B, __C),
                                      (__v16hi)__A);
}
````
- **L501 EN**: Opens a new lexical scope or compound statement.
  **L501 CN**: 打开一个新的词法作用域或复合语句块。
- **L502 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectd_128(__U,`.
  **L502 CN**: 以 `(__m128i)__builtin_ia32_selectd_128(__U,` 从当前函数返回。
- **L503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4si)_mm_shldv_epi32(__A, __B, __C),`.
  **L503 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4si)_mm_shldv_epi32(__A, __B, __C),`。
- **L504 EN**: Executes a call or declaration centered on `statement`.
  **L504 CN**: 执行以 `statement` 为核心的调用或声明。
- **L505 EN**: Closes the current lexical scope or compound statement.
  **L505 CN**: 结束当前词法作用域或复合语句块。
- **L506 EN**: Blank line separating nearby declarations or logic blocks.
  **L506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L507 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L507 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L508 EN**: Continues logic associated with callable symbol `_mm256_shldv_epi16`.
  **L508 CN**: 继续与可调用符号 `_mm256_shldv_epi16` 相关的逻辑。
- **L509 EN**: Opens a new lexical scope or compound statement.
  **L509 CN**: 打开一个新的词法作用域或复合语句块。
- **L510 EN**: Returns from the current function with `(__m256i)__builtin_elementwise_fshl((__v16hu)__A, (__v16hu)__B,`.
  **L510 CN**: 以 `(__m256i)__builtin_elementwise_fshl((__v16hu)__A, (__v16hu)__B,` 从当前函数返回。
- **L511 EN**: Executes a call or declaration centered on `statement`.
  **L511 CN**: 执行以 `statement` 为核心的调用或声明。
- **L512 EN**: Closes the current lexical scope or compound statement.
  **L512 CN**: 结束当前词法作用域或复合语句块。
- **L513 EN**: Blank line separating nearby declarations or logic blocks.
  **L513 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L514 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L514 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L515 EN**: Continues logic associated with callable symbol `_mm256_mask_shldv_epi16`.
  **L515 CN**: 继续与可调用符号 `_mm256_mask_shldv_epi16` 相关的逻辑。
- **L516 EN**: Opens a new lexical scope or compound statement.
  **L516 CN**: 打开一个新的词法作用域或复合语句块。
- **L517 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__U,`.
  **L517 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__U,` 从当前函数返回。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_shldv_epi16(__A, __B, __C),`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_shldv_epi16(__A, __B, __C),`。
- **L519 EN**: Executes a call or declaration centered on `statement`.
  **L519 CN**: 执行以 `statement` 为核心的调用或声明。
- **L520 EN**: Closes the current lexical scope or compound statement.
  **L520 CN**: 结束当前词法作用域或复合语句块。

### Lines 521-540

````c

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_shldv_epi16(__mmask16 __U, __m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectw_256(__U,
                                      (__v16hi)_mm256_shldv_epi16(__A, __B, __C),
                                      (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_shldv_epi16(__m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_elementwise_fshl((__v8hu)__A, (__v8hu)__B,
                                             (__v8hu)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_shldv_epi16(__m128i __A, __mmask8 __U, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectw_128(__U,
````
- **L521 EN**: Blank line separating nearby declarations or logic blocks.
  **L521 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L522 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L522 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L523 EN**: Continues logic associated with callable symbol `_mm256_maskz_shldv_epi16`.
  **L523 CN**: 继续与可调用符号 `_mm256_maskz_shldv_epi16` 相关的逻辑。
- **L524 EN**: Opens a new lexical scope or compound statement.
  **L524 CN**: 打开一个新的词法作用域或复合语句块。
- **L525 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__U,`.
  **L525 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__U,` 从当前函数返回。
- **L526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_shldv_epi16(__A, __B, __C),`.
  **L526 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_shldv_epi16(__A, __B, __C),`。
- **L527 EN**: Executes a call or declaration centered on `statement`.
  **L527 CN**: 执行以 `statement` 为核心的调用或声明。
- **L528 EN**: Closes the current lexical scope or compound statement.
  **L528 CN**: 结束当前词法作用域或复合语句块。
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L530 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L531 EN**: Continues logic associated with callable symbol `_mm_shldv_epi16`.
  **L531 CN**: 继续与可调用符号 `_mm_shldv_epi16` 相关的逻辑。
- **L532 EN**: Opens a new lexical scope or compound statement.
  **L532 CN**: 打开一个新的词法作用域或复合语句块。
- **L533 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshl((__v8hu)__A, (__v8hu)__B,`.
  **L533 CN**: 以 `(__m128i)__builtin_elementwise_fshl((__v8hu)__A, (__v8hu)__B,` 从当前函数返回。
- **L534 EN**: Executes a call or declaration centered on `statement`.
  **L534 CN**: 执行以 `statement` 为核心的调用或声明。
- **L535 EN**: Closes the current lexical scope or compound statement.
  **L535 CN**: 结束当前词法作用域或复合语句块。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L537 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L538 EN**: Continues logic associated with callable symbol `_mm_mask_shldv_epi16`.
  **L538 CN**: 继续与可调用符号 `_mm_mask_shldv_epi16` 相关的逻辑。
- **L539 EN**: Opens a new lexical scope or compound statement.
  **L539 CN**: 打开一个新的词法作用域或复合语句块。
- **L540 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__U,`.
  **L540 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__U,` 从当前函数返回。

### Lines 541-560

````c
                                         (__v8hi)_mm_shldv_epi16(__A, __B, __C),
                                         (__v8hi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_shldv_epi16(__mmask8 __U, __m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectw_128(__U,
                                         (__v8hi)_mm_shldv_epi16(__A, __B, __C),
                                         (__v8hi)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_shrdv_epi64(__m256i __A, __m256i __B, __m256i __C)
{
  // Ops __A and __B are swapped.
  return (__m256i)__builtin_elementwise_fshr((__v4du)__B, (__v4du)__A,
                                             (__v4du)__C);
}

````
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_shldv_epi16(__A, __B, __C),`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_shldv_epi16(__A, __B, __C),`。
- **L542 EN**: Executes a call or declaration centered on `statement`.
  **L542 CN**: 执行以 `statement` 为核心的调用或声明。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L545 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L545 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L546 EN**: Continues logic associated with callable symbol `_mm_maskz_shldv_epi16`.
  **L546 CN**: 继续与可调用符号 `_mm_maskz_shldv_epi16` 相关的逻辑。
- **L547 EN**: Opens a new lexical scope or compound statement.
  **L547 CN**: 打开一个新的词法作用域或复合语句块。
- **L548 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__U,`.
  **L548 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__U,` 从当前函数返回。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_shldv_epi16(__A, __B, __C),`.
  **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_shldv_epi16(__A, __B, __C),`。
- **L550 EN**: Executes a call or declaration centered on `statement`.
  **L550 CN**: 执行以 `statement` 为核心的调用或声明。
- **L551 EN**: Closes the current lexical scope or compound statement.
  **L551 CN**: 结束当前词法作用域或复合语句块。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L553 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L553 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L554 EN**: Continues logic associated with callable symbol `_mm256_shrdv_epi64`.
  **L554 CN**: 继续与可调用符号 `_mm256_shrdv_epi64` 相关的逻辑。
- **L555 EN**: Opens a new lexical scope or compound statement.
  **L555 CN**: 打开一个新的词法作用域或复合语句块。
- **L556 EN**: Comment explains nearby logic, constraints, or intent: `Ops __A and __B are swapped.`.
  **L556 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Ops __A and __B are swapped.`。
- **L557 EN**: Returns from the current function with `(__m256i)__builtin_elementwise_fshr((__v4du)__B, (__v4du)__A,`.
  **L557 CN**: 以 `(__m256i)__builtin_elementwise_fshr((__v4du)__B, (__v4du)__A,` 从当前函数返回。
- **L558 EN**: Executes a call or declaration centered on `statement`.
  **L558 CN**: 执行以 `statement` 为核心的调用或声明。
- **L559 EN**: Closes the current lexical scope or compound statement.
  **L559 CN**: 结束当前词法作用域或复合语句块。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 561-580

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_shrdv_epi64(__m256i __A, __mmask8 __U, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectq_256(__U,
                                      (__v4di)_mm256_shrdv_epi64(__A, __B, __C),
                                      (__v4di)__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_shrdv_epi64(__mmask8 __U, __m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectq_256(__U,
                                      (__v4di)_mm256_shrdv_epi64(__A, __B, __C),
                                      (__v4di)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_shrdv_epi64(__m128i __A, __m128i __B, __m128i __C)
{
  // Ops __A and __B are swapped.
````
- **L561 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L561 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L562 EN**: Continues logic associated with callable symbol `_mm256_mask_shrdv_epi64`.
  **L562 CN**: 继续与可调用符号 `_mm256_mask_shrdv_epi64` 相关的逻辑。
- **L563 EN**: Opens a new lexical scope or compound statement.
  **L563 CN**: 打开一个新的词法作用域或复合语句块。
- **L564 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectq_256(__U,`.
  **L564 CN**: 以 `(__m256i)__builtin_ia32_selectq_256(__U,` 从当前函数返回。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di)_mm256_shrdv_epi64(__A, __B, __C),`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di)_mm256_shrdv_epi64(__A, __B, __C),`。
- **L566 EN**: Executes a call or declaration centered on `statement`.
  **L566 CN**: 执行以 `statement` 为核心的调用或声明。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L569 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L569 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L570 EN**: Continues logic associated with callable symbol `_mm256_maskz_shrdv_epi64`.
  **L570 CN**: 继续与可调用符号 `_mm256_maskz_shrdv_epi64` 相关的逻辑。
- **L571 EN**: Opens a new lexical scope or compound statement.
  **L571 CN**: 打开一个新的词法作用域或复合语句块。
- **L572 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectq_256(__U,`.
  **L572 CN**: 以 `(__m256i)__builtin_ia32_selectq_256(__U,` 从当前函数返回。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4di)_mm256_shrdv_epi64(__A, __B, __C),`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4di)_mm256_shrdv_epi64(__A, __B, __C),`。
- **L574 EN**: Executes a call or declaration centered on `statement`.
  **L574 CN**: 执行以 `statement` 为核心的调用或声明。
- **L575 EN**: Closes the current lexical scope or compound statement.
  **L575 CN**: 结束当前词法作用域或复合语句块。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L577 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L577 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L578 EN**: Continues logic associated with callable symbol `_mm_shrdv_epi64`.
  **L578 CN**: 继续与可调用符号 `_mm_shrdv_epi64` 相关的逻辑。
- **L579 EN**: Opens a new lexical scope or compound statement.
  **L579 CN**: 打开一个新的词法作用域或复合语句块。
- **L580 EN**: Comment explains nearby logic, constraints, or intent: `Ops __A and __B are swapped.`.
  **L580 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Ops __A and __B are swapped.`。

### Lines 581-600

````c
  return (__m128i)__builtin_elementwise_fshr((__v2du)__B, (__v2du)__A,
                                             (__v2du)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_shrdv_epi64(__m128i __A, __mmask8 __U, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectq_128(__U,
                                         (__v2di)_mm_shrdv_epi64(__A, __B, __C),
                                         (__v2di)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_shrdv_epi64(__mmask8 __U, __m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectq_128(__U,
                                         (__v2di)_mm_shrdv_epi64(__A, __B, __C),
                                         (__v2di)_mm_setzero_si128());
}

````
- **L581 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshr((__v2du)__B, (__v2du)__A,`.
  **L581 CN**: 以 `(__m128i)__builtin_elementwise_fshr((__v2du)__B, (__v2du)__A,` 从当前函数返回。
- **L582 EN**: Executes a call or declaration centered on `statement`.
  **L582 CN**: 执行以 `statement` 为核心的调用或声明。
- **L583 EN**: Closes the current lexical scope or compound statement.
  **L583 CN**: 结束当前词法作用域或复合语句块。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L585 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L585 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L586 EN**: Continues logic associated with callable symbol `_mm_mask_shrdv_epi64`.
  **L586 CN**: 继续与可调用符号 `_mm_mask_shrdv_epi64` 相关的逻辑。
- **L587 EN**: Opens a new lexical scope or compound statement.
  **L587 CN**: 打开一个新的词法作用域或复合语句块。
- **L588 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectq_128(__U,`.
  **L588 CN**: 以 `(__m128i)__builtin_ia32_selectq_128(__U,` 从当前函数返回。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di)_mm_shrdv_epi64(__A, __B, __C),`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di)_mm_shrdv_epi64(__A, __B, __C),`。
- **L590 EN**: Executes a call or declaration centered on `statement`.
  **L590 CN**: 执行以 `statement` 为核心的调用或声明。
- **L591 EN**: Closes the current lexical scope or compound statement.
  **L591 CN**: 结束当前词法作用域或复合语句块。
- **L592 EN**: Blank line separating nearby declarations or logic blocks.
  **L592 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L593 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L593 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L594 EN**: Continues logic associated with callable symbol `_mm_maskz_shrdv_epi64`.
  **L594 CN**: 继续与可调用符号 `_mm_maskz_shrdv_epi64` 相关的逻辑。
- **L595 EN**: Opens a new lexical scope or compound statement.
  **L595 CN**: 打开一个新的词法作用域或复合语句块。
- **L596 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectq_128(__U,`.
  **L596 CN**: 以 `(__m128i)__builtin_ia32_selectq_128(__U,` 从当前函数返回。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v2di)_mm_shrdv_epi64(__A, __B, __C),`.
  **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v2di)_mm_shrdv_epi64(__A, __B, __C),`。
- **L598 EN**: Executes a call or declaration centered on `statement`.
  **L598 CN**: 执行以 `statement` 为核心的调用或声明。
- **L599 EN**: Closes the current lexical scope or compound statement.
  **L599 CN**: 结束当前词法作用域或复合语句块。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 601-620

````c
static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_shrdv_epi32(__m256i __A, __m256i __B, __m256i __C)
{
  // Ops __A and __B are swapped.
  return (__m256i)__builtin_elementwise_fshr((__v8su)__B, (__v8su)__A,
                                             (__v8su)__C);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_shrdv_epi32(__m256i __A, __mmask8 __U, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectd_256(__U,
                                      (__v8si)_mm256_shrdv_epi32(__A, __B, __C),
                                      (__v8si)__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_shrdv_epi32(__mmask8 __U, __m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectd_256(__U,
````
- **L601 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L601 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L602 EN**: Continues logic associated with callable symbol `_mm256_shrdv_epi32`.
  **L602 CN**: 继续与可调用符号 `_mm256_shrdv_epi32` 相关的逻辑。
- **L603 EN**: Opens a new lexical scope or compound statement.
  **L603 CN**: 打开一个新的词法作用域或复合语句块。
- **L604 EN**: Comment explains nearby logic, constraints, or intent: `Ops __A and __B are swapped.`.
  **L604 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Ops __A and __B are swapped.`。
- **L605 EN**: Returns from the current function with `(__m256i)__builtin_elementwise_fshr((__v8su)__B, (__v8su)__A,`.
  **L605 CN**: 以 `(__m256i)__builtin_elementwise_fshr((__v8su)__B, (__v8su)__A,` 从当前函数返回。
- **L606 EN**: Executes a call or declaration centered on `statement`.
  **L606 CN**: 执行以 `statement` 为核心的调用或声明。
- **L607 EN**: Closes the current lexical scope or compound statement.
  **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L609 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L610 EN**: Continues logic associated with callable symbol `_mm256_mask_shrdv_epi32`.
  **L610 CN**: 继续与可调用符号 `_mm256_mask_shrdv_epi32` 相关的逻辑。
- **L611 EN**: Opens a new lexical scope or compound statement.
  **L611 CN**: 打开一个新的词法作用域或复合语句块。
- **L612 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectd_256(__U,`.
  **L612 CN**: 以 `(__m256i)__builtin_ia32_selectd_256(__U,` 从当前函数返回。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8si)_mm256_shrdv_epi32(__A, __B, __C),`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8si)_mm256_shrdv_epi32(__A, __B, __C),`。
- **L614 EN**: Executes a call or declaration centered on `statement`.
  **L614 CN**: 执行以 `statement` 为核心的调用或声明。
- **L615 EN**: Closes the current lexical scope or compound statement.
  **L615 CN**: 结束当前词法作用域或复合语句块。
- **L616 EN**: Blank line separating nearby declarations or logic blocks.
  **L616 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L617 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L617 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L618 EN**: Continues logic associated with callable symbol `_mm256_maskz_shrdv_epi32`.
  **L618 CN**: 继续与可调用符号 `_mm256_maskz_shrdv_epi32` 相关的逻辑。
- **L619 EN**: Opens a new lexical scope or compound statement.
  **L619 CN**: 打开一个新的词法作用域或复合语句块。
- **L620 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectd_256(__U,`.
  **L620 CN**: 以 `(__m256i)__builtin_ia32_selectd_256(__U,` 从当前函数返回。

### Lines 621-640

````c
                                      (__v8si)_mm256_shrdv_epi32(__A, __B, __C),
                                      (__v8si)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_shrdv_epi32(__m128i __A, __m128i __B, __m128i __C)
{
  // Ops __A and __B are swapped.
  return (__m128i)__builtin_elementwise_fshr((__v4su)__B, (__v4su)__A,
                                             (__v4su)__C);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_shrdv_epi32(__m128i __A, __mmask8 __U, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectd_128(__U,
                                         (__v4si)_mm_shrdv_epi32(__A, __B, __C),
                                         (__v4si)__A);
}

````
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8si)_mm256_shrdv_epi32(__A, __B, __C),`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8si)_mm256_shrdv_epi32(__A, __B, __C),`。
- **L622 EN**: Executes a call or declaration centered on `statement`.
  **L622 CN**: 执行以 `statement` 为核心的调用或声明。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L625 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L625 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L626 EN**: Continues logic associated with callable symbol `_mm_shrdv_epi32`.
  **L626 CN**: 继续与可调用符号 `_mm_shrdv_epi32` 相关的逻辑。
- **L627 EN**: Opens a new lexical scope or compound statement.
  **L627 CN**: 打开一个新的词法作用域或复合语句块。
- **L628 EN**: Comment explains nearby logic, constraints, or intent: `Ops __A and __B are swapped.`.
  **L628 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Ops __A and __B are swapped.`。
- **L629 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshr((__v4su)__B, (__v4su)__A,`.
  **L629 CN**: 以 `(__m128i)__builtin_elementwise_fshr((__v4su)__B, (__v4su)__A,` 从当前函数返回。
- **L630 EN**: Executes a call or declaration centered on `statement`.
  **L630 CN**: 执行以 `statement` 为核心的调用或声明。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L633 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L634 EN**: Continues logic associated with callable symbol `_mm_mask_shrdv_epi32`.
  **L634 CN**: 继续与可调用符号 `_mm_mask_shrdv_epi32` 相关的逻辑。
- **L635 EN**: Opens a new lexical scope or compound statement.
  **L635 CN**: 打开一个新的词法作用域或复合语句块。
- **L636 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectd_128(__U,`.
  **L636 CN**: 以 `(__m128i)__builtin_ia32_selectd_128(__U,` 从当前函数返回。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4si)_mm_shrdv_epi32(__A, __B, __C),`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4si)_mm_shrdv_epi32(__A, __B, __C),`。
- **L638 EN**: Executes a call or declaration centered on `statement`.
  **L638 CN**: 执行以 `statement` 为核心的调用或声明。
- **L639 EN**: Closes the current lexical scope or compound statement.
  **L639 CN**: 结束当前词法作用域或复合语句块。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 641-660

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_shrdv_epi32(__mmask8 __U, __m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectd_128(__U,
                                         (__v4si)_mm_shrdv_epi32(__A, __B, __C),
                                         (__v4si)_mm_setzero_si128());
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_shrdv_epi16(__m256i __A, __m256i __B, __m256i __C)
{
  // Ops __A and __B are swapped.
  return (__m256i)__builtin_elementwise_fshr((__v16hu)__B, (__v16hu)__A,
                                             (__v16hu)__C);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_mask_shrdv_epi16(__m256i __A, __mmask16 __U, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectw_256(__U,
````
- **L641 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L641 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L642 EN**: Continues logic associated with callable symbol `_mm_maskz_shrdv_epi32`.
  **L642 CN**: 继续与可调用符号 `_mm_maskz_shrdv_epi32` 相关的逻辑。
- **L643 EN**: Opens a new lexical scope or compound statement.
  **L643 CN**: 打开一个新的词法作用域或复合语句块。
- **L644 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectd_128(__U,`.
  **L644 CN**: 以 `(__m128i)__builtin_ia32_selectd_128(__U,` 从当前函数返回。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v4si)_mm_shrdv_epi32(__A, __B, __C),`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v4si)_mm_shrdv_epi32(__A, __B, __C),`。
- **L646 EN**: Executes a call or declaration centered on `statement`.
  **L646 CN**: 执行以 `statement` 为核心的调用或声明。
- **L647 EN**: Closes the current lexical scope or compound statement.
  **L647 CN**: 结束当前词法作用域或复合语句块。
- **L648 EN**: Blank line separating nearby declarations or logic blocks.
  **L648 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L649 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L649 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L650 EN**: Continues logic associated with callable symbol `_mm256_shrdv_epi16`.
  **L650 CN**: 继续与可调用符号 `_mm256_shrdv_epi16` 相关的逻辑。
- **L651 EN**: Opens a new lexical scope or compound statement.
  **L651 CN**: 打开一个新的词法作用域或复合语句块。
- **L652 EN**: Comment explains nearby logic, constraints, or intent: `Ops __A and __B are swapped.`.
  **L652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Ops __A and __B are swapped.`。
- **L653 EN**: Returns from the current function with `(__m256i)__builtin_elementwise_fshr((__v16hu)__B, (__v16hu)__A,`.
  **L653 CN**: 以 `(__m256i)__builtin_elementwise_fshr((__v16hu)__B, (__v16hu)__A,` 从当前函数返回。
- **L654 EN**: Executes a call or declaration centered on `statement`.
  **L654 CN**: 执行以 `statement` 为核心的调用或声明。
- **L655 EN**: Closes the current lexical scope or compound statement.
  **L655 CN**: 结束当前词法作用域或复合语句块。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L657 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L658 EN**: Continues logic associated with callable symbol `_mm256_mask_shrdv_epi16`.
  **L658 CN**: 继续与可调用符号 `_mm256_mask_shrdv_epi16` 相关的逻辑。
- **L659 EN**: Opens a new lexical scope or compound statement.
  **L659 CN**: 打开一个新的词法作用域或复合语句块。
- **L660 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__U,`.
  **L660 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__U,` 从当前函数返回。

### Lines 661-680

````c
                                     (__v16hi)_mm256_shrdv_epi16(__A, __B, __C),
                                     (__v16hi)__A);
}

static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR
_mm256_maskz_shrdv_epi16(__mmask16 __U, __m256i __A, __m256i __B, __m256i __C)
{
  return (__m256i)__builtin_ia32_selectw_256(__U,
                                     (__v16hi)_mm256_shrdv_epi16(__A, __B, __C),
                                     (__v16hi)_mm256_setzero_si256());
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_shrdv_epi16(__m128i __A, __m128i __B, __m128i __C)
{
  // Ops __A and __B are swapped.
  return (__m128i)__builtin_elementwise_fshr((__v8hu)__B, (__v8hu)__A,
                                             (__v8hu)__C);
}

````
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_shrdv_epi16(__A, __B, __C),`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_shrdv_epi16(__A, __B, __C),`。
- **L662 EN**: Executes a call or declaration centered on `statement`.
  **L662 CN**: 执行以 `statement` 为核心的调用或声明。
- **L663 EN**: Closes the current lexical scope or compound statement.
  **L663 CN**: 结束当前词法作用域或复合语句块。
- **L664 EN**: Blank line separating nearby declarations or logic blocks.
  **L664 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L665 EN**: Continues the surrounding expression or declaration: `static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L665 CN**: 继续构造周围的表达式或声明：`static __inline__ __m256i __DEFAULT_FN_ATTRS256_CONSTEXPR`。
- **L666 EN**: Continues logic associated with callable symbol `_mm256_maskz_shrdv_epi16`.
  **L666 CN**: 继续与可调用符号 `_mm256_maskz_shrdv_epi16` 相关的逻辑。
- **L667 EN**: Opens a new lexical scope or compound statement.
  **L667 CN**: 打开一个新的词法作用域或复合语句块。
- **L668 EN**: Returns from the current function with `(__m256i)__builtin_ia32_selectw_256(__U,`.
  **L668 CN**: 以 `(__m256i)__builtin_ia32_selectw_256(__U,` 从当前函数返回。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v16hi)_mm256_shrdv_epi16(__A, __B, __C),`.
  **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v16hi)_mm256_shrdv_epi16(__A, __B, __C),`。
- **L670 EN**: Executes a call or declaration centered on `statement`.
  **L670 CN**: 执行以 `statement` 为核心的调用或声明。
- **L671 EN**: Closes the current lexical scope or compound statement.
  **L671 CN**: 结束当前词法作用域或复合语句块。
- **L672 EN**: Blank line separating nearby declarations or logic blocks.
  **L672 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L673 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L673 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L674 EN**: Continues logic associated with callable symbol `_mm_shrdv_epi16`.
  **L674 CN**: 继续与可调用符号 `_mm_shrdv_epi16` 相关的逻辑。
- **L675 EN**: Opens a new lexical scope or compound statement.
  **L675 CN**: 打开一个新的词法作用域或复合语句块。
- **L676 EN**: Comment explains nearby logic, constraints, or intent: `Ops __A and __B are swapped.`.
  **L676 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Ops __A and __B are swapped.`。
- **L677 EN**: Returns from the current function with `(__m128i)__builtin_elementwise_fshr((__v8hu)__B, (__v8hu)__A,`.
  **L677 CN**: 以 `(__m128i)__builtin_elementwise_fshr((__v8hu)__B, (__v8hu)__A,` 从当前函数返回。
- **L678 EN**: Executes a call or declaration centered on `statement`.
  **L678 CN**: 执行以 `statement` 为核心的调用或声明。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 681-700

````c
static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_mask_shrdv_epi16(__m128i __A, __mmask8 __U, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectw_128(__U,
                                         (__v8hi)_mm_shrdv_epi16(__A, __B, __C),
                                         (__v8hi)__A);
}

static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR
_mm_maskz_shrdv_epi16(__mmask8 __U, __m128i __A, __m128i __B, __m128i __C)
{
  return (__m128i)__builtin_ia32_selectw_128(__U,
                                         (__v8hi)_mm_shrdv_epi16(__A, __B, __C),
                                         (__v8hi)_mm_setzero_si128());
}

#undef __DEFAULT_FN_ATTRS128
#undef __DEFAULT_FN_ATTRS256
#undef __DEFAULT_FN_ATTRS128_CONSTEXPR
#undef __DEFAULT_FN_ATTRS256_CONSTEXPR
````
- **L681 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L681 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L682 EN**: Continues logic associated with callable symbol `_mm_mask_shrdv_epi16`.
  **L682 CN**: 继续与可调用符号 `_mm_mask_shrdv_epi16` 相关的逻辑。
- **L683 EN**: Opens a new lexical scope or compound statement.
  **L683 CN**: 打开一个新的词法作用域或复合语句块。
- **L684 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__U,`.
  **L684 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__U,` 从当前函数返回。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_shrdv_epi16(__A, __B, __C),`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_shrdv_epi16(__A, __B, __C),`。
- **L686 EN**: Executes a call or declaration centered on `statement`.
  **L686 CN**: 执行以 `statement` 为核心的调用或声明。
- **L687 EN**: Closes the current lexical scope or compound statement.
  **L687 CN**: 结束当前词法作用域或复合语句块。
- **L688 EN**: Blank line separating nearby declarations or logic blocks.
  **L688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L689 EN**: Continues the surrounding expression or declaration: `static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L689 CN**: 继续构造周围的表达式或声明：`static __inline__ __m128i __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L690 EN**: Continues logic associated with callable symbol `_mm_maskz_shrdv_epi16`.
  **L690 CN**: 继续与可调用符号 `_mm_maskz_shrdv_epi16` 相关的逻辑。
- **L691 EN**: Opens a new lexical scope or compound statement.
  **L691 CN**: 打开一个新的词法作用域或复合语句块。
- **L692 EN**: Returns from the current function with `(__m128i)__builtin_ia32_selectw_128(__U,`.
  **L692 CN**: 以 `(__m128i)__builtin_ia32_selectw_128(__U,` 从当前函数返回。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__v8hi)_mm_shrdv_epi16(__A, __B, __C),`.
  **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__v8hi)_mm_shrdv_epi16(__A, __B, __C),`。
- **L694 EN**: Executes a call or declaration centered on `statement`.
  **L694 CN**: 执行以 `statement` 为核心的调用或声明。
- **L695 EN**: Closes the current lexical scope or compound statement.
  **L695 CN**: 结束当前词法作用域或复合语句块。
- **L696 EN**: Blank line separating nearby declarations or logic blocks.
  **L696 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L697 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128`.
  **L697 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128`。
- **L698 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256`.
  **L698 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256`。
- **L699 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`.
  **L699 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS128_CONSTEXPR`。
- **L700 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`.
  **L700 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS256_CONSTEXPR`。

### Lines 701-702

````c

#endif
````
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Closes the current preprocessor conditional block.
  **L702 CN**: 结束当前预处理条件块。

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
- **Conditional macros / 条件宏**: `__IMMINTRIN_H`, `__AVX512VLVBMI2INTRIN_H`, `__cplusplus`
- **External builtins / 外部 builtin**: `__builtin_ia32_compresshi128_mask`, `__builtin_ia32_compressqi128_mask`, `__builtin_ia32_compressstorehi128_mask`, `__builtin_ia32_compressstoreqi128_mask`, `__builtin_ia32_expandhi128_mask`, `__builtin_ia32_expandqi128_mask`, `__builtin_ia32_expandloadhi128_mask`, `__builtin_ia32_expandloadqi128_mask`, `__builtin_ia32_compresshi256_mask`, `__builtin_ia32_compressqi256_mask`, `__builtin_ia32_compressstorehi256_mask`, `__builtin_ia32_compressstoreqi256_mask`
